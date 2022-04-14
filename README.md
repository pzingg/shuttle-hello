# shuttle-hello

Just trying out `shuttle-service`.

# Steps

### Create new project

1. `cargo new shuttle_hello --lib`
2. Add `crate-type = ["cydlib"]` to `[lib]` key in Cargo.toml.
3. Add dependencies to `[dependencies]` key in Cargo.toml:

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
