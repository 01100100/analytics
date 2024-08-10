# Analytics

[Umami](https://umami.is/) is a simple, fast, privacy-focused, open-source analytics solution.

This repo contains the deployment configuration to host Umami on [fly.io](https://fly.io), along with a Postgres instance for persistent storage.

Note: The Fly app name is `umami` and the Postgres instance is called `elefant`.

The deployment can be managed using the `fly` CLI.

```bash
# Ensure you set the correct values in the .env file
source .env
fly secrets set APP_SECRET=$APP_SECRET
fly secrets set DATABASE_URL=$DATABASE_URL
fly deploy
```

The instance can be accessed at [https://umani.fly.dev](https://umani.fly.dev).

## Backup and Restore

There is a Postgres instance deployed on [fly.io](https://fly.io). To backup the database to a local pg_dump file, run the following command:

```bash
# Ensure you set the correct values in the .env file
source .env
fly proxy 15432:5433 -a elefant &
while ! nc -z localhost 15432; do
  sleep 0.1
done
pg_dump "postgresql://postgres:${PG_PASSWORD}@localhost:15432/umani" -c -f db_backup_$(date '+%Y-%m-%d').sql &&
echo "Database backed up to file: db_backup_$(date '+%Y-%m-%d').sql"
```

To restore the database from a local pg_dump file, run the following command:

```bash
# Ensure you set the correct values in the .env file
source .env
# note: set the correct date in the pg_dump file name
export PG_DUMP_FILE=db_backup_$(date '+%Y-%m-%d').sql
fly proxy 15432:5433 -a elefant &
while ! nc -z localhost 15432; do
  sleep 0.1
done
psql -v -d postgresql://postgres:${PG_PASSWORD}@localhost:15432/umani < ${PG_DUMP_FILE}
```

