# Site Builder — Block Schema and Backgrounds

The site builder stores each (site, subsite) page as a single JSON document.
The backend (`internal/server/nexus_service_site_documents.go`) treats the
document as opaque except where noted below; NexusWeb renders it.

## Document shape

```json
{
  "blocks": [
    { "id": "abc123", "type": "hero", "props": { ... }, "children": [ ... ] }
  ],
  "meta": { "version": 1, "...": "..." }
}
```

- `blocks` — top-level array of blocks. Order is rendering order.
- Each block MUST have a stable `id` (string) and a `type` (string).
- `props` is type-specific JSON. `children` is an optional nested array of
  blocks for container types.

The only contract the backend enforces: `blocks` is an array on the
top-level document object. Snippet apply (`ApplySiteDocumentSnippet`) reads
and writes `blocks`; everything else passes the JSON through unchanged.

## Supported block types

These are the types the NexusWeb editor and renderer understand. Adding a
new type is a NexusWeb change — the backend doesn't need to know about it.

| `type`             | Purpose                                    | Notable props                                  |
| ------------------ | ------------------------------------------ | ---------------------------------------------- |
| `section`          | Container; usually carries a background    | `background`, `padding`, `children`            |
| `columns`          | N-column layout                            | `columns`, `gap`, `children`                   |
| `hero`             | Big top-of-page banner                     | `title`, `subtitle`, `cta`, `background`       |
| `rich-text`        | Markdown / HTML rich text                  | `markdown`                                     |
| `cta-banner`       | Call-to-action band                        | `label`, `href`, `background`                  |
| `gallery`          | Grid of images                             | `assetIds[]`, `columns`                        |
| `embed`            | External embed (YouTube, Twitch, iframe)   | `url`, `aspectRatio`                           |
| `server-status`    | Live status of a Minecraft server          | `serverName`                                   |
| `leaderboard-card` | A leaderboard widget                       | `metricKey`, `top`                             |
| `modpack-card`     | A modpack card linking to the modpack page | `modpackId`                                    |
| `spacer`           | Vertical space                             | `size`                                         |

## Backgrounds

A background is a JSON object with the same shape at every scope:

```json
{
  "assetId": 42,
  "url": "https://cdn.example.com/sky.jpg",
  "fit": "cover",
  "position": "center",
  "overlayColor": "#000000",
  "overlayOpacity": 0.4,
  "blur": 0
}
```

- `assetId` (preferred) — a `site_assets.id`. The renderer resolves it to
  the current asset URL. Use this so swapping the asset doesn't require
  editing every document that uses it.
- `url` — fallback for assets not tracked in `site_assets` (CDN, external
  image). Ignored when `assetId` is set and the asset exists.
- `fit` — one of `cover`, `contain`, `tile`, `center`.
- `position` — CSS `background-position` value.
- `overlayColor` / `overlayOpacity` — solid colour overlay drawn on top.
- `blur` — px of Gaussian blur applied to the image.

Backgrounds can live in three places, in increasing specificity:

1. **Site** — `sites.theme_config_json.background`. Default for every page
   on the site.
2. **Subsite** — `subsites.theme_config_json.background`. Overrides the
   site background for that subsite.
3. **Block** — any block's `props.background`. Scoped to that block (most
   common for `section` and `hero`).

Backend storage is the existing `theme_config_json` columns and the
document JSON — no schema changes were required for backgrounds.

## Snippets

A snippet (`SiteDocumentSnippet`) is a reusable block-tree fragment.
`blocks_json` is a JSON array of blocks matching the schema above.
`ApplySiteDocumentSnippet` inserts those blocks into a target draft's
top-level `blocks` array, either at the end (default) or immediately after
the block whose `id` equals `after_block_id`.

Categories are free-form strings used by the editor's picker. Common
values: `LAYOUT`, `HERO`, `CARDS`, `FOOTER`, `GENERAL`.
