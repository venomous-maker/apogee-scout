# apogee-scout

Full-text search for [Orbit](https://github.com/venomous-maker/grav-orbit)
models in [Apogee](https://github.com/venomous-maker) apps: a model's
searchable fields go into an index when it is saved and leave when it is
deleted, and a search gives back the models, best match first. Indexes live in
memory, in the app's own database (SQLite FTS5, PostgreSQL full-text search,
MySQL FULLTEXT), or in [Meilisearch](https://www.meilisearch.com).

```grav
import "std"
import "apogee-scout"
use apogee.*

class Post extends Model implements Searchable {
    public id: long = 0
    public title: string = ""
    public body: string = ""
    public published: bool = false
    fn toSearchableArray() -> HashMap<string, any> { return {"title": self.title, "body": self.body, "published": self.published} }
}

Scout.use(new DatabaseEngine(pool))          // then engine.createTables() (in an app: the migration)
Scout.observe::<Post>()                      // keep the index in step with saves and deletes
Scout.importAll::<Post>()                    // index what is already there
let posts = Scout.search::<Post>("grav compiler").where("published", true).take(10).get()
```

## In an apogee-framework app

Importing the module is all it takes: `ScoutPackage` (apogee-package) connects it when
the app is created. The engine comes from `scout.driver`.

**Migration:** `CreateScoutIndexTable` makes the `scout_index` table (only with the `database` driver). It is written in each database's own SQL: an FTS5 table, a `tsvector` column, a `FULLTEXT` index. Run `migrate` after adding the module.

| Setting | Default |
|---|---|
| `scout.driver` | `SCOUT_DRIVER`: `database` (default), `meilisearch` or `memory` |
| `scout.prefix` | `SCOUT_PREFIX`: before every index name |
| `scout.meilisearch.host`, `.key` | `MEILISEARCH_HOST` (`http://127.0.0.1:7700`), `MEILISEARCH_KEY` |

Your own configuration wins over these defaults.

## Models

A model is indexed with the fields `toSearchableArray()` gives (`Searchable`),
or all its fields. One that implements `ConditionallySearchable` stays out of
the index while `shouldBeSearchable()` is false (a draft), and joins when it
turns true. The index is named after the model's table (with `Scout.prefix`
in front, to share a search server between apps).

## Searching

`Scout.search::<T>(text)`, then:

| | |
|---|---|
| `where(field, value)`, `whereIn(field, values)` | filters on the indexed fields |
| `take(n)`, `skip(n)` | |
| `get()` | the models, best first (rows deleted since they were indexed are left out) |
| `first()`, `keys()`, `count()`, `raw()` | |
| `paginate(perPage, page)` | `items`, `total`, `page`, `lastPage`, `hasMore()` |

Every word must match; the last may be the start of a word, so a search box
can search as the user types. An empty search matches everything.

## Keeping indexes up to date

`Scout.observe::<T>()` updates the index on every save and delete.
`Scout.makeSearchable(m)` and `Scout.unsearchable(m)` do it by hand;
`Scout.importAll::<T>(chunk)` indexes every row; `Scout.flushAll::<T>()`
empties the index; `Scout.withoutSyncing(f)` runs `f` without touching it.

## Engines

| | |
|---|---|
| `MemoryEngine` | an inverted index in this process (the default; tests, small apps) |
| `DatabaseEngine(pool, table = "scout_index")` | the app's database through grav-db: SQLite FTS5 (bm25 ranking), PostgreSQL `tsvector` with a GIN index (`ts_rank`), MySQL `FULLTEXT` (boolean mode). `createTables()` makes the table. |
| `MeilisearchEngine(host, key, wait = false)` | a Meilisearch server: typo tolerance and its ranking rules; filter fields are made filterable on first use; `wait` waits for each write to be applied |

A custom engine implements `SearchEngine` (`update`, `delete`, `search`,
`flush`).

## Tests

```bash
grav test tests/scout.grav
grav test tests/package.grav
MEILI_URL=http://127.0.0.1:7700 MEILI_KEY=... grav test tests/scout.grav   # with Meilisearch too
```

The same checks run against each engine (SQLite for the database one;
PostgreSQL and MySQL were not tested here).
