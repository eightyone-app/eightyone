# Commission a factory

Goal: take an empty (or partly set up) EightyOne company to a working factory setup, in one reviewed change.

## Before you start

1. **Check where you are.** Call `me_get`: check the company and that the user can manage what you'll set up. Then call `config_export`. An empty factory has a few defaults (its own client, an administrator group); anything else means someone has started. Ask before replacing their work.
2. **Read** `guide_get` topic `commissioning`, and topic `company` (the company's own guidance, if any).
3. **Gather the facts from the user,** or from documents they give you:
   - **areas** (floors, cells) and the **machines** in each, as a tree: a line with its machines;
   - **work centers:** groups of machines that can run the same operations;
   - **products,** by SKU: what's bought (with vendors), what's made (with its recipe), units, lot or serial tracking;
   - **operations:** the task types, the checks each records (its form), standard setup and run minutes;
   - **routings:** the steps each product goes through, and which steps gate the next ones;
   - **machine statuses** and **downtime and failure reason codes;**
   - **roles** and what each may do (permission groups), **teams** and **skills.**

   Never ask about prices or costs.

## Build the spec

- **From a template.** Start from `templates/brazos-metalworks.json` (or an export of a similar factory). Rename what's specific, delete what doesn't apply, add what's missing.
- **From scratch.** Write only the sections you need. A section you leave out isn't touched.

Everything refers to everything else **by name**, so keep the names consistent:

| Thing | Referred to by |
| --- | --- |
| machine (SSU) | its path: `"Site / Line / Machine"` |
| storage unit | `"Site or warehouse / Unit"` |
| product | its SKU (capitals don't matter) |
| task type, routing, work center, status, role, team, skill, permission group | its name |
| person | their email (existing users only) |

What the system checks, so get it right the first time:

- **Products:**
  - A product is **bought** (`vendors`) or **made** (`recipe`), never both. Recipe quantities are whole units per unit made.
  - Units are one of: pcs, kg, g, lb, l, ml, oz, m, cm, mm, yd, ft, in.
- **Machines:** sibling machines need distinct names. Machine `make`/`model` come from a shared catalog: search it first (`machine_catalog_search_makes`) and reuse existing spellings.
- **Every reference must exist,** in the spec or in the company: statuses, work centers, task types, SKUs, storage. `config_validate` lists anything missing, with its path in the spec.
- **Roles** grant exactly one permission group. A person's permissions are the union of their roles' groups.
  - You can only hand out roles within the permissions of the user you act for.
  - The company always keeps an administrator.
- **Routings and parameter sets** describe the released version. Changing them releases a new version.

## Review with the user

1. `config_validate`, and fix every problem.
2. `config_plan`. Summarize it per section: how many creates and updates, and the notable ones.
3. **Call out:**
   - anything that changes existing data (`update`), field by field;
   - permission changes: groups' permissions, roles' groups, people's roles;
   - routings and parameter sets that will release a new version;
   - anything `blocked` (the user lacks the permission: they need someone who has it);
   - `extra` items, which are in the company but not in the spec. They're left alone.
4. **Wait for an explicit go-ahead.** Then call `config_apply` with the same spec.

## Large specs: go in stages

A whole factory is tens of kilobytes; don't pass it in one call. Split it into these stages and do them in order. Each stage only names what the earlier stages created, so each one validates on its own:

| Stage | Sections |
| --- | --- |
| 1. Plant | `ssuStatuses`, `reasonCodes`, `workCenters`, `sites`, `warehouses`, `settings` (without the task-type settings) |
| 2. Supply | `productCategories`, `vendors`, `products` (large: split it by category if needed) |
| 3. Organisation | `permissionGroups`, `roles`, `teams`, `skills` |
| 4. Work | `parameterSets`, `taskTypes`, `routings`, then `settings` with `ncrTaskType` and the like |
| 5. Machines and quality | `machineData`, `flows`, `issues`, `inspectionPlans`, `guidance` |
| 6. People | `people` (only once they've been invited) |

Before the first stage, tell the user what the whole template will set up: counts per section and anything unusual. Get the go-ahead for the plan as a whole. Then for each stage, `config_plan` and check it holds no surprises (a fresh company shows only creates), and `config_apply`. Stop and ask if a plan shows updates you didn't expect.

## Apply and verify

- `config_apply` runs change by change and stops at the first failure. The result names it (`failed.section`, `failed.key`, `failed.error`). The changes before it stay applied.
- **On a failure,** fix the spec and apply again: unchanged items are skipped, so re-applying is safe.
- **Verify** with `config_plan` on the same spec: it should show no creates or updates. Tell the user what was set up.

## Leave for the user

A spec can't do these; list them at the end:

- **Invite people** (dashboard → Users). Then a spec can give them roles, teams and skills.
- **Service accounts** for integrations (dashboard → API keys), and **machine connectors**.
- **Documents** (procedures, drawings): upload them in the dashboard.
- **Removing anything:** the REST API (with prune) or the dashboard.
