# EightyOne capabilities

<!-- Generated from src/capabilities/registry.ts by `pnpm docs:api`. Do not edit by hand. -->

Every operation EightyOne exposes to code and agents. Each is one capability: the REST API (`/api/v1`, OpenAPI at `/api/v1/openapi.json`) and the MCP server (`/mcp`) are generated from these definitions, so they cannot disagree.

- **MCP:** read capabilities only for now. A capability is either a listed tool or reached through `capabilities_search` / `capabilities_call`; an agent sees only what its credential may run.

- **Auth:** `Authorization: Bearer <API key>` (or `X-API-Key`). The key's company is the company; a key acts as its owner, narrowed to the key's permissions.
- **Errors:** RFC 9457 `application/problem+json` with a stable `code`.
- **Writes:** send `Idempotency-Key: <unique>` to make a retry safe (24 h). Where marked, `Prefer: dry-run` runs the write, rolls it back and returns the planned changes.

## Account

### `me.get`: Who am I

The caller: the user this credential acts as, the company it is bound to, how it is calling (REST or MCP; human, agent or service account) and the permissions it actually has here (the union of the user's roles' groups ∩ the credential's permissions). Call it first to know what you can do.

**REST:** `GET /api/v1/me` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `me_get`

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `user` | object | yes |  |
| `company` | object | yes |  |
| `actorType` | `human` \| `agent` \| `service` | yes |  |
| `channel` | string | yes |  |
| `permissions` | string[] | yes |  |

### `me.companies`: My companies

Every EightyOne company you belong to, and which one this connection works in. A connection (an API key, or an MCP connection approved over OAuth) works in exactly one company: to work in another, use that company's own MCP URL (connect it as a separate connection and approve it for that company) or an API key created there.

**REST:** `GET /api/v1/me/companies` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `me_companies`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `shortName` | string \| null | yes |  |
| `current` | boolean | yes | The company this connection works in |
| `mcpUrl` | string \| null | yes | This company's MCP server URL (null if MCP isn't set up) |
| `apiKeysUrl` | string \| null | yes | Where to create an API key for this company |

## Guides

### `guide.get`: Read a guide

EightyOne concepts and how to read the data: the glossary, downtime, maintenance, production, inventory, quality, commissioning, what deserves attention, and this company's own guidance (topic "company"). Without a topic, lists the topics. Read the glossary before working on an unfamiliar area.

**REST:** `GET /api/v1/guides` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `guide_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `topic` | `glossary` \| `downtime` \| `maintenance` \| `production` \| `inventory` \| `quality` \| `commissioning` \| `flows` \| `signals` \| `company` |  | "company": what this company's administrators tell agents, including notes on specific machines |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `topic` | string \| null | yes |  |
| `title` | string | yes |  |
| `body` | string | yes | Markdown |
| `topics` | object[] | yes |  |

### `guidance.list`: List company guidance for agents

What the company's administrators tell every AI agent: company-wide guidance and notes on specific machines, work centers, products and task types, with any thresholds they set.

**REST:** `GET /api/v1/guidance` · **Permission:** any member of the company · **Effect:** read · **MCP:** via `capabilities_call`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `subject` | `company` \| `ssu` \| `work_center` \| `product` \| `task_type` | yes |  |
| `ref` | string \| null | yes | The SSU / product / task type external id, or the work center name |
| `name` | string \| null | yes | What it's about, by name |
| `body` | string | yes |  |
| `thresholds` | object | yes |  |
| `version` | integer | yes |  |
| `reviewedAt` | string (date-time) \| null | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `guidance.set`: Set company guidance for agents

Write the guidance agents get for the company, or for one machine, work center, product or task type. Replaces that subject's text; the previous version is kept in the audit log. Every agent connected to the company follows it.

**REST:** `PUT /api/v1/guidance` · **Permission:** `ai_guidance:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `subject` | `company` \| `ssu` \| `work_center` \| `product` \| `task_type` | yes |  |
| `ref` | string |  | Which one (not for company): SSU external id, work center name, product external id or SKU, task type external id |
| `body` | string | yes | Plain text or Markdown. Company: up to 4000 characters; one subject: 1000 |
| `thresholds` | object |  |  |
| `reviewed` | boolean \| `true` \| `false` |  | Mark it as reviewed now (agents see how fresh it is) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `subject` | `company` \| `ssu` \| `work_center` \| `product` \| `task_type` | yes |  |
| `ref` | string \| null | yes | The SSU / product / task type external id, or the work center name |
| `name` | string \| null | yes | What it's about, by name |
| `body` | string | yes |  |
| `thresholds` | object | yes |  |
| `version` | integer | yes |  |
| `reviewedAt` | string (date-time) \| null | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `guidance.delete`: Remove company guidance for agents

Remove the guidance for the company or one subject. The audit log keeps what it said.

**REST:** `DELETE /api/v1/guidance` · **Permission:** `ai_guidance:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `subject` | `company` \| `ssu` \| `work_center` \| `product` \| `task_type` | yes |  |
| `ref` | string |  |  |

## Tasks

### `tasks.list`: List tasks

Tasks, newest first. Everything in EightyOne is a task (work-order operations, maintenance, inspections, NCRs, approvals…). Open tasks only unless a status is given or includeCompleted=true. Each task names its assignees (personnel id, name, photo, color); assigneeId keeps one person's tasks, and the total counts them. Without the 'See all tasks' permission you see only tasks assigned to you, plus your teams' tasks with 'See tasks for my team'.

**REST:** `GET /api/v1/tasks` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `tasks_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |
| `status` | `open` \| `preparation` \| `ready` \| `dispatched` \| `submitted` \| `incomplete` \| `done` \| `declined` |  |  |
| `typeId` | string |  | Task type external id |
| `ssuId` | string |  | SSU external id |
| `assigneeId` | integer |  | Personnel id: only tasks this person is on |
| `includeCompleted` | boolean \| `true` \| `false` |  | Include done and declined tasks |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `title` | string | yes |  |
| `details` | string \| null | yes |  |
| `status` | string | yes |  |
| `startDate` | string (date-time) \| null | yes |  |
| `endDate` | string (date-time) \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `priorityId` | integer | yes |  |
| `typeId` | string \| null | yes | Task type external id |
| `team` | object \| null | yes | The team whose members see and take this task: its task type's team (change it on the type) |
| `assignees` | object[] | yes | The people the task is on (personnel), by name |
| `ssuId` | string \| null | yes | SSU external id |
| `externalSystemId` | string \| null | yes | Your id for the task |
| `incompleteReason` | string \| null | yes |  |
| `declinedReason` | string \| null | yes |  |
| `isPartialCompletion` | boolean | yes | Completed in part; a follow-up task carries the rest |
| `handoffRole` | `proxy` \| `mirror` \| null | yes | proxy: you sent it to a partner; mirror: a partner sent it to you |
| `problemCodeId` | integer \| null | yes | Failure coding: what was wrong |
| `causeCodeId` | integer \| null | yes | Failure coding: why |
| `remedyCodeId` | integer \| null | yes | Failure coding: what fixed it |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `tasks.get`: Get a task

One task by its id.

