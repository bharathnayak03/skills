---
name: chat-embeddable-web-apps
description: Use when the user asks to create any small HTML/web app or interactive browser artifact, run it from the local container, expose it publicly, and embed or preview it in chat/Discord/Telegram.
version: 1.1.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [html, web-app, interactive, preview, embed, localtunnel, discord, telegram]
    related_skills: [visual-design-artifacts]
---

# Chat-Embeddable Web Apps

## Overview

Use this skill when the user wants any small browser artifact — static HTML, a prototype, form, calculator, visual demo, interactive widget, status page, chart, game, landing-page mock, or generated preview — and asks to “embed it here”, “run a web server”, “share the app in chat”, or “make it accessible from Discord/Telegram”.

The key lesson: **localhost is not accessible from the user’s chat client**. A local server must either be exposed through a public tunnel, deployed to a public host, or accompanied by a media preview/screenshot attachment. For quick one-off previews inside a Hermes session, a local static server plus a public tunnel is usually the fastest path.

## When to Use

- User asks for a small HTML page, prototype, or web app and wants to see/open it from chat.
- User asks to run a local web server and embed/share the link.
- User says they cannot access localhost from Discord/Telegram/WhatsApp.
- User wants a public preview URL for an artifact produced in the container.
- User wants a chat-friendly preview image plus a clickable public URL.
- User wants a one-off app without full production deployment.

Don’t use this skill for production deployments with persistence, authentication, custom domains, CI/CD, or long-term hosting. For those, use a normal deployment workflow such as Vercel, Netlify, Fly.io, Render, or the project’s established hosting process, and verify the deployed URL.

## Artifact Types This Skill Covers

| User asks for | Good deliverable |
|---|---|
| “Make a small HTML page” | Self-contained `index.html` with inline CSS and minimal JS |
| “Make a mini app / demo” | Static app folder with `index.html`, optional `app.js`, `style.css`, assets |
| “Make it interactive” | Browser-side JavaScript; avoid backend unless required |
| “Show project data” | HTML generated from real CLI/API/file data, plus refresh instructions |
| “Embed it here” | Public tunnel URL + optional preview screenshot/media attachment |
| “I need to keep using it later” | Persistent deployment instead of temporary tunnel |

## Quick Workflow

1. **Define the artifact boundary**
   - If the user’s request is clear, proceed without asking.
   - Choose a simple default: static HTML/CSS/JS in an artifact folder.
   - If data is needed, gather it from real sources first: project CLI, API, local files, user-provided content, or browser-accessible endpoints. Do not invent data.

2. **Generate the artifact**
   - Prefer a self-contained `index.html` with inline CSS/JS for simple previews.
   - For slightly larger demos, use an app folder:

   ```text
   /absolute/project-or-temp-path/web-preview/
     index.html
     style.css       # optional
     app.js          # optional
     assets/         # optional
   ```

   - Include a unique marker string such as the page title or `data-preview-marker="..."` so verification can check the right page loaded.
   - If the artifact is data-driven, include generation metadata or a “last updated” label where useful.

3. **Start a local static server**
   - Use `terminal(background=true)` for long-lived servers.
   - Bind to `0.0.0.0` when possible.
   - Prefer `--directory` over relying on shell working-directory persistence.

   ```bash
   python3 -m http.server 8787 --bind 0.0.0.0 --directory /absolute/path/to/web-preview
   ```

   If the app needs a dev server instead of a static server, run the project’s dev command in background, bind to `0.0.0.0`, and use the printed/listening port.

4. **Verify locally from inside the container**

   ```bash
   python3 - <<'PY'
   import urllib.request
   url = 'http://127.0.0.1:8787/'
   text = urllib.request.urlopen(url, timeout=15).read().decode('utf-8', 'ignore')
   print('bytes', len(text))
   print(text[:120])
   print('expected marker?', 'Expected page title or marker' in text)
   PY
   ```

5. **Expose the server publicly**
   - First check for existing tunnel tools:

   ```bash
   command -v cloudflared || command -v ngrok || command -v localtunnel || command -v lt || true
   ```

   - If none are installed, `npx --yes localtunnel --port <port>` works well for one-off public links. Run it as a background process and wait briefly for the URL:

   ```bash
   npx --yes localtunnel --port 8787
   ```

   In Hermes tools:

   - Start via `terminal(background=true, notify_on_complete=true)`.
   - Then `process(action='wait', timeout=5)` or `process(action='poll')`.
   - Extract the line `your url is: https://...loca.lt`.

6. **Verify the public URL**

   ```bash
   python3 - <<'PY'
   import urllib.request
   url = 'https://example.loca.lt'
   response = urllib.request.urlopen(url, timeout=20)
   text = response.read(1000).decode('utf-8', 'ignore')
   print('status', response.status)
   print(text[:200])
   print('expected marker?', 'Expected page title or marker' in text)
   PY
   ```

   Also verify with a real browser snapshot when possible, not only `urllib`/`curl`. Some tunnel providers, especially localtunnel, can return the app to programmatic requests while showing a human-facing interstitial/password page in the browser. If the browser sees an interstitial, switch tunnel providers (for example `npx --yes cloudflared tunnel --url http://127.0.0.1:<port>`) and re-verify both terminal and browser access before sharing.

