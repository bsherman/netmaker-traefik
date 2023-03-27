# netmaker-traefik

This is a sample traefik configuration for running Netmaker. It's mostly based on the [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) but using [Traefik Proxy 2.9](https://traefik.io/blog/announcing-traefik-proxy-2-9/) instead of Caddy.

## Version 0.17.1

This version of the config supports Netmaker 0.17.1. Please reivew [Netmaker upgrade documentation](https://netmaker.readthedocs.io/en/master/upgrades.html) to determine any required upgrade process.

## Quick Start

Note you can mostly follow the instructons from [Netmaker Quick Start](https://netmaker.readthedocs.io/en/master/quick-start.html) except for a few differences.

Note: This example uses the community version of Netmaker
0. Prerequisites - as instructed
1. Prepare DNS - as instructed
2. Install Dependencies - as instructed
3. Open Firewall - as instructed (though this config does expect you'll have firewall allowing private access to your traefik dashboard)
4. Prepare MQ - as specified but NOTE: our `docker-compose.yml` uses `/PATHTO/` as a placeholder rather than assuming `/root/` so you may want the following instead of the default:

```
wget -O /PATHTO/mosquitto.conf https://raw.githubusercontent.com/gravitl/netmaker/master/docker/mosquitto.conf
wget -q -O /PATHTO/wait.sh https://raw.githubusercontent.com/gravitl/netmaker/develop/docker/wait.sh
chmod +x wait.sh
```

5. Install Netmaker - Instead of downloading and using `sed` commands to modify the `docker-config.yml` I suggest using the provided (in this repo) `docker-compose.yml` and `sample.env` file to store your private/config vars.
- So, `cp sample.env .env`.
- Modify this `.env` file similarly to how it is suggested by "Quick Start" step 5, though don't change anything in the `docker-compose.yml` file, and only change VALUES in the `.env` file, not the key/variable names themselves.
- get the server IP `ip route get 1 | sed -n 's/^.*src \([0-9.]*\) .*$/\1/p'`
- generate 2 unique values for MASTER_KEY/MQ_ADMIN_PASSWORD: `tr -dc A-Za-z0-9 </dev/urandom | head -c 30 ; echo ''`
- Finally, ensure the `/PATHTO` values are modified in `docker-compose.yml` to be where you want to store specified volume data and your `acme.json` (the file Traefik uses to track certificate management).

Assuming you use `/PATHTO`, prepare the docker volumes like so:

```
mkdir -p /PATHTO/netmaker_sqldata
mkdir -p /PATHTO/netmaker_dnsconfig
mkdir -p /PATHTO/netmaker_mosquitto_data
mkdir -p /PATHTO/netmaker_mosquitto_logs
touch /PATHTO/traefik_acme.json
chmod 600 /PATHTO/traefik_acme.json
```

## Commentary

For your security, it is VERY IMPORTANT that your firewall (`ufw` in the Ubuntu/Debian case) ONLY allows inbound traffic on the ports desired, unless you know why you've allowed other ports.

As mentioned in "Quick Start" that is:

- 80 (tcp): for LetsEncrypt certificate creation
- 443 (tcp): for Dashboard and REST API
- 51821-518XX (udp): for WireGuard

Note that though port 80 is open, the Traefik configuration auto-redirects any non-secure HTTP requests to HTTPS. The port IS required, though, to enable LetsEncrypt certificate creation.

## Differences from Caddy Reference

This `docker-compose.yml` for Traefik differs from the reference `docker-compose.caddy.yml` in a few ways.
This detail is provided for the curious.

1. Traefik replaces Caddy and Traefik `labels` are added where appropriate, which Caddy does not use
2. Traefik versions of the Caddyfile basic security headers are included in the docker-compose.yml as of version `0.11`
3. Docker definitions for `sqldata`, `dnsconfig`, and `mosquitto` volumes are fleshed out as local volume bind mounts
4. All other changes are to support the use of `.env` instead of requiring edits to the `docker-compose.yml` file.

## Default Configuration Functionality

In this default configuration the `netmaker` server automatically registers itself as a client named `netmaker-server-1` for each network created. However, instead of running a `netclient` process like typical clients, `CLIENT_MODE: on` means its client is embedded in the server. This allows simple automated behavior and enablement of both the the UDP hole punching and egress gateway routing features at the expense of the ability to connect to the host machine via a `netmaker` managed network.