**REST:** `GET /api/v1/tasks/:id` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `tasks_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `title` | string | yes |  |
| `details` | string \| null | yes |  |
| `status` | string | yes |  |
| `startDate` | string (date-time) \| null | yes |  |
| `endDate` | string (date-time) \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `priorityId` | integer | yes |  |
| `typeId` | string \| null | yes | Task type external id |
| `team` | object \| null | yes | The team whose members see and take this task: its task type's team (change it on the type) |
| `assignees` | object[] | yes | The people the task is on (personnel), by name |
| `ssuId` | string \| null | yes | SSU external id |
| `externalSystemId` | string \| null | yes | Your id for the task |
| `incompleteReason` | string \| null | yes |  |
| `declinedReason` | string \| null | yes |  |
| `isPartialCompletion` | boolean | yes | Completed in part; a follow-up task carries the rest |
| `handoffRole` | `proxy` \| `mirror` \| null | yes | proxy: you sent it to a partner; mirror: a partner sent it to you |
| `problemCodeId` | integer \| null | yes | Failure coding: what was wrong |
| `causeCodeId` | integer \| null | yes | Failure coding: why |
| `remedyCodeId` | integer \| null | yes | Failure coding: what fixed it |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `tasks.create`: Create a task

Create a task of a task type, exactly as the dashboard does: the type's defaults apply (SLA due date, default assignee, transit, created-as-ready), and the task.created webhook fires. Needs 'Create tasks' unless the type is user-generated.

**REST:** `POST /api/v1/tasks` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `tasks_create`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `typeId` | string | yes | Task type external id |
| `title` | string | yes |  |
| `details` | string |  |  |
| `status` | `open` \| `preparation` \| `ready` \| `dispatched` \| `submitted` \| `incomplete` \| `done` |  |  |
| `startDate` | string (date-time) \| string (date) |  |  |
| `endDate` | string (date-time) \| string (date) |  |  |
| `dueDate` | string (date-time) \| string (date) |  | Defaults to the type's SLA for the priority |
| `priorityId` | integer |  |  |
| `ssuId` | string |  | SSU external id |
| `externalSystemId` | string |  |  |
| `fromStorageUnitId` | integer |  |  |
| `toStorageUnitId` | integer |  |  |
| `fromVendorId` | integer |  |  |
| `toVendorId` | integer |  |  |
| `assigneeIds` | integer[] |  | Personnel ids |
| `incompleteReason` | string |  | Required when status is 'incomplete' |
| `after` | object |  | Link the new task to an existing one: dependency (it waits for that task) or follow |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `title` | string | yes |  |
| `details` | string \| null | yes |  |
| `status` | string | yes |  |
| `startDate` | string (date-time) \| null | yes |  |
| `endDate` | string (date-time) \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `priorityId` | integer | yes |  |
| `typeId` | string \| null | yes | Task type external id |
| `team` | object \| null | yes | The team whose members see and take this task: its task type's team (change it on the type) |
| `assignees` | object[] | yes | The people the task is on (personnel), by name |
| `ssuId` | string \| null | yes | SSU external id |
| `externalSystemId` | string \| null | yes | Your id for the task |
| `incompleteReason` | string \| null | yes |  |
| `declinedReason` | string \| null | yes |  |
| `isPartialCompletion` | boolean | yes | Completed in part; a follow-up task carries the rest |
| `handoffRole` | `proxy` \| `mirror` \| null | yes | proxy: you sent it to a partner; mirror: a partner sent it to you |
| `problemCodeId` | integer \| null | yes | Failure coding: what was wrong |
| `causeCodeId` | integer \| null | yes | Failure coding: why |
| `remedyCodeId` | integer \| null | yes | Failure coding: what fixed it |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `tasks.update`: Update a task

Change a task, with the dashboard's rules: status changes need 'Change task status from mobile' (assignees may still complete their own work), reassigning needs 'Reassign tasks', blocked tasks can't progress, producing tasks complete with goodQuantity / scrapQuantity (counts during the run: tasks_record_counts), maintenance types may need failure codes (problemCodeId, causeCodeId, remedyCodeId) to complete, 'incomplete' needs a reason and can create a follow-up (createFollowUp), 'declined' needs a reason. Tasks sent to a partner only accept details, dueDate and priorityId.

**REST:** `PATCH /api/v1/tasks/:id` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `tasks_update`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `title` | string |  |  |
| `details` | string |  |  |
| `status` | `open` \| `preparation` \| `ready` \| `dispatched` \| `submitted` \| `incomplete` \| `done` \| `declined` |  |  |
| `startDate` | string (date-time) \| string (date) \| null |  |  |
| `endDate` | string (date-time) \| string (date) \| null |  |  |
| `dueDate` | string (date-time) \| string (date) \| null |  |  |
| `priorityId` | integer |  |  |
| `ssuId` | string \| null |  | SSU external id, or null to clear |
| `externalSystemId` | string |  |  |
| `assigneeIds` | integer[] |  | Personnel ids: replaces the assignees |
| `incompleteReason` | string |  |  |
| `declinedReason` | string |  |  |
| `createFollowUp` | boolean |  | With status 'incomplete': create a follow-up task for the rest |
| `goodQuantity` | number |  |  |
| `scrapQuantity` | number |  |  |
| `scrapReasonCodeId` | integer |  |  |
| `problemCodeId` | integer |  | Failure coding (reason_codes_list kind failure_problem); types that require it can't complete without all three |
| `causeCodeId` | integer |  | Failure coding: kind failure_cause |
| `remedyCodeId` | integer |  | Failure coding: kind failure_remedy |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `title` | string | yes |  |
| `details` | string \| null | yes |  |
| `status` | string | yes |  |
| `startDate` | string (date-time) \| null | yes |  |
| `endDate` | string (date-time) \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `priorityId` | integer | yes |  |
| `typeId` | string \| null | yes | Task type external id |
| `team` | object \| null | yes | The team whose members see and take this task: its task type's team (change it on the type) |
| `assignees` | object[] | yes | The people the task is on (personnel), by name |
| `ssuId` | string \| null | yes | SSU external id |
| `externalSystemId` | string \| null | yes | Your id for the task |
| `incompleteReason` | string \| null | yes |  |
| `declinedReason` | string \| null | yes |  |
| `isPartialCompletion` | boolean | yes | Completed in part; a follow-up task carries the rest |
| `handoffRole` | `proxy` \| `mirror` \| null | yes | proxy: you sent it to a partner; mirror: a partner sent it to you |
| `problemCodeId` | integer \| null | yes | Failure coding: what was wrong |
| `causeCodeId` | integer \| null | yes | Failure coding: why |
| `remedyCodeId` | integer \| null | yes | Failure coding: what fixed it |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |
| `_meta` | object | yes |  |

### `tasks.record_counts`: Record production counts

Add good and scrap quantities to a running work-order operation or producing task, as operators do from the app. Counts add up and feed OEE; the output posts to stock once, when the task completes. To count at completion instead, pass goodQuantity / scrapQuantity to tasks.update with the status.

**REST:** `POST /api/v1/tasks/:id/counts` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `tasks_record_counts`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `goodQuantity` | number |  |  |
| `scrapQuantity` | number |  |  |
| `scrapReasonCodeId` | integer |  | A scrap reason (reason_codes_list, kind scrap) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `goodQuantity` | number | yes |  |
| `scrapQuantity` | number | yes |  |
| `scrapReasonCodeId` | integer \| null | yes |  |

### `tasks.connect`: Connect two tasks

Link two tasks. 'dependency': the second waits for the first. It can't progress until the first is done, and goes back to open if it was already ready or dispatched (it becomes ready if the first is already done). 'follow': only records that the second follows the first. Connections are removed in the dashboard.

**REST:** `POST /api/v1/tasks/:from/connections` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `tasks_connect`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | yes | The first task's external id |
| `to` | string | yes | The external id of the task that waits for or follows it |
| `type` | `dependency` \| `follow` | yes |  |
| `description` | string |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string | yes |  |
| `to` | string | yes |  |
| `type` | string | yes |  |
| `description` | string \| null | yes |  |
| `toStatus` | string | yes | The second task's status after connecting |

### `tasks.delete`: Delete a task

Delete a task, as the dashboard does: a task you sent to a partner withdraws the handoff; a task a partner sent you can't be deleted (decline it instead).

**REST:** `DELETE /api/v1/tasks/:id` · **Permission:** `tasks:delete` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

### `tasks.get_form`: A task's form

The fields a task's form records, in order: type, whether it's required, the choices of a select, the limits of a measurement, whether a condition hides it, and the current answer. Answer them with tasks_submit_form.

**REST:** `GET /api/v1/tasks/:id/form` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `tasks_get_form`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `fields` | object[] | yes |  |

### `tasks.submit_form`: Fill in a task's form

Answer a task's form fields by title (tasks_get_form lists them), and optionally complete the task (status done or submitted) or mark it incomplete, as the app does. Completing needs every required field; measurements outside their limits open an NCR; machine-status fields set the machine's status. Counts and failure codes go with the completion, as in tasks_update. Preview it first.

**REST:** `POST /api/v1/tasks/:id/form` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `tasks_submit_form`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `answers` | object[] | yes | Answers by field title; fields left out keep their answer |
| `status` | `done` \| `submitted` \| `incomplete` |  | Complete it (done, or submitted for review) or mark it incomplete; leave out to only save the answers |
| `incompleteReason` | string |  |  |
| `createFollowUp` | boolean |  |  |
| `goodQuantity` | number |  |  |
| `scrapQuantity` | number |  |  |
| `scrapReasonCodeId` | integer |  |  |
| `problemCodeId` | integer |  |  |
| `causeCodeId` | integer |  |  |
| `remedyCodeId` | integer |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `title` | string | yes |  |
| `details` | string \| null | yes |  |
| `status` | string | yes |  |
| `startDate` | string (date-time) \| null | yes |  |
| `endDate` | string (date-time) \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `priorityId` | integer | yes |  |
| `typeId` | string \| null | yes | Task type external id |
| `team` | object \| null | yes | The team whose members see and take this task: its task type's team (change it on the type) |
| `assignees` | object[] | yes | The people the task is on (personnel), by name |
| `ssuId` | string \| null | yes | SSU external id |
| `externalSystemId` | string \| null | yes | Your id for the task |
| `incompleteReason` | string \| null | yes |  |
| `declinedReason` | string \| null | yes |  |
| `isPartialCompletion` | boolean | yes | Completed in part; a follow-up task carries the rest |
| `handoffRole` | `proxy` \| `mirror` \| null | yes | proxy: you sent it to a partner; mirror: a partner sent it to you |
| `problemCodeId` | integer \| null | yes | Failure coding: what was wrong |
| `causeCodeId` | integer \| null | yes | Failure coding: why |
| `remedyCodeId` | integer \| null | yes | Failure coding: what fixed it |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |
| `failedChecks` | string[] | yes |  |
| `ncrId` | string \| null | yes |  |

## Task types

### `task_types.list`: List task types

The company's task types: the templates every task is created from (work orders' operations, maintenance, NCRs, inspections, approvals…), alphabetically. Use a type's `id` as `typeId` when creating a task.

**REST:** `GET /api/v1/task-types` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `task_types_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task type external id |
| `typeName` | string | yes |  |
| `color` | string \| null | yes |  |
| `isQuote` | boolean | yes | A done task sends its products and quantities as an estimate to the connected books |
| `requestInvoiceOnDone` | boolean | yes | A done task sends an invoice request to the connected books |
| `isUserGenerated` | boolean | yes | Field users may create tasks of this type themselves |
| `isDataRequest` | boolean | yes | Tasks of this type ask a partner company for traceability data |
| `isRemote` | boolean | yes |  |
| `isInventory` | boolean | yes |  |
| `defaultUseTransit` | boolean | yes | Inventory tasks stage stock through the transit bin by default |
| `isBulk` | boolean | yes |  |
| `workCenterId` | integer \| null | yes |  |
| `team` | object \| null | yes | The team whose members see and take this type's tasks. Changing it moves every task of the type |
| `parentProductId` | integer \| null | yes | The product a production operation of this type makes |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `task_types.get`: Get a task type

One task type by its id.

**REST:** `GET /api/v1/task-types/:id` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `task_types_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task type external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task type external id |
| `typeName` | string | yes |  |
| `color` | string \| null | yes |  |
| `isQuote` | boolean | yes | A done task sends its products and quantities as an estimate to the connected books |
| `requestInvoiceOnDone` | boolean | yes | A done task sends an invoice request to the connected books |
| `isUserGenerated` | boolean | yes | Field users may create tasks of this type themselves |
| `isDataRequest` | boolean | yes | Tasks of this type ask a partner company for traceability data |
| `isRemote` | boolean | yes |  |
| `isInventory` | boolean | yes |  |
| `defaultUseTransit` | boolean | yes | Inventory tasks stage stock through the transit bin by default |
| `isBulk` | boolean | yes |  |
| `workCenterId` | integer \| null | yes |  |
| `team` | object \| null | yes | The team whose members see and take this type's tasks. Changing it moves every task of the type |
| `parentProductId` | integer \| null | yes | The product a production operation of this type makes |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

## Teams

### `teams.list`: List teams

The company's teams, alphabetically, with their members' personnel ids. A task belongs to its task type's team: team members with 'See tasks for my team' see those tasks and can take them in the field.

**REST:** `GET /api/v1/teams` · **Permission:** any member of the company · **Effect:** read · **MCP:** via `capabilities_call`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Team id |
| `name` | string | yes |  |
| `color` | string | yes |  |
| `memberIds` | integer[] | yes | Personnel ids of the members |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `teams.create`: Create a team

Create a team, optionally with its members. Point task types at it to give it work.

**REST:** `POST /api/v1/teams` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes |  |
| `color` | string |  | Hex color, e.g. #026485 |
| `memberIds` | integer[] |  | Personnel ids: replaces the members |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Team id |
| `name` | string | yes |  |
| `color` | string | yes |  |
| `memberIds` | integer[] | yes | Personnel ids of the members |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `teams.update`: Update a team

Rename or recolor a team, or replace its members (memberIds). Only the fields sent change. Members see the team's tasks at once.

**REST:** `PATCH /api/v1/teams/:id` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string |  |  |
| `color` | string |  | Hex color, e.g. #026485 |
| `memberIds` | integer[] |  | Personnel ids: replaces the members |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Team id |
| `name` | string | yes |  |
| `color` | string | yes |  |
| `memberIds` | integer[] | yes | Personnel ids of the members |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `teams.delete`: Delete a team

Delete a team and its memberships. Refused while a task type still belongs to it: move those types to another team (or none) first.

**REST:** `DELETE /api/v1/teams/:id` · **Permission:** `settings:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

## Personnel

### `personnel.list`: List personnel

The company's people, by name: their personnel id (the id tasks' assignees, teams' memberIds and assigneeIds use), photo, color, roles, teams, rank and addresses (home and current temporary bases). Active people only unless includeInactive=true. To count a person's work, use tasks_list with assigneeId. Needs 'View personnel'.

**REST:** `GET /api/v1/personnel` · **Permission:** `personnel:view` · **Effect:** read · **MCP:** tool `personnel_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |
| `includeInactive` | boolean \| `true` \| `false` |  | Include deactivated people |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Personnel id |
| `name` | string | yes |  |
| `imageUrl` | string \| null | yes |  |
| `color` | string | yes | The person's color on the dashboard's boards |
| `isActive` | boolean | yes |  |
| `rank` | integer | yes | Seniority 0–5; task types may require a minimum |
| `roles` | string[] | yes | Role (personnel title) names |
| `teams` | object[] | yes |  |
| `address` | object \| null | yes | Where the person starts from |
| `temporaryAddresses` | object[] | yes | Current temporary bases (expired ones are left out) |

### `personnel.get`: Get a person

One person by personnel id (a task's assignee id, a team's memberId): name, photo, color, roles, teams, rank and addresses. Needs 'View personnel'.

**REST:** `GET /api/v1/personnel/:id` · **Permission:** `personnel:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Personnel id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Personnel id |
| `name` | string | yes |  |
| `imageUrl` | string \| null | yes |  |
| `color` | string | yes | The person's color on the dashboard's boards |
| `isActive` | boolean | yes |  |
| `rank` | integer | yes | Seniority 0–5; task types may require a minimum |
| `roles` | string[] | yes | Role (personnel title) names |
| `teams` | object[] | yes |  |
| `address` | object \| null | yes | Where the person starts from |
| `temporaryAddresses` | object[] | yes | Current temporary bases (expired ones are left out) |

## Clients

### `clients.list`: List clients

The company's clients (the customers whose sites and SSUs work happens at), newest first. Filter by name or active flag.

**REST:** `GET /api/v1/clients` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `clients_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |
| `search` | string |  | Case-insensitive name contains |
| `isActive` | boolean \| `true` \| `false` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string \| null | yes | Your id for the client (its external system id). null for clients created in EightyOne without one: they are listed, but can't be fetched or changed by id over the API |
| `name` | string | yes |  |
| `isActive` | boolean | yes |  |
| `contactPersonName` | string \| null | yes |  |
| `contactPersonEmail` | string \| null | yes |  |
| `contactPersonPhone` | string \| null | yes |  |
| `freeText` | string \| null | yes |  |
| `color` | string \| null | yes |  |
| `isDefault` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `clients.get`: Get a client

One client by your id for it.

**REST:** `GET /api/v1/clients/:id` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `clients_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string \| null | yes | Your id for the client (its external system id). null for clients created in EightyOne without one: they are listed, but can't be fetched or changed by id over the API |
| `name` | string | yes |  |
| `isActive` | boolean | yes |  |
| `contactPersonName` | string \| null | yes |  |
| `contactPersonEmail` | string \| null | yes |  |
| `contactPersonPhone` | string \| null | yes |  |
| `freeText` | string \| null | yes |  |
| `color` | string \| null | yes |  |
| `isDefault` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `clients.create`: Create a client

Create a client with your own id for it (used to address it afterwards). Fires the client.created webhook.

