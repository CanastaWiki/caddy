# canasta-caddy

A custom [Caddy](https://caddyserver.com/) image for [Canasta](https://canasta.wiki) with the [`caddy-dns/cloudflare`](https://github.com/caddy-dns/cloudflare) module built in.

Published to: **`ghcr.io/canastawiki/caddy`**

---

## Why this exists

Caddy obtains TLS certificates automatically using the ACME **HTTP-01** challenge. When your domain is proxied through Cloudflare, the validation request from Let's Encrypt hits Cloudflare instead of your origin server and is blocked, so the challenge fails and Caddy cannot provision a certificate. Cloudflare then returns a **525 SSL Handshake Error** because it cannot complete a TLS connection to the origin.

This image adds the Cloudflare DNS module, enabling Caddy to use the **DNS-01** challenge instead. The challenge is answered by writing a temporary DNS TXT record via the Cloudflare API — no inbound HTTP traffic is required, so the Cloudflare proxy is no longer an obstacle.

---

## Image tags

| Tag | Description |
|-----|-------------|
| `ghcr.io/canastawiki/caddy:latest` | Latest build (tracks `main`) |
| `ghcr.io/canastawiki/caddy:2.10.2` | Pinned to a specific Caddy version |

Supported architectures: `linux/amd64`, `linux/arm64`

---

## Using with Canasta

### 1. Switch the Caddy image

In your Canasta `docker-compose.override.yml`, replace the stock Caddy image:

```yaml
# Override the default caddy image
services:
  caddy:
    image: ghcr.io/canastawiki/caddy:2.10.2
```

### 2. Get a Cloudflare API token

In the [Cloudflare dashboard](https://dash.cloudflare.com/profile/api-tokens), create an API token with the following permission:

- **Zone → DNS → Edit** (scoped to the zone for your wiki's domain)

### 3. Configure Caddy to use DNS-01

Add the following to your instance's `Caddyfile.global` (this file is not overwritten by upgrades):

```caddy
{
    acme_dns cloudflare <YOUR_API_TOKEN>
}
```

Replace `<YOUR_API_TOKEN>` with the token from step 2.

### 4. Restart Caddy

```bash
canasta restart
```

Caddy will now obtain and automatically renew Let's Encrypt certificates using DNS-01 challenges through the Cloudflare API, regardless of whether your domain is proxied.

---

## Cloudflare SSL/TLS setting

Set your Cloudflare SSL/TLS encryption mode to **Full (Strict)**. This ensures end-to-end encryption between Cloudflare's edge and your origin server using the valid Let's Encrypt certificate Caddy provisions.

---

## How the image is built

The image is built and published automatically via GitHub Actions on every push to `main`. The version tag is derived from the `FROM caddy:<version>-alpine` line in the Dockerfile — updating that line and merging to `main` publishes a new versioned tag alongside `latest`.
