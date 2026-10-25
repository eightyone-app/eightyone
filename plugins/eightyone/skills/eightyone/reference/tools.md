# EightyOne MCP tools

Generated from the capability registry (`pnpm docs:api`): don't edit by hand.

Every tool acts as the connected user, in one company, with that user's permissions (`me_get` shows them). Tools the user may not use aren't listed by the server.

## Tools

- **`accounting_inventory_differences`**: Where the books' quantity on hand differs from EightyOne's, per linked tracked product, largest first. EightyOne counts its own storage (not stock at clients' sites). Reported only: neither side is changed.
- **`analytics_downtime`**: Time each machine (SSU) spent DOWN or in MAINTENANCE in the window, most first, with how many times it entered that state. For the reasons, follow up with downtime_events.list (reason-coded stoppages) or storylines.ssu (the machine's story).
- **`analytics_failure_pareto`**: What breaks most: completed failure-coded tasks grouped by problem code, most frequent first (top 25), with the machine that suffers each mode most.
- **`analytics_oee`**: Overall equipment effectiveness per machine: availability, performance and quality (0–1) with good / scrap quantities. A factor is null when its source data is missing (no routed work, no counts).
- **`analytics_overview`**: Headline numbers for a window: total downtime, failures, average availability and OEE, and how many machines are monitored. `unavailableMetrics` names OEE factors with no source data yet. Start here, then drill into analytics.downtime / analytics.reliability.
- **`analytics_reliability`**: Per machine: failures (entries into DOWN), mean time to repair and mean time between failures, in seconds, most failures first.
- **`clients_get`**: One client by your id for it.
- **`clients_list`**: The company's clients (the customers whose sites and SSUs work happens at), newest first. Filter by name or active flag.
- **`config_apply`** (changes data): Converges the company's setup to the spec, through the same rules as the dashboard; every change is audited. Safe to re-run: unchanged items are skipped. Not one transaction: if a change fails, the ones before it stay and the result says where it stopped; fix the cause and apply again. Always run config.plan first and get the user's go-ahead.
- **`config_export`**: The company's setup as a factory spec: settings, SSU statuses, reason codes, work centers, sites with their storage units and SSU trees, warehouses, product categories, vendors, products by SKU with their vendors, recipe and stock policies, parameter sets, task types with their forms, SLAs and gates, routings, per-machine signals, meters, condition rules and spare parts, flows, issue reports, inspection plans, and people (permission groups, roles, teams, skills, existing people by email, subcontractors). Everything is referenced by name, so the spec can be applied to another company as a template. Start here before changing setup: edit the export, then config.plan and config.apply.
- **`config_plan`**: What config.apply would do, change by change: create, update (with each field's current and new value), unchanged, and extra (in the company but not in the spec). Doesn't change anything. Show this plan to the user before applying.
- **`config_validate`**: Checks a spec beyond its schema: duplicate names or SKUs, references to statuses, work centers, reason codes, storage, categories, vendors, SKUs, task types, parameter sets or permission groups that don't exist, ambiguous names, a product both bought and made, gates naming fields the form doesn't have. Doesn't change anything.
- **`customers_list`**: A factory's customers, who shipments go to: name, reference and ship-to address.
- **`downtime_events_classify`** (changes data): Classify a machine stoppage with a downtime reason code (reason_codes_list, kind downtime), optionally with a note. Re-classifying replaces the reason. Find stops waiting for one with downtime_events_list and unclassifiedOnly.
- **`downtime_events_close`** (changes data): End a stoppage now. Closing one that already ended changes nothing. Needs 'Manage machine data'.
- **`downtime_events_list`**: Individual downtime events (newest first, up to 500): when each machine stopped, for how long, and the reason code someone classified it with. Use it to explain the totals from analytics.downtime. Filter by machine and window; `unclassifiedOnly` lists stops still waiting for a reason.
- **`downtime_events_open`** (changes data): Record that a machine stopped now, for machines without telemetry (those report stops themselves). Close it with downtime_events_close when it runs again, and classify it with a reason. Needs 'Manage machine data'.
- **`downtime_events_summary`**: Recorded stoppages per machine in a window (default: the last 30 days), most stopped time first, each broken down by reason code: the direct answer to "which machines were down most, and why?". Counts only the part of each stop inside the window; a stop still open counts up to the window end. Complements analytics.downtime, which measures time in a DOWN / MAINTENANCE status.
- **`external_refs_lookup`**: Translate between an outside system's id and an EightyOne record. Send externalId to find the record it stands for, or entityType + entityId to find the record's id in that system. 404 when they aren't linked.
- **`guide_get`**: EightyOne concepts and how to read the data: the glossary, downtime, maintenance, production, inventory, quality, commissioning, what deserves attention, and this company's own guidance (topic "company"). Without a topic, lists the topics. Read the glossary before working on an unfamiliar area.
- **`handoffs_decline`** (changes data): Decline a task a partner sent you, with a reason they will see.
- **`handoffs_send`** (changes data): Hand one of your tasks to another EightyOne company (by its short name, or a partner subcontractor). The partner gets its own copy; your people come off the task. Goods can travel with it (items), and a data-request task carries the scopes it asks for.
- **`inventory_counts_create`** (changes data): Open a count of one storage unit. It snapshots what the system expects there, and when the company has an inventory task type it also creates the count task operators fill in from the app. Needs 'Manage inventory'.
- **`inventory_counts_finalize`** (changes data): Close counting: each line's expected quantity becomes what's on hand now, discrepancies are computed, and anything on hand the count never saw becomes a line counted 0. Nothing is posted to stock until it's reconciled. Needs 'Manage inventory'.
- **`inventory_counts_get`**: A cycle count with its lines: expected (on hand by the system), counted, the discrepancy once finalized, and whether stock moved while it was open (a reason to recount rather than adjust).
- **`inventory_counts_list`**: Cycle counts, newest first: which storage unit, status (draft, in_progress, completed = finalized and waiting to be reconciled, reconciled) and totals. inventory_counts_get shows the lines and discrepancies.
- **`inventory_counts_reconcile`** (changes data): Decide each line of a finalized count and post it. accepted: stock becomes the counted quantity; adjusted: becomes adjustedQuantity; ignored (and any line not listed): stock stays as the system has it. Every write-off needs an adjustment reasonCodeId (reason_codes_list kind adjustment). Needs 'Manage inventory' and 'Adjust inventory'. Preview it first: it lists each addition, write-off and bin move.
- **`inventory_counts_record_lines`** (changes data): Add what someone counted to an open count, by SKU (and bin, lot or serial where it matters). Quantities add to what was already counted for that line, as in the dashboard. Then finalize it. Needs 'Manage inventory'.
- **`inventory_reorder_suggestions`**: Products below their minimum or at their dynamic reorder point (lead time × measured consumption), with the suggested quantity and vendor. Quantities and days only.
- **`inventory_stock_levels`**: Goods on hand per product and storage site (up to 2000 rows), by the same rule reordering uses: good stock only (not the bads bin), not counting stock held by a partner. Filter by product id, SKU or storage site.
- **`line_items_find_ssu_by_barcode`**: Resolve a scanned line-item barcode to the SSU (machine / location) whose storage unit holds it. 404 when the unit, its storage unit or that storage unit's SSU is unknown.
- **`machine_catalog_search_makes`**: Search the global catalog of machine manufacturers (e.g. 'Trumpf'). Use the existing spelling when it matches: everyone shares one catalog.
- **`machine_catalog_search_models`**: Search the models of one make in the global catalog.
- **`me_companies`**: Every EightyOne company you belong to, and which one this connection works in. A connection (an API key, or an MCP connection approved over OAuth) works in exactly one company: to work in another, use that company's own MCP URL (connect it as a separate connection and approve it for that company) or an API key created there.
- **`me_get`**: The caller: the user this credential acts as, the company it is bound to, how it is calling (REST or MCP; human, agent or service account) and the permissions it actually has here (the union of the user's roles' groups ∩ the credential's permissions). Call it first to know what you can do.
- **`meters_list`**: Usage meters on machines (run hours, strokes, cycles…) and their current readings. Usage-based maintenance rules open a task each time a meter advances by their interval. Filter by machine.
- **`meters_record_reading`** (changes data): Enter a meter's current total (not the change), as a technician reads it off the machine. It can open usage-based maintenance tasks: preview it with dryRun first to see which. Needs 'Manage maintenance'.
- **`ncrs_disposition`** (changes data): Decide what happens to nonconforming material, once per NCR. use_as_is: recorded only. rework: creates a rework task (of reworkTaskTypeId, or the NCR's own type). scrap: moves `quantity` of the lot to the bads bin and blocks the lot. return: blocks the lot for return to the vendor. Needs 'Disposition NCRs'.
- **`ncrs_open`** (changes data): Open a nonconformance report: a task of the company's NCR task type (Settings), linked to the task it was found on and carrying its machine and work order, with the affected lot or product recorded on it. A failed in-line check opens one by itself; use this for what people find. Disposition it with ncrs_disposition.
- **`partner_deliveries_receive`** (changes data): Book goods a partner sent you into stock, line by line (damaged quantities go to the bads bin), optionally reporting the rest missing.
- **`purchase_orders_list`**: Purchase orders (purchase requests): vendor, status, and each line's product, quantity ordered and received. Quantities only: EightyOne holds no prices.
- **`purchase_orders_receive`** (changes data): Book what arrived: per PO line, the quantity and the storage unit it goes to (with the lot for lot-tracked products). An inspection plan for the product or vendor puts it in quarantine and opens an inspection task. Can't receive more than ordered. Needs 'Receive inventory'. Preview it first.
- **`shipments_cancel`** (changes data): Cancel a shipment that hasn't shipped. Nothing moves: a draft holds no stock. Needs 'Manage inventory'.
- **`shipments_create`** (changes data): Draft a shipment to a customer: per line the SKU, quantity and the storage unit it ships from (and the lot for lot-tracked products). Nothing leaves stock until shipments_ship. Factories only. Needs 'Manage inventory'.
- **`shipments_get`**: A shipment with its lines.
- **`shipments_list`**: Shipments to customers, newest first: draft (being prepared), shipped or cancelled.
- **`shipments_ship`** (changes data): Confirm a draft shipment left: each line's goods come out of stock, written to the customer. For a customer that is an EightyOne company, the goods arrive in their transit. Refused if any line lacks the stock, names a lot that isn't available, or takes another customer's consigned stock. Needs 'Manage inventory'. Preview it first.
- **`ssus_get`**: One SSU by its external id, with the company's guidance note on it for agents (if any): read it before drawing conclusions about this machine.
- **`ssus_list`**: The company's SSUs (sub-site units: machines, assets, locations within a site), newest first.
- **`stock_transactions_list`**: The stock movement ledger (consumption, receipts, transfers, shipments), newest first, for pull-based ERP sync. Quantities only.
- **`stock_transfer`** (changes data): Move stock between storage units, or between bins in one unit, as the dashboard's transfers do: name the product by SKU, and the lot or serial when it's tracked. It can't move more than is there. Needs 'Manage inventory'.
- **`storage_units_get`**: A storage unit and everything in it: per product, bin (goods / bads / quarantine), lot and serial.
- **`storage_units_list`**: Where stock sits: each storage unit (shelf, rack, bay, dock…) with its storage site and whose it is: warehouse or site (the company's own), client, personnel (a technician's van), customer_consigned, partner.
- **`storylines_ssu`**: What happened on one machine (SSU) in a window (default: the last 7 days), as a chronological narrative: orders run, stops and their reasons, maintenance done, scrap and NCRs, meter readings, plus headline numbers. The best way to explain a machine's downtime.
- **`storylines_work_order`**: The life of one work order as a narrative: release, each operation started and finished, what was consumed and produced, scrap, quality checks and NCRs, shipment; with lead time and yield.
- **`task_types_get`**: One task type by its id.
- **`task_types_list`**: The company's task types: the templates every task is created from (work orders' operations, maintenance, NCRs, inspections, approvals…), alphabetically. Use a type's `id` as `typeId` when creating a task.
- **`tasks_connect`** (changes data): Link two tasks. 'dependency': the second waits for the first. It can't progress until the first is done, and goes back to open if it was already ready or dispatched (it becomes ready if the first is already done). 'follow': only records that the second follows the first. Connections are removed in the dashboard.
- **`tasks_create`** (changes data): Create a task of a task type, exactly as the dashboard does: the type's defaults apply (SLA due date, default assignee, transit, created-as-ready), and the task.created webhook fires. Needs 'Create tasks' unless the type is user-generated.
- **`tasks_get`**: One task by its id.
- **`tasks_get_form`**: The fields a task's form records, in order: type, whether it's required, the choices of a select, the limits of a measurement, whether a condition hides it, and the current answer. Answer them with tasks_submit_form.
- **`tasks_list`**: Tasks, newest first. Everything in EightyOne is a task (work-order operations, maintenance, inspections, NCRs, approvals…). Open tasks only unless a status is given or includeCompleted=true. Without the 'See all tasks' permission you see only tasks assigned to you, plus your teams' tasks with 'See tasks for my team'.
- **`tasks_record_counts`** (changes data): Add good and scrap quantities to a running work-order operation or producing task, as operators do from the app. Counts add up and feed OEE; the output posts to stock once, when the task completes. To count at completion instead, pass goodQuantity / scrapQuantity to tasks.update with the status.
- **`tasks_submit_form`** (changes data): Answer a task's form fields by title (tasks_get_form lists them), and optionally complete the task (status done or submitted) or mark it incomplete, as the app does. Completing needs every required field; measurements outside their limits open an NCR; machine-status fields set the machine's status. Counts and failure codes go with the completion, as in tasks_update. Preview it first.
- **`tasks_update`** (changes data): Change a task, with the dashboard's rules: status changes need 'Change task status from mobile' (assignees may still complete their own work), reassigning needs 'Reassign tasks', blocked tasks can't progress, producing tasks complete with goodQuantity / scrapQuantity (counts during the run: tasks_record_counts), maintenance types may need failure codes (problemCodeId, causeCodeId, remedyCodeId) to complete, 'incomplete' needs a reason and can create a follow-up (createFollowUp), 'declined' needs a reason. Tasks sent to a partner only accept details, dueDate and priorityId.
- **`work_orders_get`**: One work order with its operations (the traveler's tasks) and their status.
- **`work_orders_list`**: The company's work orders, newest first, optionally by status. Use for ERP pull-sync or to see what production is working on.
- **`work_orders_release`** (changes data): Release a draft work order to the floor: one operation task per routing step, each on the only machine of its work center when there is just one, with the type's parameters pinned and later steps waiting for gates. Needs 'Release work orders'. Preview it with dryRun first: it lists the operations it creates.

## More capabilities

Not listed as tools, to keep the list short. Find them with `capabilities_search` and run them with `capabilities_call` (pass the name below).

- `accounting.list_documents`: Documents sent to the books
- `accounting.providers`: Accounting providers
- `accounting.test`: Test the books connection
- `data_packages.get`: Get a data package
- `data_packages.list`: List data packages
- `external_refs.list`: List external ids
- `guidance.list`: List company guidance for agents
- `handoffs.get`: Get a handoff
- `handoffs.list`: List handoffs
- `handoffs.timeline`: Handoff timeline
- `machine_catalog.lookup_sku`: Find a machine by SKU
- `partner_deliveries.get`: Get a partner delivery
- `partner_deliveries.list`: List partner deliveries
- `reason_codes.list`: List reason codes
- `teams.list`: List teams
- `webhooks.delivery_logs`: Webhook delivery log
- `webhooks.event_types`: List webhook event types
- `webhooks.get`: Get a webhook
- `webhooks.list`: List webhooks