**REST:** `POST /api/v1/clients` · **Permission:** `clients:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Your id for the client; must be unique in the company |
| `name` | string | yes |  |
| `contactPersonName` | string \| null |  |  |
| `contactPersonEmail` | string (email) \| null |  |  |
| `contactPersonPhone` | string \| null |  |  |
| `freeText` | string \| null |  |  |
| `color` | string |  |  |
| `isDefault` | boolean |  | Make this the company's default client (unsets the previous default) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string \| null | yes | Your id for the client (its external system id). null for clients created in EightyOne without one: they are listed, but can't be fetched or changed by id over the API |
| `name` | string | yes |  |
| `isActive` | boolean | yes |  |
| `contactPersonName` | string \| null | yes |  |
| `contactPersonEmail` | string \| null | yes |  |
| `contactPersonPhone` | string \| null | yes |  |
| `freeText` | string \| null | yes |  |
| `color` | string \| null | yes |  |
| `isDefault` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `clients.update`: Update a client

Change any of a client's fields; only the fields sent change.

**REST:** `PATCH /api/v1/clients/:id` · **Permission:** `clients:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string |  |  |
| `isActive` | boolean |  |  |
| `contactPersonName` | string \| null |  |  |
| `contactPersonEmail` | string (email) \| null |  |  |
| `contactPersonPhone` | string \| null |  |  |
| `freeText` | string \| null |  |  |
| `color` | string |  |  |
| `isDefault` | boolean |  | Make this the company's default client (unsets the previous default) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string \| null | yes | Your id for the client (its external system id). null for clients created in EightyOne without one: they are listed, but can't be fetched or changed by id over the API |
| `name` | string | yes |  |
| `isActive` | boolean | yes |  |
| `contactPersonName` | string \| null | yes |  |
| `contactPersonEmail` | string \| null | yes |  |
| `contactPersonPhone` | string \| null | yes |  |
| `freeText` | string \| null | yes |  |
| `color` | string \| null | yes |  |
| `isDefault` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `clients.delete`: Delete a client

Delete (archive) a client. Its history is kept.

**REST:** `DELETE /api/v1/clients/:id` · **Permission:** `clients:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

## SSUs

### `ssus.list`: List SSUs

The company's SSUs (sub-site units: machines, assets, locations within a site), newest first.

**REST:** `GET /api/v1/ssus` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `ssus_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | SSU external id (also its QR identity) |
| `name` | string | yes |  |
| `imageUrl` | string \| null | yes |  |
| `isActive` | boolean | yes |  |
| `isDefault` | boolean | yes |  |
| `location` | any \| null | yes | GeoJSON point (PostGIS geometry), or null |
| `make` | string \| null | yes | Machine make, from the global machine catalog (factories) |
| `model` | string \| null | yes | Machine model, from the global machine catalog |
| `sku` | string \| null | yes | Model-level SKU / part number (not a serial number) |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `ssus.get`: Get an SSU

One SSU by its external id, with the company's guidance note on it for agents (if any): read it before drawing conclusions about this machine.

**REST:** `GET /api/v1/ssus/:id` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `ssus_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | SSU external id (also its QR identity) |
| `name` | string | yes |  |
| `imageUrl` | string \| null | yes |  |
| `isActive` | boolean | yes |  |
| `isDefault` | boolean | yes |  |
| `location` | any \| null | yes | GeoJSON point (PostGIS geometry), or null |
| `make` | string \| null | yes | Machine make, from the global machine catalog (factories) |
| `model` | string \| null | yes | Machine model, from the global machine catalog |
| `sku` | string \| null | yes | Model-level SKU / part number (not a serial number) |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |
| `guidance` | object \| null | yes | What the company's administrators tell agents about this SSU |

### `ssus.update`: Update an SSU

Change an SSU's name, image, active flag, default flag or location. Making it the default unsets the default on the other SSUs of its sites.

**REST:** `PUT /api/v1/ssus/:id` · **Permission:** `clients:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string |  |  |
| `imageUrl` | string |  |  |
| `isActive` | boolean |  |  |
| `isDefault` | boolean |  |  |
| `location` | object |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | SSU external id (also its QR identity) |
| `name` | string | yes |  |
| `imageUrl` | string \| null | yes |  |
| `isActive` | boolean | yes |  |
| `isDefault` | boolean | yes |  |
| `location` | any \| null | yes | GeoJSON point (PostGIS geometry), or null |
| `make` | string \| null | yes | Machine make, from the global machine catalog (factories) |
| `model` | string \| null | yes | Machine model, from the global machine catalog |
| `sku` | string \| null | yes | Model-level SKU / part number (not a serial number) |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `ssus.deletion_blockers`: What keeps an SSU from being deleted

The history that keeps an SSU from being deleted (tasks, stock, downtime, signals, …), as labelled counts. Empty when it has none and may be deleted. A machine with history is kept; take it out of service with an offline status instead.

**Permission:** `clients:manage` · **Effect:** read · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | SSU external id |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `label` | string | yes |  |
| `count` | integer | yes | Stops at 1000 ("1000+") |

## Inventory

### `line_items.find_ssu_by_barcode`: Find the SSU holding a scanned unit

Resolve a scanned line-item barcode to the SSU (machine / location) whose storage unit holds it. 404 when the unit, its storage unit or that storage unit's SSU is unknown.

**REST:** `GET /api/v1/line-items/ssu` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `line_items_find_ssu_by_barcode`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `barcode` | string | yes | Line item barcode (code) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `imageUrl` | string \| null | yes |  |
| `isActive` | boolean | yes |  |
| `location` | any \| null | yes | GeoJSON point (PostGIS geometry), or null |

### `stock_transactions.list`: Export stock movements

The stock movement ledger (consumption, receipts, transfers, shipments), newest first, for pull-based ERP sync. Quantities only.

**REST:** `GET /api/v1/stock-transactions` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `stock_transactions_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) |  | Movements at or after this time |
| `to` | string (date-time) |  | Movements before this time |
| `limit` | integer |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `time` | string (date-time) | yes |  |
| `quantity` | number | yes |  |
| `bin` | string \| null | yes | goods, bads, in_use, transit, quarantine |
| `itemId` | integer \| null | yes | Line item (unit) id |
| `fromStorageUnitId` | integer \| null | yes |  |
| `toStorageUnitId` | integer \| null | yes |  |
| `fromVendorId` | integer \| null | yes |  |
| `toVendorId` | integer \| null | yes |  |
| `toClientId` | integer \| null | yes |  |
| `toCustomerId` | integer \| null | yes |  |
| `taskId` | integer \| null | yes |  |
| `lotId` | integer \| null | yes |  |
| `serialId` | integer \| null | yes |  |
| `reasonCodeId` | integer \| null | yes |  |
| `stockTransactionGroupId` | integer \| null | yes |  |

### `inventory.stock_levels`: Stock on hand

Goods on hand per product and storage site (up to 2000 rows), by the same rule reordering uses: good stock only (not the bads bin), not counting stock held by a partner. Filter by product id, SKU or storage site.

**REST:** `GET /api/v1/inventory/stock-levels` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `inventory_stock_levels`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `productId` | string |  | Product external id |
| `sku` | string |  | Product SKU (case-insensitive) |
| `storageSiteId` | string |  | Storage site external id |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `productId` | string | yes |  |
| `sku` | string \| null | yes |  |
| `name` | string | yes |  |
| `uom` | string \| null | yes |  |
| `itemClass` | string | yes | raw, wip, finished, operations or spare_part |
| `storageSiteId` | string \| null | yes |  |
| `storageSiteName` | string \| null | yes |  |
| `onHand` | number | yes |  |

### `inventory.reorder_suggestions`: Reorder suggestions

Products below their minimum or at their dynamic reorder point (lead time × measured consumption), with the suggested quantity and vendor. Quantities and days only.

**REST:** `GET /api/v1/inventory/reorder-suggestions` · **Permission:** `products:view` · **Effect:** read · **MCP:** tool `inventory_reorder_suggestions`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `productName` | string | yes |  |
| `sku` | string \| null | yes |  |
| `onHand` | number | yes |  |
| `minQty` | number | yes |  |
| `suggestedQty` | number | yes |  |
| `suggestedVendorName` | string \| null | yes |  |
| `vendorLeadTime` | number \| null | yes | Days |
| `dailyConsumption` | number \| null | yes |  |
| `reorderPoint` | number \| null | yes |  |
| `triggerReason` | `below_min` \| `below_reorder_point` | yes |  |

### `storage_units.list`: List storage units

Where stock sits: each storage unit (shelf, rack, bay, dock…) with its storage site and whose it is: warehouse or site (the company's own), client, personnel (a technician's van), customer_consigned, partner.

**REST:** `GET /api/v1/storage-units` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `storage_units_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `storageSiteId` | string |  | Storage site external id |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Storage unit external id |
| `name` | string | yes |  |
| `site` | object \| null | yes |  |

### `storage_units.get`: What's in a storage unit

A storage unit and everything in it: per product, bin (goods / bads / quarantine), lot and serial.

**REST:** `GET /api/v1/storage-units/:id` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `storage_units_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Storage unit external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Storage unit external id |
| `name` | string | yes |  |
| `site` | object \| null | yes |  |
| `contents` | object[] | yes |  |

### `stock.transfer`: Move stock

Move stock between storage units, or between bins in one unit, as the dashboard's transfers do: name the product by SKU, and the lot or serial when it's tracked. It can't move more than is there. Needs 'Manage inventory'.

**REST:** `POST /api/v1/stock/transfers` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `stock_transfer`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `sku` | string | yes |  |
| `quantity` | number | yes |  |
| `from` | string | yes | Storage unit external id it leaves |
| `to` | string | yes | Storage unit external id it goes to |
| `bin` | `goods` \| `bads` \| `quarantine` |  | The bin it's taken from |
| `toBin` | `goods` \| `bads` \| `quarantine` |  | The bin it lands in (default: the same) |
| `lotCode` | string |  | Which lot, for lot-tracked products |
| `serialCode` | string |  | Which unit, for serialized products |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `sku` | string | yes |  |
| `quantity` | number | yes |  |
| `from` | object | yes |  |
| `to` | object | yes |  |
| `bin` | string | yes |  |
| `toBin` | string | yes |  |
| `lotCode` | string \| null | yes |  |

### `purchase_orders.list`: List purchase orders

Purchase orders (purchase requests): vendor, status, and each line's product, quantity ordered and received. Quantities only: EightyOne holds no prices.

**REST:** `GET /api/v1/purchase-orders` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `purchase_orders_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | string |  | e.g. open, partially_received, received |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `name` | string \| null | yes |  |
| `status` | string | yes |  |
| `vendor` | string \| null | yes |  |
| `externalSystemId` | string \| null | yes |  |
| `lines` | object[] | yes |  |

### `purchase_orders.receive`: Receive against a purchase order

Book what arrived: per PO line, the quantity and the storage unit it goes to (with the lot for lot-tracked products). An inspection plan for the product or vendor puts it in quarantine and opens an inspection task. Can't receive more than ordered. Needs 'Receive inventory'. Preview it first.

**REST:** `POST /api/v1/purchase-orders/:id/receive` · **Permission:** `inventory:receive` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `purchase_orders_receive`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Purchase order id |
| `lines` | object[] | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `name` | string \| null | yes |  |
| `status` | string | yes |  |
| `vendor` | string \| null | yes |  |
| `externalSystemId` | string \| null | yes |  |
| `lines` | object[] | yes |  |

### `customers.list`: List customers

A factory's customers, who shipments go to: name, reference and ship-to address.

**REST:** `GET /api/v1/customers` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `customers_list`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `externalRef` | string \| null | yes |  |
| `shipToAddress` | string \| null | yes |  |
| `isEightyOneCompany` | boolean | yes | Shipments arrive in their EightyOne stock |

### `shipments.list`: List shipments

Shipments to customers, newest first: draft (being prepared), shipped or cancelled.

**REST:** `GET /api/v1/shipments` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `shipments_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `shipped` \| `cancelled` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `code` | string | yes |  |
| `status` | string | yes | draft / shipped / cancelled |
| `customer` | object \| null | yes |  |
| `shipToAddress` | string \| null | yes |  |
| `trackingReference` | string \| null | yes |  |
| `notes` | string \| null | yes |  |
| `shippedAt` | string (date-time) \| null | yes |  |
| `partnerDeliveryState` | string \| null | yes | For a customer that is an EightyOne company |
| `lines` | object[] | yes |  |

### `shipments.get`: Get a shipment

A shipment with its lines.

**REST:** `GET /api/v1/shipments/:id` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `shipments_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Shipment external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `code` | string | yes |  |
| `status` | string | yes | draft / shipped / cancelled |
| `customer` | object \| null | yes |  |
| `shipToAddress` | string \| null | yes |  |
| `trackingReference` | string \| null | yes |  |
| `notes` | string \| null | yes |  |
| `shippedAt` | string (date-time) \| null | yes |  |
| `partnerDeliveryState` | string \| null | yes | For a customer that is an EightyOne company |
| `lines` | object[] | yes |  |

