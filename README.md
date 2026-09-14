# Autotask Read — Cursor Plugin

A small, publishable [Cursor Plugin](https://cursor.com/docs/plugins) that
connects Cursor to a remote, read-only **Datto Autotask** MCP server over HTTP
with bearer authentication.

Install the plugin, enter your MCP URL and bearer token, and the agent gets
Autotask read tools (tickets, companies, counts, attachment metadata, entity
field metadata, threshold information) plus a skill that teaches it how to use
them safely.

- **Read-only.** The plugin exposes no write tools. No ticket updates, no time
  entries, no notes.
- **Bring your own server.** This plugin is only a connector. You need an
  Autotask MCP HTTP endpoint that speaks Streamable HTTP (or SSE) and accepts a
  bearer token.
- **No secrets in this repo.** Your URL and token live in Cursor's plugin
  configuration, not in these files.

## Layout

```
.cursor-plugin/plugin.json   plugin manifest (name, variables, components)
mcp.json                     HTTP MCP server definition using ${AUTOTASK_MCP_URL} / ${AUTOTASK_MCP_BEARER}
skills/autotask-read/SKILL.md  agent guidance: read-only, known tool list, thresholds, no secrets
assets/logo.svg              plugin mark
LICENSE                      MIT
```

## Setup

After installing, Cursor prompts for two variables (also editable later under
**Plugins → Configure**):

| Variable | Title | Value |
| --- | --- | --- |
| `AUTOTASK_MCP_URL` | Autotask MCP URL | HTTPS MCP endpoint with **no trailing slash**, e.g. `https://example.com/autotask-read` |
| `AUTOTASK_MCP_BEARER` | Autotask MCP bearer | Bearer token for that MCP path. Never commit it. |

### No trailing slash on the URL

Enter `https://example.com/autotask-read`, not `https://example.com/autotask-read/`.
Many bearer/JWT gateways match the path exactly; a trailing slash produces a
different path, which can fail auth (401/403) or route to a different handler
even though the MCP server itself is fine.

The plugin sends:

```
Authorization: Bearer <AUTOTASK_MCP_BEARER>
Accept: application/json, text/event-stream
```

## Tools exposed by a compatible server

The skill assumes the server exposes exactly these read tools and tells the
agent not to invent others:

`autotask_zone_information`, `autotask_threshold_information`,
`autotask_entity_information`, `autotask_entity_fields`, `autotask_entity_udfs`,
`autotask_get`, `autotask_read_path`, `autotask_query`,
`autotask_query_next_page`, `autotask_count`, `autotask_query_all`,
`autotask_list_attachments`, `autotask_get_attachment_metadata`,
`autotask_get_attachment_content`, `autotask_get_document_plain_text`.

The skill also tells the agent to check `autotask_threshold_information` before
bulk reads (Autotask allows 10,000 API requests per hour per tenant), to look up
field names with `autotask_entity_fields` instead of guessing, and to never print
bearers or Autotask API keys.

## Test locally

1. Clone this repo.
2. Link or copy it into Cursor's local plugin folder:

   ```bash
   mkdir -p ~/.cursor/plugins/local
   ln -s "$(pwd)" ~/.cursor/plugins/local/autotask-read
   ```

3. Restart Cursor, or run **Developer: Reload Window** from the command palette.
4. Open **Cursor Settings → Plugins** (or Customize), find **Autotask Read**,
   and enter `AUTOTASK_MCP_URL` and `AUTOTASK_MCP_BEARER`.
5. Check **MCP** in settings: the `autotask-read` server should show as
   connected with the tools listed above.
6. In chat, ask something scoped, for example "check Autotask threshold usage"
   or "count open tickets for company 12345", and confirm the agent uses the
   `autotask_*` tools.

Notes:

- Teams/Enterprise admins may need to enable **Allow Local Plugin Imports**
  (Dashboard → Settings → Security & Identity → Marketplace and Plugins).
- If a marketplace plugin with the same name is already installed, it takes
  precedence over the local copy.

## Submit / publish

The plugin must remain **public and open source** so it can be reviewed.

- **Community listing:** <https://cursor.directory/plugins/new>
- **Curated Cursor Marketplace:** <https://cursor.com/marketplace/publish>
  (manually reviewed and frequently closed to new submissions; submit the
  public repository URL). Updates are re-reviewed and are not pulled
  automatically.

`homepage` and `repository` in `.cursor-plugin/plugin.json` point at this
public repository: <https://github.com/abeyaruna-lgtm/autotask-read-cursor-plugin>.
If you fork it, update both fields to your fork before submitting.

## Security

- The bearer token is a secret. It is never stored in this repository and is
  only injected by Cursor into the `Authorization` header at runtime.
- Do not put production MSP URLs, tenant identifiers, or tokens anywhere in this
  repo, including examples, issues, or commit messages. Use `example.com`
  placeholders.
- The connector is read-only by design. If you fork this and add write tools,
  rename the plugin so users are not misled about what it can do.
- Rotate the bearer if it is ever pasted into a chat, log, or screenshot.

## License

[MIT](LICENSE) © Anti Chaos
