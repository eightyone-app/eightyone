# EightyOne developer docs

EightyOne runs factories and field-service companies. Everything that happens there is a task: machines, work orders, stock, quality, maintenance. These docs are for people and AI agents who build on it: system integrators who set up and run client factories, and teams who connect their own systems.

## Ways in

| | For | Start with |
| --- | --- | --- |
| **REST API** (`/api/v1`) | Integrations, scripts, ERP and MES connectors | [Getting started](getting-started.md), then the [API reference](../reference/capabilities.md) |
| **MCP server** (`/mcp`) | AI agents: Claude, Cursor, any MCP client | [Agents and MCP](agents-and-mcp.md) |
| **The `eightyone` skill** | Agents that set up or run a factory | [Agents and MCP](agents-and-mcp.md#the-skill) |
| **Webhooks** | Being told when something happens | [Webhooks](webhooks.md), then the [event reference](../reference/events.md) |
| **External ids** | Connectors that map their own ids to EightyOne records | [Integrations](integrations.md) |
| **Factory specs** | Setting up a whole company from one file | [Commissioning](commissioning.md) |

The REST API and the MCP server are generated from one registry of capabilities. The same operation has the same name, input, rules and permissions on both, and both are logged the same way.

## Concepts

- [Tasks: everything is a task](tasks.md)
- [Production: work orders, routings, counts, lots](production.md)
- [Inventory: storage, bins, moves, counts](inventory.md)
- [Quality, maintenance and downtime](quality-and-maintenance.md)
- [Factory or managed services](factory-and-managed-services.md)

## Reference (generated)

- [Capabilities](../reference/capabilities.md): every operation, with its input, output, permission and REST path
- [OpenAPI 3.1](../reference/openapi.json): the REST API and its webhooks, machine-readable
- [Webhook events](../reference/events.md)
- [Errors](../reference/errors.md)
- [Permissions](../reference/permissions.md)
- [Factory spec schema](../reference/factory-spec.schema.json)
- [MCP tools](../skill/eightyone/reference/tools.md)

## Two rules that shape everything

- **No financial data.** EightyOne holds no prices, costs or rates: quantities and hours only. Integrations with accounting systems exchange ids, products and quantities.
- **An agent or a key acts as a person.** It has that person's permissions in one company, it is logged as them, and it can never do more than they could in the dashboard.
