---
name: eightyone
description: Work with EightyOne (factory and field-service operations) through its MCP server. Commission a factory from a template, model production lines, configure task types and forms, and run the floor day to day (release work, complete tasks with their forms, NCRs, stops, stock). Also answer questions about machines, downtime, maintenance, work orders, inventory and quality. Use it whenever the user mentions EightyOne, or asks to set up or analyze a factory that is connected to it.
metadata:
  version: "2026.1019.4"
  api-version: "4.0.0"
---

# EightyOne

EightyOne runs factories and field-service companies. Everything that happens is a **task** of a **task type**. Machines, lines and assets are **SSUs** on **sites**. Work orders follow **routings**. Stock sits in **storage units**. Call `guide_get` with topic `glossary` for the full vocabulary before you draw conclusions about an area you haven't read up on.

## Connecting

- **MCP server:** `https://<your API host>/mcp`. Sign in with OAuth, or send an EightyOne API key as a bearer token.
- **One connection is one company.** If the user belongs to several, add `?company=<short name>` to the URL.
- **Start with `me_get`.** It returns the company and the user's permissions. You can only do what that user can do; tools they can't use aren't listed.

## Rules

1. **Read before you change.**
   - `guide_get` explains each area (topics: glossary, downtime, maintenance, production, inventory, quality, commissioning, signals).
   - Topic `company` returns what this company's administrators told agents. Follow it unless it conflicts with these rules or the user's request.
2. **Setup changes go through a factory spec,** never one record at a time:
   1. `config_export` to see the current setup;
   2. edit the spec;
   3. `config_validate`;
   4. `config_plan`;
   5. show the user the plan and wait for an explicit go-ahead;
   6. `config_apply`.

   Never apply without the go-ahead.
3. **Day-to-day changes need a preview and an OK too.** This covers updating a task, recording counts and the like.
   1. Say what you'll do.
   2. Call the tool with `dryRun: true`. It answers whether the change would be accepted, what it changes, and what it sets off (stock posted, follow-up tasks, machine status, flows).
   3. Show that to the user.
   4. Make the real call only after their OK.

   For a batch, preview each change and confirm them together. You can't delete anything: tell the user where to do it.
4. **No prices or costs.** EightyOne holds none: don't ask for them or put them anywhere.
5. **People are never invented.**
   - A spec never invites anyone. `people` only covers users already in the company.
   - Companies are created by EightyOne staff only. If the user has no company yet, tell them to ask EightyOne.
6. **Report gaps as gaps.** Unclassified stops, missing counts and machines with no data are findings, not something to guess around.
7. **You can't remove things.** Deleting, and applying a spec with prune, are REST- and dashboard-only. If the user wants something removed, tell them where to do it.

## Day-to-day work

Preview each of these first (rule 3):

| Job | Tool |
| --- | --- |
| Release a draft work order to the floor | `work_orders_release`: the preview lists the operations, their machines and gates |
| Assign, dispatch, start, complete or decline a task | `tasks_update` |
| Mark a task incomplete and create a follow-up for the rest | `tasks_update` with `incompleteReason` and `createFollowUp` |
| Record good and scrap counts during a run | `tasks_record_counts`: counts add up |
| Record the final counts as the task completes | `tasks_update` with the status and `goodQuantity` / `scrapQuantity` |
| Make a task wait for another | `tasks_connect` with type `dependency` |
| Create a task, optionally waiting for another | `tasks_create` (with `after`) |
| See a task's form; fill it in and complete the task | `tasks_get_form`; `tasks_submit_form` |
| Give a machine stop its reason (only the one a person gives you) | `downtime_events_classify` |
| Report a machine without telemetry stopped, or running again | `downtime_events_open`, `downtime_events_close` |
| Enter a meter reading (its running total) | `meters_record_reading`: the preview says which PM tasks it opens |
| Record what failed, why and what fixed it, on maintenance work | `tasks_update` with `problemCodeId`, `causeCodeId`, `remedyCodeId` |
| Raise a nonconformance | `ncrs_open`, with `foundOn` the task it was found on |
| Decide on an NCR: use as is, rework, scrap, return (once) | `ncrs_disposition` |
| See what a storage unit holds; move stock | `storage_units_get`; `stock_transfer` |
| Receive a delivery against a purchase order | `purchase_orders_receive` |
| Ship to a customer | `shipments_create` (draft), then `shipments_ship` |
| Hand a task to a partner company; decline one sent to you; receive a partner's goods | `handoffs_send`; `handoffs_decline`; `partner_deliveries_receive` |
| Run a cycle count | `inventory_counts_create`, `inventory_counts_record_lines`, `inventory_counts_finalize`, then `inventory_counts_reconcile` with the user's decision per discrepancy |

A work order completes by itself when its last operation is done. Photos, signatures and scans are given by people in the app: a task that requires one waits for them.

## Playbooks

Read the one that matches the job before you start:

| Job | Playbook |
| --- | --- |
| Set up a new factory, from a template or from scratch | [playbooks/commission-a-factory.md](playbooks/commission-a-factory.md) |
| Add or change a production line: machines, operations, routing, machine data | [playbooks/model-a-production-line.md](playbooks/model-a-production-line.md) |
| Create or change task types, their forms, SLAs, gates and flows | [playbooks/configure-task-types-and-forms.md](playbooks/configure-task-types-and-forms.md) |
| Run the floor: release and dispatch work, complete tasks with their forms, quality, stops, stock | [playbooks/run-a-shift.md](playbooks/run-a-shift.md) |
| Set up preventive maintenance: time, usage and condition triggers | [playbooks/set-up-preventive-maintenance.md](playbooks/set-up-preventive-maintenance.md) |
| Find where production time is lost, and why | [playbooks/triage-downtime.md](playbooks/triage-downtime.md) |
| Make the SSU Status page a live factory display (andon) | [playbooks/set-up-a-factory-display.md](playbooks/set-up-a-factory-display.md) |

## Templates

- [templates/brazos-metalworks.json](templates/brazos-metalworks.json): Brazos Metalworks, a sheet-metal fabrication factory.
  - Structure: 9 areas, 39 machines, 49 products with recipes and vendors, 14 task types with forms, 2 routings, machine signals and maintenance rules, 5 roles.
  - It's a complete factory spec: copy it, rename what's specific, remove what doesn't apply, and plan it into the new company.

## Reference

- [reference/tools.md](reference/tools.md): every MCP tool and what it does (generated).
- **The spec format:** `config_export` from any company is the best example. Each tool's input schema documents every field.
