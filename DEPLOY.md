# Deploying KinderBay to a DigitalOcean droplet (nginx)

Static site — no build step. Deploy = copy files to the web root.

## 1. One-time server setup

```bash
sudo apt update && sudo apt install -y nginx
```

## 2. Upload the site

From your local machine, from this project folder:

```bash
rsync -avz --delete index.html css js assets DEPLOY.md root@YOUR_DROPLET_IP:/var/www/html/
```

(Or `scp -r index.html css js assets root@YOUR_DROPLET_IP:/var/www/html/`.)

## 3. nginx server block

Create `/etc/nginx/sites-available/kinderbay.ae`:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name kinderbay.ae www.kinderbay.ae;

    root /var/www/html;
    index index.html;

    # Long cache for static assets, short for HTML
    location ~* \.(png|jpg|jpeg|webp|svg|ico|woff2?)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    location = /index.html {
        add_header Cache-Control "no-cache";
    }

    gzip on;
    gzip_types text/css application/javascript image/svg+xml;
}
```

Enable it and reload:

```bash
sudo ln -s /etc/nginx/sites-available/kinderbay.ae /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

## 4. HTTPS with certbot

Point the `kinderbay.ae` and `www.kinderbay.ae` DNS A records at the droplet IP first, then:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d kinderbay.ae -d www.kinderbay.ae
```

Certbot rewrites the server block for HTTPS and installs an auto-renew timer. Verify renewal works:

```bash
sudo certbot renew --dry-run
```

## 5. Updating the site later

Re-run the `rsync` command from step 2. That's it.

## Before going live (placeholders to replace)

- WhatsApp number `+971 4 123 4567` (`wa.me/97141234567` links in `index.html`)
- `hello@kinderbay.ae` and `@kinderbay.dubai`
- Pricing amounts (see footnote in the Pricing section)
- `og:image` / canonical URLs assume `https://kinderbay.ae/`
