# Accounting: QuickBooks, Xero and more

**What it is:** EightyOne connects to a company's accounting system ("the books"). The books stay the place where money lives, and EightyOne never holds a price, a cost or a total. What moves between them is names, ids and quantities:

- **From the books:** customers, vendors and products, and the quantities on hand of tracked items.
- **To the books:** estimates, invoices and purchase orders, as items and quantities. The books price them.

**Providers:** QuickBooks today. Xero and others are coming. `accounting.providers` lists them all, with which ones are available. A company connects one at a time.

**Who:** connecting, importing and sending documents need **Manage company settings & integrations**. Seeing the connection and the documents needs **View company settings & integrations**. The stock comparison needs **View inventory**.

## Connecting

1. `accounting.connect` with the provider returns its sign-in page (`authorizationUrl`).
2. A person opens it, signs in to their books and approves.
3. The provider sends them back to EightyOne's **Integrations** page, connected.

An agent can't approve for the company. It can only hand the link to a person. `accounting.test` answers with the books' company name. `accounting.disconnect` erases the stored tokens. Ids already linked are kept, so reconnecting the same books carries on where it left off. Connecting *different* books unlinks the old ids, because they mean nothing in the new books.

## Importing

`accounting.sync` imports `customers`, `vendors` and `products` (all three by default):

- **Customers** become a factory's **customers**, or a managed-services company's **clients**.
- A record already linked to the books is updated (its name, and a product's description and SKU).
- A record with the same name, or a product with the same SKU, is **linked** rather than duplicated.
- Everything else is created and linked. Nothing is deleted because it's gone from the books.
- A new tracked product starts with the books' quantity on hand, in the company's default storage.

The reply counts what was created, linked, updated and skipped, and lists problems: a SKU two items share, a quantity that isn't whole. Every link is an [external ref](integrations.md), so `external_refs.lookup` translates in both directions.

## Documents

| Document | From | Sent when |
| --- | --- | --- |
| Estimate | A quote task (task type **is quote**) | The task is done, or someone asks |
| Invoice | A done task, or a shipped shipment (factories) | Someone asks, or the task's type has **Request an invoice when done** |
| Purchase order | A purchase order | It's created |

Asking is `accounting.request_document` with the `kind` and one of `taskId`, `shipmentId` or `purchaseOrderId`. Documents are queued and sent in the background, so completing a task never waits on the books. A failure is retried later: after 1 minute, then 5, 30 and 120, then once more after 12 hours. A refusal that won't change on its own fails at once with the reason, for example a product that isn't linked to an item.

- **One per source:** asking again returns the same document, and asking again for a failed one queues it again. `accounting.retry_document` does the same by id.
- **What goes over:** the customer or vendor, each product as its item in the books, and the quantities. A task's quantities are what was recorded on its form (parts removed or only counted aren't billed). A shipment's are its lines, a PO's are its lines. Each document is marked with EightyOne's reference ("Fix press (task id)", "PO 12: Restock").
- **What comes back:** the document's number, its status in the books, and a link that opens it there. `accounting.list_documents` lists them, by task, shipment, purchase order or status.

If the company's setting **Send invoices and quotes automatically** is on, the books also email each new estimate and invoice to the customer.

## Stock

The books own the **capitalized inventory**: what the stock is worth. EightyOne owns the **operational inventory**: what is where, by unit, bin, lot and serial. `accounting.inventory_differences` compares, item by item, the books' quantity on hand with EightyOne's stock in the company's own storage. It lists the largest differences first. Nothing is changed on either side: the report is for a person to decide.

## What is never kept

- **No amounts.** Some providers need a price on every line to accept a document. EightyOne reads the item's own price from the books for that one request and never stores, logs or returns it. Error messages from the books are stored with any amount cut out.
- **No passwords.** The connection keeps the provider's refresh token, encrypted, and nothing else.
