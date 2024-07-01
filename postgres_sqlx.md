# Persistence for our server: PostgreSQL
postgres is the best well rounded choice for a database when starting a project.
sqlx provides compile-tim safety, async and an query interface that uses SQL.

## Spinning up a database
```bash
#!/usr/bin/env bash
# print all executed commands
set -x
# exit immediatlly if any command fails
set -e
# the exit status of a pipe '|' is the exit status of the last command that fails
set -o pipefail

# Check if a custom user has been set, otherwise default to 'postgres'
DB_USER="${POSTGRES_USER:=postgres}"

# Check if a custom password has been set, otherwise default to 'password'
DB_PASSWORD="${POSTGRES_PASSWORD:=password}"

# Check if a custom database name has been set, otherwise default to 'newsletter'
DB_NAME="${POSTGRES_DB:=newsletter}"

# Check if a custom port has been set, otherwise default to '5432'
DB_PORT="${POSTGRES_PORT:=5432}"

# Check if a custom host has been set, otherwise default to 'localhost'
DB_HOST="${POSTGRES_HOST:=localhost}"

# Launch postgres using Docker
docker run \
	-e POSTGRES_USER=${DB_USER} \
	-e POSTGRES_PASSWORD=${DB_PASSWORD} \
	-e POSTGRES_DB=${DB_NAME} \
	-p "${DB_PORT}":5432 \
	-d postgres \
	postgres -N 1000
# ^ Increased maximum number of connections for testing purposes

# Keep pinging Postgres until it's ready to accept commands
export PGPASSWORD="${DB_PASSWORD}"
until psql -h "${DB_HOST}" -U "${DB_USER}" -p "${DB_PORT}" -d "postgres" -c '\q'; do
	echo >&2 "Postgres is still unavailable - sleeping"
	sleep 1
done

echo >&2 "Postgres is up and running on port ${DB_PORT}!"

# using sqlx to create a db named $DB_NAME (although the docker run command has already created it)
DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@${DB_HOST}:${DB_PORT}/${DB_NAME}
export DATABASE_URL
sqlx database create
```

## Migration: a bad word to refer to database schemas
They provide a way to, incrementally through time, design our database.
### Creating a migration
We can use `sqlx` to do so:
```bash
export DATABASE_URL=postgres://postgres:password@127.0.0.1:5432/newsletter
sqlx migrate add create_subscriptions_table
```
Which will create a file (and a folder if didn't exist) at `migrations/{timestamp}_create_subscriptions_table.sql`

And then we can use SQL to create the table we need:
```sql
-- Create Subscriptions Table
CREATE TABLE subscriptions(
id uuid NOT NULL,
PRIMARY KEY (id), -- the table primary key, it has to be unique
email TEXT NOT NULL UNIQUE, -- UNIQUE ensures uniquness at that db level, but introduces some memory and processing overhead
name TEXT NOT NULL,
subscribed_at timestamptz NOT NULL -- timezone aware timestamp
);
```

And we can the migration with `sqlx migrate run`.