### `shipments.create`: Prepare a shipment

Draft a shipment to a customer: per line the SKU, quantity and the storage unit it ships from (and the lot for lot-tracked products). Nothing leaves stock until shipments_ship. Factories only. Needs 'Manage inventory'.

**REST:** `POST /api/v1/shipments` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `shipments_create`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `customerId` | string | yes | Customer external id (customers_list) |
| `lines` | object[] | yes |  |
| `shipToAddress` | string |  | Defaults to the customer's |
| `code` | string |  |  |
| `trackingReference` | string |  |  |
| `notes` | string |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `code` | string | yes |  |
| `status` | string | yes | draft / shipped / cancelled |
| `customer` | object \| null | yes |  |
| `shipToAddress` | string \| null | yes |  |
| `trackingReference` | string \| null | yes |  |
| `notes` | string \| null | yes |  |
| `shippedAt` | string (date-time) \| null | yes |  |
| `partnerDeliveryState` | string \| null | yes | For a customer that is an EightyOne company |
| `lines` | object[] | yes |  |

### `shipments.ship`: Ship it

Confirm a draft shipment left: each line's goods come out of stock, written to the customer. For a customer that is an EightyOne company, the goods arrive in their transit. Refused if any line lacks the stock, names a lot that isn't available, or takes another customer's consigned stock. Needs 'Manage inventory'. Preview it first.

**REST:** `POST /api/v1/shipments/:id/ship` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `shipments_ship`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Shipment external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `code` | string | yes |  |
| `status` | string | yes | draft / shipped / cancelled |
| `customer` | object \| null | yes |  |
| `shipToAddress` | string \| null | yes |  |
| `trackingReference` | string \| null | yes |  |
| `notes` | string \| null | yes |  |
| `shippedAt` | string (date-time) \| null | yes |  |
| `partnerDeliveryState` | string \| null | yes | For a customer that is an EightyOne company |
| `lines` | object[] | yes |  |

### `shipments.cancel`: Cancel a draft shipment

Cancel a shipment that hasn't shipped. Nothing moves: a draft holds no stock. Needs 'Manage inventory'.

**REST:** `POST /api/v1/shipments/:id/cancel` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `shipments_cancel`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Shipment external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `code` | string | yes |  |
| `status` | string | yes | draft / shipped / cancelled |
| `customer` | object \| null | yes |  |
| `shipToAddress` | string \| null | yes |  |
| `trackingReference` | string \| null | yes |  |
| `notes` | string \| null | yes |  |
| `shippedAt` | string (date-time) \| null | yes |  |
| `partnerDeliveryState` | string \| null | yes | For a customer that is an EightyOne company |
| `lines` | object[] | yes |  |

### `inventory_counts.list`: List cycle counts

Cycle counts, newest first: which storage unit, status (draft, in_progress, completed = finalized and waiting to be reconciled, reconciled) and totals. inventory_counts_get shows the lines and discrepancies.

**REST:** `GET /api/v1/inventory-counts` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `inventory_counts_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `in_progress` \| `completed` \| `reconciled` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `storageUnit` | object \| null | yes |  |
| `status` | string | yes | draft / in_progress / completed (finalized, awaiting reconciliation) / reconciled |
| `countType` | string \| null | yes |  |
| `totalExpected` | number \| null | yes |  |
| `totalCounted` | number \| null | yes |  |
| `taskId` | string \| null | yes | The count task operators fill in, if any |
| `dueDate` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `reconciledAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |

### `inventory_counts.get`: Get a cycle count

A cycle count with its lines: expected (on hand by the system), counted, the discrepancy once finalized, and whether stock moved while it was open (a reason to recount rather than adjust).

**REST:** `GET /api/v1/inventory-counts/:id` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `inventory_counts_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `storageUnit` | object \| null | yes |  |
| `status` | string | yes | draft / in_progress / completed (finalized, awaiting reconciliation) / reconciled |
| `countType` | string \| null | yes |  |
| `totalExpected` | number \| null | yes |  |
| `totalCounted` | number \| null | yes |  |
| `taskId` | string \| null | yes | The count task operators fill in, if any |
| `dueDate` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `reconciledAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `inventory_counts.create`: Start a cycle count

Open a count of one storage unit. It snapshots what the system expects there, and when the company has an inventory task type it also creates the count task operators fill in from the app. Needs 'Manage inventory'.

**REST:** `POST /api/v1/inventory-counts` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `inventory_counts_create`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `storageUnitId` | string | yes | Storage unit external id (storage_units_list) |
| `title` | string |  | The count task's title |
| `dueDate` | string (date-time) \| string (date) |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `storageUnit` | object \| null | yes |  |
| `status` | string | yes | draft / in_progress / completed (finalized, awaiting reconciliation) / reconciled |
| `countType` | string \| null | yes |  |
| `totalExpected` | number \| null | yes |  |
| `totalCounted` | number \| null | yes |  |
| `taskId` | string \| null | yes | The count task operators fill in, if any |
| `dueDate` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `reconciledAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `inventory_counts.record_lines`: Record counted quantities

Add what someone counted to an open count, by SKU (and bin, lot or serial where it matters). Quantities add to what was already counted for that line, as in the dashboard. Then finalize it. Needs 'Manage inventory'.

**REST:** `POST /api/v1/inventory-counts/:id/lines` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `inventory_counts_record_lines`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Count id |
| `lines` | object[] | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `storageUnit` | object \| null | yes |  |
| `status` | string | yes | draft / in_progress / completed (finalized, awaiting reconciliation) / reconciled |
| `countType` | string \| null | yes |  |
| `totalExpected` | number \| null | yes |  |
| `totalCounted` | number \| null | yes |  |
| `taskId` | string \| null | yes | The count task operators fill in, if any |
| `dueDate` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `reconciledAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `inventory_counts.finalize`: Finish a cycle count

Close counting: each line's expected quantity becomes what's on hand now, discrepancies are computed, and anything on hand the count never saw becomes a line counted 0. Nothing is posted to stock until it's reconciled. Needs 'Manage inventory'.

**REST:** `POST /api/v1/inventory-counts/:id/finalize` · **Permission:** `inventory:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `inventory_counts_finalize`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `storageUnit` | object \| null | yes |  |
| `status` | string | yes | draft / in_progress / completed (finalized, awaiting reconciliation) / reconciled |
| `countType` | string \| null | yes |  |
| `totalExpected` | number \| null | yes |  |
| `totalCounted` | number \| null | yes |  |
| `taskId` | string \| null | yes | The count task operators fill in, if any |
| `dueDate` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `reconciledAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `inventory_counts.reconcile`: Post a cycle count to stock

Decide each line of a finalized count and post it. accepted: stock becomes the counted quantity; adjusted: becomes adjustedQuantity; ignored (and any line not listed): stock stays as the system has it. Every write-off needs an adjustment reasonCodeId (reason_codes_list kind adjustment). Needs 'Manage inventory' and 'Adjust inventory'. Preview it first: it lists each addition, write-off and bin move.

**REST:** `POST /api/v1/inventory-counts/:id/reconcile` · **Permission:** `inventory:manage` + `inventory:adjust` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `inventory_counts_reconcile`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `decisions` | object[] | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `storageUnit` | object \| null | yes |  |
| `status` | string | yes | draft / in_progress / completed (finalized, awaiting reconciliation) / reconciled |
| `countType` | string \| null | yes |  |
| `totalExpected` | number \| null | yes |  |
| `totalCounted` | number \| null | yes |  |
| `taskId` | string \| null | yes | The count task operators fill in, if any |
| `dueDate` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `reconciledAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

## Production

### `work_orders.list`: List work orders

The company's work orders, newest first, optionally by status. Use for ERP pull-sync or to see what production is working on.

**REST:** `GET /api/v1/work-orders` · **Permission:** `production:view` · **Effect:** read · **MCP:** tool `work_orders_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `draft` \| `released` \| `in_progress` \| `completed` \| `closed` \| `cancelled` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Work order external id |
| `code` | string | yes | Human code, e.g. WO-1009 |
| `externalSystemId` | string \| null | yes | Your (ERP) id for the order |
| `source` | string | yes | dashboard or erp |
| `status` | `draft` \| `released` \| `in_progress` \| `completed` \| `closed` \| `cancelled` | yes |  |
| `product` | object \| null | yes |  |
| `quantity` | number | yes |  |
| `uom` | string \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `routing` | object \| null | yes |  |
| `releasedAt` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `work_orders.get`: Get a work order

