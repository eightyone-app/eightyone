# Downloads

## The `eightyone` skill

- **Claude Code:** install it as a plugin, which keeps it up to date and connects the MCP server:

  ```
  /plugin marketplace add eightyone-app/eightyone
  /plugin install eightyone@eightyone
  ```

- **Claude apps and other agents:** download [eightyone-skill.zip](../downloads/eightyone-skill.zip) (the latest release) and upload it as a custom skill. Each release is also kept as `eightyone-skill-<version>.zip`.

The skill's version is in its `SKILL.md` (`metadata.version`). It's released with the API, so its tool reference matches the server.

## Machine-readable docs

- [llms.txt](../llms.txt): an index of these docs for agents.
- [llms-full.txt](../llms-full.txt): everything in one file.
- [OpenAPI 3.1](../reference/openapi.json), [factory spec schema](../reference/factory-spec.schema.json).
