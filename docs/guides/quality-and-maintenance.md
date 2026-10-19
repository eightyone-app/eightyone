# Quality, maintenance and downtime

## Quality

- **Inspections** are tasks of inspection types. They're created:
  - by a routing step;
  - by an inspection plan on receiving (the receipt is quarantined until the inspection passes);
  - by people.
- **In-line checks:** a number field with limits (`lsl`, `usl`) is a measurement. A completed task with a measurement outside its limits opens an **NCR** of the company's NCR type, linked to it and carrying its machine and work order.
- **NCRs** are tasks too:
  - `ncrs.open` raises one from the task where a problem was found, with the affected lot or product;
  - `ncrs.disposition` records the decision, once per NCR, and needs **Disposition NCRs**.
- **Dispositions:**
  - **use as is:** recorded only;
  - **rework:** creates a rework task;
  - **scrap:** moves the quantity of the lot to the bads bin and blocks the lot;
  - **return:** blocks the lot for return to the vendor.

## Maintenance

- **Corrective work** is a task of a maintenance type, opened by a person, an issue report (from the app or a QR code), or a condition rule on machine data.
- **Failure coding:** maintenance types can require a problem, a cause and a remedy code to complete (`tasks.update` or `tasks.submit_form` with `problemCodeId`, `causeCodeId`, `remedyCodeId`). `analytics.failure_pareto` ranks failure modes.
- **Preventive maintenance** has three triggers:
  - **time-based:** a task type's recurrence schedules the next task when one completes;
  - **usage-based:** a meter (run hours, strokes, cycles) with a rule "every N" opens a task each time the meter advances by N;
  - **condition-based:** a rule on a live signal (temperature above 80) opens a task when it's crossed.
- **Meters:** manual meters take readings (`meters.record_reading`, the running total), and signal meters count from machine data. A reading's preview says which preventive tasks it would open.
- **Spare parts:** each machine lists its spare parts.

## Downtime

- **A stop** is one downtime event: when the machine stopped, when it ran again, and the reason it was given.
  - Machines with telemetry report their stops themselves (`auto`).
  - For others, people report them (`downtime_events.open` and `.close`).
- **Classifying** a stop (`downtime_events.classify`) gives it a downtime reason code. Agents classify only with the reason a person gives: an unclassified stop is a gap to report, not a guess to fill.
- **Analysis:**
  - `downtime_events.summary` gives stopped time per machine and reason;
  - `analytics.downtime` measures time spent in a DOWN or MAINTENANCE status;
  - `analytics.reliability` gives MTBF and MTTR.
