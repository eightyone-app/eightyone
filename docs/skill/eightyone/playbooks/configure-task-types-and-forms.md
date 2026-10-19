# Configure task types and forms

Goal: create or change the kinds of work (task types): what people record (forms), how fast it's due (SLAs), what must be true before it moves on (gates), and what happens next (flows).

Start with `config_export` and copy the `taskTypes` you're changing. Plan only those.

## A task type

```json
{
  "name": "Press bracket",
  "workCenter": "Assembly", "product": "BR-100", "setupMinutes": 15, "runMinutesPerUnit": 0.5,
  "ssuStatusOnOpen": "Running", "ssuStatusOnComplete": "Idle",
  "requiresFailureCoding": false, "useSubmittedStatus": true,
  "team": "Day shift", "requiredSkill": "Press setup", "minRank": 2,
  "slas": [{ "priority": "standard", "hours": 24 }, { "priority": "urgent", "hours": 4 }],
  "gates": [{ "status": "done", "requiredFields": ["Die checked"], "requiredSignature": true }],
  "form": [
    { "title": "Die checked", "type": "checkbox", "required": true },
    { "title": "First-piece height", "type": "number",
      "options": [{ "id": "config", "uom": "mm", "target": 10, "lsl": 9.8, "usl": 10.2, "decimals": 2 }] },
    { "title": "Result", "type": "select",
      "options": [{ "id": "1", "title": "Pass", "type": "select" }, { "id": "2", "title": "Fail", "type": "select" }] },
    { "title": "Photo", "type": "camera" }
  ]
}
```

- **Names are keys.** Renaming a task type in a spec creates a new one. Rename in the dashboard instead.
- **Flags.**
  - `isInventory`: the task moves stock. List its from / to / bin / products fields in `form`: the dashboard adds them when the flag is switched on there, a spec doesn't.
  - `useSubmittedStatus`: completion is reviewed (through "submitted").
  - `requiresFailureCoding`: can't complete without problem / cause / remedy codes (use it for maintenance).
  - `recurrenceHours`: completing a task schedules the next one (time-based preventive maintenance).
- **`ssuStatusOnOpen` / `ssuStatusOnComplete`** set the machine's status when a task opens or completes. Statuses are the company's names.
- **People.** `team` makes it the team's work: members see and can take it. `defaultAssignee` and `defaultSubcontractor` are emails of existing people. `requiredSkill` and `minRank` limit who may do it.

## Forms

- **`form` replaces the whole form.** List every field, in order.
- **Field types:** text, short_text, number, checkbox, select, datetime, camera, signature, location, qr, segment, products, storage, bin, ssus, ssu_status, tag_chooser, tags, list, target_time.
- **`options` hold type-specific settings, as the Mobile Builder stores them.** Copy the shape from an export:
  - a `select` lists its choices;
  - a `number` its unit, target and limits (`lsl` / `usl`), which feed SPC charts.
- **Titles are how gates, reports and comparisons find a field.** Renaming one breaks the gates that name it and splits its history. Keep titles stable.
- **Data-request types** (`isDataRequest`) get a system "Data package" field automatically: don't list it.

## SLAs, gates, flows, issue reports

- **`slas`:** due time in hours per priority (`low`, `standard`, `high`, `urgent`; use the level number where names repeat).
- **`gates`:** rules for moving into a status (`done`, `submitted`, …). A gate can require:
  - form fields (by title);
  - a signature;
  - references;
  - a member of certain permission groups.

  `"gates": []` removes them.
- **`flows`** (top level): when a task of one type reaches a status, create a task of another.

  ```json
  "flows": [{ "name": "NCR follow-up", "startsWith": "Final inspection",
    "nodes": [{ "taskType": "Final inspection" }, { "taskType": "NCR" }],
    "edges": [{ "from": "Final inspection", "to": "NCR", "onStatus": "incomplete" }] }]
  ```

  One flow per starting task type. The canvas (`nodes`, `edges`) is replaced as a whole.
- **`issues`** (top level): what people can report from the app or a QR code. Each has the task type it opens, a priority and its own `form`.
- **Settings** can name the task types used for NCRs, kitting, document approval and ECNs: `settings.ncrTaskType` and so on.

## Check your work

1. **`config_validate`.** It reports, for example, a gate naming a field the form doesn't have, an unknown priority or team, or a person who isn't in the company.
2. **`config_plan`.** Show the form changes field by field. Values already recorded are kept; tasks not filled in yet get the new form.
3. **Get the go-ahead,** `config_apply`, then re-plan to confirm.
