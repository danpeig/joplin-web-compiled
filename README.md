# Joplin Web: compiled static releases

This repository builds the [Joplin](https://joplinapp.org/) web app with a GitHub Actions workflow and packages the compiled files as a zip. You can extract the zip on any static web server and self-host it anywhere.

The build steps are the same ones the official [joplin/web-app](https://github.com/joplin/web-app) repository uses to deploy <https://app.joplincloud.com/>. The difference is that this workflow does not deploy to GitHub Pages. It produces a downloadable release archive instead.

## Download the compiled packages
- [Releases page](https://github.com/danpeig/joplin-web-compiled/releases/)

## Requirements to host

| Requirement | Notes |
|---|---|
| Any static web server | nginx, Apache, Caddy, Lighttpd, or object storage with static hosting. No server-side runtime is needed. |
| HTTPS | Required for the service worker (offline/PWA support). The only exception is `localhost`. |
| Correct MIME type for `.wasm` | Files must be served as `application/wasm`. |
| A domain root or subdomain | Recommended. The official build is served from a domain root, and serving from a subpath such as `/joplin/` has not been verified. |

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| App stays on a blank or loading screen | Check the browser console. Wrong `.wasm` MIME types and failed asset loads are the most common causes. |
| Blank page on `localhost` (React Refresh error in console) | Edit `environment.js` and set `window.__DEV__ = false`  |
| Works on `localhost` but not when deployed | The site must be served over HTTPS. |
| Old version still shows after an update | The service worker cache is still serving it. Do a hard refresh or close all tabs of the app, and make sure `index.html` and the service worker file are not cached long-term. |
| Remote sync problems | Joplin Cloud refuses connections from other web apps rather than their own. Others are typically related to CORS, the simplest solution is changing the server configurations to allow connections from a specific host: https://github.com/laurent22/joplin/pull/16760 |
| The package hosted in this repository is too old | The build script is programmed to run once per day. Open an [issue](https://github.com/danpeig/joplin-web-compiled/issues) if it is failing to build the latest versions.|

## License

The workflow in this repository only automates the build (AGPL. Joplin itself is developed by Laurent Cozic and contributors at [laurent22/joplin](https://github.com/laurent22/joplin) and is distributed under its own license (AGPL-3.0-or-later at the time of writing). Check the license file in the Joplin repository and follow it when redistributing the built files.

This project is not affiliated with or endorsed by the Joplin project.
