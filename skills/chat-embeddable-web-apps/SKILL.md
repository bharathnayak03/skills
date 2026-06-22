---
name: chat-embeddable-web-apps
description: Publish produced HTML as a temporary chat-openable UI by serving it locally, exposing it with Cloudflare Tunnel, and returning a verified URL.
version: 2.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [html, web-app, ui, preview, cloudflared, cloudflare-tunnel, discord, telegram]
    related_skills: [visual-design-artifacts]
---

# Chat-Embeddable Web Apps

## Purpose

Use this skill when an agent has produced an HTML UI and the user needs to open it from a chat app such as Discord, Telegram, or WhatsApp.

The job of this skill is narrow and generic:

1. take the produced HTML,
2. write it to an artifact folder,
3. serve it from a local HTTP server,
4. expose that server through Cloudflare Tunnel using `cloudflared`,
5. verify the public URL,
6. return the URL to the user.

Do not make this skill about the kind of content inside the HTML. Treat the HTML as an opaque browser artifact unless the user asks for changes to the UI itself.

## When to use

Use this skill when:

- the agent has generated HTML and the user wants to open it in chat,
- the user asks to publish, preview, embed, or share a generated UI,
- a chat client cannot access `localhost`,
- the expected result is a temporary public URL for a browser artifact.

Do not use this as a production deployment workflow. Cloudflare quick tunnels are temporary and should be treated as preview links.

## Core rule

Never give the user a `localhost` or `127.0.0.1` URL. That address points to the user's device, not the Hermes runtime. Always publish through Cloudflare Tunnel and return the public `https://...trycloudflare.com` URL.

## Required shape

Use a folder like this:

```text
/path/to/artifact/
  index.html
```

The produced HTML should be saved as `index.html`. If the generated UI has supporting files, keep them next to it:

```text
/path/to/artifact/
  index.html
  style.css
  app.js
  assets/
```

For simple one-off UIs, prefer a single self-contained `index.html` with inline CSS and JavaScript.

## Workflow

### 1. Write the produced HTML

Create a dedicated artifact directory and write the HTML there.

Recommended temporary path:

```bash
mkdir -p /tmp/chat-ui-preview
```

Save the produced HTML as:

```text
/tmp/chat-ui-preview/index.html
```

Include a unique marker in the HTML so verification can prove the correct UI loaded:

```html
<body data-preview-marker="chat-ui-preview-v1">
```

Use a marker that matches the artifact, for example `medicine-tracker-preview-v1` or `invoice-ui-preview-v1`.

### 2. Serve the HTML locally

Start a local HTTP server from the artifact folder. Bind to `0.0.0.0`.

```bash
python3 -m http.server 8787 --bind 0.0.0.0 --directory /tmp/chat-ui-preview
```

In Hermes, start this as a background process because the server must keep running:

```text
terminal(command="python3 -m http.server 8787 --bind 0.0.0.0 --directory /tmp/chat-ui-preview", background=true)
```

If port `8787` is busy, choose another port instead of killing unknown processes.

### 3. Verify the local server

Before tunneling, verify that the local server returns the produced HTML and contains the marker.

```bash
python3 - <<'PY'
import urllib.request
url = 'http://127.0.0.1:8787/'
text = urllib.request.urlopen(url, timeout=15).read().decode('utf-8', 'ignore')
print('bytes', len(text))
print('marker', 'chat-ui-preview-v1' in text)
print(text[:160].replace('\n', ' '))
PY
```

Do not proceed if the marker check fails. Fix the artifact path, server directory, or HTML first.

### 4. Publish with Cloudflare Tunnel

Prefer an installed `cloudflared` binary:

```bash
cloudflared tunnel --url http://127.0.0.1:8787
```

If `cloudflared` is not installed, use the npm package for a one-off tunnel:

```bash
npx --yes cloudflared tunnel --url http://127.0.0.1:8787
```

In Hermes, run the tunnel as a background process and watch or poll its output:

```text
terminal(command="cloudflared tunnel --url http://127.0.0.1:8787", background=true)
```

or:

```text
terminal(command="npx --yes cloudflared tunnel --url http://127.0.0.1:8787", background=true)
```

Extract the public URL from the tunnel logs. It normally looks like:

```text
https://something.trycloudflare.com
```

### 5. Verify the public URL

Verify the Cloudflare URL before giving it to the user.

```bash
python3 - <<'PY'
import urllib.request
url = 'https://something.trycloudflare.com'
text = urllib.request.urlopen(url, timeout=20).read().decode('utf-8', 'ignore')
print('bytes', len(text))
print('marker', 'chat-ui-preview-v1' in text)
print(text[:160].replace('\n', ' '))
PY
```

If browser tools are available, also open the public URL in the browser and check that it renders the intended UI. Terminal verification proves the route works; browser verification catches visual or tunnel issues.

### 6. Return the result

Reply with:

- the Cloudflare Tunnel URL,
- a note that the URL is temporary,
- optional screenshot/media preview if useful,
- a short reminder that the server/tunnel process must keep running while the user uses the UI.

Example final response:

```text
Published the UI here:
https://example.trycloudflare.com

This is a temporary Cloudflare Tunnel URL. It will work while the local server and tunnel process are running.
```

## Process management

- Use `terminal(background=true)` for the local server and Cloudflare tunnel.
- Use `process(action="poll")` to read tunnel output and confirm both processes are still running.
- Use `process(action="kill")` only for processes you started or when the user asks you to stop the preview.
- If a port is busy, prefer another port such as `8788`, `8789`, or `8790`.

## Cloudflared troubleshooting

### `cloudflared` not found

Use:

```bash
npx --yes cloudflared tunnel --url http://127.0.0.1:<port>
```

If that fails because Node/npm is missing or network access is blocked, report the blocker and ask whether to install `cloudflared` or use another publishing method.

### Tunnel starts but no URL appears

Poll the process logs for a few seconds. Cloudflared can take time to print the `trycloudflare.com` URL. If it exits, read the full log and fix the reported issue.

### Public URL loads the wrong thing

Check the local server directory. The most common mistake is serving the parent folder instead of the folder containing `index.html`.

Restart the server with an explicit directory:

```bash
python3 -m http.server <port> --bind 0.0.0.0 --directory /path/to/artifact
```

### Marker check fails

Do not share the link. The marker check is the guardrail that prevents sending the user a stale or wrong UI.

Fix one of these:

- wrong artifact folder,
- old server still running on the port,
- `index.html` missing or not updated,
- tunnel pointing at the wrong port.

## Verification checklist

Before finalizing, confirm:

- [ ] produced HTML is saved as `index.html` in a known artifact folder,
- [ ] HTML contains a unique marker,
- [ ] local server returns HTTP 200 and contains the marker,
- [ ] Cloudflare Tunnel printed a `trycloudflare.com` URL,
- [ ] public URL returns HTTP 200 and contains the marker,
- [ ] final response includes the public URL and says it is temporary.
