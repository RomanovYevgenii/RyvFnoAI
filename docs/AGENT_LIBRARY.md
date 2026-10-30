# Agent Library

Ready-made RyvSoft AI agents for Dynamics 365 Finance & Operations. Each agent is a set of instructions plus the tools it may use, delivered as a data package and imported through **Data management**. The library is included in the RyvSoft AI subscription; configuring the agents for a customer's own processes (policies, thresholds, report layouts, knowledge base) is part of the implementation.

Every agent runs under the signed-in user's F&O security: it sees and changes only what the user can see and change. Agents marked **writes** prepare changes as a preview, and nothing is saved until the user clicks **Confirm** in the panel.

All agents are built and tested on the Contoso demo data (company USMF).

| Agent | What it does | Writes |
|---|---|---|
| [Credit & Collections](#credit--collections) | Overdue balances, credit limit breaches, collection priorities, customer blocking | ✔ with Confirm |
| [AP Invoice Check](#ap-invoice-check) | Checks a vendor invoice (PDF / scan / pending invoice) against the PO and receipts | – |
| [Master Data Quality](#master-data-quality) | Finds and fixes incomplete customer and vendor data in bulk | ✔ with Confirm |
| [Sales Order Desk](#sales-order-desk) | Order status, late and held orders, replies to customers | – |
| [Inventory](#inventory) | On hand, shortages, slow-moving items | – |
| [Period Close Assistant](#period-close-assistant) | Month-end checklist with counts and amounts | – |
| [F&O Helpdesk](#fo-helpdesk) | "How do I…" answers from the company knowledge base, opens the right form | – |

For administrators the library also includes **[Agent Architect](#agent-architect)**, which helps design an agent for your own process.

---

## Credit & Collections

Overdue balances by aging bucket, credit limit breaches, who to call first, and blocking customers with confirmation.

**Try:** "Which customers are over 60 days overdue? Top 10 by amount." · "Block all customers in group 30 that are more than 90 days overdue."

| Report | Columns | Output |
|---|---|---|
| Customer aging | Account, Name, Group, Currency, Credit limit, Balance, Not due, 1–30, 31–60, 61–90, Over 90, Oldest due date, Blocked | Excel |
| Credit limit breaches | Account, Name, Credit limit, Balance, Over limit, Over limit %, Blocked | Excel / chat |
| Collection call list | Rank, Account, Name, Overdue > 60, Total overdue, Oldest due date, Contact | Excel |

## AP Invoice Check

Reads a vendor invoice attached to the chat (PDF, image, CSV) or a pending invoice in F&O and compares it line by line with the purchase order and product receipts: price and quantity differences, items not on the PO, duplicate invoice numbers, vendor on hold. Read-only.

**Try:** "Check this invoice against the PO." (with the PDF attached) · "Are there invoices from vendor US-101 with the same number?"

| Report | Columns | Output |
|---|---|---|
| Invoice match result | Invoice, Vendor, PO, Line, Item, Invoice qty, Received qty, Ordered qty, Invoice price, PO price, Difference, Status | Chat / Excel |
| Pending invoices with exceptions | Invoice, Vendor, PO, Exception type, Amount affected | Excel |

## Master Data Quality

Checks customers and vendors for missing tax numbers, addresses, payment terms, payment methods, e-invoicing settings and possible duplicates, and fixes them in bulk with confirmation. Never merges or deletes records.

**Try:** "Which vendors have no payment terms?" · "Set payment terms Net30 for customers in group 30 that have none."

| Report | Columns | Output |
|---|---|---|
| Data quality summary | Check, Customers affected, Vendors affected | Chat |
| Data quality details | Type, Account, Name, Issue, Current value, Suggested value | Excel |
| Possible duplicates | Group, Account, Name, Tax number, Address, Created | Excel |

## Sales Order Desk

Order status, requested and confirmed dates, delivered vs ordered quantities, late orders, holds and their reasons, and a short neutral reply the user can paste into an e-mail. Read-only.

**Try:** "Which open orders for customer US-003 are late?" · "List all orders on hold and why."

| Report | Columns | Output |
|---|---|---|
| Late orders | Order, Customer, Name, Requested ship date, Confirmed ship date, Days late, Open lines, Open amount | Excel |
| Orders on hold | Order, Customer, Hold code, Reason, Held since, Amount | Excel / chat |

## Inventory

Physical, reserved and available stock by site and warehouse, shortages against minimum stock, slow-moving and non-moving items. Large lists go to a file, in batch for very large volumes. Read-only.

**Try:** "How much of item 1000 do we have, by warehouse?" · "Which items are below minimum stock at site 1?"

| Report | Columns | Output |
|---|---|---|
| On hand by warehouse | Item, Name, Site, Warehouse, Physical, Reserved, Available, Ordered in, Unit | Excel |
| Shortage list | Item, Name, Site/Warehouse, Available, Ordered in, Minimum, Shortage | Excel |
| Slow-moving items | Item, Name, On hand, Stock value, Last issue date, Days without issue | Excel |

## Period Close Assistant

A month-end checklist with counts and amounts: unposted journals, received not invoiced, delivered not invoiced, pending vendor invoices, ledger period status. Reports only; posting and closing stay with the user.

**Try:** "Run the month-end checklist for USMF for last month." · "Which general journals are not posted and who created them?"

| Report | Columns | Output |
|---|---|---|
| Close checklist | Check, Count, Amount, Status | Chat |
| Unposted journals | Journal, Name, Type, Created by, Created, Lines, Amount | Excel |
| Received not invoiced | PO, Vendor, Item, Received qty, Value, Receipt date | Excel |
| Delivered not invoiced | Order, Customer, Item, Delivered qty, Value, Delivery date | Excel |

## F&O Helpdesk

Answers "how do I…" questions from the company knowledge base first, then from F&O metadata and Microsoft documentation; gives the navigation path and opens the form. Does not change data.

**Try:** "How do I put a customer on hold?" · "What does the Blocked field on the customer mean?"

## Agent Architect

For administrators. Helps design a new agent, or adapt a library agent, for your own environment: the list of tasks it should solve, the steps checked on your data, the instructions, the tools it needs and test questions. It only reads; the administrator reviews the result and creates the agent.

---

Want an agent for your own process? Custom agents are designed as part of an implementation. See [Contact](../README.md#contact).
