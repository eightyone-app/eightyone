# Model a production line

Goal: add a production line (or change one) so work orders can run on it, with machine data and maintenance.

Always use a spec that holds **only the sections you're changing.**
- A section you leave out isn't touched.
- Inside a section you include, items you don't list stay as they are. They show as `extra` in the plan.

## What a line needs

1. **The machines,** as a tree under their site. `status` and `workCenter` must exist (in the spec or the company). `criticality` goes from 1 to 5.

   ```json
   "sites": [{ "name": "Floor 2", "ssus": [
     { "name": "Line A", "workCenter": "Assembly", "status": "Running", "children": [
       { "name": "Press A1", "criticality": 4, "make": "Trumpf", "model": "TruBend 5130" }
     ]}
   ]}]
   ```

2. **A work center** per kind of operation (`workCenters: [{ "name": "Assembly" }]`). Task types and machines both point at it.

3. **The products:** what the line makes, with its `recipe` of components by SKU, and the bought components with their `vendors`. A made product has no vendors.

4. **One task type per operation,** with:
   - `workCenter` and `product` (the SKU it makes);
   - `setupMinutes` and `runMinutesPerUnit`;
   - optionally a `parameterSet` (machine settings shown at the point of work) and an `outputStorageUnit`;
   - its `form`: the checks the operator records (see [configure-task-types-and-forms.md](configure-task-types-and-forms.md)).

5. **A routing:** the steps, in order, each naming its task type. `blocksNext: true` makes a step a gate: later steps wait for it when the work order is released.

   ```json
   "routings": [{ "name": "Bracket line", "product": "BR-100", "steps": [
     { "name": "Cut", "taskType": "Laser cut" },
     { "name": "Bend", "taskType": "Press brake", "blocksNext": true },
     { "name": "Inspect", "taskType": "Final inspection" }
   ]}]
   ```

   A routing's product is set when the routing is created. Changing steps later **releases a new version**: work orders already released keep theirs. Say so in the plan you show.

## Machine data and maintenance

Use `machineData`, one entry per machine path:

- **`signals`:** what the machine connector reports. Declare them before go-live so everything is ready when data arrives.
  - Mark the one state signal downtime detection reads as `isRunState: true`, with the `runningValue` that means running.
  - Mark counters that drive usage-based maintenance as `isMeter: true`.
- **`meters`:** hand-read meters (hour meters).
- **`conditionRules`:** open a maintenance task automatically.
  - `threshold`: a signal above (`gt`) or below (`lt`) a limit;
  - `meter_delta`: every N units of a meter.
  - Each needs a `taskType`; `cooldownHours` limits repeats.
- **`spareParts`:** the machine's spares, by SKU.

## Check your work

- **`config_validate`** catches broken references. For example, a rule naming a signal the machine doesn't have, or a routing step whose task type doesn't exist.
- **`config_plan`:** show it and get the go-ahead.
- **`config_apply`,** then re-plan: no creates or updates means it converged.
- **Afterwards:**
  - `work_orders_list` shows released orders;
  - `ssus_get` on a machine shows its setup;
  - `storylines_ssu` shows a machine's history once it runs.
