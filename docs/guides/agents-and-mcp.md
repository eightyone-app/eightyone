# Agents and MCP

EightyOne runs an [MCP](https://modelcontextprotocol.io) server. Any MCP client, such as Claude, Cursor or your own agent, can read a company's data and work in it, as the person who connected it.

## Connecting

- **URL:** `https://api.eightyone.app/mcp`. One connection is one company: if the person belongs to several, add `?company=<short name>`.
- **Sign in (OAuth):** interactive clients add the URL as a custom connector. EightyOne's sign-in page opens, and the person approves.
- **API key:** headless agents send a key as `Authorization: Bearer <key>` ([Getting started](getting-started.md#credentials)).

Start with `me_get`: it shows the company, the person and their permissions. Tools the person can't use aren't listed.

### Working for several companies

An integrator usually serves several client companies, as an invited user in each. Every connection works in exactly one company:

- `me_companies` lists the person's companies, marks the one this connection works in, and gives each company's own MCP URL (`…/mcp?company=<short name>`).
- **Over OAuth,** add each company as its own connection, with its own URL, and approve it for that company when the sign-in page asks. A connection approved for one company is refused in another, and the refusal names the URL to use.
- **With API keys,** create a key in each company (**Platform → API**). A key never works outside the company it was made in.

## What the company's administrator sees

Administrators see everyone who reaches their company programmatically on **Platform → API → Access**. That's each person's API keys and the agent apps they've connected (Claude, Cursor…), with when each was last used. People who also belong to other EightyOne companies, such as an integrator's staff, are marked and listed first. An administrator can **revoke** a key, or **block** one agent app for one person. A blocked app is refused for that person from its next call; their other apps, their keys and their own access are untouched, and the block can be undone. The same is available as `access.*` over REST, never to agents.

## What an agent can do

- **Read:** tasks, machines and their status, work orders, downtime, analytics, storylines (a machine's or an order's history), inventory, quality, partner handoffs.
- **Set up a company:** `config_export`, `config_validate`, `config_plan`, `config_apply` ([Commissioning](commissioning.md)).
- **Automate with flows:** `flows_list`, `flows_get`, `flows_simulate` (what a status change sets off), `flows_create` and `flows_update` (rename, change the canvas, pause or resume), with the same rules as the Flow Creator. A flow is never deleted over MCP: an agent pauses it and tells the person to delete it in the Flow Creator.
- **Run the floor:**
  - release work orders, and move, assign and complete tasks, filling in their forms;
  - record production counts;
  - open NCRs and disposition them;
  - report and classify machine stops, and enter meter readings;
  - move, receive, ship and count stock;
  - send work to partner companies.

The [tool list](../skill/eightyone/reference/tools.md) has the details. Anything not listed as a tool can still be reached with `capabilities_search` and `capabilities_call`.

## The rules agents work under

- **Preview, then confirm.** Every write an agent can call takes `dryRun: true`. The answer says whether the change would be accepted, what it changes, and what it sets off (stock posted, tasks created, notifications). The skill tells agents to show that to the person and wait for their OK before the real call.
- **Never destructive.** Deleting anything, cancelling a work order, withdrawing a handoff and pruning a factory spec are not available over MCP. An agent tells the person where to do them.
- **People give what only people can give:**
  - photos, signatures and scans are filled in in the app;
  - stop reasons, dispositions and count decisions come from a person, never from the agent's guess.
- **Company guidance.** Administrators can tell every agent working in their company something (**Settings → AI guidance**), and agents read it with `guide_get` topic `company`.

## The skill

The `eightyone` skill teaches an agent the domain and the jobs: commissioning a factory from a template, modelling a production line, configuring task types and forms, and running a shift. It's an [Agent Skill](https://agentskills.io): a `SKILL.md` with playbooks, a factory template and the generated tool reference.

- **Claude Code:** `/plugin marketplace add eightyone-app/eightyone`, then `/plugin install eightyone@eightyone`. The plugin also connects the MCP server.
- **Claude apps and other agents:** download `eightyone-skill-<version>.zip` from [Downloads](downloads.md) and upload it as a custom skill.

The skill is versioned with the API: each release's tool reference matches the server it was published from.
