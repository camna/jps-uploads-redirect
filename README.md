# Uploads Redirect (Virtuozzo / Jelastic JPS Add-on)

AppServer add-on for staging/dev WordPress sites. Missing media under `/wp-content/uploads/` is 301-redirected to your live/production domain; files that exist locally are served as usual.

## What it does

1. Asks for your **live / production domain** (e.g. `www.nigelbeauty.com`)  
2. Inserts these blocks into Nginx `default.conf` as the **first `location`** (after `root`, `index`, `add_header`, `auth_basic`, etc.):

```nginx
# Serve local uploads when present; otherwise 301 to production
location ^~ /wp-content/uploads/ {
    try_files $uri $uri/ @missing_upload;
}

location @missing_upload {
    return 301 https://www.example.com$request_uri;
}
```

3. Validates config with `nginx -t` and reloads Nginx  

## Install (dashboard)

Until this package is in Marketplace, install it via **Import** or Cluster **Marketplace → Add**:

1. Open your Virtuozzo dashboard (or Cluster Admin → Marketplace)  
2. **Import** / **Add** the package  
3. Paste a raw URL to `manifest.jps`, or upload / paste the file contents  
4. Choose the environment and **AppServer** (`cp`) layer  
5. Enter the **Live / production domain**  
6. Click **Install**  

The tile appears under **AppServer : Add-ons** (same place as HTTP Basic Auth, Let's Encrypt, FTP, etc.).

### After install

| Button | Action |
|--------|--------|
| **Configure** | Re-open the form and change the live domain |
| **Disable** | Remove the uploads `location` blocks from Nginx |
| **Uninstall** (tile menu) | Same cleanup as Disable |

## Supported stacks

Nginx-based AppServers where site config lives under `conf.d` / `sites-enabled`:

- `nginxphp` / `nginxphp-dockerized` (typical WordPress AppServer)
- `lemp`
- `nginx` / `nginx-dockerized`
- `nginx-ruby` / `nginxruby`

The script looks for `default.conf` in:

- `/etc/nginx/conf.d/sites-enabled/default.conf`
- `/etc/nginx/conf.d/sites_enabled/default.conf`
- `/etc/nginx/sites-enabled/default.conf`
- `/etc/nginx/conf.d/default.conf`

## Domain input

Enter hostname only, or with protocol — both work:

- `www.nigelbeauty.com`
- `https://www.nigelbeauty.com`

The add-on strips `http(s)://` and any path, then always redirects with `https://`.

## Manual equivalent (what this automates)

Add the `location` blocks above inside the site `server { }` in `default.conf`, then:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

## Notes

- Insertion is marker-based (`UPLOADS_REDIRECT_ADDON_BEGIN` / `_END`), so Configure / reinstall safely replaces the previous block.  
- The add-on expects at least one existing `location` directive in `default.conf` (standard on Virtuozzo WordPress/Nginx templates) so it knows where to insert.  
- LiteSpeed AppServers are not covered; this targets Nginx only.  
- Pair with **HTTP Basic Auth** on staging if you want the site gated while still allowing production media redirects.
