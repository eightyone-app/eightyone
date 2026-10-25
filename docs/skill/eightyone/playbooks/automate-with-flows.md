# Automate a process with flows

Goal: when a task reaches a status, the next task is created by itself: an inspection that fails creates rework; finished rework creates a re-inspection; a declined job creates a re-planning task. Read `guide_get` topic `flows` first.

## 1. Understand what exists

1. `flows_list`: the company's flows, which are paused, how big each is.
2. `flows_get` on the ones that touch the process. Keep each one's `version`.
3. `flows_simulate` for the status changes the user cares about (for example `Inspection` reaching `incomplete`). It shows what gets created, what that sets off next, loops, and arrows in paused flows.

Tell the user what happens today, in their words: "When an inspection is marked incomplete, a Rework task is created for the same machine."

## 2. Propose the change

- **A new process:** `flows_create` with a `name`, the task type it `startsWith`, and its `edges` (`from`, `to`, `onStatus`). Nodes and their positions are filled in for you. Set `inheritAssignee` on a node when the same person should carry the work on.
- **A change to a flow:** `flows_update` with `changes` (`addNodes`, `removeNodes`, `addEdges`, `removeEdges`) and `expectedVersion`. Removing a task type removes its arrows. The starting task type can't change.
- **Stop a flow:** `flows_update` with `enabled: false` (pause). You never delete a flow: if the user wants it gone, tell them to delete it in the Flow Creator.

The task types must exist (`task_types_list`). If one is missing, create it first (playbook [configure-task-types-and-forms.md](configure-task-types-and-forms.md)).

## 3. Preview, confirm, apply

1. Make the call with `dryRun: true`. The answer is the flow as it would be, and `changed`: arrows and task types added and removed.
2. Show the user the changes and what they set off (run `flows_simulate` again on a draft if the chain is long). Point out:
   - arrows on `open` or `ready`: they fire whenever a task moves into that status, which may be more often than they think;
   - loops (A → B → A): they're fine when people close each step, but say so;
   - task types whose "created as ready" or default assignee decide who gets the new task.
3. On their OK, make the same call without `dryRun`.

If the answer says an arrow already exists in another flow, the same trigger would fire twice: offer to move it, not duplicate it. If it says the flow changed since you read it (409), get it again and redo the change from the new version.

## Notes

- Arrows fire on status *changes*. A task a flow creates is inserted already in its starting status, so it fires nothing as it's created: an arrow from a type that starts "ready" on `ready` won't fire for tasks a flow creates.
- Flows are also part of the factory spec (`flows` section). For a whole factory, use the spec. For one process, the `flows_*` tools are simpler and preview exactly what changes.