One work order with its operations (the traveler's tasks) and their status.

**REST:** `GET /api/v1/work-orders/:externalId` · **Permission:** `production:view` · **Effect:** read · **MCP:** tool `work_orders_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `externalId` | string | yes | Work order external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Work order external id |
| `code` | string | yes | Human code, e.g. WO-1009 |
| `externalSystemId` | string \| null | yes | Your (ERP) id for the order |
| `source` | string | yes | dashboard or erp |
| `status` | `draft` \| `released` \| `in_progress` \| `completed` \| `closed` \| `cancelled` | yes |  |
| `product` | object \| null | yes |  |
| `quantity` | number | yes |  |
| `uom` | string \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `routing` | object \| null | yes |  |
| `releasedAt` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |
| `operations` | object[] | yes |  |

### `work_orders.release`: Release a work order

Release a draft work order to the floor: one operation task per routing step, each on the only machine of its work center when there is just one, with the type's parameters pinned and later steps waiting for gates. Needs 'Release work orders'. Preview it with dryRun first: it lists the operations it creates.

**REST:** `POST /api/v1/work-orders/:externalId/release` · **Permission:** `work_orders:release` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `work_orders_release`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `externalId` | string | yes | Work order external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Work order external id |
| `code` | string | yes | Human code, e.g. WO-1009 |
| `externalSystemId` | string \| null | yes | Your (ERP) id for the order |
| `source` | string | yes | dashboard or erp |
| `status` | `draft` \| `released` \| `in_progress` \| `completed` \| `closed` \| `cancelled` | yes |  |
| `product` | object \| null | yes |  |
| `quantity` | number | yes |  |
| `uom` | string \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `routing` | object \| null | yes |  |
| `releasedAt` | string (date-time) \| null | yes |  |
| `completedAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `work_orders.upsert`: Create or update a work order (ERP orders in)

Idempotent on externalSystemId: a new id creates a draft order; re-sending updates it while it is still a draft and is a no-op afterwards. Lands on the product's released routing when there is exactly one. Release it with work_orders.release.

**REST:** `POST /api/v1/work-orders` · **Permission:** `production:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `externalSystemId` | string | yes | Your (ERP) id for the order: the upsert key |
| `productId` | integer |  | EightyOne product id (or give productSku) |
| `productSku` | string |  | Product SKU (case-insensitive) |
| `quantity` | number |  |  |
| `uom` | string |  |  |
| `dueDate` | string (date-time) \| string (date) |  |  |
| `code` | string |  | Human code; defaults to the next WO-n |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `workOrder` | object | yes |  |
| `created` | boolean | yes |  |

### `work_orders.batch_upsert`: Create or update many work orders

Upsert up to 500 orders; each succeeds or fails on its own (results in input order).

**REST:** `POST /api/v1/work-orders/batch` · **Permission:** `production:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `orders` | any[] | yes | Each item is a work_orders.upsert input; an invalid item fails on its own |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results` | object[] | yes |  |

## Webhooks

### `webhooks.list`: List webhooks

The company's webhook subscriptions.

**REST:** `GET /api/v1/webhooks` · **Permission:** `api_keys:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |
| `isActive` | boolean \| `true` \| `false` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `targetUrl` | string | yes |  |
| `eventTypes` | string[] | yes |  |
| `isActive` | boolean | yes |  |
| `failureCount` | integer | yes |  |
| `disabledAt` | string (date-time) \| null | yes | Set when repeated failures switched the webhook off |
| `headers` | object \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `webhooks.event_types`: List webhook event types

Every event a webhook can subscribe to, with what triggers it.

**REST:** `GET /api/v1/webhooks/events` · **Permission:** `api_keys:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `eventTypes` | object[] | yes |  |

### `webhooks.create`: Create a webhook

Subscribe a URL to events. Deliveries are Standard Webhooks messages signed with EightyOne's Ed25519 key: verify them with the public keys at /api/v1/webhook-signing-keys. There is no secret to store.

**REST:** `POST /api/v1/webhooks` · **Permission:** `webhooks:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes |  |
| `targetUrl` | string (uri) | yes | HTTPS endpoint; private and internal addresses are refused |
| `eventTypes` | `task.status_changed` \| `task.created` \| `client.created` \| `work_order.released` \| `work_order.completed` \| `downtime.started` \| `downtime.ended` \| `lot.created` \| `stock.reorder_triggered` \| `handoff.received` \| `handoff.status_changed` \| `handoff.declined` \| `handoff.deleted` \| `partner_delivery.incoming` \| `partner_delivery.received` \| `partner_delivery.shortfall` \| `data_package.received`[] | yes |  |
| `headers` | object |  | Extra headers sent with each delivery |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `targetUrl` | string | yes |  |
| `eventTypes` | string[] | yes |  |
| `isActive` | boolean | yes |  |
| `failureCount` | integer | yes |  |
| `disabledAt` | string (date-time) \| null | yes | Set when repeated failures switched the webhook off |
| `headers` | object \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |
| `verification` | object | yes | Verify each delivery's signature with EightyOne's public key |

### `webhooks.get`: Get a webhook

One webhook subscription.

**REST:** `GET /api/v1/webhooks/:id` · **Permission:** `api_keys:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `targetUrl` | string | yes |  |
| `eventTypes` | string[] | yes |  |
| `isActive` | boolean | yes |  |
| `failureCount` | integer | yes |  |
| `disabledAt` | string (date-time) \| null | yes | Set when repeated failures switched the webhook off |
| `headers` | object \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `webhooks.update`: Update a webhook

Change a webhook's name, URL, events, headers or active flag.

**REST:** `PATCH /api/v1/webhooks/:id` · **Permission:** `webhooks:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string |  |  |
| `targetUrl` | string (uri) |  |  |
| `eventTypes` | `task.status_changed` \| `task.created` \| `client.created` \| `work_order.released` \| `work_order.completed` \| `downtime.started` \| `downtime.ended` \| `lot.created` \| `stock.reorder_triggered` \| `handoff.received` \| `handoff.status_changed` \| `handoff.declined` \| `handoff.deleted` \| `partner_delivery.incoming` \| `partner_delivery.received` \| `partner_delivery.shortfall` \| `data_package.received`[] |  |  |
| `isActive` | boolean |  |  |
| `headers` | object |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `targetUrl` | string | yes |  |
| `eventTypes` | string[] | yes |  |
| `isActive` | boolean | yes |  |
| `failureCount` | integer | yes |  |
| `disabledAt` | string (date-time) \| null | yes | Set when repeated failures switched the webhook off |
| `headers` | object \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |

### `webhooks.delete`: Delete a webhook

Remove a webhook subscription.

**REST:** `DELETE /api/v1/webhooks/:id` · **Permission:** `webhooks:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

### `webhooks.delivery_logs`: Webhook delivery log

Recent deliveries of one webhook with their status, response and timing.

**REST:** `GET /api/v1/webhooks/:id/logs` · **Permission:** `api_keys:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |
| `status` | `pending` \| `success` \| `failed` \| `retrying` |  |  |
| `eventType` | `task.status_changed` \| `task.created` \| `client.created` \| `work_order.released` \| `work_order.completed` \| `downtime.started` \| `downtime.ended` \| `lot.created` \| `stock.reorder_triggered` \| `handoff.received` \| `handoff.status_changed` \| `handoff.declined` \| `handoff.deleted` \| `partner_delivery.incoming` \| `partner_delivery.received` \| `partner_delivery.shortfall` \| `data_package.received` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `eventType` | string | yes |  |
| `eventId` | string | yes |  |
| `payload` | any | yes |  |
| `status` | string | yes |  |
| `attemptCount` | integer | yes |  |
| `responseStatusCode` | integer \| null | yes |  |
| `responseBody` | string \| null | yes |  |
| `responseTimeMs` | integer \| null | yes |  |
| `errorMessage` | string \| null | yes |  |
| `nextRetryAt` | string (date-time) \| null | yes |  |
| `deliveredAt` | string (date-time) \| null | yes |  |
| `createdAt` | string (date-time) | yes |  |

### `webhooks.test`: Send a test event

Queue a test delivery to THIS webhook only: a signed message of type webhook.test, to check your endpoint and its signature verification.

**REST:** `POST /api/v1/webhooks/:id/test` · **Permission:** `webhooks:manage` · **Effect:** write · **Idempotency-Key:** not accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `message` | string | yes |  |
| `webhookId` | string | yes |  |

### `webhooks.retry_delivery`: Retry a delivery

Re-send one logged delivery of this webhook.

**REST:** `POST /api/v1/webhooks/:id/logs/:logId/retry` · **Permission:** `webhooks:manage` · **Effect:** write · **Idempotency-Key:** not accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `logId` | integer | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `message` | string | yes |  |
| `deliveryLogId` | integer | yes |  |

## Partner network

### `handoffs.send`: Send a task to a partner company

Hand one of your tasks to another EightyOne company (by its short name, or a partner subcontractor). The partner gets its own copy; your people come off the task. Goods can travel with it (items), and a data-request task carries the scopes it asks for.

**REST:** `POST /api/v1/handoffs` · **Permission:** `handoffs:send` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `handoffs_send`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `taskId` | string | yes | Your task's id |
| `recipientShortName` | string |  |  |
| `subContractorId` | integer |  | A subcontractor that is an EightyOne company |
| `items` | object[] |  |  |
| `dataRequest` | object |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string | yes | This company's own task (never the partner's) |
| `status` | string | yes |  |
| `declinedReason` | string \| null | yes |  |
| `requestedItems` | object[] | yes |  |
| `dataRequest` | object \| null | yes |  |
| `deliveries` | object[] | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |

### `handoffs.list`: List handoffs

Tasks you sent to, or received from, partner companies (newest first, up to 500).

**REST:** `GET /api/v1/handoffs` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `direction` | `incoming` \| `outgoing` \| `all` |  |  |
| `state` | `active` \| `withdrawn` \| `deleted` |  |  |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string | yes | This company's own task (never the partner's) |
| `status` | string | yes |  |
| `declinedReason` | string \| null | yes |  |
| `requestedItems` | object[] | yes |  |
| `dataRequest` | object \| null | yes |  |
| `deliveries` | object[] | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |

### `handoffs.get`: Get a handoff

One handoff from your side.

**REST:** `GET /api/v1/handoffs/:id` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string | yes | This company's own task (never the partner's) |
| `status` | string | yes |  |
| `declinedReason` | string \| null | yes |  |
| `requestedItems` | object[] | yes |  |
| `dataRequest` | object \| null | yes |  |
| `deliveries` | object[] | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |

### `handoffs.timeline`: Handoff timeline

What happened on a handoff, in order; your own people are named, the partner's are not.

**REST:** `GET /api/v1/handoffs/:id/timeline` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `type` | string | yes |  |
| `at` | string (date-time) | yes |  |
| `by` | `you` \| `partner` | yes |  |
| `actorName` | string \| null | yes | Only your own people are named |
| `payload` | any | yes |  |

### `handoffs.decline`: Decline a received handoff

Decline a task a partner sent you, with a reason they will see.

**REST:** `POST /api/v1/handoffs/:id/decline` · **Permission:** `handoffs:send` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `handoffs_decline`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `reason` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string | yes | This company's own task (never the partner's) |
| `status` | string | yes |  |
| `declinedReason` | string \| null | yes |  |
| `requestedItems` | object[] | yes |  |
| `dataRequest` | object \| null | yes |  |
| `deliveries` | object[] | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |

### `handoffs.withdraw`: Withdraw a sent handoff

Take back an active handoff you sent; the partner's copy is removed.

**REST:** `DELETE /api/v1/handoffs/:id` · **Permission:** `handoffs:send` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

### `partner_deliveries.list`: List partner deliveries

Goods sent to or received from partner companies, from your side.

**REST:** `GET /api/v1/partner-deliveries` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `direction` | `incoming` \| `outgoing` \| `all` |  |  |
| `state` | `in_transit` \| `partially_received` \| `shortfall` \| `received` \| `reversed` \| `refused` \| `unmapped` \| `held` \| `settled` |  |  |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `kind` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string \| null | yes |  |
| `handoffId` | string \| null | yes |  |
| `shipmentCode` | string \| null | yes |  |
| `returnOf` | string \| null | yes |  |
| `receivingStorageUnit` | object \| null | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `partner_deliveries.get`: Get a partner delivery

One delivery from your side.

**REST:** `GET /api/v1/partner-deliveries/:id` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `kind` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string \| null | yes |  |
| `handoffId` | string \| null | yes |  |
| `shipmentCode` | string \| null | yes |  |
| `returnOf` | string \| null | yes |  |
| `receivingStorageUnit` | object \| null | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `partner_deliveries.receive`: Receive a partner delivery

Book goods a partner sent you into stock, line by line (damaged quantities go to the bads bin), optionally reporting the rest missing.

**REST:** `POST /api/v1/partner-deliveries/:id/receive` · **Permission:** `handoffs:send` + `inventory:receive` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `partner_deliveries_receive`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `lines` | object[] |  |  |
| `reportRestMissing` | boolean |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `kind` | string | yes |  |
| `direction` | `incoming` \| `outgoing` | yes |  |
| `state` | string | yes |  |
| `partner` | object | yes |  |
| `taskId` | string \| null | yes |  |
| `handoffId` | string \| null | yes |  |
| `shipmentCode` | string \| null | yes |  |
| `returnOf` | string \| null | yes |  |
| `receivingStorageUnit` | object \| null | yes |  |
| `linkActive` | boolean | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `lines` | object[] | yes |  |

### `data_packages.list`: List data packages

Traceability data packages you released to, or received from, partners.

**REST:** `GET /api/v1/data-packages` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `role` | `released` \| `received` |  |  |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `role` | string | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `from` | any | yes |  |
| `object` | any | yes |  |

### `data_packages.get`: Get a data package

A data package's content, with short-lived links to its photos.

**REST:** `GET /api/v1/data-packages/:id` · **Permission:** `handoffs:send` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `role` | string | yes |  |
| `createdAt` | string (date-time) | yes |  |
| `content` | any | yes |  |
| `files` | object[] | yes |  |

## Analytics

### `analytics.overview`: Factory performance overview

Headline numbers for a window: total downtime, failures, average availability and OEE, and how many machines are monitored. `unavailableMetrics` names OEE factors with no source data yet. Start here, then drill into analytics.downtime / analytics.reliability.

**REST:** `GET /api/v1/analytics/overview` · **Permission:** `insights:view` · **Effect:** read · **MCP:** tool `analytics_overview`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) \| string (date) |  | Window start (ISO 8601). Default: 30 days before `to` |
| `to` | string (date-time) \| string (date) |  | Window end, exclusive (ISO 8601). Default: now |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `range` | object | yes |  |
| `downtimeSeconds` | number | yes |  |
| `downtimeHours` | number | yes |  |
| `failures` | integer | yes |  |
| `avgAvailability` | number \| null | yes | 0–1 |
| `avgOee` | number \| null | yes | 0–1 |
| `monitoredSsus` | integer | yes |  |
| `unavailableMetrics` | string[] | yes |  |

### `analytics.downtime`: Downtime by machine

Time each machine (SSU) spent DOWN or in MAINTENANCE in the window, most first, with how many times it entered that state. For the reasons, follow up with downtime_events.list (reason-coded stoppages) or storylines.ssu (the machine's story).

**REST:** `GET /api/v1/analytics/downtime` · **Permission:** `insights:view` · **Effect:** read · **MCP:** tool `analytics_downtime`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) \| string (date) |  | Window start (ISO 8601). Default: 30 days before `to` |
| `to` | string (date-time) \| string (date) |  | Window end, exclusive (ISO 8601). Default: now |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string \| null | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `category` | string | yes | DOWN or MAINTENANCE |
| `seconds` | number | yes |  |
| `hours` | number | yes |  |
| `transitions` | integer | yes | Times the machine entered this state |

### `analytics.reliability`: Machine reliability (MTTR / MTBF)

Per machine: failures (entries into DOWN), mean time to repair and mean time between failures, in seconds, most failures first.

**REST:** `GET /api/v1/analytics/reliability` · **Permission:** `insights:view` · **Effect:** read · **MCP:** tool `analytics_reliability`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) \| string (date) |  | Window start (ISO 8601). Default: 30 days before `to` |
| `to` | string (date-time) \| string (date) |  | Window end, exclusive (ISO 8601). Default: now |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string \| null | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `failures` | integer | yes |  |
| `mttrSeconds` | number \| null | yes |  |
| `mtbfSeconds` | number \| null | yes |  |

### `analytics.oee`: OEE by machine

Overall equipment effectiveness per machine: availability, performance and quality (0–1) with good / scrap quantities. A factor is null when its source data is missing (no routed work, no counts).

