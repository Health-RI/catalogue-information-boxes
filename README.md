# Information box help text

This folder is the storage location for the text (and optional links) shown in the
information icons ("i" tooltips) next to search filters and dataset detail properties
in the GDI User Portal frontend. It exists so this content can be edited without
needing a frontend developer or a frontend deploy.

- `filters.yaml` — help text for filter information icons on the search page, keyed by
  filter key (e.g. `access_rights`).
- `detail-view.yaml` — help text for property information icons on the dataset detail
  page, keyed by Discovery API dataset property name (e.g. `title`, `publisher`; see
  `RetrievedDataset.helpText` in the Discovery Service's `discovery.yaml`).

Both files currently contain **dummy placeholder content only**. Getting the real copy
into these files is out of scope for this ticket and left for a follow-up.

## File shape

Each top-level key maps to an entry with:
- `text` — a map of language code (`en`, `nl`) to the help text string. Optional.
- `link` — optional. `label` is a map of language code to a list of link labels;
  `value` is a flat list of URLs (not per-language). If `label`/`value` have different
  lengths after resolving the language, the Discovery Service pads the shorter one with
  a placeholder (`"More info"` / `"#"`) so the two always line up.

```yaml
access_rights:
  text:
    en: "..."
    nl: "..."
  link:
    label:
      en: ["More info"]
      nl: ["Meer info"]
    value:
      - "https://example.com/access-rights"
```

## How the Discovery Service consumes this

The Discovery Service (`gdi-userportal-dataset-discovery-service`) has a pluggable
help-text source per endpoint, configured via:

- `helptext.filters-source` — for the filters endpoint.
- `helptext.dataset-source` — for the dataset detail endpoint.

Each is either left empty (default: help text comes from CKAN's scheming schema, text
only, no links — today's original behavior) or points at one of:

- A **local file path** mounted into the container, e.g.
  `helptext.filters-source=/config/helptexts/filters.yaml` — a checkout of this repo (or
  just this folder) mounted at that path.
- An **external URL**, e.g.
  `helptext.filters-source=https://raw.githubusercontent.com/Health-RI/catalog-backend-deploy/main/config/helptexts/filters.yaml` —
  fetched at runtime over HTTP and cached in memory (`helptext.cache-ttl`, default 5
  minutes) so the service doesn't hit GitHub on every request.

Either way, editing is purely a matter of committing to this repo — no code change or
Discovery Service rebuild required, only (for the URL case) waiting out the cache TTL,
or (for the local-file case) whatever redeploy/remount step refreshes the mounted file.
