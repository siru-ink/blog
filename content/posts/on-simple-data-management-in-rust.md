+++
title = "On Simple Data Management in Rust"
date = "2025-11-18"
+++
## Documentation Teaches Some but Not All

I was recently working on expanding my knowledge of rust, and along those lines
wanted to learn more about data management. This took me down the path of
learning about serde (a powerful crate for serializing/deserializing structs in
rust) as well as rusqlite (a crate to interact with the ubiquitous SQLite
database system from rust). I would say that both of these crates have done a
great job in providing beginner-friendly documentation online, however, there
are almost always some pitfalls that you can only come accross in practice. This
will be a listing -- mostly for my future self -- about some of the mistakes I
made, and am likely to make again, that were rather confusing to me at first.

### Database File Not Found Error

```log
Error: SqliteFailure(Error { code: CannotOpen, extended_code: 14 }, Some("unable to open database file: ./data/test.db3"))
```

This was the first error I came accross using the
[`rusqlite` crate](https://crates.io/crates/rusqlite). Supposedly, using the
`Connection::open("filename");` should automatically create a new file if the
file does not exist, but instead I was getting a file not found error.
Apparently, this error message is thrown if the directory/directory tree for the
file is not accesible. In my case this was the `data` folder. While rusqlite is
capable of automatically creating the file if it is missing, it can not create
the missing directory structure.

In order to create the parent directory structure for a file, one can use the
following in rust:

```rust
use std::fs;

// The "let _" simply discards the potential error for now
// Don't use this directly in production code
let _ = fs::create_dir_all(&test_path);
```

### Table Already Exists

This one is mostly my fault, but if you try to run the following while the table
`ingredients` already exists, sqlite will throw an error that crashes the
surround rust program:

```rust
database.execute(
        "CREATE TABLE ingredient (
            id TEXT PRIMARY KEY,
            name TEXT NOT NULL,
        )",
        (),
    )?;
```

### Type Safety in the Database

Writing in rust gives us programmers a lot of confidence that we will not have
to worry about mismatched types because `rustc`, our *glorious* compiler, will
always catch them for us. However, this does not seem to extend to rusqlite's
ability to type check the data being put into a database.

In the project I am currently working on, I had the following database setup.

{% <box> %}
| id TEXT | name TEXT|
| - | - |
| ... | ... |
{% </box> %}

Set up via the following code through rusqlite:

```rust
database.execute(
        "CREATE TABLE ingredient (
            id TEXT PRIMARY KEY,
            name TEXT NOT NULL,
        )",
        (),
    )?;
```

My plan was to use a
[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier) as the id
for each entry so that I could garantee that it would always be unique. Creating
these uuid in rust is fairly straight forward using the `uuid` crate's
`Uuid::new_v4()`. This, gives us a new uuid in the `Uuid` struct type. To insert
a new item into the database I was then calling rusqlite's `execute` to insert a
new record as such:

```rust
database.execute(
        "INSERT INTO ingredient (id, name) VALUES (?1, ?2)",
        (Uuid::new_v4(), &name_var),
    )?;
```

Which works without throwing any warnings or errors. **However**, since this
newly created uuid is technically a `Uuid` struct type and not a string, what is
actually stored into the database is a blob object even though the id column is
technically marked as having the `TEXT` datatype. This can be verified using a
database viewer, such as [DB Browser for SQLite](https://sqlitebrowser.org).
This shocked me, because I (a) did not believe that the database itself would
allow this, and (b) that rusqlite did not throw any mismatched type errors (or
at least warnings). This particular issue can easily be mitigated by simply
using `Uuid::new_v4().to_string()` instead, but it does highlight the fact that
one must be particularly careful in what is being added into the database, in
order not to corrupt it unintentionally.

### Serde's Derive Macros

[`serde` crate](https://crates.io/crates/serde) provides an all around framework
for serializing and deserializing structs into a variety of different storage
formats (JSON, etc.). It's API is designed in a, to me, incredibly intuitive
manner, in that it simply provides rust macros that can be applied to most
common user-made structs, which then automatically add serialization abilities
to the struct. Such as:

```rust
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct Ingredient {
    name: String,
    count: u8,
}
```

However, it has the small caveat that this is not turned on in the `serde` crate
by default, but rather requires it to be enabled as a feature in the
`Cargo.toml`. (Easy enough to do, but why not as default?):

```toml
serde = { version = "1.0.228", features = ["derive"] }
```

### Serde Can't Serialize or Deserialize

But anyone trying to actually serialize anything using `serde` will quickly
notice that this is not trivially possible. Trying to run the example they
provide on their homepage will also simply not compile. Instead, the different
possible (de-)serializers are split into individual crates. For example, to use
the JSON serializer, one also needs to actually add the
[`serde_json` crate](https://crates.io/crates/serde_json) into the projects
[TOML](https://en.wikipedia.org/wiki/TOML) file, such as:

```toml
serde_json = "1.0.145"
```

I do appreciate, that the serde team directly links to these different
serializers from their homepage, but maybe putting in a single sentance
explaining this separation would make getting `serde` set up the first time a
more fluid experience.