**REST:** `GET /api/v1/analytics/oee` · **Permission:** `insights:view` · **Effect:** read · **MCP:** tool `analytics_oee`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) \| string (date) |  | Window start (ISO 8601). Default: 30 days before `to` |
| `to` | string (date-time) \| string (date) |  | Window end, exclusive (ISO 8601). Default: now |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string \| null | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `plannedSeconds` | number | yes |  |
| `downtimeSeconds` | number | yes |  |
| `availability` | number \| null | yes |  |
| `performance` | number \| null | yes | Can exceed 1 when the line beats its standards |
| `quality` | number \| null | yes |  |
| `goodQuantity` | number | yes |  |
| `scrapQuantity` | number | yes |  |
| `oee` | number \| null | yes |  |

### `analytics.failure_pareto`: Failure modes (Pareto)

What breaks most: completed failure-coded tasks grouped by problem code, most frequent first (top 25), with the machine that suffers each mode most.

**REST:** `GET /api/v1/analytics/failure-pareto` · **Permission:** `insights:view` · **Effect:** read · **MCP:** tool `analytics_failure_pareto`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) \| string (date) |  | Window start (ISO 8601). Default: 30 days before `to` |
| `to` | string (date-time) \| string (date) |  | Window end, exclusive (ISO 8601). Default: now |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `problemCodeId` | integer | yes |  |
| `code` | string \| null | yes |  |
| `label` | string | yes |  |
| `count` | integer | yes |  |
| `worstSsuName` | string \| null | yes |  |

## Machines

### `downtime_events.list`: List machine stoppages

Individual downtime events (newest first, up to 500): when each machine stopped, for how long, and the reason code someone classified it with. Use it to explain the totals from analytics.downtime. Filter by machine and window; `unclassifiedOnly` lists stops still waiting for a reason.

**REST:** `GET /api/v1/downtime-events` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `downtime_events_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string |  | SSU external id |
| `from` | string (date-time) \| string (date) |  | Stops that started at or after this time |
| `to` | string (date-time) \| string (date) |  | Stops that started before this time |
| `unclassifiedOnly` | boolean \| `true` \| `false` |  |  |
| `openOnly` | boolean \| `true` \| `false` |  | Only machines still stopped |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `startedAt` | string (date-time) | yes |  |
| `endedAt` | string (date-time) \| null | yes | null while the machine is still stopped |
| `durationSeconds` | number \| null | yes |  |
| `source` | `auto` \| `manual` | yes | auto: detected from machine telemetry; manual: reported by a person |
| `reason` | object \| null | yes | The downtime reason code, once someone classified the stop |
| `note` | string \| null | yes |  |

### `downtime_events.classify`: Give a stop its reason

Classify a machine stoppage with a downtime reason code (reason_codes_list, kind downtime), optionally with a note. Re-classifying replaces the reason. Find stops waiting for one with downtime_events_list and unclassifiedOnly.

**REST:** `POST /api/v1/downtime-events/:id/classify` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `downtime_events_classify`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Downtime event id |
| `reasonCodeId` | integer | yes | A downtime reason code id |
| `note` | string |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `startedAt` | string (date-time) | yes |  |
| `endedAt` | string (date-time) \| null | yes | null while the machine is still stopped |
| `durationSeconds` | number \| null | yes |  |
| `source` | `auto` \| `manual` | yes | auto: detected from machine telemetry; manual: reported by a person |
| `reason` | object \| null | yes | The downtime reason code, once someone classified the stop |
| `note` | string \| null | yes |  |

### `downtime_events.open`: Report a machine stopped

Record that a machine stopped now, for machines without telemetry (those report stops themselves). Close it with downtime_events_close when it runs again, and classify it with a reason. Needs 'Manage machine data'.

**REST:** `POST /api/v1/downtime-events` · **Permission:** `machine_data:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `downtime_events_open`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string | yes | SSU external id |
| `note` | string |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `startedAt` | string (date-time) | yes |  |
| `endedAt` | string (date-time) \| null | yes | null while the machine is still stopped |
| `durationSeconds` | number \| null | yes |  |
| `source` | `auto` \| `manual` | yes | auto: detected from machine telemetry; manual: reported by a person |
| `reason` | object \| null | yes | The downtime reason code, once someone classified the stop |
| `note` | string \| null | yes |  |

### `downtime_events.close`: Report a machine running again

End a stoppage now. Closing one that already ended changes nothing. Needs 'Manage machine data'.

**REST:** `POST /api/v1/downtime-events/:id/close` · **Permission:** `machine_data:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `downtime_events_close`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Downtime event id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `startedAt` | string (date-time) | yes |  |
| `endedAt` | string (date-time) \| null | yes | null while the machine is still stopped |
| `durationSeconds` | number \| null | yes |  |
| `source` | `auto` \| `manual` | yes | auto: detected from machine telemetry; manual: reported by a person |
| `reason` | object \| null | yes | The downtime reason code, once someone classified the stop |
| `note` | string \| null | yes |  |

### `downtime_events.summary`: Downtime by machine and reason

Recorded stoppages per machine in a window (default: the last 30 days), most stopped time first, each broken down by reason code: the direct answer to "which machines were down most, and why?". Counts only the part of each stop inside the window; a stop still open counts up to the window end. Complements analytics.downtime, which measures time in a DOWN / MAINTENANCE status.

**REST:** `GET /api/v1/downtime-events/summary` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `downtime_events_summary`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `from` | string (date-time) \| string (date) |  | Window start (ISO 8601). Default: 30 days before `to` |
| `to` | string (date-time) \| string (date) |  | Window end, exclusive (ISO 8601). Default: now |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `seconds` | number | yes |  |
| `hours` | number | yes |  |
| `stops` | integer | yes |  |
| `openStops` | integer | yes | Stops not yet ended |
| `byReason` | object[] | yes |  |

### `reason_codes.list`: List reason codes

The company's reason codes: why machines stop (downtime), failure problem / cause / remedy codes on maintenance tasks, scrap and stock-adjustment reasons.

**REST:** `GET /api/v1/reason-codes` · **Permission:** any member of the company · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `kind` | `downtime` \| `failure_problem` \| `failure_cause` \| `failure_remedy` \| `scrap` \| `adjustment` |  |  |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `kind` | string | yes |  |
| `code` | string \| null | yes |  |
| `label` | string | yes |  |
| `parentId` | integer \| null | yes |  |
| `isActive` | boolean | yes |  |

### `meters.list`: List machine meters

Usage meters on machines (run hours, strokes, cycles…) and their current readings. Usage-based maintenance rules open a task each time a meter advances by their interval. Filter by machine.

**REST:** `GET /api/v1/meters` · **Permission:** `maintenance:view` · **Effect:** read · **MCP:** tool `meters_list`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `ssuId` | string |  | SSU external id |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `key` | string | yes |  |
| `name` | string | yes |  |
| `uom` | string \| null | yes |  |
| `source` | `manual` \| `signal` | yes | signal: counted from machine data; manual: people enter readings |
| `currentValue` | number | yes |  |

### `meters.record_reading`: Record a meter reading

Enter a meter's current total (not the change), as a technician reads it off the machine. It can open usage-based maintenance tasks: preview it with dryRun first to see which. Needs 'Manage maintenance'.

**REST:** `POST /api/v1/meters/:id/readings` · **Permission:** `maintenance:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `meters_record_reading`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes | Meter id (meters_list) |
| `value` | number | yes | The meter's reading: its running total |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `ssuId` | string | yes | SSU external id |
| `ssuName` | string \| null | yes |  |
| `key` | string | yes |  |
| `name` | string | yes |  |
| `uom` | string \| null | yes |  |
| `source` | `manual` \| `signal` | yes | signal: counted from machine data; manual: people enter readings |
| `currentValue` | number | yes |  |

### `machine_catalog.search_makes`: Search machine makes

Search the global catalog of machine manufacturers (e.g. 'Trumpf'). Use the existing spelling when it matches: everyone shares one catalog.

**REST:** `GET /api/v1/machine-catalog/makes` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `machine_catalog_search_makes`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `q` | string |  | Part of the make's name; empty lists common makes |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `name` | string | yes |  |
| `matchType` | `exact` \| `prefix` \| `contains` \| `similar` \| null | yes | similar = a 'did you mean' suggestion for a likely misspelling |

### `machine_catalog.search_models`: Search machine models

Search the models of one make in the global catalog.

**REST:** `GET /api/v1/machine-catalog/models` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `machine_catalog_search_models`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `makeId` | integer | yes | Make id from machine_catalog.search_makes |
| `q` | string |  |  |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | integer | yes |  |
| `name` | string | yes |  |
| `matchType` | `exact` \| `prefix` \| `contains` \| `similar` \| null | yes | similar = a 'did you mean' suggestion for a likely misspelling |

### `machine_catalog.lookup_sku`: Find a machine by SKU

The make and model registered for a model-level SKU / part number (not a serial number).

**REST:** `GET /api/v1/machine-catalog/sku-lookup` · **Permission:** any member of the company · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `sku` | string |  |  |

**Output:** an array of

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `make` | object | yes |  |
| `model` | object | yes |  |

## Quality

### `ncrs.open`: Open an NCR

Open a nonconformance report: a task of the company's NCR task type (Settings), linked to the task it was found on and carrying its machine and work order, with the affected lot or product recorded on it. A failed in-line check opens one by itself; use this for what people find. Disposition it with ncrs_disposition.

**REST:** `POST /api/v1/ncrs` · **Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** preview (checks the change and lists its effects, without making it) · **MCP:** tool `ncrs_open`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `title` | string | yes | What's wrong, in a line |
| `details` | string |  | What was found: measurements, quantities, how it was noticed |
| `foundOn` | string |  | External id of the task it was found on (an operation or inspection): the NCR takes its machine and work order |
| `ssuId` | string |  | SSU external id, when not taken from foundOn |
| `workOrderId` | string |  | Work order external id, when not taken from foundOn |
| `lotCode` | string |  | The affected lot |
| `productSku` | string |  | The affected product |
| `priorityId` | integer |  |  |
| `assigneeIds` | integer[] |  | Personnel ids |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Task external id |
| `title` | string | yes |  |
| `details` | string \| null | yes |  |
| `status` | string | yes |  |
| `startDate` | string (date-time) \| null | yes |  |
| `endDate` | string (date-time) \| null | yes |  |
| `dueDate` | string (date-time) \| null | yes |  |
| `priorityId` | integer | yes |  |
| `typeId` | string \| null | yes | Task type external id |
| `team` | object \| null | yes | The team whose members see and take this task: its task type's team (change it on the type) |
| `assignees` | object[] | yes | The people the task is on (personnel), by name |
| `ssuId` | string \| null | yes | SSU external id |
| `externalSystemId` | string \| null | yes | Your id for the task |
| `incompleteReason` | string \| null | yes |  |
| `declinedReason` | string \| null | yes |  |
| `isPartialCompletion` | boolean | yes | Completed in part; a follow-up task carries the rest |
| `handoffRole` | `proxy` \| `mirror` \| null | yes | proxy: you sent it to a partner; mirror: a partner sent it to you |
| `problemCodeId` | integer \| null | yes | Failure coding: what was wrong |
| `causeCodeId` | integer \| null | yes | Failure coding: why |
| `remedyCodeId` | integer \| null | yes | Failure coding: what fixed it |
| `createdAt` | string (date-time) | yes |  |
| `updatedAt` | string (date-time) | yes |  |
| `foundOn` | string \| null | yes |  |
| `lotCode` | string \| null | yes |  |
| `productSku` | string \| null | yes |  |

### `ncrs.disposition`: Disposition an NCR

Decide what happens to nonconforming material, once per NCR. use_as_is: recorded only. rework: creates a rework task (of reworkTaskTypeId, or the NCR's own type). scrap: moves `quantity` of the lot to the bads bin and blocks the lot. return: blocks the lot for return to the vendor. Needs 'Disposition NCRs'.

**REST:** `POST /api/v1/ncrs/:id/disposition` · **Permission:** `ncr:disposition` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `ncrs_disposition`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | The NCR's task external id |
| `disposition` | `use_as_is` \| `rework` \| `scrap` \| `return` | yes |  |
| `lotCode` | string |  | The affected lot (scrap and return act on it) |
| `quantity` | number |  | For scrap: how much of the lot |
| `reworkTaskTypeId` | string |  | For rework: the task type's external id |
| `notes` | string |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `disposition` | `use_as_is` \| `rework` \| `scrap` \| `return` | yes |  |
| `lot` | object \| null | yes |  |
| `reworkTaskId` | string \| null | yes | The rework task's external id |

## Storylines

### `storylines.ssu`: A machine's story

What happened on one machine (SSU) in a window (default: the last 7 days), as a chronological narrative: orders run, stops and their reasons, maintenance done, scrap and NCRs, meter readings, plus headline numbers. The best way to explain a machine's downtime.

**REST:** `GET /api/v1/storylines/ssus/:id` · **Permission:** `maintenance:view` · **Effect:** read · **MCP:** tool `storylines_ssu`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | SSU external id |
| `from` | string (date-time) \| string (date) |  |  |
| `to` | string (date-time) \| string (date) |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `subject` | object | yes |  |
| `plotPoints` | object | yes | The story's headline numbers |
| `chapters` | object[] | yes |  |

### `storylines.work_order`: A work order's story

The life of one work order as a narrative: release, each operation started and finished, what was consumed and produced, scrap, quality checks and NCRs, shipment; with lead time and yield.

**REST:** `GET /api/v1/storylines/work-orders/:id` · **Permission:** `production:view` · **Effect:** read · **MCP:** tool `storylines_work_order`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | Work order external id |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `subject` | object | yes |  |
| `plotPoints` | object | yes | The story's headline numbers |
| `chapters` | object[] | yes |  |

## Users

### `invitations.revoke`: Revoke an invitation

Withdraw a pending invitation: it is revoked on WorkOS (the emailed link stops working) and the invitee is removed from the company, so the same address can be invited again.

**Permission:** `users:manage` · **Effect:** destructive · **Idempotency-Key:** not accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string (uuid) | yes | The invited user's id (from the company's invitee list) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | string | yes |  |

