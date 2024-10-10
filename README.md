# Add Flood as a modern UI for Deluge

This is part 4 of the series: [Deluge via Proton VPN with port forwarding](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward).

This part adds Flood to the stack providing an alternative modern UI.

## Technologies
- [Flood](https://github.com/jesec/flood)

## How it works

### Flood

The `flood` service runs [Flood](https://github.com/jesec/flood) a modern UI for Deluge.

## Getting started

### 1. Set the DNS records

Add an `A` record for the subdomain you intend to use pointing to your server's IP address.

For example: `flood.example.com`

### 2. Set the Caddy environment variables

Update your `caddy/.env` file to include `FLOOD_HOST` from `caddy/.env.example`.

### 3. Start the services

Start flood with:

```bash
docker compose up -d --wait flood
```

Give caddy a moment to obtain a TLS/HTTPS certificate and then navigate to your subdomain to access Flood.

### 4. Configure Flood

Create a unique username and password for flood.

From the `Client` dropdown select `Deluge` and enter the following details:

- Host: `localhost`
- Port: `58846` - The deluge daemon port
- Username: `localclient`
- Password: The value contained in the `/srv/deluge/auth` file

> [!NOTE]
> Flood connects with the deluge daemon itself so the username and password must be obtained from `/srv/deluge/auth` rather than the web UI. That also means the port number is the daemon port, not the web UI.

## Troubleshooting

1. Check the logs
2. Re-read the guide
3. [Ask for help in GitHub Discussions](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/discussions)
4. [Create an issue](https://github.com/RogueOneEcho/how-to-setup-deluge-with-protonvpn-portforward/issues)
