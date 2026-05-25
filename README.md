# 🛒 E-Commerce Checkout Incident Detection Agent

An autonomous AI agent that monitors e-commerce checkout failure events in real time, classifies each event by severity using 4 business rules, fires webhook alerts, writes a full audit log, and tracks its own state to prevent duplicate processing — all without human intervention.

---

## 🎯 Problem Statement

E-commerce businesses lose significant revenue when customers abandon carts at checkout. Payment failures, shipping errors, and address issues each have different revenue impacts. Without automated monitoring, engineering teams receive no real-time alerts and must manually review logs — too slow to prevent revenue loss.

This agent solves that by classifying every checkout failure the moment it occurs and routing ALERT or LOG decisions to downstream systems instantly.

---

## ⚡ How It Works

The agent runs on a scheduled trigger and processes all new checkout events since the last run:

```
Trigger → Read State → Read Baselines → Read New Events → Evaluate Severity → Write Log → Fire Webhook → Update State
```

### The 4 Severity Rules

| Rule | Condition | Severity |
|------|-----------|----------|
| Rule A | checkout_step = PAYMENT | HIGH |
| Rule B | cart_value > $1,000 | HIGH |
| Rule C | 2+ MOBILE devices in current batch | HIGH |
| Rule D | Failures exceed baseline rate for that step | HIGH |
| None | No rules triggered | LOW |

- **Any rule true** → `action = ALERT`, `severity = HIGH`
- **No rules true** → `action = LOG`, `severity = LOW`

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────┐
│               Google Sheets (Data Layer)            │
│  ┌──────────────┐ ┌─────────────────┐ ┌──────────┐ │
│  │  Cart Events │ │Baseline Metrics │ │  State   │ │
│  └──────────────┘ └─────────────────┘ └──────────┘ │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│         Checkout Incident Detection Agent           │
│              (Zapier Automation)                    │
│                                                     │
│  1. Read LAST_TIMESTAMP from State sheet            │
│  2. Load baseline rates for all steps               │
│  3. Filter new events (timestamp > LAST_TIMESTAMP)  │
│  4. Apply 4 severity rules per event                │
│  5. Write 1 row per event to Agent sheet            │
│  6. Fire 1 webhook POST per event (9-field JSON)    │
│  7. Update State sheet with latest timestamp        │
└─────────────────────────────────────────────────────┘
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
┌─────────────────────┐   ┌──────────────────────┐
│    Agent Sheet      │   │   Webhook Endpoint   │
│  (Audit Log)        │   │   (JSON Payload)     │
└─────────────────────┘   └──────────────────────┘
```

---

## 📊 Google Sheets Structure

**Spreadsheet Name:** `E-commerce Operations Agent`

| Tab | Purpose |
|-----|---------|
| `Cart Events` | Input — checkout failure events |
| `Baseline Metrics` | Reference — normal failure rates per step |
| `State` | Memory — last processed timestamp |
| `Agent` | Output — one row per event processed |

### Cart Events Schema
```
event_id | timestamp | checkout_step | cart_value | device
```

### Baseline Metrics Schema
```
metric | baseline_rate
PAYMENT | 0.02
SHIPPING | 0.05
ADDRESS | 0.03
```

### State Schema
```
key | value
last_processed_timestamp | 2026-05-20 09:00:00
```

### Agent Output Schema
```
command_id | action | severity | reason | checkout_step | cart_value | device | baseline_rate | timestamp
```

---

## 🔧 Zapier Tools Used

| Tool | Purpose |
|------|---------|
| `Google Sheets: Get Many Spreadsheet Rows (Advanced)` | Read State sheet |
| `Google Sheets: Get Data Range` | Read Baseline Metrics |
| `Google Sheets: Lookup Spreadsheet Rows (Advanced)` | Read Cart Events |
| `Google Sheets: Create Spreadsheet Row` | Write to Agent sheet |
| `Google Sheets: Update Spreadsheet Row` | Update State sheet |
| `Webhooks by Zapier: POST` | Fire JSON webhook per event |

---

## 📦 Webhook Payload (9 Fields)

```json
{
  "command_id": "CMD_20260520_001",
  "action": "ALERT",
  "severity": "HIGH",
  "reason": "PAYMENT step failure - critical revenue impact",
  "checkout_step": "PAYMENT",
  "cart_value": 1250.00,
  "device": "MOBILE",
  "baseline_rate": 0.02,
  "timestamp": "2026-05-20 09:15:00"
}
```

---

## 🔒 Strict Processing Rules

- **Never** process events with timestamp ≤ `LAST_TIMESTAMP`
- **Always** write to Agent sheet **before** firing webhook
- **Always** update State sheet **last** after all events processed
- **Always** use POST method — never GET
- **Never** batch multiple events into one webhook call
- **Never** write raw serial numbers as timestamps

---

## 🚀 How to Run

1. Set up the Google Sheet with 4 tabs (see schema above)
2. Add sample data to Cart Events tab
3. Configure Zapier agent with 6 tools
4. Set trigger to Scheduled (daily or every 15 minutes)
5. Run agent and verify:
   - Agent sheet has new rows
   - Webhook endpoint received POST requests
   - State sheet timestamp updated

---

## 🛠️ Tech Stack

![Zapier](https://img.shields.io/badge/Zapier-FF4A00?style=flat&logo=zapier&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-34A853?style=flat&logo=google-sheets&logoColor=white)
![Webhooks](https://img.shields.io/badge/Webhooks-JSON-blue)

---

## 👤 Author

**Praveen Kumar Jatta** — AI Automation Consultant & Technical Program Manager

- 🌐 [jattaai.com](https://jattaai.com)
- 💼 [linkedin.com/in/praveenjatta](https://linkedin.com/in/praveenjatta)
- 📧 jattaaihq@gmail.com

---

*Built as part of the EdgeUp for TPMs — Applied Agentic AI program by Interview Kickstart*
