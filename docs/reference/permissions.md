# Permissions

Generated from the permission vocabulary and the capability registry: don't edit by hand.

A user's permissions are the union of their roles' permission groups (Settings → Permission groups). An API key acts as its owner, optionally narrowed to fewer permissions; an agent connected over MCP acts as the user who connected it. Capabilities marked *any member* need no permission beyond belonging to the company; some of them check more as they run (a task's type, its assignee).

## `tasks:view_all`

See all tasks (not only own)

## `tasks:view_my_teams`

See tasks for my teams

## `tasks:view_pre_active`

See pre-active tasks on mobile

## `tasks:create`

Create tasks

## `tasks:delete`

Delete tasks

Capabilities: `tasks.delete`

## `tasks:reassign`

Reassign tasks

## `tasks:change_status_mobile`

Change task status from mobile

## `task_types:view`

View Task types & work centers

## `task_types:manage`

Manage Task types & work centers

Capabilities: `config.apply`

## `mobile_builder:view`

View Mobile Builder

## `mobile_builder:manage`

Manage Mobile Builder

Capabilities: `config.apply`

## `flows:view`

View Flows

## `flows:manage`

Manage Flows

Capabilities: `config.apply`

## `clients:view`

View Clients, sites & SSUs

## `clients:manage`

Manage Clients, sites & SSUs

Capabilities: `clients.create`, `clients.delete`, `clients.update`, `config.apply`, `ssus.deletion_blockers`, `ssus.update`

## `geographic:view`

View Geographic

## `geographic:manage`

Manage Geographic

## `personnel:view`

View Personnel & skills

## `personnel:manage`

Manage Personnel & skills

Capabilities: `config.apply`

## `subcontractors:view`

View Subcontractors

## `subcontractors:manage`

Manage Subcontractors

Capabilities: `config.apply`

## `users:view`

View Users & invitations

## `users:manage`

Manage Users & invitations

Capabilities: `invitations.resend`, `invitations.revoke`

## `access_requests:approve`

Approve access requests

## `permissions:view`

View Permission groups

## `permissions:manage`

Manage Permission groups (Manage makes an administrator)

Capabilities: `access.block_agent`, `access.list`, `access.revoke_key`, `access.unblock_agent`, `config.apply`

## `settings:view`

View Company settings & integrations

Capabilities: `accounting.list_documents`, `accounting.providers`, `accounting.test`, `external_refs.list`, `external_refs.lookup`

## `settings:manage`

Manage Company settings & integrations

Capabilities: `accounting.connect`, `accounting.disconnect`, `accounting.request_document`, `accounting.retry_document`, `accounting.sync`, `config.apply`, `external_refs.delete`, `external_refs.upsert`, `teams.create`, `teams.delete`, `teams.update`

## `api_keys:view`

View API keys & webhooks

Capabilities: `webhooks.delivery_logs`, `webhooks.event_types`, `webhooks.get`, `webhooks.list`

## `api_keys:manage`

Manage API keys & webhooks

## `webhooks:manage`

Manage webhooks

Capabilities: `webhooks.create`, `webhooks.delete`, `webhooks.retry_delivery`, `webhooks.test`, `webhooks.update`

## `audit_logs:view`

View audit logs

## `data:export`

Export data

## `data:import`

Import data (CSV)

## `products:view`

View Products, vendors & purchasing

Capabilities: `inventory.reorder_suggestions`

## `products:manage`

Manage Products, vendors & purchasing

Capabilities: `config.apply`

## `inventory:view`

View Inventory

Capabilities: `accounting.inventory_differences`, `customers.list`, `inventory.stock_levels`, `inventory_counts.get`, `inventory_counts.list`, `shipments.get`, `shipments.list`, `stock_transactions.list`, `storage_units.get`, `storage_units.list`

## `inventory:manage`

Manage Inventory

Capabilities: `config.apply`, `inventory_counts.create`, `inventory_counts.finalize`, `inventory_counts.reconcile`, `inventory_counts.record_lines`, `shipments.cancel`, `shipments.create`, `shipments.ship`, `stock.transfer`

## `inventory:adjust`

Adjust inventory (post counts)

Capabilities: `inventory_counts.reconcile`

## `inventory:receive`

Receive goods (POs, partner deliveries)

Capabilities: `partner_deliveries.receive`, `purchase_orders.receive`

## `production:view`

View Production

Capabilities: `storylines.work_order`, `work_orders.get`, `work_orders.list`

## `production:manage`

Manage Production

Capabilities: `config.apply`, `work_orders.batch_upsert`, `work_orders.upsert`

## `work_orders:release`

Release work orders

Capabilities: `work_orders.release`

## `machine_data:view`

View Machine data

## `machine_data:manage`

Manage Machine data

Capabilities: `config.apply`, `downtime_events.close`, `downtime_events.open`

## `maintenance:view`

View Maintenance

Capabilities: `meters.list`, `storylines.ssu`

## `maintenance:manage`

Manage Maintenance

Capabilities: `config.apply`, `meters.record_reading`

## `quality:view`

View Quality

## `quality:manage`

Manage Quality

Capabilities: `config.apply`

## `ncr:disposition`

Disposition NCRs

Capabilities: `ncrs.disposition`

## `documents:view`

View Documents

## `documents:manage`

Manage Documents

## `documents:publish`

Publish documents

## `insights:view`

View insights

Capabilities: `analytics.downtime`, `analytics.failure_pareto`, `analytics.oee`, `analytics.overview`, `analytics.reliability`

## `handoffs:send`

Send tasks to partner companies

Capabilities: `data_packages.get`, `data_packages.list`, `handoffs.decline`, `handoffs.get`, `handoffs.list`, `handoffs.send`, `handoffs.timeline`, `handoffs.withdraw`, `partner_deliveries.get`, `partner_deliveries.list`, `partner_deliveries.receive`

## `partner_data:request`

Request data from partners

## `partner_data:release`

Release data to partners

## `ai_guidance:manage`

Edit AI guidance (what agents are told)

Capabilities: `config.apply`, `guidance.delete`, `guidance.set`
