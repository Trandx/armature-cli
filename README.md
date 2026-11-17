# armature-cli

CLI tool for the Armature framework.

## Features

- **Project Generation** - Create new projects
- **Code Generation** - Generate controllers, models, etc.
- **Development Server** - Hot reload development
- **Build Tools** - Production builds and optimization

## Installation

```bash
cargo install armature-cli --locked
```

## Commands

### Create Project

```bash
armature new my-app
cd my-app
```

### Generate Code

```bash
# Generate a controller
armature generate controller users

# Generate a model
armature generate model user

# Generate a full CRUD scaffold with fields
armature generate scaffold post --fields "title:string,body:text,published:bool"
```

### Development Server

```bash
# Start with hot reload
armature dev

# Specify port
armature dev --port 8080

# Extra `cargo build` arguments, then arguments for the application
armature dev -- --release --bin api -- --config dev.toml

# Explain every event, build and restart
armature --verbose dev
```

`armature dev` builds the project with `cargo build`, runs the produced
binary directly (with `PORT` and `HOST` set) and rebuilds when a build input
changes. If `cargo-watch` is installed it is used instead; pass
`--no-cargo-watch` to force the built-in watcher.

How the built-in watcher behaves:

- **One save, one rebuild.** Events are filtered by kind: reads/opens (which
  the compiler itself produces while building) and metadata-only changes
  (`touch`, permissions) are ignored. Events close in time are grouped over a
  stability window (400 ms by default) and a file whose content did not
  actually change is ignored.
- **One build at a time.** Changes made while a build runs are queued and
  built once, right after it — never lost, never in parallel.
- **A failed build is not retried** until something changes again; the
  previous server keeps running meanwhile.
- **Clean restarts.** The old server and every process it started are stopped
  (SIGTERM, then SIGKILL after the grace period; a Job Object on Windows)
  before the new one starts, so the port is free. `Ctrl+C`, `SIGTERM` and
  closing the terminal stop everything as well.

Watched (build inputs, derived from `cargo metadata`):

| Path | |
|---|---|
| `src/**` of the package and of its workspace path dependencies | rebuild when content changes |
| `Cargo.toml`, `Cargo.lock`, `build.rs`, `.cargo/config.toml`, `rust-toolchain(.toml)` | rebuild |
| `armature.toml` | rebuild (no-op) and restart |
| `dev.watcher.watch_paths` | rebuild and restart |

Ignored: the Cargo target directory, `.git/` (and other VCS/IDE/cache
directories such as `.idea/`, `.vscode/`, `node_modules/`), editor swap and
backup files (`*.swp`, `*~`, `.#*`, `4913`, `*___jb_tmp___`…) and
`dev.watcher.exclude_paths`. Other hidden files are *not* ignored.

### Build

```bash
# Development build
armature build

# Production build
armature build --release
```

## Configuration

Create `armature.toml` in your project root:

```toml
[project]
name = "my-app"

[server]
port = 3000
host = "127.0.0.1"

[database]
url = "postgres://localhost/mydb"

# `armature dev` (all optional; these are the defaults)
[dev.watcher]
enabled = true          # false: build and run once, without watching
debounce_ms = 400       # stability window; also `armature dev --debounce-ms`
watch_paths = []        # extra files/directories, relative to the project root
exclude_paths = []      # paths that never trigger a rebuild
verbose = false         # same as `armature --verbose dev`

[dev.server]
graceful_shutdown_timeout_ms = 3000   # SIGTERM / CTRL_BREAK → wait
force_kill_timeout_ms = 2000          # then forced kill → wait
```

## License

MIT OR Apache-2.0

