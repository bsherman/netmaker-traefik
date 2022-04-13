# netmaker-traefik

This is a sample traefik configuration for running Netmaker. It's mostly based on the [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) but using [Traefik Proxy 2.5](https://traefik.io/blog/announcing-traefik-proxy-2-5/) instead of Caddy or Nginx.

## Version 0.12.2

This version of the config supports Netmaker 0.12.2. Per Netmaker documentation, it is NOT advised to upgrade a previous version to 0.12.

## Quick Start

Note you can mostly follow the instructons from [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) except for a few differences.

1. Prepare DNS - as instructed
2. Install Dependencies - as instructed
3. Open Firewall - as instructed (though this config does expect you'll have firewall allowing private access to your traefik dashboard)
4. Install Netmaker - Instead of using `sed` commands to modify the `docker-config.yml` I suggest using a `.env` file to store your private/config vars.
So, `cp sample.env .env`.
Modify this `.env` file similarly to how it is suggested by "Quick Start" step 4, though don't change anything in the `docker-compose.yml` file, and only change VALUEs in the `.env` file, not the key/variable names themselves.
Finally, ensure the `/PATHTO` values are modified in `docker-compose.yml` to be where you want to store netmaker data and your `acme.json` (the file Traefik uses to track certificate management).

You can skip the **Prepare Caddy** and **Prepare MQ** steps as you aren't using Caddy and you will get the MQ config in the code block below.

Assuming you use `/PATHTO`, prepare the docker volumes like so:

```
mkdir -p /PATHTO/netmaker_sqldata
mkdir -p /PATHTO/netmaker_dnsconfig
mkdir -p /PATHTO/netmaker_mosquitto_data
mkdir -p /PATHTO/netmaker_mosquitto_logs
wget -O /PATHTO/mosquitto.conf https://raw.githubusercontent.com/gravitl/netmaker/master/docker/mosquitto.conf
touch /PATHTO/traefik_acme.json
chmod 600 /PATHTO/traefik_acme.json
```


## Commentary

It is VERY IMPORTANT that your firewall (`ufw` in the Ubuntu/Debian case) ONLY allows inbound traffic on the ports desired.

As mentioned in "Quick Start" that is:

- 443 (tcp): for Dashboard, REST API, and gRPC
- 53 (udp and tcp): for CoreDNS
- 51821-518XX (udp): for WireGuard


## Differences from Caddy Reference

This `docker-compose.yml` for Traefik differs from the reference `docker-compose.caddy.yml` in a few ways.
This detail is provided for the curious.

1. Traefik replaces Caddy and Traefik `labels` are added where appropriate, which Caddy does not use
2. Traefik versions of the Caddyfile basic security headers are included in the docker-compose.yml as of version `0.11`
3. Docker definitions for `sqldata`, `dnsconfig`, and `mosquitto` volumes are fleshed out as local volume bind mounts
4. For `netmaker-ui` and `netmaker`, ports have been removed to limit any possible external exposure where Traefik can instead access them directly on the internal docker network.
5. All other changes are to support the use of `.env` instead of requiring edits to the `docker-compose.yml` file.


## Default Configuration Functionality

In this default configuration the `netmaker` server automatically registers itself as a client named `netmaker-1` for each network created. However, instead of running a `netclient` process like typical clients, `CLIENT_MODE: on` means its client is embedded in the server. This allows simple automated behavior and enablement of both the the UDP hole punching and egress gateway routing features at the expense of the ability to connect to the host machine via a `netmaker` managed network.

