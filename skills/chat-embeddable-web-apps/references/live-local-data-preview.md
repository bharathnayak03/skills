# Live Local Data Preview Pattern

Use this reference when a chat-embeddable preview must update from a local project data source instead of a static HTML snapshot.

## Pattern

1. Keep the browser UI as static files in a preview directory, e.g. `web-preview/index.html`.
2. Add a tiny local backend in the same preview directory, e.g. `web-preview/server.ts` or `server.js`.
3. The backend reads the real local source on each request:
   - SQLite DB via existing project classes/queries.
   - JSON/CSV/local files via filesystem reads.
   - Project CLI/API via a subprocess only if no direct library call exists.
4. Expose API routes such as:
   - `/api/health` — returns `{ ok: true, marker: "..." }`.
   - `/api/summary` or similar domain route — reads fresh data on every request.
5. Serve `index.html` from the same backend so the public tunnel only needs one port.
6. In the browser, fetch APIs with cache-busting/no-cache and auto-refresh on an interval.
7. Verify local HTML + local API, then public tunnel HTML + public API.

## TypeScript/Node Example Shape

```ts
import { createServer, type ServerResponse } from 'node:http';
import { readFile } from 'node:fs/promises';
import { dirname, join, resolve } from 'node:path';
import { fileURLToPath } from 'node:url';
import { ProjectReader } from '../src/project-reader.js';

const host = process.env.HOST ?? '0.0.0.0';
const port = Number(process.env.PORT ?? '8790');
const rootDir = dirname(fileURLToPath(import.meta.url));
const dataPath = resolve(rootDir, '..', 'data', 'app.sqlite');

function sendJson(res: ServerResponse, status: number, body: unknown) {
  res.writeHead(status, {
    'content-type': 'application/json; charset=utf-8',
    'cache-control': 'no-store, no-cache, must-revalidate, proxy-revalidate',
  });
  res.end(JSON.stringify(body, null, 2));
}

const server = createServer(async (req, res) => {
  const url = new URL(req.url ?? '/', `http://${req.headers.host ?? 'localhost'}`);
  if (url.pathname === '/api/health') return sendJson(res, 200, { ok: true, marker: 'live-preview-v1' });
  if (url.pathname === '/api/summary') {
    const reader = new ProjectReader({ dataPath });
    const body = reader.getSummary();
    reader.close?.();
    return sendJson(res, 200, body);
  }
  if (url.pathname === '/' || url.pathname === '/index.html') {
    const html = await readFile(join(rootDir, 'index.html'), 'utf8');
    res.writeHead(200, { 'content-type': 'text/html; charset=utf-8', 'cache-control': 'no-store' });
    return res.end(html);
  }
  sendJson(res, 404, { error: 'Not found' });
});

server.listen(port, host, () => console.log(`Live preview listening at http://${host}:${port}`));
```

When using `tsx` in an ESM TypeScript project, import local TypeScript source with the runtime-compatible `.js` specifier if the project already does that (for example `../src/tracker.js`), rather than `../src/tracker.ts`, unless the project explicitly enables `allowImportingTsExtensions`.

## Browser Fetch Pattern

```js
async function load() {
  const summary = await fetch(`/api/summary?t=${Date.now()}`, { cache: 'no-store' }).then(r => r.json());
  render(summary);
}
load();
setInterval(load, 10000);
```

## Verification Commands

```bash
# Start backend
PORT=8790 HOST=0.0.0.0 npx tsx web-preview/server.ts

# Local verification
python3 - <<'PY'
import urllib.request, json
base='http://127.0.0.1:8790'
for path in ['/', '/api/health', '/api/summary']:
    r=urllib.request.urlopen(base+path, timeout=15)
    text=r.read().decode('utf-8', 'ignore')
    print(path, r.status, len(text), text[:100].replace('\n',' '))
PY

# Public tunnel
npx --yes localtunnel --port 8790
```

If Vitest or another JS test runner fails due to worker/thread limits in a constrained container, rerun with explicit low concurrency (for Vitest: `npx vitest run --maxWorkers=1 --minWorkers=1`). Capture this as a verification retry, not as a permanent claim that tests are broken.
