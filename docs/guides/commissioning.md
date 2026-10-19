# Commissioning: a company from one file

A **factory spec** is a company's whole setup as one JSON document: plant, supply, work, machine data, quality and people.

- Everything in it refers to everything else **by name**, so a spec from one company applies to another.
- A section you leave out isn't touched.

## The sections

| Stage | Sections |
| --- | --- |
| Plant | `ssuStatuses`, `reasonCodes`, `workCenters`, `sites` (with their storage and the SSU tree), `warehouses`, `settings` |
| Supply | `productCategories`, `vendors`, `products` (by SKU, with vendors, recipe and stock policies) |
| Organisation | `permissionGroups`, `roles`, `teams`, `skills` |
| Work | `parameterSets`, `taskTypes` (forms, SLAs, gates, flags), `routings` |
| Machines and quality | `machineData` (signals, meters, condition rules, spare parts), `flows`, `issues`, `inspectionPlans`, `guidance` |
| People | `people`: existing users only, with their roles, teams and skills |

The [schema](../reference/factory-spec.schema.json) has every field.

## The calls

1. **`config.export`:** the company's setup as a spec. It's the best example of the format, and the starting point for a change.
2. **`config.validate`:** problems, each with its path in the file (a status that doesn't exist, a gate naming a field the form lacks).
3. **`config.plan`:** what applying would do, item by item: creates, updates field by field, what's blocked by permissions, and what's in the company but not in the spec (left alone).
4. **`config.apply`:** applies the plan through the same rules as the dashboard, audited. If a change fails, the earlier ones stay and the answer names the failure. Fix the file and apply again: unchanged items are skipped.

Over REST, `prune` also removes what's in the company but not in the spec. Agents can't prune.

## Working as an SI

- **With an agent:**
  - the SI describes the factory or hands over documents;
  - the agent, with the `eightyone` skill, starts from a template, builds the spec, and shows the plan;
  - it applies after the SI's go-ahead, in stages for a large factory.
- **With files:**
  1. commission one reference factory;
  2. export it, and keep the spec in your own repository as your template;
  3. for each client, rename what's specific, validate, plan and apply.

  The skill's Brazos Metalworks template, a sheet-metal fabricator, was made that way.

## What a spec doesn't do

- **People:** a spec never invites anyone. Invite them in the dashboard; a spec can then give them roles, teams and skills.
- **Permissions:** nobody can grant more than they have themselves, and the company always keeps an administrator.
- **Versions:** changing a routing or a parameter set releases a new version, and orders already released keep theirs.
- **Companies:** only EightyOne staff create them.
- **Prices:** none, anywhere.