7. **Optionally create a preview image**
   - A media preview is useful because Discord/Telegram unfurling can vary and public tunnels are temporary.
   - Use browser screenshot tools when browser access can reach the URL.
   - If the browser stack cannot reach the container’s localhost, use a public tunnel URL or generate a static preview image separately.
   - Attach media in the final reply with:

   ```text
   MEDIA:/absolute/path/to/preview.png
   ```

8. **Return a chat-friendly result**
   - Include the public URL plainly so Discord/Telegram can unfurl it.
   - Include a one-line description of what is running.
   - Mention whether the URL is temporary.
   - Attach a preview image if available.

## Generic Generation Recipe

Use this recipe for any one-off static web artifact:

1. Choose an output path:

```bash
mkdir -p /tmp/chat-web-preview
```

2. Write the HTML/CSS/JS. Include a marker:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Preview App</title>
</head>
<body data-preview-marker="preview-app-v1">
  <main>
    <h1>Preview App</h1>
  </main>
</body>
</html>
```

3. Serve the folder:

```bash
python3 -m http.server 8787 --bind 0.0.0.0 --directory /tmp/chat-web-preview
```

4. Verify local HTML contains `preview-app-v1`.

5. Start a public tunnel:

```bash
npx --yes localtunnel --port 8787
```

6. Verify the tunnel URL contains `preview-app-v1`.

7. Reply with the tunnel URL and optional `MEDIA:` preview.

## Dynamic App Notes

For interactive apps, prefer client-side JavaScript unless a backend is necessary. If a backend is needed:

- Bind the backend server to `0.0.0.0`.
- Expose the backend’s actual port through the tunnel.
- Add a `/health` endpoint or a static marker route when practical.
- Verify both the HTML and any required API routes.
- Do not expose secrets in frontend code or public tunnel URLs.

For project-specific dev servers:

- Use the project’s documented start command.
- Pass host/port flags explicitly, e.g. `--host 0.0.0.0 --port 8787` where supported.
- If the dev server prints a different port, tunnel that port.
- Check the logs with `process(action='poll')` before sharing the URL.

## Live Local Data Previews

When the user wants the preview to update from a local data source in real time, do **not** keep serving a static snapshot. Turn the preview into a tiny single-port app:

- Keep `index.html` as the browser UI.
- Add a small backend (`server.ts`, `server.js`, etc.) that serves the HTML and exposes API routes.
- Read the real data source on every API request: SQLite DB, JSON/CSV files, existing project classes, or the project API.
- Add `cache-control: no-store` on HTML/API responses and use browser `fetch(..., { cache: 'no-store' })` plus a cache-busting query string.
- Add a manual Refresh button and a short auto-refresh interval when the user expects ongoing updates.
- Verify both HTML and API routes locally, then verify both through the public tunnel.

See `references/live-local-data-preview.md` for a reusable TypeScript/Node pattern, browser fetch loop, and verification commands.

## Important Notes for Discord/Telegram/Chat

- **Do not tell the user to open `localhost` or `127.0.0.1`**. That address points to their device, not the Hermes container.
- Discord may unfurl a public URL automatically, but Hermes cannot force an iframe-style embed in a Discord message.
- The safest “embed here” response is:
  1. public URL,
  2. short description,
  3. attached preview image if available.
- Public tunnel URLs are temporary. If the user comes back later, recreate the server/tunnel or deploy to a persistent host.

## Server Process Management

- Use `terminal(background=true)` for servers/tunnels.
- Use `notify_on_complete=false` only for genuine long-lived servers/tunnels.
- Use `process(action='poll')` to confirm a server is still running.
- Kill stale processes only when you started them or the user approves; otherwise choose another port.
- Prefer changing ports if killing is risky.

If `python3 -m http.server` serves a directory listing instead of `index.html`, the server is pointed at the wrong directory. Restart it with the explicit `--directory /absolute/path/to/web-preview` flag.

## Public Tunnel Troubleshooting

- `npx localtunnel` can take a few seconds before printing the URL; wait/poll the process output.
- Some localtunnel URLs may show an interstitial in browsers or require a client IP password. If that happens, try another tunnel provider (`cloudflared tunnel --url http://127.0.0.1:<port>` if installed) or deploy the static files.
- If `Address already in use`, inspect/poll known process sessions first. If it is a stale server you started, kill it and restart. Otherwise choose another port.
- Always verify the public URL from the container with `urllib.request`/`curl` before claiming it works.
- For localtunnel specifically, terminal verification can be misleading: the app may load for `urllib` while a browser still sees the “Tunnel website ahead” page asking for the tunnel host IP. Treat browser interstitials as a failed share link; use `cloudflared tunnel --url http://127.0.0.1:<port>` or another provider and verify again.
- If the browser tool cannot access `127.0.0.1`, verify via terminal and use the public URL for browser checks.

## Verification Checklist

- [ ] Artifact files exist at the expected absolute path.
- [ ] Local server returns HTTP 200 and contains an expected marker string.
- [ ] Public tunnel URL was captured from tunnel process output.
- [ ] Public URL returns HTTP 200 and contains the expected marker string.
- [ ] Interactive routes/API calls needed by the app were verified.
- [ ] Final response includes the public URL, notes that it is temporary unless deployed, and attaches a preview image if created.
- [ ] No fabricated data: content came from user instructions, real project data, or clearly labeled placeholders.
