# postgresql_timescaledb
A Dockerfile integrating Postgresql with TimescaleDB including some "base" extensions

# Extensions integrated
* Everything in https://github.com/spitzenidee/postgresql_base
* TimescaleDB 0.10.1 (https://github.com/timescale/timescaledb)

# How to start and set up a container of "spitzenidee/postgresql_timescaledb"
* `docker pull spitzenidee/postgresql_timescaledb:latest`
* `docker run -d --name postgresql_timescaledb --restart unless-stopped -p 5432:5432 -v <static_pgsql_files>:/var/lib/postgresql/data spitzenidee/postgresql_timescaledb:latest`

# Edit "postgresql.conf"
* `docker stop postgresql_timescaledb`
* Edit "<static_pgsql_files>/postgresql.conf" and the parameters "shared_preload_libraries" and "track_io_timing" as follows:
* `shared_preload_libraries = 'pg_stat_statements,powa,pg_qualstats,pg_stat_kcache,pg_cron,timescaledb'`
* `track_io_timing = on`
* `docker start postgresql_timescaledb`

# Set up PG_CRON and PGSQL_HTTP
Use your favorite SQL command line or UI tool to create extensions in your selected database (as a Postgresql superuser, such as "postgres", and possibly in the database "postgres" if you're on defaults from this container image).
* `SET AUTOCOMMIT=ON;`
* `CREATE IF NOT EXISTS EXTENSION http;`
* `CREATE IF NOT EXISTS EXTENSION pg_cron;`
* If applicable: `GRANT USAGE ON SCHEMA cron TO regular_pgsql_user;`
* `CREATE IF NOT EXISTS EXTENSION timescaledb;`, or if upgrading from a previous version `ALTER EXTENSION timescaledb UPDATE;`

# Now for setting up POWA
The following commands where taken from "https://github.com/dalibo/docker/blob/master/powa/powa-archivist/install_all.sql".
* `-- Connect to your database as a superuser:`
* `SET AUTOCOMMIT=ON;`
* `CREATE IF NOT EXISTS EXTENSION hypopg;`
* `CREATE database powa;`
* `SET AUTOCOMMIT=OFF;`
* `-- Reconnect to database "powa"`
* `SET AUTOCOMMIT=ON;`
* `CREATE IF NOT EXISTS EXTENSION pg_stat_statements;`
* `CREATE IF NOT EXISTS EXTENSION btree_gist;`
* `CREATE IF NOT EXISTS EXTENSION pg_qualstats;`
* `CREATE IF NOT EXISTS EXTENSION pg_stat_kcache;`
* `CREATE IF NOT EXISTS EXTENSION pg_track_settings;`
* `CREATE IF NOT EXISTS EXTENSION powa;`
* `SET AUTOCOMMIT=OFF;`
* `-- Reconnect to database "template1"`
* `SET AUTOCOMMIT=ON;`
* `CREATE IF NOT EXISTS EXTENSION hypopg;`
* `SET AUTOCOMMIT=OFF;`

Also make sure to read the documentations (via the links above) of the individual extensions if you run into any woes.

# Connect to POWA-Archivist via POWA-Web
* `docker pull dalibo/powa-web`
* `docker run -d --name powa_web --restart unless-stopped -p 8888:8888 --link postgresql_timescaledb:powa-archivist dalibo/powa-web:latest`
You can now access the POWA web interface via "http://yourdockerhost:8888/" and log in e.g. via the superuser "postgres" (highly insecure, please read the POWA documentation on how to secure it properly: http://powa.readthedocs.io/en/latest/security.html).

# Have fun!
Do it! :-)
