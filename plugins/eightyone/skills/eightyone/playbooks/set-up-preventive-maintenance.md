# Set up preventive maintenance

Goal: machines get serviced before they fail, on time, on usage, or on condition, with the work recorded so it can be measured.

Start with `config_export`. You'll change `taskTypes` and `machineData` for the machines in scope. Read `guide_get` topic `maintenance`.

## 1. The work itself

A maintenance **task type** per kind of service, for example "Laser monthly service" or "Press 1000-stroke service". For each:

- a **form** with what the technician checks and records: checkboxes for each step, numbers for readings, a photo where it helps;
- `requiresFailureCoding: true` on **corrective** types (repairs), so every repair records a problem, a cause and a remedy. Leave it off for routine PM;
- the team or required skill, and SLAs per priority;
- `ssuStatusOnOpen` / `ssuStatusOnComplete` if the machine's status should show it's under maintenance.

## 2. The trigger

| Trigger | Where in the spec | Example |
| --- | --- | --- |
| **Time** | `recurrenceHours` on the task type: completing one schedules the next | Every 720 h (30 days) |
| **Usage** | `machineData[].meters` (manual) or a counter signal with `isMeter: true`, plus a `conditionRules` entry with `mode: "meter_delta"` | Every 1000 strokes, every 250 run hours |
| **Condition** | `machineData[].signals` plus a `conditionRules` entry with `operator` `gt` or `lt` and a `threshold` | Spindle temperature above 80 °C |

- Rules name a task type and a cooldown.
- A rule never stacks tasks: while its task is open, it doesn't open another.
- Ask the user for the intervals and limits. Don't invent them: they come from the machine's manual or the plant's practice.

## 3. Spare parts

`machineData[].spareParts` lists the parts each machine uses (SKU and quantity), so the parts for a service can be checked before it's due.

## 4. Review and apply

1. `config_validate`, then `config_plan`.
2. Show the user, per machine: the trigger, the interval or limit, and the task type it opens.
3. After their go-ahead, `config_apply`, then re-plan to confirm.

## Afterwards

- Technicians enter manual meter readings in the app, or you do with `meters_record_reading` from what they tell you. Its preview says whether it opens a task.
- `analytics_reliability` (MTBF and MTTR) and `analytics_failure_pareto` show whether it's working.
