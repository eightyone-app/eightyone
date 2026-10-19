# Integrations: mapping your ids

**What it is:** when you connect another system (an ERP, an MES, an accounting package) to EightyOne, the same customer, product or order has an id on each side. EightyOne can keep that mapping for you: you link your id to an EightyOne record once, then find either side from the other. Your connector then doesn't need a mapping table of its own.

**Where:** the API's `external_refs.*` capabilities ([reference](../reference/capabilities.md#external-refs)). Looking an id up needs **View company settings & integrations**. Linking and unlinking needs **Manage company settings & integrations**. Agents can look ids up over MCP, but linking is REST-only.

## A link

```json
{
    "system": "acme-erp",
    "externalType": "item",
    "externalId": "ITEM-77",
    "entityType": "product",
    "entityId": "SKU-1001"
}
```

- **`system`** names your system. Use a short lowercase slug: `sap`, `netsuite`, `acme-erp`. `quickbooks` and `xero` are used by EightyOne's own integrations.
- **`externalType`** is the kind of object on your side: `customer`, `item`, `order`, `invoice`. One record can have one id of each type in each system, so a customer and an invoice can both point at the same task.
- **`externalId`** is your id, up to 255 characters, as you send it.
- **`entityType`** is the kind of EightyOne record: `client`, `customer`, `vendor`, `product`, `task`, `work_order`, `purchase_order`, `ssu` (machine) or `storage_unit`.
- **`entityId`** is the record's API id, the same one the rest of the API uses. That's a product's SKU (matched without case), a client's id, a purchase order's number, and the `externalId` of anything else.

## Linking

`PUT /api/v1/external-refs` with the link. Linking is idempotent: sending the same link again only refreshes it (and its `status`, if you send one). Each side maps to one record:

- If your object was linked to another record, it moves to this one.
- If this record already had a different object of that type in your system, that link is replaced.

`status` is optional, up to 32 characters, and kept as given. Use it for documents EightyOne asked your system to create, such as `draft`, `sent` or `paid`.

## Looking up

- **Your id → EightyOne:** `GET /api/v1/external-refs/lookup?system=acme-erp&externalType=item&externalId=ITEM-77`
- **EightyOne → your id:** `GET /api/v1/external-refs/lookup?system=acme-erp&externalType=item&entityType=product&entityId=SKU-1001`

Both answer the link, or `404` when there is none. `GET /api/v1/external-refs` lists links, newest first. Filter it by `system` and `externalType`, or by one record (`entityType` + `entityId`) to see every id it has elsewhere.

## Unlinking

`DELETE /api/v1/external-refs/{system}/{externalType}/{externalId}` removes the link only. Neither the EightyOne record nor your object is changed. Every link and unlink is recorded in the audit log.

## What isn't kept

Ids and statuses only. EightyOne keeps no prices, amounts or totals from any system it is connected to.
