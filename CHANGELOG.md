# Changelog

Every release of apogee-scout, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.2.2 (2026-11-28)

### Fixes

- A Redis connection shared by several threads (the cache, session and queue-dashboard stores, used by a multi-threaded server or queue worker) no longer mixes up replies or stalls: every command is one locked round trip (grav-redis 0.2.2). Uses the new releases of apogee-package.

## 0.2.1 (2026-11-27)

### Fixes

- Database connections that the server dropped while idle (MySQL's `wait_timeout`, a restart, a proxy's idle limit) reconnect on their next use instead of failing with "server has gone away" (grav-db 0.2.1). Uses the new releases of apogee-package, grav-db, grav-orbit.

## 0.2.0 (2026-11-25)

Needs grav 1.12.0 or newer.

- `ScoutPackage`: the module connects itself to an apogee-framework app (apogee-package):
  its settings, wiring moved here from the framework.
- Its tables as a migration (`CreateScoutIndexTable`, run by `migrate`); the store's own
  table-making method uses the same definition.

### Dependencies

- Uses apogee-package 0.1.0, grav-orbit 0.3.0, apogee-http-client 0.2.0 (from their GitHub tags).

## 0.1.1 (2026-11-21)

- Laid out by `grav fmt` and tidied as grav 1.11's lints suggest (`isEmpty()`
  over `size() == 0`, compound assignments); no change in behaviour.

## 0.1.0 (2026-11-21)

The first release. Needs grav 1.10.1 or newer.

- `Scout`: search Orbit models (best first), filters, pages; indexes kept in
  step with saves and deletes; import, flush, syncing off; `Searchable` and
  `ConditionallySearchable`.
- Engines: memory, database (SQLite FTS5, PostgreSQL full-text search, MySQL
  FULLTEXT) and Meilisearch.
