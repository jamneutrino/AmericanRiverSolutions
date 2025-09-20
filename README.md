# American River Solutions Website

This repository contains the static marketing site for American River Solutions (`index.html`). The site is designed for self-hosting on your VPS with Cloudflare providing TLS, CDN, and reverse-proxy protection.

## Deployment on the VPS

1. **Install Nginx** (or your preferred web server) if it is not already available:
   ```bash
   sudo apt update && sudo apt install nginx
   ```
2. **Create a web root** for the site, e.g. `/var/www/americanriversolutions`:
   ```bash
   sudo mkdir -p /var/www/americanriversolutions
   sudo chown $USER:$USER /var/www/americanriversolutions
   ```
3. **Copy the site files** into that directory:
   ```bash
   cp /home/jmflu/apps/AmericanRiverSolutions/index.html /var/www/americanriversolutions/
   ```
   Add future pages as additional HTML/CSS/JS assets in the same folder or subdirectories (e.g. `/var/www/americanriversolutions/immigration/index.html`).
4. **Configure Nginx** to serve the site. Create `/etc/nginx/sites-available/americanriversolutions` with the following contents:
   ```nginx
   server {
       listen 80;
       listen [::]:80;
       server_name americanriversolutions.com www.americanriversolutions.com;

       root /var/www/americanriversolutions;
       index index.html;

       location / {
           try_files $uri $uri/ =404;
       }
   }
   ```
   Enable the site and reload Nginx:
   ```bash
   sudo ln -s /etc/nginx/sites-available/americanriversolutions /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## Cloudflare Reverse Proxy Setup

1. In the Cloudflare dashboard, open the `americanriversolutions.com` zone.
2. Under **DNS**, create an `A` record for `@` (root) pointing to your VPS's public IP. Enable the orange-cloud proxy.
3. Optional: add a `CNAME` record for `www` pointing to `americanriversolutions.com`, also proxied.
4. After DNS propagates, Cloudflare issues an SSL certificate automatically. All traffic to `https://americanriversolutions.com` will reach Cloudflare first and then be forwarded to your VPS over HTTP.

### Enforcing HTTPS

Once the proxy is active, go to **SSL/TLS → Edge Certificates** and enable **Always Use HTTPS** and **Automatic HTTPS Rewrites** so visitors are redirected to HTTPS.

### Future Subapps

To expose applications at paths such as `/immigration`, place the assets or reverse-proxy configuration under matching locations in your Nginx config. Because Cloudflare is already fronting the domain, no additional Workers/Pages setup is required—just redeploy updated files to the VPS and reload Nginx.

## Local Testing

Before copying files to `/var/www/americanriversolutions`, you can preview the site by opening `index.html` directly in a browser or by serving the directory with any static file server, e.g. `python3 -m http.server`.
