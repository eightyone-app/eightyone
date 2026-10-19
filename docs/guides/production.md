# Production: work orders, routings, counts, lots

## The model

- **Products** have an item class (raw, wip, finished, operations, spare part), a unit, and a tracking mode: none, lot, or serial. A product is either **bought** (it has vendors) or **made** (it has a **recipe**: the components one unit consumes, in whole units).
- **Work centers** group machines that can run the same operations.
- **Task types** are the operations: cut, form, weld, inspect. Each has its standard setup and run minutes, the work center it runs in, its parameter set (machine settings, versioned) and its form.
- **Routings** list the steps a product goes through, each a task type. A step can be a **gate**: the steps after it wait until it's done. Changing a routing creates a new version; released orders keep the version they started with.
- **Work orders** ask for a quantity of a product. Status: `draft` → `released` → `in progress` → `completed` → `closed` (or `cancelled`).

## Release

Releasing a work order (`work_orders.release`, which needs **Release work orders**) creates one operation task per routing step:

- on the step's machine, when its work center has exactly one;
- with the parameter set's released version pinned;
- each step after a gate waiting on it, through a dependency.

Its preview lists each operation it would create. Work orders usually come from an ERP: `work_orders.upsert` and `work_orders.batch_upsert` create and update draft orders by your system's id.

## Counts

- **During a run,** `tasks.record_counts` adds good and scrap quantities. Counts add up and feed OEE.
- **At completion,** a producing task needs its counts. They come either with the completion or from the counts recorded during the run.
- **Completing posts the output:**
  - the good quantity goes into stock;
  - the recipe is consumed (backflushed) from the order's kit cart, then the machine's line-side storage, then any goods bin, earliest lot expiry first.
- A work order completes by itself when its last operation is done.

## Lots and genealogy

- Receiving a lot-tracked product records its lot.
- Completing a work order for a lot-tracked product creates the produced lot.
- Consumption links one to the other, so a finished lot traces back to its supplier lots.
- Blocked and quarantined lots are never consumed or shipped.

## Downtime and OEE

Machines report stops from telemetry, or people report them. Stops are classified with reason codes. `analytics.oee`, `analytics.downtime` and `downtime_events.summary` turn counts, stops and standard times into availability, performance and quality.
