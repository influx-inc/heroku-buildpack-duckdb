# heroku-buildpack-duckdb

Heroku buildpack that installs the [DuckDB](https://duckdb.org/) C library (`duckdb.h` + `libduckdb.so`) so Ruby gems like [ruby-duckdb](https://github.com/suketa/ruby-duckdb) can compile their native extensions.

## Usage

Add this buildpack **before** the language buildpack:

```bash
heroku buildpacks:add --index 1 \
  https://github.com/<org>/heroku-buildpack-duckdb \
  -a your-app-name
```

Verify the order:

```bash
heroku buildpacks -a your-app-name
# 1. https://github.com/<org>/heroku-buildpack-duckdb
# 2. heroku/ruby
```

## How it works

1. **`bin/compile`** downloads the DuckDB C library from [GitHub releases](https://github.com/duckdb/duckdb/releases) and installs `duckdb.h` and `libduckdb.so` into `$BUILD_DIR/.duckdb/`. Downloads are cached between builds.

2. **`bin/export`** sets `LIBRARY_PATH`, `LD_LIBRARY_PATH`, and `BUNDLE_BUILD__DUCKDB` so the Ruby buildpack can find the header and library when compiling the `duckdb` gem.

3. **`.profile.d/duckdb.sh`** sets `LD_LIBRARY_PATH` at runtime so the dyno can load `libduckdb.so`.

Follows the [heroku-buildpack-apt](https://github.com/heroku/heroku-buildpack-apt) pattern of installing into the app directory rather than system paths.

## Configuration

The DuckDB version is pinned in `bin/compile`:

```bash
DUCKDB_VERSION="1.4.3"
```

To upgrade, update this value and push.

## Stack compatibility

Tested on Heroku-24 (Ubuntu 24.04).
