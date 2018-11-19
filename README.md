Database Janitor
---

Highly-configurable database dumper

## Features

 - First-party support for Drupal databases
 - Configuration-first
 - PHP with minimal dependencies

## Usage

This application can either be used as a CLI app or a library that other applications can build on top of, e.g a drush
command or another custom application.

### Testing

There is a sample Lando config you can use to spin up two separate mysql databases for testing, as well as a sample SQL
file containing random data.

```bash
lando start
lando db-import sampledata.sql --host real_database; lando db-import sampledata.sql --host trim_database
```

### Configuration

Configuration can be done in three ways, and it's recommended you use all three. First, values are loaded from the
command-line arguments. The CLI command then looks for specific environment variables, finally falling back to the
`janitor.json` file.

|JSON Key|ENV Key|CLI Key|What it does|Default value|
|---|------------|-------------|---|---|
|`sanitize_users`|none|none|Tells janitor whether or not to run drupal-specific sanitation on dump.|`true`|
|`trim`|none|`--trim`|Whether or not we should attempt to cut down results in dump.|`false`|
|`sanitize_tables`|none|none|Specific tables and their columns to sanitize.| |
|**not recommended** `host` |`DB_JANITOR_HOST`|`--host=[host]`|Specifies database host to connect to.| |
|**not recommended** `password` |`DB_JANITOR_PASSWORD`|`--password=[password]`|Specifies database user password.| |
|**not recommended** `database` |`DB_JANITOR_DATABASE`|`--database=[database]`|Specific database to dump.| |
|**not recommended** `user` |`DB_JANITOR_USER`|`--username=[username]`|Database user.| |
| none | |`--config=[config file]`|Custom configuration file.| |
| `trim_database` | none | none | The server to use when trimming data (see: [#Trimming](#trimming)).|Lando|
| `trim_tables` | none | none | Tables to trim prior to dumping data.| |
| `excluded_tables` | none | none | Tables to exlude when dumping data.| |

```json
{
  "config": {
    "sanitize_users": true,
    "trim": false,
    "trim_database": {
      "host": "localhost:8787",
      "user": "trim",
      "password": "trim",
      "database": "trim"
    }
  },
  "sanitize_tables": {
    "some_table": [
      "some_col"
    ]
  },
  "trim_tables": [
    "some_table"
  ],
  "excluded_tables": [
    "some_table"
  ]
}
```

```bash
export DB_JANITOR_HOST='localhost'
export DB_JANITOR_USER='[sql user]'
export DB_JANITOR_PASSWORD='[sql password]'
export DB_JANITOR_DATABASE='[sql database]'
```

Remember, Database Janitor prefers arguments > env variables > json configuration.

### CLI

First you'll want to copy `janitor.example.json` to `janitor.json`, which will serve as the primary configuration file.
You can technically define all your configuration within, but it's strongly encouraged you move more sensitive values
(passwords, usersnames) to either the command line args or ENV variables.

Then install our dependencies with `composer install`.

#### Dumping

This will produce a gzip'd .sql file in the `output/` directory.

```bash
php src/DatabaseJanitorCli.php --host=localhost --username=[sql username] --password=[sql password] --database=[sql database]
```

#### Trimming

Trimming is an experimental feature to try and reduce the amount of data in a dump, allowing for smaller 
local databases for development.

**Warning** 
You'll want to edit the `trim_database` configuration to point to a _new server_. **DO NOT** run the trim
command on your actual database - it's recommended you dump the data first, then import it into a seperate
database since this command is destructive.

It is not currently possible pass this database as CLI arguments, so it's recommenced you run this under
a Docker environment so you don't leak your actual server credentials (even better, have this as part of
your CI).

```bash
php src/DatabaseJanitorCli.php --trim=true
```
