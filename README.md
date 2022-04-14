# shuttle-hello

Some notes on my trial of the new alpha-status [Shuttle](https://www.shuttle.rs/)
deployment platform for Rust server applications.

Here's what I did:

## Create new project

1. `cargo new shuttle-hello --lib`
2. Add `description`, `authors`, `license`, and optionally `homepage` and
    `repository` fields to the `[package]` section of Cargo.toml.
3. Add `crate-type = ["cydlib"]` to the `[lib]` section of Cargo.toml.
4. Add rocket dependencies to the `[dependencies]` section of Cargo.toml:

```
rocket = "0.5.0-rc.1"
shuttle-service = { version = "0.2", features = ["web-rocket"] }
```

## Add (rocket) server code to lib.rs

Your server's entry point must have the
`#[shuttle_service::main]` attribute.

For apps based on the [rocket](https://docs.rs/rocket/0.5.0-rc.1)
framework the entry point's signature is:

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

For apps written for the [axum](https://docs.rs/axum/0.5) framework:

```
async fn init() -> Result<SyncWrapper<Router>, shuttle_service::Error> {
```

with dependencies:

```
axum = "0.5"
shuttle-service = { version = "0.2", features = ["web-axum"] }
sync_wrapper = "0.1"
```

Apparently, support for the [tide](https://docs.rs/tide/0.16.0),
[tower](https://docs.rs/tower/0.4.12/), and other servers may be coming.

## Install `cargo-shuttle` and authorize with your GitHub identity

```
cargo install cargo-shuttle
cargo shuttle login
```

After authorizing, you will be redirected to a page displaying your API key.
You can copy the API key and paste it into the terminal window, which
is prompting for input, or copy the full login command displayed,
quit the terminal and re-run `cargo shuttle login` with the supplied API key:

```
cargo shuttle login --api-key {API_KEY}
```

Logging in (or creating another API key with `cargo shuttle auth`),
will create or update the file at `{CONFIG_DIR}/shuttle/config.toml`
where `CONFIG_DIR` is the user's system config directory
(see https://docs.rs/dirs/4.0.0/dirs/fn.config_dir.html).

`config.toml` will contain one line:

```
api_key = '{API_KEY}'
```

## Deploy your server

You must have all changes committed to your local git repository. Then:

```
cargo shuttle deploy
```

If you absolutely need to deploy when there are uncommitted changes,
you can add the `--allow-dirty` flag to the command above.

The `deploy` command will package your project using cargo's
utilities into a local file, upload the file to Shuttle's server, and
then have it built, linked, and started there.

The output shows:

```
   Packaging shuttle-hello v0.1.0 (/home/pzingg/Projects/rust/shuttle_hello)
   Archiving .cargo_vcs_info.json
   Archiving .gitignore
   Archiving Cargo.toml
   Archiving Cargo.toml.orig
   Archiving README.md
   Archiving src/lib.rs
   Updating crates.io index
   Compiling shuttle-hello v0.1.0 (/opt/unveil/crates/shuttle-hello)
   ...
   [compilation steps here]
   ...
   Compiling shuttle-hello v0.1.0 (/opt/unveil/crates/shuttle-hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1m 24s

        Project:            shuttle-hello
        Deployment Id:      {DEPLOYMENT_UUID}
        Deployment Status:  DEPLOYED
        Host:               shuttle-hello.shuttleapp.rs
        Created At:         2022-04-14 19:16:22.577369173 UTC
        Database URI:       postgres://{USER:PASSWORD}@pg.shuttle.rs/db-shuttle-hello
```

You can obtain the above project information (including the deployment id,
host, and database URI) at any time, using `cargo shuttle status`

Other commands are:

* `cargo shuttle auth` - create user credentials for the shuttle platform
* `cargo shuttle delete` - delete the latest deployment for a shuttle project


### Success:

Visit the host displayed in the deployment, e.g. https://shuttle-hello.shuttleapp.rs/hello
