# netmaker-traefik

This is a sample traefik configuration for running Netmaker. It's mostly based on the [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) but using [Traefik Proxy 2.5](https://traefik.io/blog/announcing-traefik-proxy-2-5/) instead of Caddy or Nginx.

## Quick Start

Note you can mostly follow the instructons from [Netmaker Quick Start](https://docs.netmaker.org/quick-start.html) except for a few differences.

1. Prepare DNS - no change
2. Install Dependencies - no change
3. Open Firewall - no change (though this config does expect you'll have firewall allowing private access to your traefik dashboard)
4. Install Netmaker
Instead of using `sed` commands to modify the `docker-config.yaml` I suggest using a `.env` file to store your private/config vars. 
So, `cp sample.env .env`.
Modify this `.env` file similarly to how it is suggested by "Quick Start" step 4, though don't change the key/variable names in the `.env` file
Finally, ensure the `/PATHTO` values are modified in `docker-compose.yaml` to make where you want to store netmaker data and your `acme.json` (the file Traefik uses to track certificate management).

## Commentary

Note that typically one would not run a Traefik proxy with `network_mode: host`, but it's required in this case as we need to proxy `netmaker` (api/grpc) which is also `network_mode: host`. 

Doing it this way allows the proxy to function without other odd configs.

VERY IMPORTANT that your firewall (`ufw` in the Ubuntu case) ONLY allows inbound traffic on the ports desired.

As mentioned in "Quick Start" that is:

- 443 (tcp): for Dashboard, REST API, and gRPC
- 53 (udp and tcp): for CoreDNS
- 51821-518XX (udp): for WireGuard
