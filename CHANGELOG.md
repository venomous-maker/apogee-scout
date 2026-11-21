# Changelog

Every release of apogee-scout, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

- Laid out by `grav fmt` and tidied as grav 1.11's lints suggest (`isEmpty()`
  over `size() == 0`, compound assignments); no change in behaviour.

## 0.1.0 (2026-11-21)

The first release. Needs grav 1.10.1 or newer.

- `Scout`: search Orbit models (best first), filters, pages; indexes kept in
  step with saves and deletes; import, flush, syncing off; `Searchable` and
  `ConditionallySearchable`.
- Engines: memory, database (SQLite FTS5, PostgreSQL full-text search, MySQL
  FULLTEXT) and Meilisearch.