### `invitations.resend`: Resend an invitation

Email the invitee a new invitation with a fresh expiry, for one that expired or got lost. The previous invitation is revoked if it is still pending; the invitee keeps their role.

**Permission:** `users:manage` · **Effect:** write · **Idempotency-Key:** not accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string (uuid) | yes | The invited user's id (from the company's invitee list) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `email` | string | yes |  |
| `expiresAt` | string | yes |  |

## Issues

### `issues.list`: List reportable issues

The issues a member can report (e.g. Breakdown, Safety hazard). Each one is a form that files a task of its configured type on the chosen SSU.

**Permission:** any member of the company · **Effect:** read · **MCP:** no

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `issues` | object[] | yes |  |

### `issues.get_form`: Get an issue's report form

The fields of an issue's report form, in order, as configured in the form builder: the same form the issue's QR code opens.

**Permission:** any member of the company · **Effect:** read · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string | yes | The issue's id (from issues.list) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `items` | object[] | yes |  |

### `issues.upload_url`: Get an upload URL for an issue photo

A short-lived URL to upload one photo for an issue report, stored where the QR flow stores its photos. PUT the file to it, then submit the URL without its query string as the answer.

**Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** not accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string | yes | The issue's id (from issues.list) |
| `fileName` | string | yes |  |
| `contentType` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `uploadUrl` | string | yes |  |

### `issues.report`: Report an issue

File an issue report on an SSU, as its QR code would: creates a task of the issue's type with the answers attached, and applies any SSU status the form's machine-status fields set.

**Permission:** any member of the company · **Effect:** write · **Idempotency-Key:** not accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `issueId` | string | yes | The issue's id (from issues.list) |
| `ssuId` | string | yes | The SSU's id |
| `answers` | object[] | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `taskId` | string | yes |  |

### `issues.site_qr_codes`: Download a site's issue QR codes

A ZIP with the issue's QR code for every SSU on a site, one PNG per SSU named after it, to print and post at the machines. Returns a download URL that expires after an hour.

**Permission:** `clients:view` · **Effect:** read · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `siteId` | string | yes | The site's id |
| `issueId` | string | yes | The issue's id (from issues.list) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `zipUrl` | string | yes |  |

## Commissioning

### `config.export`: Export the factory setup

The company's setup as a factory spec: settings, SSU statuses, reason codes, work centers, sites with their storage units and SSU trees, warehouses, product categories, vendors, products by SKU with their vendors, recipe and stock policies, parameter sets, task types with their forms, SLAs and gates, routings, per-machine signals, meters, condition rules and spare parts, flows, issue reports, inspection plans, and people (permission groups, roles, teams, skills, existing people by email, subcontractors). Everything is referenced by name, so the spec can be applied to another company as a template. Start here before changing setup: edit the export, then config.plan and config.apply.

**REST:** `GET /api/v1/config/export` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `config_export`

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `version` | number | yes |  |
| `settings` | object |  |  |
| `ssuStatuses` | object[] |  |  |
| `reasonCodes` | object[] |  |  |
| `workCenters` | object[] |  |  |
| `sites` | object[] |  |  |
| `warehouses` | object[] |  |  |
| `productCategories` | object[] |  |  |
| `vendors` | object[] |  |  |
| `products` | object[] |  | Products by SKU, with their vendors, recipe and stock policies. No prices or costs |
| `parameterSets` | object[] |  |  |
| `taskTypes` | object[] |  | Kinds of work, each with its form, SLAs and gates. People (default assignee, skills) come later |
| `routings` | object[] |  | Step sequences that work orders follow |
| `machineData` | object[] |  | Per machine: signals, meters, condition rules and spare parts |
| `flows` | object[] |  | Automations: a task's status change creates the next task |
| `issues` | object[] |  | Issue reports people can file (from the app or a QR code) |
| `permissionGroups` | object[] |  |  |
| `roles` | object[] |  | What people are; each role grants one permission group |
| `teams` | object[] |  |  |
| `skills` | object[] |  |  |
| `people` | object[] |  | Users already in the company, by email: their roles, teams, skills. Specs never invite |
| `subcontractors` | object[] |  |  |
| `inspectionPlans` | object[] |  | Which receipts or products must be inspected, and how |
| `guidance` | object[] |  | What the company tells every AI agent: company-wide and on specific machines, work centers, products, task types |

### `config.validate`: Validate a factory spec

Checks a spec beyond its schema: duplicate names or SKUs, references to statuses, work centers, reason codes, storage, categories, vendors, SKUs, task types, parameter sets or permission groups that don't exist, ambiguous names, a product both bought and made, gates naming fields the form doesn't have. Doesn't change anything.

**REST:** `POST /api/v1/config/validate` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `config_validate`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `spec` | object | yes | The factory spec (export one with config.export to see the shape) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `valid` | boolean | yes |  |
| `problems` | object[] | yes |  |

### `config.plan`: Preview applying a factory spec

