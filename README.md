# `schemaconv`: Tools for converting between database table schemas (WIP)

[![Build Status](https://travis-ci.org/faradayio/schemaconv.svg)](https://travis-ci.org/faradayio/schemaconv)

This tool is intended to help convert between schema formats. It's still very incomplete. Right now, `schemaconv` is most useful for moving data from PostgreSQL to Google's BigQuery.

## What we currently support

We currently support many small pieces which are helpful when converting from PostgreSQL to BigQuery:

- Parsing (some) Postgre `CREATE TABLE` statements, or extracting schemas from `information_schema.columns` in a running database.
- Generating SQL to dump a Postgres table as CSV.
- Generating a "temporary" BigQuery JSON schema allowing the CSV to be loaded into BigQuery.
- Generating BigQuery SQL to tranform the temporary table into a final table, parsing any data types which can't be directly loaded from SQL.
- Generating a BigQuery JSON schema for the final table. This isn't normally necessary, but it's nice to have for the sake of completeness.
- Transforming the schema in several ways

However, we do not yet have any higher-level interface that just transfers data directly.

## Installation

```sh
# Install Rust compiler.
curl https://sh.rustup.rs -sSf | sh

# Install schemaconv.
cargo install -f --git https://github.com/faradayio/schemaconv schemaconv
```

## Examples

Run `schemaconv --help` for more documentation.

```sh
# Given a `postgres:` URL, dump a table schema as JSON.
schemaconv "$DATABASE_URL#mytable" > schema.json

# Dump a table schema as BigQuery schema JSON.
schemaconv "$DATABASE_URL#mytable" -O bq:schema > bigquery-schema.json

# Ditto, but using PostgreSQL `CREATE TABLE` SQL as input.
schemaconv -I pg -O bq:schema < table.sql > bigquery-schema.json

# Dump a table schema as quoted PostgreSQL `SELECT ...` arguments.
schemaconv "$DATABASE_URL#mytable" -O pg:select > select-args.txt
```

You can also edit the default schema JSON (generated with no `-O` flag, or with `-O json`), and then run it back through to generate another format:

```sh
schemaconv "$DATABASE_URL#mytable" > schema.json
# (Edit schema.json.)

schemaconv -O bq < schema.json > bigquery-schema.json
```

## "Interchange" table schemas

In order to make `schemaconv` work, we define a "interchange" table schema format. This uses a highly-simplied and carefully curated set of column types that make sense when passing data between databases. This represents a compromise between the richness of PostgreSQL data types, and the relative poverty of BigQuery data types, while still preserving as much information as possible. It includes timestamps, geodata, etc.

Seee [`table.rs`](./schemaconvlib/src/table.rs) for the details of this "interchange" schema.

## Contributing

For more instructions about building `schemaconv`, running tests, and contributing code, see [CONTRIBUTING.md](./CONTRIBUTING.md).