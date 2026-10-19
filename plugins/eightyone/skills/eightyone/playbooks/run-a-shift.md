# Run a shift

Goal: help the people on the floor get the day's work done. Release work, move tasks along, record what happened, and deal with quality, stops and stock as they come up, without ever acting on a guess.

## Before you change anything

- **Every change is previewed and confirmed** (SKILL.md rule 3). Call the tool with `dryRun: true`, tell the user what it will do and set off, and make the real call only after their OK. Batch related changes and confirm them together.
- **You act as the user.** You can do what they can do, and you see the tasks they see. When a tool is refused for a permission, say which permission and who usually has it. Don't look for another way round.
- **Facts come from people or the data, never from you.** Counts, measurements, stop reasons and dispositions are what someone tells you or what EightyOne records. If you don't have one, ask.

## Start of shift

1. `me_get`, then `guide_get` topic `company` for this company's own rules.
2. **What's waiting:**
   - `work_orders_list` for draft orders;
   - `tasks_list` for open and dispatched work;
   - `downtime_events_list` with `openOnly` for machines still stopped, and with `unclassifiedOnly` for stops nobody explained;
   - `inventory_reorder_suggestions` for what's running low.
3. Summarize it for the user before proposing anything.

## Release and dispatch

1. **Release:** `work_orders_release`. Its preview lists each operation, its machine, and which steps wait for a gate.
   - A step whose work center has several machines gets none. Ask which machine, then set it with `tasks_update` (`ssuId`).
2. **Dispatch:** `tasks_update` with `assigneeIds` and `status: "dispatched"`. People come from `personnel` ids (`me_get` shows your own).
3. **Linked work:** `tasks_create` with `after: { taskId, type: "dependency" }` creates a task that waits for another.

## Doing the work

- **During a run:** `tasks_record_counts` adds good and scrap counts. They add up, so record each new count once.
- **Completing a task:** start with `tasks_get_form` to list the fields. Then `tasks_submit_form` with the answers and `status: "done"`:
  - required fields must be filled;
  - a producing task also needs its counts (or counts recorded during the run);
  - a maintenance task needs failure codes: `problemCodeId`, `causeCodeId`, `remedyCodeId`.
- **What the preview tells you:**
  - a measurement outside its limits opens an NCR;
  - a machine-status field changes the machine's status;
  - the work order moves on.
- **Fields you can't fill:** photos, signatures, scans, stock and location fields are filled in the app. When one is required, the task waits for a person: tell the user who and what.
- **Partial work:** `tasks_submit_form` with `status: "incomplete"`, an `incompleteReason`, and `createFollowUp: true` creates a follow-up for the rest.

## Quality

- **A failed check** opens an NCR by itself. `ncrs_open` is for problems people find: give `foundOn` (the task where it was found) and the lot or product.
- **Disposition:** `ncrs_disposition`, once per NCR, and only with the decision of someone who holds 'Disposition NCRs'.
  - Scrap moves stock to the bads bin and blocks the lot.
  - Rework creates a rework task.
  - A blocked lot can't ship.

## Stops

- **Reporting:** `downtime_events_open` for a machine without telemetry; `downtime_events_close` when it runs again.
- **Classifying:** `downtime_events_classify`, only with the reason the operator gives. An unexplained stop is a gap to report, not a guess to fill.
- **Maintenance:** a technician's meter reading goes in with `meters_record_reading`. Its preview says whether it opens a maintenance task.

## Stock

- **Receiving:** `purchase_orders_receive`, per line, into the unit the user names, with the lot for lot-tracked products.
- **Moving:** `stock_transfer`, never more than `storage_units_get` shows is there.
- **Shipping:** `shipments_create` (a draft holds no stock), then `shipments_ship`.
- **Cycle count:**
  1. `inventory_counts_create`;
  2. counts from the count task in the app, or `inventory_counts_record_lines` from what someone reports;
  3. `inventory_counts_finalize`;
  4. show each difference, and get a decision for each one;
  5. `inventory_counts_reconcile`. Write-offs need an adjustment reason. Recount a line that moved during the count rather than adjusting it.

## Partners

- **Handing work over:** `handoffs_send` gives a task to a partner company.
- **Work sent to you:** `handoffs_decline` turns it down, with a reason the partner sees.
- **Goods from a partner:** `partner_deliveries_receive` books them.
- **Withdrawing** a handoff is done in the dashboard.

## End of shift

Summarize:
- what was released, completed and left open, and why;
- counts;
- NCRs opened and their dispositions;
- stops: hours, reasons, and which are still unclassified;
- stock changes;
- anything waiting for a person: a signature, a photo, a decision.
