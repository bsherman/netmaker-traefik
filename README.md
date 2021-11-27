# netmaker-traefik

This is a sample traefik configuration for running Netmaker. It's mostly based on the [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) but using [Traefik Proxy 2.5](https://traefik.io/blog/announcing-traefik-proxy-2-5/) instead of Caddy or Nginx.

## Quick Start

Note you can mostly follow the instructons from [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) except for a few differences.

1. Prepare DNS - no change
2. Install Dependencies - no change
3. Open Firewall - no change (though this config does expect you'll have firewall allowing private access to your traefik dashboard)
4. Install Netmaker - Instead of using `sed` commands to modify the `docker-config.yml` I suggest using a `.env` file to store your private/config vars.
So, `cp sample.env .env`.
Modify this `.env` file similarly to how it is suggested by "Quick Start" step 4, though don't change anything in the `docker-compose.yml` file, and only change VALUEs in the `.env` file, not the key/variable names themselves.
Finally, ensure the `/PATHTO` values are modified in `docker-compose.yml` to be where you want to store netmaker data and your `acme.json` (the file Traefik uses to track certificate management).

Assuming you use `/PATHTO`, prepare the docker volumes like so:

```
mkdir -p /PATHTO/netmaker_sqldata
mkdir -p /PATHTO/netmaker_dnsconfig
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
2. Docker volumes for `sqldata` and `dnsconfig` are fleshed out as local volume bind mounts
3. For the `netmaker` container `VERBOSITY: 0` has been added to allow quick changes to logging verbosity if more logging is requred for troubleshooting
4. For `netmaker-ui` and `netmaker`, ports have been removed to limit any possible external exposure where Traefik can instead access them directly on the internal docker network.
5. All other changes are to support the use of `.env` instead of requiring edits to the `docker-compose.yml` file.


## Default Configuration Functionality and Limitations

It is important to note that in this default configuration the `netmaker` server automatically registers itself as a client named `netmaker` for each network created. However, instead of running a `netclient` process like typical clients, `CLIENT_MODE: on` means its client is embedded in the server. This allows simple automated behavior and enablement of both the the UDP hole punching and egress gateway routing features. The simple mode is accomplished by providing the `netmaker` container privileged access to the host system, thus allowing it to manage all wireguard and iptables packet handling for the system.

The previous release had pulled all this management into the container itself, however, it led to one potentially significant limitation for the default configuration... remote client members of a managed network were NOT able to access the netmaker host system via the netmaker managed networks. For example, if you had a client at `10.10.10.2` it would not be able to SSH to netmaker host system at `10.10.10.1`, the user would need to SSH to the host's public IP.

