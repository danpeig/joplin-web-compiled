# Joplin Web – compiled static releases

This repository builds the [Joplin](https://joplinapp.org/) web app with a GitHub Actions workflow and packages the compiled files as a zip. You can extract the zip on any static web server and self-host it.

The build steps are the same ones the official [joplin/web-app](https://github.com/joplin/web-app) repository uses to deploy <https://app.joplincloud.com/>. The difference is that this workflow does not deploy to GitHub Pages. It produces a downloadable release archive instead.

---
## Requirements to host

| Requirement | Notes |
|---|---|
| Any static web server | nginx, Apache, Caddy, Lighttpd, or object storage with static hosting. No server-side runtime is needed. |
| HTTPS | Required for the service worker (offline/PWA support). The only exception is `localhost`. |
| Correct MIME type for `.wasm` | Files must be served as `application/wasm`. |
| A domain root or subdomain | Recommended. The official build is served from a domain root, and serving from a subpath such as `/joplin/` has not been verified. |

## Configure the web server

### nginx

```nginx
server {
    listen 443 ssl http2;
    server_name joplin.example.com;

    ssl_certificate     /etc/letsencrypt/live/joplin.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/joplin.example.com/privkey.pem;

    root  /var/www/joplin;
    index index.html;

    # Only needed if your mime.types does not already include wasm
    types {
        application/wasm wasm;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Always fetch the newest service worker and entry page
    location ~* ^/(index\.html|.*serviceWorker.*\.js|sw\.js)$ {
        add_header Cache-Control "no-cache";
    }
}
```

### Apache

Enable `mod_mime` and `mod_headers`, then add to the virtual host or a `.htaccess` file:

```apache
AddType application/wasm .wasm

<FilesMatch "^(index\.html|.*serviceWorker.*\.js|sw\.js)$">
    Header set Cache-Control "no-cache"
</FilesMatch>
```

### Caddy

Caddy provides HTTPS automatically and already serves `.wasm` with the correct type:

```caddy
joplin.example.com {
    root * /var/www/joplin
    file_server
}
```

### Local (no HTTPS needed)

```bash
cd /var/www/joplin
python3 -m http.server 8080
# open http://localhost:8080
```

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| App stays on a blank or loading screen | Check the browser console. Wrong `.wasm` MIME types and failed asset loads are the most common causes. |
| Works on `localhost` but not when deployed | The site must be served over HTTPS. |
| Old version still shows after an update | The service worker cache is still serving it. Do a hard refresh or close all tabs of the app, and make sure `index.html` and the service worker file are not cached long-term. |
| Very slow start in Firefox | This is a known limitation of Joplin Web in Firefox. Use Chrome or Safari. |

---

## License

The workflow in this repository only automates the build (AGPL. Joplin itself is developed by Laurent Cozic and contributors at [laurent22/joplin](https://github.com/laurent22/joplin) and is distributed under its own license (AGPL-3.0-or-later at the time of writing). Check the license file in the Joplin repository and follow it when redistributing the built files.

This project is not affiliated with or endorsed by the Joplin project.
