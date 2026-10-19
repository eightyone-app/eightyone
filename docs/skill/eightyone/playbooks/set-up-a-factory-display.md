# Set up a factory display (andon)

Goal: the **SSU Status** page in the dashboard (factories) shows every machine's state live, on a screen the floor can see. The statuses have to be right and update by themselves.

Start with `config_export`. You'll change `ssuStatuses`, `taskTypes` and `machineData`.

## 1. The statuses

`ssuStatuses` are the company's own names for a machine's state, each in one of five **categories**: `RUNNING`, `IDLE`, `DOWN`, `MAINTENANCE`, `OFFLINE`.

- **The categories drive analytics:** downtime, availability, OEE. The names and colours are what people see.
- **Keep the list short:** "Running", "Idle", "Setup" (IDLE), "Down", "Maintenance", "Off". Set one as the default (`isDefault`).
- Ask the user what the floor calls these states.

## 2. What changes them

| Source | Where in the spec |
| --- | --- |
| Machine data | `machineData[].signals`: one state signal per machine with `isRunState: true` and its `runningValue`. Downtime is detected from it. |
| Work | `taskTypes[].ssuStatusOnOpen` / `ssuStatusOnComplete`: an operation opening sets the machine to "Running", and a repair opening sets it to "Maintenance". |
| People | A `ssu_status` field on a form, which sets the machine's status when the task completes. People can also change it on the page. |

A machine with neither telemetry nor tasks that set its status shows whatever was last set by hand. Say so to the user, and suggest a status field on the operators' forms.

## 3. The machine tree

The page groups machines as `sites` nest them: floors, then lines, then machines. Check the tree and the **criticality** (1–5) of each machine. Critical machines are the ones a display should make obvious.

## 4. Review and apply

`config_validate`, `config_plan`, then show the user each status and what sets it. Apply after their go-ahead. Then open **SSU Status** in the dashboard to check it, and put it on the floor screen.
