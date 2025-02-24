# syncstorage-rs-docker

A simple Docker container and Docker Compose configuration to get started with [mozilla-services/syncstorage-rs](https://github.com/mozilla-services/syncstorage-rs) to self-host a Firefox sync server.

I didn't have much luck with the existing documentation and wrote this for my own infrastructure. There's no guarantee it will work for you.

## Getting started

So far these steps have been shown to work on Debian based distros including Raspbian and Debian Bullseye.

To get started clone this repository. You will need to have [Docker](https://docs.docker.com/engine/install/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.

### Environment Variables

The Docker Compose file makes use of environment variables. To configure them, make a copy of example.env

```bash
cp example.env .env
```

Now edit the new `.env` file to add configuration and secrets. Keep in mind the `SYNC_MASTER_SECRET` and `METRICS_HASH_SECRET` require 64 characters.

### Initial Run

```bash
docker compose up -d --build && docker compose logs -f
```

The first time you run the application, it will do a few things:

1. MariaDB container will be pulled and on first run it will load the `./data/init/init.sql` script that creates the required databases and user permissions. This will only run during the initial setup.

2. Next the Dockerfile will build the syncserver app. This is a Rust app and all of the required dependencies will be loaded into the environment, as well as cloning the Mozilla syncstorage-rs repo. This will take several minutes.

3. Once everything is compiled and configured you should see startup logs begin to appear. Subsequent runs of `docker compose up -d` will happen much faster because the build artifacts are cached. Data is persisted in the database (`./data/config`) between restarts.

### Rebuilding Everything

In the course of setting this up, you may need to tear down and rebuild your instance. To remove persisted data and artifacts, run the following.

```bash
docker compose down
docker image rm app-syncserver
docker builder prune -af
rm -rf ./data/config
```

This will delete the compiled Rust app and any cached layers, and also delete the database data.

### Firefox Setup

Once your app is running, you can configure Firefox by updating the `about:config` settings.

`identity.sync.tokenserver.uri` needs to be set to the `SYNC_URL` configured in your `.env` file followed by `/token/1.0/sync/1.5`. 

>Example: http://sync.example.com:8000/token/1.0/sync/1.5

To confirm the sync is working you can enable success logs in `about:config` also. Set `services.sync.log.appender.file.logOnSuccess` to true. Now you should see sync logs in `about:sync-log`

Syncing is usually very quick, and when a sync occurs you can see logs in `docker compose logs -f` also.
