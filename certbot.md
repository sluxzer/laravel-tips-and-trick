renew certbot manually

certbot certonly -d *.elite-pos.site --manual --preferred-challenges dns

Reference:
https://teletype.in/@alteregor/nginx-certbot-wildcard



# Wildcard TLS with Certbot (Manual / DNS-01)

Issue and install a Let’s Encrypt **wildcard** certificate for `*.example.com` (and the apex `example.com`) using **manual DNS verification**.

> **What “manual” means**  
> You’ll add a temporary **TXT** record in your DNS by hand whenever you issue/renew. Manual DNS-01 **cannot auto-renew** without DNS API hooks. If you need automation later, see the [Automation](#automation-options-later) section.

---

## Prerequisites

- A Linux server (Ubuntu 20.04/22.04/24.04 recommended) with `sudo`.
- A registered domain (e.g. `example.com`) and access to its DNS control panel.
- Ports **80/443** open (only needed to serve HTTPS after issuance).
- Email address for Let’s Encrypt notices (expiry, issues).

---

## 1) Install Certbot

### Ubuntu (snap)

```bash
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
certbot --version
```

---

## 2) Request a Wildcard Certificate (Manual DNS-01)

> Replace `example.com` and the email before running.

```bash
sudo certbot certonly   --manual   --preferred-challenges dns   -d *.example.com -d example.com   --agree-tos -m you@example.com --no-eff-email   --manual-public-ip-logging-ok
```

Certbot will pause and show a **TXT record** you must create, something like:

- **Name / Host**: `_acme-challenge.example.com`  
- **Type**: `TXT`  
- **Value**: `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`

### Add the TXT record in DNS

1. Go to your DNS provider (Cloudflare, Route 53, registrar panel, etc.).
2. Create the TXT record exactly as shown.
3. If there is already an `_acme-challenge` TXT record **for the same issuance** (e.g., adding both `example.com` and `*.example.com`), you can **add multiple TXT values** to the same `_acme-challenge` name.

### Wait for DNS to propagate, then verify

Check from a public resolver (e.g., Google 8.8.8.8):

```bash
dig TXT _acme-challenge.example.com +short @8.8.8.8
```

You should see **exactly** the value(s) Certbot printed.  
When ready, **press Enter** back in the Certbot prompt to continue validation.

If validation succeeds, Certbot will place your certs here:

```
/etc/letsencrypt/live/example.com/
  ├─ cert.pem        # leaf cert
  ├─ chain.pem       # intermediate
  ├─ fullchain.pem   # cert + chain (use this)
  └─ privkey.pem     # private key
```

> Keep `privkey.pem` secret and permissions restricted (root:root, 600).

---

## 3) Configure Your Web Server (Nginx example)

Install Nginx if needed:

```bash
sudo apt-get update && sudo apt-get install -y nginx
```

Create an HTTPS server block (e.g., `/etc/nginx/sites-available/example.com.conf`):

```nginx
server {
  listen 80;
  listen [::]:80;
  server_name  example.com  *.example.com;
  # Optional: redirect all HTTP to HTTPS
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name  example.com  *.example.com;

  # TLS certs from Certbot
  ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

  # Recommended TLS settings (modern defaults)
  ssl_session_timeout 1d;
  ssl_session_cache shared:SSL:50m;
  ssl_session_tickets off;

  # OCSP Stapling (optional)
  ssl_stapling on;
  ssl_stapling_verify on;
  resolver 1.1.1.1 8.8.8.8 valid=300s;
  resolver_timeout 5s;

  # Your app
  root /var/www/html;
  index index.html index.htm;

  location / {
    try_files $uri $uri/ =404;
  }
}
```

Enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Visit `https://example.com` and a subdomain like `https://app.example.com`.

---

## 4) Renewal (Manual)

Wildcard certificates from Let’s Encrypt are valid for **90 days**.

Because we used **`--manual`**, you must repeat issuance (and DNS TXT creation) **each time**:

```bash
sudo certbot certonly   --manual   --preferred-challenges dns   -d *.example.com -d example.com   --agree-tos -m you@example.com --no-eff-email   --manual-public-ip-logging-ok
```

- Certbot will again prompt for a **new TXT value**.  
- After successful issuance, Nginx can keep using the same paths; just reload if needed:

```bash
sudo systemctl reload nginx
```

> **Note**: `certbot renew --dry-run` **does not work** with `--manual` challenges.

---

## Automation Options (Later)

If you get tired of manual TXT updates, switch to one of these:

- **DNS plugins** (preferred):  
  Use your provider’s Certbot plugin (e.g., `python3-certbot-dns-cloudflare`, `...-dns-route53`, `...-dns-digitalocean`) and supply API credentials. Then Certbot can add/remove `_acme-challenge` automatically and **auto-renew** via cron/systemd timers.
- **acme.sh**: Lightweight ACME client with many DNS APIs supported.

---

## Troubleshooting

- **Propagation delay**: TXT records can take minutes. Keep TTL low (e.g., 60s) if your DNS supports it.
- **Multiple TXT values**: When validating both `*.example.com` and `example.com` together, you may need **two TXT values** at the **same** `_acme-challenge.example.com` name.
- **CAA records**: If you use CAA, ensure Let’s Encrypt is allowed, e.g.:
  ```
  example.com.  CAA 0 issue "letsencrypt.org"
  ```
- **DNSSEC**: Misconfigured DNSSEC can break validation—verify your zone health if challenges fail.
- **Wildcard limits**: `*.example.com` covers `api.example.com`, `www.example.com`, etc., but **not** `a.b.example.com`. For deeper subdomains, you need another wildcard (e.g., `*.b.example.com`) or an explicit cert.
- **File permissions**: Ensure Nginx (root at startup) can read `/etc/letsencrypt/live/...` paths. Default Certbot permissions are correct; avoid changing ownership to `www-data`.

---

## Useful Commands

```bash
# Show installed certificates
sudo certbot certificates

# Test Nginx config
sudo nginx -t

# Reload Nginx after renewing
sudo systemctl reload nginx

# Check TXT via public resolver
dig TXT _acme-challenge.example.com +short @8.8.8.8
```

---

## Uninstall / Revoke (If Needed)

```bash
# Revoke a certificate
sudo certbot revoke --cert-path /etc/letsencrypt/live/example.com/cert.pem

# Delete from local store
sudo certbot delete --cert-name example.com
```

---

## Quick Reference (Checklist)

1. Install Certbot via snap.  
2. Run `certbot certonly --manual --preferred-challenges dns -d *.example.com -d example.com ...`  
3. Add the `_acme-challenge` TXT record(s) shown by Certbot.  
4. Verify with `dig`, then continue Certbot.  
5. Point Nginx to `fullchain.pem` and `privkey.pem`, reload.  
6. Repeat issuance every ~60–75 days (or switch to DNS plugin for auto-renew).
