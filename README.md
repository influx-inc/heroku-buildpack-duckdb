# heroku-buildpack-duckdb

Heroku buildpack that installs the [DuckDB](https://duckdb.org/) C library (`duckdb.h` + `libduckdb.so`) so Ruby gems like [ruby-duckdb](https://github.com/suketa/ruby-duckdb) can compile their native extensions.

## Usage

Add this buildpack **before** the language buildpack:

```bash
heroku buildpacks:add --index 1 \
  https://github.com/<org>/heroku-buildpack-duckdb \
  -a your-app-name
```

Verify the buildpack order:

```bash
heroku buildpacks -a your-app-name
# 1. https://github.com/<org>/heroku-buildpack-duckdb
# 2. heroku/ruby
```

## How it works

1. **`bin/compile`** downloads the DuckDB C library from [GitHub releases](https://github.com/duckdb/duckdb/releases) and installs `duckdb.h` and `libduckdb.so` into `vendor/duckdb/`. Downloads are cached between builds.

2. **`bin/export`** sets `C_INCLUDE_PATH` and `LIBRARY_PATH` so the compiler can find the header and library when the Ruby buildpack compiles the `duckdb` gem. This avoids `BUNDLE_BUILD__DUCKDB`, which is [broken by bundler's backslash-escaping](https://github.com/rubygems/rubygems/issues/3727).

3. **`.profile.d/duckdb.sh`** sets `LD_LIBRARY_PATH` at runtime so the dyno can load `libduckdb.so`.

## Configuration

The DuckDB version is pinned in `bin/compile`:

```bash
DUCKDB_VERSION="1.4.3"
```

To upgrade, update this value and push.

## Stack compatibility

Tested on Heroku-24 (Ubuntu 24.04).
