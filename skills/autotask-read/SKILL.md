---
name: autotask-read
description: Use when querying Autotask PSA through this MCP (tickets, companies, counts, attachments, thresholds). Read-only Datto Autotask REST access via the autotask-read MCP server.
---

# Autotask Read

This skill covers the `autotask-read` MCP server. It is a read-only path into a
Datto Autotask PSA tenant.

## Read only

- This connector is READ ONLY. There are no write tools.
- Do not attempt to create or update tickets, complete tickets, add notes, add
  time entries, or change any Autotask record through this server. If the user
  asks for a write, say the connector cannot do it and stop.
- If a tool name that implies a write appears to exist, do not call it. It is
  not part of this connector.

## Known tools

Only these tools exist. Do not invent others.

| Tool | Use |
| --- | --- |
| `autotask_zone_information` | Resolve the tenant's API zone. |
| `autotask_threshold_information` | Current request-threshold usage for the tenant. |
| `autotask_entity_information` | Metadata for an entity (capabilities, supported operations). |
| `autotask_entity_fields` | Standard field definitions for an entity, including picklist values. |
| `autotask_entity_udfs` | User-defined field definitions for an entity. |
| `autotask_get` | Fetch a single record by entity and id. |
| `autotask_read_path` | Read an arbitrary allowlisted REST path (GET only). |
| `autotask_query` | Filtered, paged query against one entity. |
| `autotask_query_next_page` | Follow `pageDetails.nextPageUrl` from a prior query. |
| `autotask_count` | Count records matching a filter without returning them. |
| `autotask_query_all` | Drain every page of a query. Bounded use only, see below. |
| `autotask_list_attachments` | List attachments on a parent record (for example a ticket). |
| `autotask_get_attachment_metadata` | Metadata for one attachment. |
| `autotask_get_attachment_content` | Raw content of one attachment. |
| `autotask_get_document_plain_text` | Plain-text rendering of a document. |

## How to query

- Prefer `autotask_query`, `autotask_get`, and `autotask_count`, always scoped
  with a company id, ticket id, or another tight filter.
- Avoid unbounded `autotask_query_all`. Only use it after `autotask_count` shows
  the result set is small, and only with a strong filter. Never run it against
  Tickets, TimeEntries, or TicketNotes without a company or ticket filter.
- Page with `autotask_query_next_page` using the `nextPageUrl` returned by the
  previous call. Do not change the filter, field list, or page size between
  pages.
- Request only the fields you need with `IncludeFields` where the tool supports
  it.
- Do not invent Autotask REST field names, entity names, or picklist meanings.
  Call `autotask_entity_fields` (and `autotask_entity_udfs` for custom fields)
  first, then build the filter from the returned names and picklist values.
- Use `autotask_entity_information` if you are unsure which operations or
  child collections an entity supports.
- Autotask timestamps are UTC. Do not assume a local timezone.

## Thresholds

- The Autotask API threshold is 10,000 requests per hour per tenant, shared
  with every other integration on that tenant.
- Call `autotask_threshold_information` before any bulk read (multi-page
  queries, `autotask_query_all`, or fetching many attachments) and again if a
  long job is still running. Back off if usage is high.
- Batch work: one `autotask_count` first, then a paged `autotask_query`, rather
  than repeated `autotask_get` calls.

## Attachments

- List with `autotask_list_attachments`, inspect with
  `autotask_get_attachment_metadata`, and only then fetch content with
  `autotask_get_attachment_content` or `autotask_get_document_plain_text`.
- Check size in the metadata before pulling content.

## Secrets

- Never print, echo, log, or paste the MCP bearer token or any Autotask API
  key, integration code, or username/secret pair. This includes values that
  appear in tool errors, URLs, or headers.
- If a response contains a credential, redact it before showing it to the user.
