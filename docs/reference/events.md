# EightyOne webhook events

Generated from the event catalog (`src/webhooks/events/catalog.ts`, `pnpm docs:api`): don't edit by hand.

Every delivery is a [Standard Webhooks](https://www.standardwebhooks.com/) message, `POST`ed as JSON:

```json
{ "type": "task.status_changed", "timestamp": "2026-10-19T08:00:00.000Z", "data": { ... } }
```

Ids in `data` are the API's own: external ids, SKUs, and the numeric ids the API itself uses.

## Verifying a delivery

- `webhook-id`: The message id: the same for every retry of this event (de-duplicate on it)
- `webhook-timestamp`: Unix seconds when this attempt was sent (reject old ones to stop replays)
- `webhook-signature`: Space-separated "v1a,<base64 Ed25519 signature>" over "<webhook-id>.<webhook-timestamp>.<raw body>"; verify with a key from /api/v1/webhook-signing-keys

EightyOne signs with its Ed25519 key (scheme `v1a`); there is no shared secret. Fetch the public keys from `GET /api/v1/webhook-signing-keys` (no authentication; `whpk_` + base64 of the 32-byte key) and accept a delivery when any listed signature verifies with any listed key. During a key rotation two keys are listed and both sign.

A delivery answered with anything but 2xx is retried after 1 minute, 5 minutes, 30 minutes, 2 hours and 24 hours, with the same `webhook-id`.

## Events

### `client.created`

A client was created.

```json
{
  "type": "object",
  "properties": {
    "client": {
      "type": "object",
      "properties": {
        "id": {
          "description": "Your id for the client (its external system id)",
          "type": [
            "string",
            "null"
          ]
        },
        "name": {
          "type": "string"
        },
        "isActive": {
          "type": "boolean"
        },
        "contactPersonName": {
          "type": [
            "string",
            "null"
          ]
        },
        "contactPersonEmail": {
          "type": [
            "string",
            "null"
          ]
        },
        "contactPersonPhone": {
          "type": [
            "string",
            "null"
          ]
        },
        "color": {
          "type": [
            "string",
            "null"
          ]
        },
        "createdAt": {
          "anyOf": [
            {
              "type": "string",
              "format": "date-time"
            },
            {
              "type": "string"
            }
          ],
          "description": "ISO 8601"
        }
      },
      "required": [
        "id",
        "name",
        "isActive",
        "contactPersonName",
        "contactPersonEmail",
        "contactPersonPhone",
        "color",
        "createdAt"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "client"
  ],
  "additionalProperties": false
}
```

### `data_package.received`

A partner released the data package you requested.

```json
{
  "type": "object",
  "properties": {
    "dataPackage": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "dataRequestId": {
          "type": "string"
        },
        "from": {
          "type": "object",
          "properties": {
            "name": {
              "type": "string"
            },
            "shortName": {
              "type": "string"
            }
          },
          "required": [
            "name",
            "shortName"
          ],
          "additionalProperties": false
        }
      },
      "required": [
        "id",
        "dataRequestId",
        "from"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "dataPackage"
  ],
  "additionalProperties": false
}
```

### `downtime.ended`

A machine stoppage ended.

```json
{
  "type": "object",
  "properties": {
    "downtime": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "minimum": -9007199254740991,
          "maximum": 9007199254740991,
          "description": "Downtime event id, as downtime_events.list shows it"
        },
        "ssu": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "id": {
                  "type": "string",
                  "description": "SSU external id"
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "id",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "source": {
          "type": "string",
          "enum": [
            "auto",
            "manual"
          ]
        },
        "startedAt": {
          "anyOf": [
            {
              "type": "string",
              "format": "date-time"
            },
            {
              "type": "string"
            }
          ],
          "description": "ISO 8601"
        },
        "endedAt": {
          "anyOf": [
            {
              "anyOf": [
                {
                  "type": "string",
                  "format": "date-time"
                },
                {
                  "type": "string"
                }
              ],
              "description": "ISO 8601"
            },
            {
              "type": "null"
            }
          ]
        },
        "durationSeconds": {
          "type": [
            "number",
            "null"
          ]
        }
      },
      "required": [
        "id",
        "ssu",
        "source",
        "startedAt",
        "endedAt",
        "durationSeconds"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "downtime"
  ],
  "additionalProperties": false
}
```

### `downtime.started`

A machine stopped: detected from telemetry (auto) or reported by a person (manual).

```json
{
  "type": "object",
  "properties": {
    "downtime": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "minimum": -9007199254740991,
          "maximum": 9007199254740991,
          "description": "Downtime event id, as downtime_events.list shows it"
        },
        "ssu": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "id": {
                  "type": "string",
                  "description": "SSU external id"
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "id",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "source": {
          "type": "string",
          "enum": [
            "auto",
            "manual"
          ]
        },
        "startedAt": {
          "anyOf": [
            {
              "type": "string",
              "format": "date-time"
            },
            {
              "type": "string"
            }
          ],
          "description": "ISO 8601"
        },
        "endedAt": {
          "anyOf": [
            {
              "anyOf": [
                {
                  "type": "string",
                  "format": "date-time"
                },
                {
                  "type": "string"
                }
              ],
              "description": "ISO 8601"
            },
            {
              "type": "null"
            }
          ]
        },
        "durationSeconds": {
          "type": [
            "number",
            "null"
          ]
        }
      },
      "required": [
        "id",
        "ssu",
        "source",
        "startedAt",
        "endedAt",
        "durationSeconds"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "downtime"
  ],
  "additionalProperties": false
}
```

### `handoff.declined`

A handed-over task was declined (sent to both sides).

```json
{
  "type": "object",
  "properties": {
    "handoff": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        },
        "status": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "direction",
        "state",
        "status"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "handoff"
  ],
  "additionalProperties": false
}
```

### `handoff.deleted`

A handoff was withdrawn or deleted (sent to both sides).

```json
{
  "type": "object",
  "properties": {
    "handoff": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        },
        "status": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "direction",
        "state",
        "status"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "handoff"
  ],
  "additionalProperties": false
}
```

### `handoff.received`

A partner company sent you a task.

```json
{
  "type": "object",
  "properties": {
    "handoff": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        },
        "status": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "direction",
        "state",
        "status"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "handoff"
  ],
  "additionalProperties": false
}
```

### `handoff.status_changed`

A task handed between partner companies changed status (sent to both sides).

```json
{
  "type": "object",
  "properties": {
    "handoff": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        },
        "status": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "direction",
        "state",
        "status"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "handoff"
  ],
  "additionalProperties": false
}
```

### `lot.created`

A lot was created: received from a vendor, produced by a work order, or entered by hand.

```json
{
  "type": "object",
  "properties": {
    "lot": {
      "type": "object",
      "properties": {
        "code": {
          "type": "string"
        },
        "product": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "sku": {
                  "type": [
                    "string",
                    "null"
                  ]
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "sku",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "status": {
          "type": "string"
        },
        "workOrderId": {
          "description": "The work order that produced it",
          "type": [
            "string",
            "null"
          ]
        },
        "vendor": {
          "description": "The vendor it came from",
          "type": [
            "string",
            "null"
          ]
        }
      },
      "required": [
        "code",
        "product",
        "status",
        "workOrderId",
        "vendor"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "lot"
  ],
  "additionalProperties": false
}
```

### `partner_delivery.incoming`

A partner company sent you goods: they are in transit to you.

```json
{
  "type": "object",
  "properties": {
    "delivery": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "kind": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "kind",
        "direction",
        "state"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "delivery"
  ],
  "additionalProperties": false
}
```

### `partner_delivery.received`

A partner received goods you sent.

```json
{
  "type": "object",
  "properties": {
    "delivery": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "kind": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "kind",
        "direction",
        "state"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "delivery"
  ],
  "additionalProperties": false
}
```

### `partner_delivery.shortfall`

A partner reported goods you sent as missing or damaged.

```json
{
  "type": "object",
  "properties": {
    "delivery": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string"
        },
        "kind": {
          "type": "string"
        },
        "direction": {
          "type": "string"
        },
        "state": {
          "type": "string"
        }
      },
      "required": [
        "id",
        "kind",
        "direction",
        "state"
      ],
      "additionalProperties": {}
    }
  },
  "required": [
    "delivery"
  ],
  "additionalProperties": false
}
```

### `stock.reorder_triggered`

A product's stock fell to its reorder point (lead time × measured consumption). Sent once per crossing; it re-arms when stock recovers.

```json
{
  "type": "object",
  "properties": {
    "product": {
      "anyOf": [
        {
          "type": "object",
          "properties": {
            "sku": {
              "type": [
                "string",
                "null"
              ]
            },
            "name": {
              "type": "string"
            }
          },
          "required": [
            "sku",
            "name"
          ],
          "additionalProperties": false
        },
        {
          "type": "null"
        }
      ]
    },
    "onHand": {
      "type": "number"
    },
    "reorderPoint": {
      "type": "number"
    },
    "dailyConsumption": {
      "type": "number"
    },
    "leadTimeDays": {
      "type": "number"
    },
    "suggestedQuantity": {
      "type": "number"
    },
    "storageSiteId": {
      "description": "Storage site external id, when the policy is per site",
      "type": [
        "string",
        "null"
      ]
    }
  },
  "required": [
    "product",
    "onHand",
    "reorderPoint",
    "dailyConsumption",
    "leadTimeDays",
    "suggestedQuantity",
    "storageSiteId"
  ],
  "additionalProperties": false
}
```

### `task.created`

A task was created: by a person, the API, an agent, a flow, a schedule or a work order's release.

```json
{
  "type": "object",
  "properties": {
    "task": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "description": "Task external id"
        },
        "title": {
          "type": "string"
        },
        "status": {
          "type": "string"
        },
        "priorityId": {
          "anyOf": [
            {
              "type": "integer",
              "minimum": -9007199254740991,
              "maximum": 9007199254740991
            },
            {
              "type": "null"
            }
          ],
          "description": "Priority level, as the API uses it"
        },
        "type": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "id": {
                  "type": "string"
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "id",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "dueDate": {
          "anyOf": [
            {
              "anyOf": [
                {
                  "type": "string",
                  "format": "date-time"
                },
                {
                  "type": "string"
                }
              ],
              "description": "ISO 8601"
            },
            {
              "type": "null"
            }
          ]
        },
        "workOrderId": {
          "description": "Work order external id, for an operation",
          "type": [
            "string",
            "null"
          ]
        }
      },
      "required": [
        "id",
        "title",
        "status",
        "priorityId",
        "type",
        "dueDate",
        "workOrderId"
      ],
      "additionalProperties": false
    },
    "ssu": {
      "anyOf": [
        {
          "type": "object",
          "properties": {
            "id": {
              "type": "string",
              "description": "SSU external id"
            },
            "name": {
              "type": "string"
            }
          },
          "required": [
            "id",
            "name"
          ],
          "additionalProperties": false
        },
        {
          "type": "null"
        }
      ]
    }
  },
  "required": [
    "task",
    "ssu"
  ],
  "additionalProperties": false
}
```

### `task.status_changed`

A task moved to another status (open, preparation, ready, dispatched, submitted, done, incomplete, declined).

```json
{
  "type": "object",
  "properties": {
    "task": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "description": "Task external id"
        },
        "title": {
          "type": "string"
        },
        "status": {
          "type": "string"
        },
        "priorityId": {
          "anyOf": [
            {
              "type": "integer",
              "minimum": -9007199254740991,
              "maximum": 9007199254740991
            },
            {
              "type": "null"
            }
          ],
          "description": "Priority level, as the API uses it"
        },
        "type": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "id": {
                  "type": "string"
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "id",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "dueDate": {
          "anyOf": [
            {
              "anyOf": [
                {
                  "type": "string",
                  "format": "date-time"
                },
                {
                  "type": "string"
                }
              ],
              "description": "ISO 8601"
            },
            {
              "type": "null"
            }
          ]
        },
        "workOrderId": {
          "description": "Work order external id, for an operation",
          "type": [
            "string",
            "null"
          ]
        }
      },
      "required": [
        "id",
        "title",
        "status",
        "priorityId",
        "type",
        "dueDate",
        "workOrderId"
      ],
      "additionalProperties": false
    },
    "ssu": {
      "anyOf": [
        {
          "type": "object",
          "properties": {
            "id": {
              "type": "string",
              "description": "SSU external id"
            },
            "name": {
              "type": "string"
            }
          },
          "required": [
            "id",
            "name"
          ],
          "additionalProperties": false
        },
        {
          "type": "null"
        }
      ]
    },
    "previousStatus": {
      "type": [
        "string",
        "null"
      ]
    },
    "status": {
      "type": "string"
    }
  },
  "required": [
    "task",
    "ssu",
    "previousStatus",
    "status"
  ],
  "additionalProperties": false
}
```

### `work_order.completed`

A work order's last operation was done.

```json
{
  "type": "object",
  "properties": {
    "workOrder": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "description": "Work order external id"
        },
        "code": {
          "type": [
            "string",
            "null"
          ]
        },
        "product": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "sku": {
                  "type": [
                    "string",
                    "null"
                  ]
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "sku",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "quantity": {
          "type": "number"
        },
        "completedAt": {
          "anyOf": [
            {
              "anyOf": [
                {
                  "type": "string",
                  "format": "date-time"
                },
                {
                  "type": "string"
                }
              ],
              "description": "ISO 8601"
            },
            {
              "type": "null"
            }
          ]
        }
      },
      "required": [
        "id",
        "code",
        "product",
        "quantity",
        "completedAt"
      ],
      "additionalProperties": false
    }
  },
  "required": [
    "workOrder"
  ],
  "additionalProperties": false
}
```

### `work_order.released`

A work order went to the floor: its operations were created.

```json
{
  "type": "object",
  "properties": {
    "workOrder": {
      "type": "object",
      "properties": {
        "id": {
          "type": "string",
          "description": "Work order external id"
        },
        "code": {
          "type": [
            "string",
            "null"
          ]
        },
        "product": {
          "anyOf": [
            {
              "type": "object",
              "properties": {
                "sku": {
                  "type": [
                    "string",
                    "null"
                  ]
                },
                "name": {
                  "type": "string"
                }
              },
              "required": [
                "sku",
                "name"
              ],
              "additionalProperties": false
            },
            {
              "type": "null"
            }
          ]
        },
        "quantity": {
          "type": "number"
        },
        "dueDate": {
          "anyOf": [
            {
              "anyOf": [
                {
                  "type": "string",
                  "format": "date-time"
                },
                {
                  "type": "string"
                }
              ],
              "description": "ISO 8601"
            },
            {
              "type": "null"
            }
          ]
        }
      },
      "required": [
        "id",
        "code",
        "product",
        "quantity",
        "dueDate"
      ],
      "additionalProperties": false
    },
    "operations": {
      "type": "integer",
      "minimum": -9007199254740991,
      "maximum": 9007199254740991,
      "description": "How many operation tasks the release created"
    }
  },
  "required": [
    "workOrder",
    "operations"
  ],
  "additionalProperties": false
}
```
