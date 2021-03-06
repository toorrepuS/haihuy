![](https://i0.wp.com/trungquan710.com/wp-content/uploads/2015/05/postgresql.png?fit=468%2C415&ssl=1)

Postgres.app is a full-featured PostgreSQL installation packaged as a standard Mac app. It includes everything you need to get started: we’ve even included popular extensions like [PostGIS](http://postgis.net/) for geo data and [plv8](https://github.com/plv8/plv8) for JavaScript.

## Setting on Mac OS
### Postgres.app
Postgres.app has a beautiful user interface and a convenient menu bar item. You never need to touch the command line to use it – but of course we do include all the necessary [command line tools](https://postgresapp.com/documentation/cli-tools.html) and header files for advanced users.

[Link download Postgres.app](https://postgresapp.com/downloads.html)

## Setting on Ubuntu 18.04
> sudo apt-get install postgresql postgresql-contrib

- Login to postgresql cli:
> sudo -u postgres psql

### Config postgres server to client connect
> sudo su postgres

> vim /etc/postgresql/10/main/pg_hba.conf

host    all             all             0.0.0.0/0               trust

> vim /etc/postgresql/10/main/postgresql.conf

listen_addresses = '*'

> sudo service postgresql stop

> sudo service postgresql start

## Create multiple Postgres instances on same machine
### Create the clusters
> /usr/lib/postgresql/10/bin/initdb -D /var/lib/postgresql/10/main/datadb1
> /usr/lib/postgresql/10/bin/initdb -D /var/lib/postgresql/10/main/datadb2

### Run the instances
> /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main/datadb1 -o "-p 5433" -l logfile start
> /usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main/datadb2 -o "-p 5434" -l logfile start

Ref: [Create multiple Postgres instances on same machine](https://stackoverflow.com/questions/37861262/create-multiple-postgres-instances-on-same-machine)

## List Postgres packages installed
> dpkg -l | grep postgres

## Remove/Uninstall Postgres Ubuntu
> sudo apt-get --purge remove $(dpkg -l | grep postgres | awk '{print($2)}')

## How to connect from python
> pip install psycopg2

```python
from sqlalchemy import create_engine
engine = create_engine('postgresql://localhost/[DATABASE])
```

## Useful Command
* Enlisting all the available commands
	> \?

* Knowing the syntaxes of PostgreSQL statements
	> \h DROP TABLE

* Open database
	> psql testdb_1

* List of all available databases
	> \l

* List user tables
	> \dt+

* List tablespace
	> \db

* List Access privileges
	> \z
	
* Switching to other database
	> \c <database_name>

* Describing a particular table
	> \d <table_name>
	
* Knowing the version of PostgreSQL
	> select version();

* Seeing the previously executed command
	> \g

* Knowing the execution times of queries
	> \timing

* psql + text editor
	> \e
