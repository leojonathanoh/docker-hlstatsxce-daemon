# docker-hlstatsxce-daemon

[![github-actions](https://github.com/startersclan/docker-hlstatsxce-daemon/workflows/build/badge.svg)](https://github.com/startersclan/docker-hlstatsxce-daemon/actions)
[![github-tag](https://img.shields.io/github/tag/startersclan/docker-hlstatsxce-daemon)](https://github.com/startersclan/docker-hlstatsxce-daemon/releases/)
[![docker-image-size](https://img.shields.io/microbadger/image-size/startersclan/docker-hlstatsxce-daemon/latest)](https://hub.docker.com/r/startersclan/docker-hlstatsxce-daemon)
[![docker-image-layers](https://img.shields.io/microbadger/layers/startersclan/docker-hlstatsxce-daemon/latest)](https://hub.docker.com/r/startersclan/docker-hlstatsxce-daemon)

Docker image for the [HLStatsX:CE](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/) perl daemon.

## Variants

Each variant contains additional perl modules.

These below variants use `Ubuntu:16.04`. Variants with suffix `-alpine` use the `alpine:3.8`

| Name | Perl Modules |
|:-------:|:---------:|
| `:cron` | `DBI`<br>`DBD::mysql`
| `:geoip` | `Geo::IP::PurePerl` and dependencies
| `:geoip2` | `MaxMind::DB::Reader` and dependencies<br> `MaxMind::DB::Reader::XS` and dependencies
| `:emailsender` | `Email::Sender::Simple` and dependencies

NOTE: Chained tags E.g. `:geoip-geoip2-emailsender` contain the `:geoip`, `:geoip2`, and `:emailsender` Perl modules.

## Docker

```sh
docker run -d \
    -e LOG_LEVEL=1 \
    -e MODE=Normal \
    -e DB_HOST=db \
    -e DB_NAME=hlstatsxce \
    -e DB_USER=hlstatsxce \
    -e DB_PASSWORD=hlstatsxce \
    startersclan/docker-hlstatsxce-daemon:geoip

# Alternatively, if you prefer to use a config file instead of environment variables
docker run -d \
    -v /path/to/hlxce/scripts/hlstats.conf:/app/hlstats.conf \
    startersclan/docker-hlstatsxce-daemon:geoip
```

## Example (Swarm Mode with Docker Secrets):

```sh
docker service create --name hlstatsxce-daemon \
    -e LOG_LEVEL=1 \
    -e MODE=Normal \
    -e DB_HOST=db \
    -e DB_NAME=DOCKER-SECRET:secret_db_name \
    -e DB_USER=DOCKER-SECRET:secret_db_user \
    -e DB_PASSWORD=DOCKER-SECRET:secret_db_password \
    --secret secret_db_name \
    --secret secret_db_user \
    --secret secret_db_password \
    startersclan/docker-hlstatsxce-daemon:geoip
```

The entrypoint script takes care of expanding the environment variables `DB_NAME`, `DB_USER`, and `DB_PASSWORD` from the respective secret files `/run/secrets/secret_db_name`, `/run/secrets/secret_db_user`, and `/run/secrets/secret_db_password`. This is done by using the syntax `ENVVARIABLE=DOCKER-SECRET:docker_secret_name` (note the colon).

## Environment variables

Environment variables are optional. Use them only if:

1. not using a config file

2. using the config file `./hlstats.conf` in same directory as `hlstats.pl`, but want to override the config file's settings.

In general, it is better to use environment variables than a config file, because it offers more configuration options.

| Name | Default value (as in `hlstats.pl`) | Description | Corresponds to `hlstats.pl` argument |
|:-------:|:---------:|:---------:|:---------:|
| `CONFIG_FILE` | `"./hlstats.conf"` | Path to config file. May be absolute or relative | `-c,--configfile`
| `LOG_LEVEL` | `"1"` | Log level for debugging | 0 - `-n, --nodebug`<br /> 1 - Default <br />  2 - `-d, --debug`
| `MODE` | `"Normal"` | Player tracking mode (`Normal`, `LAN` or `NameTrack`) | `-m, --mode`
| `DB_HOST` | `"localhost"` | Database IP or hostname, in format `<ip>` or `<hostname>`. Port may be omitted, in which case it is `27500` by default. To use a custom port, use format `<ip>:<port>` or `<hostname>:<port>` specifed. | `--db-host`
| `DB_NAME` | `"hlstats"` | Database name | `--db-name`
| `DB_USER` | `""` | Database user | `--db-name`
| `DB_PASSWORD` | `""` | Database password | `--db-password`
| `DNS_RESOLVE_IP` | `"true"` | Resolve player IP addresses to hostnames (requires working DNS) | `--dns-resolveip`
| `DNS_RESOLVE_IP_TIMEOUT` | `"5"` | timeout DNS queries after SEC seconds | `--dns-timeout`
| `LISTEN_IP` | `""` | IP to listen on for UDP log data | `--ip`
| `LISTEN_PORT` | `"27500"` | Port to listen on for UDP log data | `--port`
| `RCON` | `"true"` | Enable rcon to gameservers | `--rcon`
| `STDIN` | `"false"` | Read log data from standard input, instead of from UDP socket. Must specify `STDIN_SERVER_IP` and `STDIN_SERVER_PORT` to indicate the generatorof the inputted log data (implies `--norcon`) | `-s, --stdin`
| `STDIN_SERVER_IP` | `""` | Data source IP address. Only required for `STDIN` | `--server-ip`
| `STDIN_SERVER_PORT` | `"27015"` | Data source port. Only required for `STDIN` | `--server-port`
| `USE_LOG_TIMESTAMP` | `"false"` for UDP; `"true"` for `STDIN` | Use the timestamp in the log data, instead of the current time on the daemon, when recording events | `-t, --timestamp`
<!-- | `EVENT_QUEUE_SIZE` | `"10"` | Event buffer size before flushing to the db (recommend 100+ for STDIN) | `--event-queue-size` -->

### Configuration via configuration file and database options

When a file `./hlstats.conf` is in the same folder as `hlstats.pl` (which by default, it is), whenever the daemon is started or reloaded, [configuration options](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/src/11cac08de8c01b7a07897562596e59b7f0f86230/scripts/hlstats.pl#lines-1755) are [read](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/src/11cac08de8c01b7a07897562596e59b7f0f86230/scripts/hlstats.pl#lines-1882) from the [database table `hlstats_options`](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/src/11cac08de8c01b7a07897562596e59b7f0f86230/sql/install.sql#lines-3901) which will [override those specified on the command line](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/src/11cac08de8c01b7a07897562596e59b7f0f86230/scripts/hlstats.pl#lines-1829) (the environment variables simply generate a command line). Therefore, be very aware that setting environment variables does not mean that configuration is actually used by the daemon.

Looking at the [`install.sql`](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/src/11cac08de8c01b7a07897562596e59b7f0f86230/sql/install.sql#lines-3901), you notice some database configuration options that may override the above environment variables:

- `--dns-resolveip` is enabled since parameter `DNSResolveIP` value is `1`
- `--dns-timeout` is `3` since parameter `DNSTimeout` value is `3`
- `--mode` is `Normal` since parameter `Mode` value is `Normal`
- `--rcon` is enabled since parameter `Rcon` value is `1`
- `--timestamp` is disabled since parameter `UseTimestamp` value is `0`

### Warning: If using `CONFIG_FILE`

There is a bug in `hlstats.pl` that does not allow the passing of `--configfile=<configfile>` properly. To fix that, find the line in [`hlstats.pl`](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/src/11cac08de8c01b7a07897562596e59b7f0f86230/scripts/hlstats.pl#lines-1821) on line `1821`:

```perl
if ($configfile && -r $configfile) {
```

Add this code line before it:

```perl
setOptionsConf(%copts);
```

Save the file. That should fix hlstats.pl's `--configfile` argument issue.

## FAQ

### How to use GeoIP2 with the perl daemon?

- As of [`HLStatsX:CE 1.6.19`](https://bitbucket.org/Maverick_of_UC/hlstatsx-community-edition/downloads/), the perl daemon scripts uses [GeoIP](https://metacpan.org/pod/Geo::IP::PurePerl), and not [GeoIP2](https://metacpan.org/pod/GeoIP2). You will have to change a bit of the code yourself to use the [GeoIP2 API](https://metacpan.org/release/GeoIP2).

### How long will this Docker Image be supported?

- As long as the repository is not marked deprecated, which should not happen.
