# Umami with Tailscale Sidecar Configuration

This Docker Compose configuration sets up [Umami](https://github.com/umami-software/umami) with Tailscale as a sidecar
container to keep the app reachable over your Tailnet.

## Umami

[Umami](https://github.com/umami-software/umami) is an open-source web analytics platform that respects user privacy. No
cookies, no tracking across sites, no personal data collection. GDPR compliant out of the box. This configuration
leverages Tailscale to securely connect to your Umami dashboards, protecting your analytics data from unauthorized
access.

## Configuration Overview

In this setup, the `tailscale-umami` service runs Tailscale, which manages secure networking for Umami. The
`Umami` service utilizes the Tailscale network stack via Docker's `network_mode: service:` configuration.

## Prerequisites

- Docker and the Compose plugin, with your user in the `docker` group (or use `sudo`).
- `/dev/net/tun` available on the host and the `NET_ADMIN` capability, both already declared in `compose.yaml`.
- A Tailscale [auth key](https://console.tailscale.com/admin/settings/keys) from the web admin console (**Settings →
  Keys → Generate auth key**). Set it to "Pre-Approved" if that option appears. The key is used only for the initial
  registration — with `TS_AUTH_ONCE=true` and the persisted `ts/state` volume, restarts reuse the stored node state — so
  a single-use key is sufficient. Tagging the device disables key expiry, which avoids re-authentication after the
  default 180 days.
- HTTPS certificates [enabled for your Tailnet](https://console.tailscale.com/admin/dns) (**DNS → HTTPS Certificates**).
  Tailscale Serve cannot issue a certificate without it, and the container will start but never serve.
- Funnel must be allowed for the node in the tailnet policy if using public mode.

## Files to check

Please verify the following files and variables before deploying:

- `.env` set `TS_AUTHKEY`, `TZ`, `APP_SECRET`, `DB_PASSWORD`, `SERVE_CONFIG`

## Usage Notes

By default, `SERVE_CONFIG` is `private`. This keeps the app Tailnet-only, and available on port 443. When `SERVE_CONFIG`
is set to `public`, the `/script.js` and `/api/send` paths are exposed by funnel on port 443, while the rest of the
app is accessible only over the Tailnet on port 8443. Public mode is useful for tracking analytics from websites
whose clients are not on the Tailnet.

Default credentials for Umami are `admin` / `umami`. Change these default admin / umami password right after the
first login.

To start collecting data, follow the Umami [documentation](https://docs.umami.is/docs/collect-data) to set up a
website, then embed the tracking script like so:

```html

<script defer src="https://umami.<tailnet>.ts.net/script.js" data-website-id="...">
```

## References

- [Umami documentation](https://umami.is/docs)
- [Umami on GitHub](https://github.com/umami-software/umami)
