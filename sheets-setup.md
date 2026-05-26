# Google Sheets Setup — E-Commerce Checkout Incident Detection Agent

## Spreadsheet Name
`E-commerce Operations Agent`

## Tab 1 — Cart Events (Input)

**Headers (Row 1):**
| event_id | timestamp | checkout_step | cart_value | device |

**Rules:**
- timestamp format: YYYY-MM-DD HH:MM:SS
- checkout_step values: PAYMENT, SHIPPING, ADDRESS
- device values: MOBILE, DESKTOP

## Tab 2 — Baseline Metrics (Reference)

**Headers (Row 1):**
| metric | baseline_rate |

**Data (set once, never change):**
| PAYMENT | 0.02 |
| SHIPPING | 0.05 |
| ADDRESS | 0.03 |

## Tab 3 — State (Agent Memory)

**Headers (Row 1):**
| key | value |

**Data (1 row only):**
| last_processed_timestamp | 2026-05-20 00:00:00 |

Set the timestamp to BEFORE your earliest Cart Events timestamp so the agent picks up all events on first run.

## Tab 4 — Agent (Output Log)

**Headers (Row 1):**
| command_id | action | severity | reason | checkout_step | cart_value | device | baseline_rate | timestamp |

Leave all rows below Row 1 empty. The agent writes here automatically.

## Zapier Tools to Add

| Tool | Sheet Used |
|------|-----------|
| Google Sheets: Get Many Spreadsheet Rows (Advanced) | State |
| Google Sheets: Get Data Range | Baseline Metrics |
| Google Sheets: Lookup Spreadsheet Rows (Advanced) | Cart Events |
| Google Sheets: Create Spreadsheet Row | Agent |
| Google Sheets: Update Spreadsheet Row | State |
| Webhooks by Zapier: POST | N/A |
