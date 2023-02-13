# Pastadon

## Introduction

This is a repository for setting up an instance of Mastodon using Docker Compose and Cloudflare's Tunnels.
This guide is avoids using the Mastodon setup tool itself, because it was not designed to work with Docker and prevents the user from doing advanced configuration.

## Instructions

### 1. Environment Files

#### .mastodon.env

First, in `.mastodon.env`, fill out all sections marked with `Set me!`. Relevant instructions and/or commands are included for each field.

NOTE: At time of writing, the ElasticSearch and S3 sections are not in this guide yet. 

#### .postgres.env

Next, fill out `.postgres.env`. (This is mostly just copying values from `.mastodon.env`.)

NOTE: If you change the default username, also update the `healthcheck:` for db service in `docker-compose.yml` (see inline comments).

#### .cloudflared.env

Finally, fill out `.cloudflared.env`. The token can be retrieved from Cloudflare's Zero Trust Dashboard > Tunnels.

NOTE: This guide utilises tokens because the `cloudflared` image does not support setting authentication parameters via environment variables as of writing.

### 2. Setup database

Run this command: `docker compose run --rm -e RAILS_ENV=production web bundle exec rake db:setup` to set up the Postgres database.  

### 3. Test starting Mastodon

To start Mastodon, run `docker compose up`. If all looks good, hit `cntrl+c` to stop the containers, then `docker compose down` to remove the containers (only for setup).

### 4. Connect Cloudflared

In the Zero Trust dashboard, configure the tunnel to point to the `web` and `streaming` services. (See `docker-compose.yml` for line instructions next to the `hostname:` fields.)

### 5. Start Mastodon (for good)

To start: `docker compose up -d`

To view logs: `docker compose logs -f`.

## Manually Creating Higher Users
After Mastodon is started, run:

```sh
docker compose exec -e RAILS_ENV=production web bin/tootctl accounts create \
    <username> \
    --email <email> \
    --confirmed \
    --role <Owner|Admin|Moderator>
```

Note that role names are case-sensitive.

## TODOs
1. Adding a cron job to automatically clean cache.
2. Add optional logging configs to limit amount of logs stored.
3. Support ElasticSearch in configs.

## Known Bugs

### 500 on search

If the logs show an error like this:
```
Errno::EACCES (Permission denied @ dir_s_mkdir - /opt/mastodon/public/system/cache):
```

Run this command (shut down the running Mastodon instance first):

`docker compose run --rm --user=root -e RAILS_ENV=production web chown -R 991:991 /opt/mastodon/public`