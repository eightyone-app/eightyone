# Tasks: everything is a task

In EightyOne, every piece of work is a **task**: a work-order operation, a repair, a preventive service, an inspection, an NCR, a stock move, a count, an approval, a request to a partner. What kind of task it is comes from its **task type**.

## Task types

A task type defines a kind of work:

- **Its form:** the fields people fill in, ordered. Field types include text, number (with target and limits), checkbox, select, date and time, photo, signature, location, barcode scan, products, storage and machine status. A field can be **required**, and it can be shown only when another field has a given answer.
- **Its rules:**
  - **SLAs:** the due time per priority.
  - **Gates:** what must be true to enter a status, such as fields filled in, a signature, or a member of certain groups.
  - **Failure coding:** a problem, a cause and a remedy, required before completion on maintenance types.
  - **Production:** the product it makes, standard setup and run minutes, the work center and the parameter set.
  - **Who:** a team, a required skill and a minimum rank.
- **Automation:**
  - the machine status it sets when a task opens or completes;
  - **flows** (when a task of this type reaches a status, create a task of another type);
  - recurrence (time-based preventive maintenance).

## Statuses

`open` → `preparation` → `ready` → `dispatched` → `submitted` (when the type is reviewed) → `done`.

- **`incomplete`:** not finished, with a reason. It can create a follow-up task for the rest.
- **`declined`:** refused, with a reason.

Status changes follow the rules people get in the dashboard and the app:
- a blocked task can't move on;
- changing a status needs **Change task status from mobile**, though people can always complete their own tasks;
- reassigning needs **Reassign tasks**.

## Forms and completion

Completing a task is filling in its form and setting it to done. Through the API or an agent, that's `tasks.submit_form` with the answers and the status (`tasks.get_form` lists the fields first).

- **Required fields** must be filled. The server checks every completed form, from any client.
- **Measurements outside their limits** fail the check. The completed task then opens an NCR of the company's NCR type, linked to it.
- **Machine-status fields** set the machine's status.
- **Inventory tasks** post their stock moves.
- **Producing tasks** post their output and consume their recipe.
- **Photos, signatures and scans** are given by people in the app. A task that requires one waits for them.

## Connections

Tasks link to each other:

- **dependency:** the second task waits for the first. It can't progress until the first is done, and goes back to open if it was already ready or dispatched.
- **follow:** the second follows the first (follow-ups use this).
- **created from:** the second was made from the first (NCRs from a failed check, kitting from a task with products, rework from a disposition).

`tasks.create` takes `after: { taskId, type }` to create a task already linked.

## Who sees what

Members see their own tasks. Seeing every task needs **See all tasks**, and team members see their teams' work. The API and agents see exactly what the person they act as sees.