What config.apply would do, change by change: create, update (with each field's current and new value), unchanged, and extra (in the company but not in the spec). Doesn't change anything. Show this plan to the user before applying.

**REST:** `POST /api/v1/config/plan` · **Permission:** any member of the company · **Effect:** read · **MCP:** tool `config_plan`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `spec` | object | yes | The factory spec (export one with config.export to see the shape) |
| `prune` | boolean \| `true` \| `false` |  | Also remove what the spec leaves out of the sections it includes. Records with history (sites, SSUs, reason codes, products) are deactivated, not deleted; vendors, categories, task types and routings are removed only when nothing uses them; parameter sets are never removed. REST only |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `valid` | boolean | yes |  |
| `problems` | object[] | yes |  |
| `summary` | object | yes |  |
| `changes` | object[] | yes |  |

### `config.apply`: Apply a factory spec

Converges the company's setup to the spec, through the same rules as the dashboard; every change is audited. Safe to re-run: unchanged items are skipped. Not one transaction: if a change fails, the ones before it stay and the result says where it stopped; fix the cause and apply again. Always run config.plan first and get the user's go-ahead.

**REST:** `POST /api/v1/config/apply` · **Permission:** `settings:manage` or `task_types:manage` or `clients:manage` or `inventory:manage` or `products:manage` or `production:manage` or `mobile_builder:manage` or `machine_data:manage` or `maintenance:manage` or `flows:manage` or `quality:manage` or `permissions:manage` or `personnel:manage` or `subcontractors:manage` or `ai_guidance:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** tool `config_apply`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `spec` | object | yes | The factory spec (export one with config.export to see the shape) |
| `prune` | boolean \| `true` \| `false` |  | Also remove what the spec leaves out of the sections it includes. Records with history (sites, SSUs, reason codes, products) are deactivated, not deleted; vendors, categories, task types and routings are removed only when nothing uses them; parameter sets are never removed. REST only |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `complete` | boolean | yes |  |
| `applied` | object[] | yes |  |
| `failed` | object |  |  |
| `skipped` | integer | yes | Changes not attempted after a failure |

## External refs

### `external_refs.lookup`: Look up an external id

Translate between an outside system's id and an EightyOne record. Send externalId to find the record it stands for, or entityType + entityId to find the record's id in that system. 404 when they aren't linked.

**REST:** `GET /api/v1/external-refs/lookup` · **Permission:** `settings:view` · **Effect:** read · **MCP:** tool `external_refs_lookup`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string | yes | The outside system, lowercase: quickbooks, xero, or your own slug (sap, netsuite, acme-mes) |
| `externalType` | string | yes | The kind of object in that system, lowercase: customer, item, invoice, order… |
| `externalId` | string |  | The object's id in that system |
| `entityType` | `client` \| `customer` \| `vendor` \| `product` \| `task` \| `work_order` \| `purchase_order` \| `ssu` \| `storage_unit` |  | The kind of EightyOne record it stands for |
| `entityId` | string |  | The record's API id: a client's id, a product's SKU, a purchase order's number, otherwise its externalId |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string | yes |  |
| `externalType` | string | yes |  |
| `externalId` | string | yes |  |
| `entityType` | string | yes |  |
| `entityId` | string \| null | yes | The record's API id (null if the record has none) |
| `status` | string \| null | yes | The object's status in that system, for documents EightyOne created there |
| `syncedAt` | string (date-time) \| null | yes | Last time it was linked or synced |
| `updatedAt` | string (date-time) | yes |  |

### `external_refs.list`: List external ids

The company's links to outside systems, newest first. Filter by system, externalType, or one record (entityType + entityId) to see every id it has elsewhere.

**REST:** `GET /api/v1/external-refs` · **Permission:** `settings:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string |  | The outside system, lowercase: quickbooks, xero, or your own slug (sap, netsuite, acme-mes) |
| `externalType` | string |  | The kind of object in that system, lowercase: customer, item, invoice, order… |
| `entityType` | `client` \| `customer` \| `vendor` \| `product` \| `task` \| `work_order` \| `purchase_order` \| `ssu` \| `storage_unit` |  | The kind of EightyOne record it stands for |
| `entityId` | string |  | The record's API id: a client's id, a product's SKU, a purchase order's number, otherwise its externalId |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string | yes |  |
| `externalType` | string | yes |  |
| `externalId` | string | yes |  |
| `entityType` | string | yes |  |
| `entityId` | string \| null | yes | The record's API id (null if the record has none) |
| `status` | string \| null | yes | The object's status in that system, for documents EightyOne created there |
| `syncedAt` | string (date-time) \| null | yes | Last time it was linked or synced |
| `updatedAt` | string (date-time) | yes |  |

### `external_refs.upsert`: Link an external id

Map an outside object to an EightyOne record. Idempotent: linking the same pair again only updates status. Each side maps to one: an object already linked elsewhere moves to this record, and the record's previous object of this type in this system is unlinked.

**REST:** `PUT /api/v1/external-refs` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string | yes | The outside system, lowercase: quickbooks, xero, or your own slug (sap, netsuite, acme-mes) |
| `externalType` | string | yes | The kind of object in that system, lowercase: customer, item, invoice, order… |
| `externalId` | string | yes | The object's id in that system |
| `entityType` | `client` \| `customer` \| `vendor` \| `product` \| `task` \| `work_order` \| `purchase_order` \| `ssu` \| `storage_unit` | yes | The kind of EightyOne record it stands for |
| `entityId` | string | yes | The record's API id: a client's id, a product's SKU, a purchase order's number, otherwise its externalId |
| `status` | string \| null |  | The object's status over there (draft, sent, paid…): kept as given |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string | yes |  |
| `externalType` | string | yes |  |
| `externalId` | string | yes |  |
| `entityType` | string | yes |  |
| `entityId` | string \| null | yes | The record's API id (null if the record has none) |
| `status` | string \| null | yes | The object's status in that system, for documents EightyOne created there |
| `syncedAt` | string (date-time) \| null | yes | Last time it was linked or synced |
| `updatedAt` | string (date-time) | yes |  |

### `external_refs.delete`: Unlink an external id

Remove the link between an outside object and its EightyOne record. Neither the record nor the object is touched.

**REST:** `DELETE /api/v1/external-refs/:system/:externalType/:externalId` · **Permission:** `settings:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `system` | string | yes | The outside system, lowercase: quickbooks, xero, or your own slug (sap, netsuite, acme-mes) |
| `externalType` | string | yes | The kind of object in that system, lowercase: customer, item, invoice, order… |
| `externalId` | string | yes | The object's id in that system |

## Accounting

### `accounting.providers`: Accounting providers

The accounting systems EightyOne connects to (QuickBooks, Xero, Accurants), which are available, and the company's current connection (one at a time).

**REST:** `GET /api/v1/accounting/providers` · **Permission:** `settings:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `logoPath` | string \| null | yes | Relative to the API host, e.g. /accounting/logos/xero.svg |
| `available` | boolean | yes | Connectable today; otherwise coming soon |
| `connected` | boolean | yes |  |

### `accounting.connect`: Connect the books

Start connecting an accounting system: returns the provider's sign-in page. A person opens it and approves; the provider sends them back to EightyOne's Integrations page. Refused while another provider is connected.

**REST:** `POST /api/v1/accounting/connect` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** not accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `provider` | `quickbooks` \| `xero` \| `accurants` | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `authorizationUrl` | string | yes |  |

### `accounting.test`: Test the books connection

Calls the connected accounting system and returns its company name.

**REST:** `GET /api/v1/accounting/connection/test` · **Permission:** `settings:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `provider` | string | yes |  |
| `name` | string \| null | yes |  |

### `accounting.disconnect`: Disconnect the books

Disconnect the accounting system: its tokens are erased and nothing more is sent. Linked ids stay, so reconnecting the same books carries on where it left off.

**REST:** `DELETE /api/v1/accounting/connection` · **Permission:** `settings:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

### `accounting.sync`: Import from the books

Import customers (a factory's customers, a service company's clients), vendors and products from the connected books, by name, SKU and quantity: no prices. Linked records are updated; one with the same name or SKU is linked instead of duplicated. A new tracked product starts with the books' quantity on hand in the default storage.

**REST:** `POST /api/v1/accounting/sync` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `scopes` | `customers` \| `vendors` \| `products`[] |  | What to import (default: all) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `customers` | object |  |  |
| `vendors` | object |  |  |
| `products` | object |  |  |
| `problems` | string[] | yes | What couldn't be imported, and why (first 50) |

### `accounting.inventory_differences`: Stock: books vs EightyOne

Where the books' quantity on hand differs from EightyOne's, per linked tracked product, largest first. EightyOne counts its own storage (not stock at clients' sites). Reported only: neither side is changed.

**REST:** `GET /api/v1/accounting/inventory-differences` · **Permission:** `inventory:view` · **Effect:** read · **MCP:** tool `accounting_inventory_differences`

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `provider` | string | yes |  |
| `rows` | object[] | yes |  |
| `unlinked` | integer | yes | Tracked items in the books not linked to a product |

### `accounting.request_document`: Send a document to the books

Ask the connected books for an invoice (a done task, or a shipped shipment), an estimate (a quote task) or a purchase order (a purchase order). It's queued and sent in the background with retries: products and quantities go over, the books price them. Asking again returns the same document (a failed one is queued again).

**REST:** `POST /api/v1/accounting/documents` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `kind` | `estimate` \| `invoice` \| `purchase_order` | yes |  |
| `taskId` | string |  |  |
| `shipmentId` | string |  |  |
| `purchaseOrderId` | string |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `provider` | string | yes |  |
| `providerName` | string | yes |  |
| `kind` | `estimate` \| `invoice` \| `purchase_order` | yes |  |
| `source` | object | yes |  |
| `status` | `queued` \| `sending` \| `sent` \| `failed` | yes | queued: waiting for the worker; sent: created in the books; failed: see error, then retry |
| `trigger` | string | yes | manual, task_done or po_created |
| `attempts` | integer | yes |  |
| `error` | string \| null | yes |  |
| `number` | string \| null | yes | The document's number in the books |
| `documentStatus` | string \| null | yes | Its status in the books when it was created |
| `url` | string \| null | yes | Opens it in the accounting system |
| `createdAt` | string (date-time) | yes |  |
| `sentAt` | string (date-time) \| null | yes |  |

### `accounting.list_documents`: Documents sent to the books

Estimates, invoices and purchase orders EightyOne asked the books for, newest first, with their number, status and a link. Filter by one task, shipment or purchase order, or by status (failed ones show why).

**REST:** `GET /api/v1/accounting/documents` · **Permission:** `settings:view` · **Effect:** read · **MCP:** via `capabilities_call`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `taskId` | string |  |  |
| `shipmentId` | string |  |  |
| `purchaseOrderId` | string |  |  |
| `status` | `queued` \| `sending` \| `sent` \| `failed` |  |  |
| `limit` | integer |  | Page size (1–100, default 50) |
| `offset` | integer |  | Rows to skip |

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `provider` | string | yes |  |
| `providerName` | string | yes |  |
| `kind` | `estimate` \| `invoice` \| `purchase_order` | yes |  |
| `source` | object | yes |  |
| `status` | `queued` \| `sending` \| `sent` \| `failed` | yes | queued: waiting for the worker; sent: created in the books; failed: see error, then retry |
| `trigger` | string | yes | manual, task_done or po_created |
| `attempts` | integer | yes |  |
| `error` | string \| null | yes |  |
| `number` | string \| null | yes | The document's number in the books |
| `documentStatus` | string \| null | yes | Its status in the books when it was created |
| `url` | string \| null | yes | Opens it in the accounting system |
| `createdAt` | string (date-time) | yes |  |
| `sentAt` | string (date-time) \| null | yes |  |

### `accounting.retry_document`: Retry a failed document

Queue a failed document again now, after fixing what its error says (for example, linking a product).

**REST:** `POST /api/v1/accounting/documents/:id/retry` · **Permission:** `settings:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `provider` | string | yes |  |
| `providerName` | string | yes |  |
| `kind` | `estimate` \| `invoice` \| `purchase_order` | yes |  |
| `source` | object | yes |  |
| `status` | `queued` \| `sending` \| `sent` \| `failed` | yes | queued: waiting for the worker; sent: created in the books; failed: see error, then retry |
| `trigger` | string | yes | manual, task_done or po_created |
| `attempts` | integer | yes |  |
| `error` | string \| null | yes |  |
| `number` | string \| null | yes | The document's number in the books |
| `documentStatus` | string \| null | yes | Its status in the books when it was created |
| `url` | string \| null | yes | Opens it in the accounting system |
| `createdAt` | string (date-time) | yes |  |
| `sentAt` | string (date-time) \| null | yes |  |

## Access

### `access.list`: Who has programmatic access

Every person in the company with API keys or connected agent apps (OAuth clients such as Claude or Cursor), outsiders first: people who also belong to other EightyOne companies, which is how a system integrator's staff show up. For each: their keys (name, permissions, last used) and agent apps (last used, calls in the last 30 days, blocked or not).

**REST:** `GET /api/v1/access` · **Permission:** `permissions:manage` · **Effect:** read · **MCP:** no

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string | yes |  |
| `name` | string | yes |  |
| `email` | string | yes |  |
| `isServiceAccount` | boolean | yes |  |
| `otherCompanies` | integer | yes | Other EightyOne companies this person belongs to (integrators show up here) |
| `keys` | object[] \| null | yes | Null when the key service couldn't be reached |
| `agents` | object[] | yes |  |

### `access.revoke_key`: Revoke someone's API key

Revoke one person's API key in this company, at once. Their other keys and their own access stay.

**REST:** `DELETE /api/v1/access/:userId/keys/:keyId` · **Permission:** `permissions:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** not supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string (uuid) | yes | The person's user id (from access.list) |
| `keyId` | string | yes |  |

### `access.block_agent`: Block an agent app for someone

Stop one agent app (an OAuth client) for one person in this company, from their next call. They keep their dashboard access, their keys and their other agent apps. Undo with access.unblock_agent.

**REST:** `POST /api/v1/access/:userId/agents/block` · **Permission:** `permissions:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string (uuid) | yes | The person's user id (from access.list) |
| `clientId` | string | yes | The agent app's client id (from access.list) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string | yes |  |
| `clientId` | string | yes |  |
| `blocked` | boolean | yes |  |

### `access.unblock_agent`: Unblock an agent app

Let one person use an agent app in this company again.

**REST:** `POST /api/v1/access/:userId/agents/unblock` · **Permission:** `permissions:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string (uuid) | yes | The person's user id (from access.list) |
| `clientId` | string | yes | The agent app's client id (from access.list) |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `userId` | string | yes |  |
| `clientId` | string | yes |  |
| `blocked` | boolean | yes |  |

## Flows

### `flows.list`: List flows

The company's flows: automations that create the next task when a task reaches a status. Each with its starting task type, whether it's paused, and its size.

**REST:** `GET /api/v1/flows` · **Permission:** `flows:view` · **Effect:** read · **MCP:** tool `flows_list`

**Output:** `{ data, pagination }`, each item:

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `name` | string | yes |  |
| `enabled` | boolean | yes |  |
| `startsWith` | string | yes |  |
| `nodes` | integer | yes |  |
| `edges` | integer | yes |  |
| `version` | string | yes |  |

### `flows.simulate`: What a status change sets off

If a task of this type reaches this status, which tasks do the company's flows create, and what could those set off next? Follows every active flow to a depth, and flags loops and arrows paused flows would add. Use it before proposing a change. Each further step only happens when someone moves the created task to that status: flows fire on status changes, never by themselves.

**REST:** `GET /api/v1/flows/simulate` · **Permission:** `flows:view` · **Effect:** read · **MCP:** tool `flows_simulate`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `taskType` | string | yes | A task type, by name or external id |
| `status` | `open` \| `preparation` \| `ready` \| `dispatched` \| `submitted` \| `incomplete` \| `done` \| `declined` | yes |  |
| `depth` | integer |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `steps` | object[] | yes |  |
| `loops` | string[] | yes |  |
| `paused` | string[] | yes | Arrows in paused flows that would otherwise fire |

### `flows.get`: Get a flow

One flow: its task types on the canvas (with positions and whether each takes the triggering task's assignees) and its arrows (when a task of `from` reaches `onStatus`, a task of `to` is created). Task types by name.

**REST:** `GET /api/v1/flows/:id` · **Permission:** `flows:view` · **Effect:** read · **MCP:** tool `flows_get`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | The flow's external id |
| `name` | string | yes |  |
| `enabled` | boolean | yes | Paused flows create nothing |
| `startsWith` | string | yes | The task type that starts the flow |
| `nodes` | object[] | yes |  |
| `edges` | object[] | yes |  |
| `version` | string | yes | Send it back as expectedVersion so an edit made meanwhile isn't overwritten |

### `flows.create`: Create a flow

Create a flow starting with a task type (one flow per starting type), with its task types and arrows. Positions are laid out for the Flow Creator when left out. Refused when an arrow already exists in another flow (both would fire, D33). Preview with dryRun first.

**REST:** `POST /api/v1/flows` · **Permission:** `flows:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `flows_create`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes |  |
| `startsWith` | string | yes | A task type, by name or external id |
| `nodes` | object[] |  | Defaults to the starting type plus every type the arrows name |
| `edges` | object[] |  |  |
| `enabled` | boolean |  | Create it paused with false |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | The flow's external id |
| `name` | string | yes |  |
| `enabled` | boolean | yes | Paused flows create nothing |
| `startsWith` | string | yes | The task type that starts the flow |
| `nodes` | object[] | yes |  |
| `edges` | object[] | yes |  |
| `version` | string | yes | Send it back as expectedVersion so an edit made meanwhile isn't overwritten |

### `flows.update`: Edit a flow

Rename a flow, pause or resume it (enabled), and change its canvas: either send the whole canvas (nodes + edges, replacing it) or a list of changes (addNodes, removeNodes, addEdges, removeEdges; removing a task type removes its arrows). The starting task type can't change. Send expectedVersion from flows.get so a change made meanwhile isn't overwritten. Preview with dryRun first: the answer lists the arrows added and removed.

**REST:** `PATCH /api/v1/flows/:id` · **Permission:** `flows:manage` · **Effect:** write · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** tool `flows_update`

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
| `expectedVersion` | string |  |  |
| `name` | string |  |  |
| `enabled` | boolean |  | false pauses the flow: it creates nothing until resumed |
| `canvas` | object |  |  |
| `changes` | object |  |  |

**Output**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes | The flow's external id |
| `name` | string | yes |  |
| `enabled` | boolean | yes | Paused flows create nothing |
| `startsWith` | string | yes | The task type that starts the flow |
| `nodes` | object[] | yes |  |
| `edges` | object[] | yes |  |
| `version` | string | yes | Send it back as expectedVersion so an edit made meanwhile isn't overwritten |
| `changed` | object | yes |  |

### `flows.delete`: Delete a flow

Delete a flow: it stops creating tasks, and tasks it created stay. Never offered to agents (D27, D33): an agent pauses a flow instead (flows.update enabled: false) and tells the person to delete it in the Flow Creator.

**REST:** `DELETE /api/v1/flows/:id` · **Permission:** `flows:manage` · **Effect:** destructive · **Idempotency-Key:** accepted · **Dry run:** supported · **MCP:** no

**Input**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | string | yes |  |
