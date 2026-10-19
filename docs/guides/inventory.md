# Inventory: storage, bins, moves, counts

## Where stock is

- **Storage sites** hold **storage units** (a shelf, a rack, a bay, a van).
- Whose a site is decides what happens to the stock in it:
  - a company warehouse or a floor's storage: the company's own;
  - a client's site;
  - a technician's;
  - a customer's consigned stock, which can only ship back to that customer;
  - a partner company's custody.
- Every machine (SSU) has its own unit too.
- Stock sits in a unit **in a bin:**
  - **goods** (usable);
  - **bads** (damaged or scrapped);
  - **quarantine** (waiting for inspection);
  - **installed** (fitted into a machine);
  - **transit** (on the way).
- Only goods count as available.
- A **line item** is one product in one unit, in one bin, of one lot (or one serial).
- `storage_units.get` shows what a unit holds; `inventory.stock_levels` gives goods on hand per product and site.

## Moving stock

| | Capability | Notes |
| --- | --- | --- |
| Move between units or bins | `stock.transfer` | Never more than the line holds. Name the lot or serial when the product is tracked. |
| Receive a delivery | `purchase_orders.receive` | Per PO line, into a unit. A lot-tracked product needs its lot. An inspection plan for the product or vendor quarantines the receipt and opens an inspection task. Never more than was ordered. |
| Ship to a customer | `shipments.create`, then `shipments.ship` | A draft holds no stock. Shipping takes goods out, checking the lot, the stock and consignment. A customer that's an EightyOne company receives them in its transit. Factories only. |
| Partner goods | `partner_deliveries.receive` | Goods a partner company sent you. |

Inventory tasks (moves done in the app) post the same way when completed. Purchase orders are quantities only: EightyOne holds no prices.

## Cycle counts

1. `inventory_counts.create`: a count of one unit. It snapshots what's expected, and creates the count task for operators when the company has an inventory task type.
2. Counts come from that task in the app, or from `inventory_counts.record_lines`.
3. `inventory_counts.finalize` freezes the expected quantities as they are now, works out each difference, and adds anything on hand the count never saw (counted 0). It flags lines whose stock moved while the count was open.
4. `inventory_counts.reconcile` posts the decisions, which needs **Adjust inventory**:
   - **accepted:** stock becomes the counted quantity;
   - **adjusted:** stock becomes a corrected quantity;
   - **ignored:** stock stays as it is.

   Every write-off needs an adjustment reason. The preview lists each addition, write-off and bin move.

## Reordering

A stock policy per product (and optionally per site) sets a minimum, or a reorder point. The reorder point is computed from lead time × measured consumption. When stock falls to it, `stock.reorder_triggered` fires once, and the product appears in `inventory.reorder_suggestions`. It re-arms when stock recovers.
