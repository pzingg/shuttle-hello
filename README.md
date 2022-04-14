# shuttle-hello

Just trying out `shuttle-service`.

# Steps

### Create new project

1. `cargo new shuttle-hello --lib`
2. Add `description`, `authors`, `license`, `homepage`, and `repository` fields
    to `[package]` section of Cargo.toml.
3. Add `crate-type = ["cydlib"]` to `[lib]` section of Cargo.toml.
4. Add rocket dependencies to `[dependencies]` section of Cargo.toml:

```
rocket = "0.5.0-rc.1"
shuttle-service = { version = "0.2", features = ["web-rocket"] }
```

### Add code to lib.rs.

Entry point should have `#[shuttle_service::main]` attribute.

For rocket-based apps the entry point's signature is:

```
async fn init() -> Result<Rocket<Build>, shuttle_service::Error>
```

or, if using a Postgres database:

```
async fn init(pool: PgPool) -> Result<Rocket<Build>, shuttle_service::Error> {
```

and the required dependencies/features are:

```
rocket = "0.5.0-rc.1"
serde = { version = "1.0", features = ["derive"] }
sqlx = { version = "0.5", features = ["runtime-tokio-native-tls", "postgres"] }
shuttle-service = { version = "0.2", features = ["sqlx-postgres", "web-rocket"] }
```

For axum-based apps:

```
async fn init() -> Result<SyncWrapper<Router>, shuttle_service::Error> {
```

with dependencies:

```
axum = "0.5"
shuttle-service = { version = "0.2", features = ["web-axum"] }
sync_wrapper = "0.1"
```

### Install `cargo-shuttle` and authorize with your GitHub identity.

```
cargo install cargo-shuttle
cargo shuttle login
```

After authorizing, you will be redirected to a page displaying your API key.
You can copy the API key and paste it into the terminal window, which
is prompting for input, or copy the full login command displayed,
quit the terminal and re-run `cargo shuttle login` with the supplied API key:

```
cargo shuttle login --api-key <API_KEY>
```

### Deploy

You must have all changes committed to your local git repository.
Then:

```
cargo shuttle deploy
```

Build output shows:

```
   Packaging shuttle-hello v0.1.0 (/home/pzingg/Projects/rust/shuttle_hello)
   Archiving .cargo_vcs_info.json
   Archiving .gitignore
   Archiving Cargo.toml
   Archiving Cargo.toml.orig
   Archiving README.md
   Archiving src/lib.rs
   ...
   Compiling shuttle-hello v0.1.0 (/opt/unveil/crates/shuttle-hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1m 24s

        Project:            shuttle-hello
        Deployment Id:      <DEPLOYMENT_UUID>
        Deployment Status:  DEPLOYED
        Host:               shuttle-hello.shuttleapp.rs
        Created At:         2022-04-14 19:16:22.577369173 UTC
        Database URI:       postgres://<USER:PASSWORD>@pg.shuttle.rs/db-shuttle-hello
```

### Success:

Visit https://shuttle-hello.shuttleapp.rs/hello
