# Site Template API (superseded)

> **Status: superseded.** The `SiteTemplate` table was dropped in
> migration `042_site_documents` when the site builder moved to block-tree
> documents. Reusable layout fragments now live in **snippets**
> (`SiteDocumentSnippet`, migration `048_site_document_snippets`). See
> `site-builder-blocks.md` for the block schema and snippet contract, and
> `docs/sites-and-templates-backend.md` for the RPC list.

The historical SiteTemplate proto messages below are kept for reference for
clients that may still hold cached generated stubs; the RPCs are not served.

This document summarizes the protobuf additions for persisted site templates and site-builder support.

## Models

`SiteTemplate` lives in `proto/models.proto`.

Fields:

- `id`
- `template_key`
- `name`
- `description`
- `enabled`
- `nav_json`
- `blocks_json`
- `settings_json`
- `created_by_uuid`
- `updated_by_uuid`
- `created_at`
- `updated_at`

`nav_json`, `blocks_json`, and `settings_json` are JSON strings so the template editor can evolve without changing the proto for every UI field.

Expected JSON shapes:

```json
[
  { "label": "Home", "href": "" },
  { "label": "Maps", "href": "/maps" }
]
```

```json
[
  { "title": "Welcome", "body": "Starter copy..." }
]
```

```json
{
  "includeNav": true,
  "includeBlocks": true,
  "blockStatus": "DRAFT"
}
```

## RPCs

`proto/nexuscore.proto` adds:

- `ListSiteTemplates(ListSiteTemplatesRequest) returns (ListSiteTemplatesResponse)`
- `UpsertSiteTemplate(UpsertSiteTemplateRequest) returns (SiteTemplate)`
- `DeleteSiteTemplate(DeleteSiteTemplateRequest) returns (Empty)`

Request/response messages:

- `ListSiteTemplatesRequest`
  - `include_disabled`
- `ListSiteTemplatesResponse`
  - repeated `SiteTemplate templates`
- `UpsertSiteTemplateRequest`
  - `SiteTemplate template`
- `DeleteSiteTemplateRequest`
  - `id`
  - `template_key`

## Compatibility

Clients should treat missing persisted templates as normal. NexusWeb has built-in defaults and merges persisted rows by `template_key`.

For deletes, callers may send either `id` or `template_key`; Core accepts both.
