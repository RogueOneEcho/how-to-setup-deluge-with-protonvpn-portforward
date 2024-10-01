# Automated cross seeding with Prowlarr, cross-seed, and fertilizer

This is part 2 of the series: [Deluge via Proton VPN with port forwarding](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward).

This part adds Prowlarr, cross-seed, and fertilizer to the stack for automatic cross seeding.

## Technologies
- [Prowlarr](https://prowlarr.com/) for managing indexers.
- [cross-seed v6](https://github.com/cross-seed/cross-seed) for cross seeding movie and TV torrents between indexers.
- [Fertilizer](https://github.com/moleculekayak/fertilizer) for cross seeding music torrents between indexers.

## How it works

### Prowlarr

The `prowlarr` service runs [Prowlarr](https://prowlarr.com/) which handles searching torrent indexers.

### cross-seed

The `cross-seed` service runs [cross-seed v6](https://www.cross-seed.org/docs/v6-migration) to cross seed movie and TV torrents between indexers.

Torrents to cross seed are identified in a few different ways:
- Past - A **search** of your existing `.torrent` files.
- Present - An API **request** from Deluge when a new torrent is downloaded.
- Future - Monitoring the **RSS** feed of each indexer to identify any new uploads that match your existing torrents.

Instead of connecting to the indexers directly all searches are done through Prowlarr.

To minimize requests a cache is used to store the results of each search.

> [!WARNING]
> cross-seed v6 is a pre-release of a major version so the configuration may change.

## Getting started

### 0. Checkout the `part-2` branch

In part 1 you checked out the `part-1` branch and modified some `.env` files.

The `.env` files are ignored by git so you should be able to switch branches without any issues.

Check the status of the working directory to ensure it's in a clean state:

```bash
git status
```

Then checkout the `part-2` branch to add the additional services to the stack:

```bash
git checkout part-2
```

### 1. Start the services

We can jump straight in and start the services:

```bash
docker compose up -d --wait
```

Everything should start up and turn `Healthy` without any *apparent* issues, although, the `cross-seed` service will quietly exit because it's not yet configured.

### 1. Setup Prowlarr

Open Prowlarr in your browser: http://localhost:9696

Configure authentication using the `Forms (Login Page)` method.

Add your indexers. If you need help with this then refer to the [Prowlarr quick start guide](https://wiki.servarr.com/prowlarr/quick-start-guide) or search for `prowlarr` in the forums of your indexer for tips.

### 2. Configure cross-seed

> [!NOTE]
> cross-seed doesn't support configuration by environment variables so as a workaround we're defining the environment variables
> that are read by Docker Compose and passed to the container command line arguments.

Copy `.env.example` to `.env`:

```bash
cp .env.example .env
```

Copy the `Torznab Url` of each indexer in Prowlarr to `CROSS_SEED_TORZNAB` with each URL separated by a space and append `?apikey=YOUR_API_KEY` to each one.

> [!TIP]
> - The `Torznab Url` can be found in Prowlarr by clicking on the indexer and copying the `Torznab Url` from the `Details` tab.
> - The API keycan be found in Prowlarr under  `Settings > General > Security > API Key`.
> - Refer to the [cross-seed documentation for further guidance](https://www.cross-seed.org/docs/basics/options#torznab)

The `.env` should look something like this:

```ini
CROSS_SEED_TORZNAB="http://localhost:9696/1/api?apikey=YOUR_API_KEY http://localhost:9696/2/api?apikey=YOUR_API_KEY"
```

Update `CROSS_SEED_DELUGE_RPC_URL` to include your Deluge web client password.

> [!IMPORTANT]
> As this uses HTTP basic auth your password can't include special characters.
> Use only letters and numbers but increase the length (at least 30 characters) to make it more secure.

Set `CROSS_SEED_API_KEY` to a random string of characters that's at least 24 characters.

Start the `cross-seed` service again and follow its logs to ensure it's running correctly:

```bash
docker compose up -d cross-seed && docker compose logs -f cross-seed
```

### 3. Configure Fertilizer

Copy `fertilizer/.env.example` to `fertilizer/.env`:

```bash
cp fertilizer/.env.example fertilizer/.env
```

Set `RED_KEY` to your RED API Key - you can use the same key you added to Prowlarr.
Set `OPS_KEY` to your OPS API Key - you can use the same key you added to Prowlarr.
Set `DELUGE_RPC_URL` to the same value you used for `CROSS_SEED_DELUGE_RPC_URL` in step 2.`

### 4. Configure Deluge to notify cross-seed and Fertilizer torrents are downloaded

> [!NOTE]
> Fertilizer and cross-seed need to be informed of new torrents downloaded by Deluge so they can immediately search for them.
>
> This is done by configuring Deluge to make an API request to cross-seed and Fertilizer when a new torrent finishes downloading.
>
> This requires a shell scripts be included in the Deluge container. Therefore, in this part we're using a custom build for
> the `deluge` service.

In the Deluge Web UI go to `Preferences > Plugins` and enable the `Execute` and `Label` plugins.

Then under `Preferences > Execute` add a couple of new commands:

1. Set `Event` to `Torrent Complete` and `Command` set to `/cross-seed`.
2. Set `Event` to `Torrent Complete` and `Command` set to `/fertilizer`.

Next we'll need to

It's that simple!

Test it out by adding a new torrent to Deluge and watch the logs of the `cross-seed` and `fertilizer` services.

```bash
docker logs -f cross-seed
docker logs -f fertilizer
```

## Next steps

-  Part 3 [Use a custom domain and HTTPS/TLS with Caddy](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/tree/part-3)

## Troubleshooting

1. Check the logs
2. Re-read the guide
3. [Ask for help in GitHub Discussions](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/discussions)
4. [Create an issue](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues)
