# CODEBASE.md

## Scope
- **Apparent purpose:** Ledger Suite, an offline-first managerial judgment and operational analysis PWA.
- **Stack/languages/frameworks:** Vanilla HTML, CSS, JavaScript (ES6+). No backend, Node.js, or external frameworks.
- **Entry points:** `index.html` (UI), `app.js` (business logic), `sw.js` (offline caching).
- **Build/run/test systems:** No automated build system; manually served via GitHub Pages. Local testing with python http.server.
- **Architectural style:** Fully local Single Page Application (SPA / PWA) utilizing IndexedDB for persistence and a strictly offline-first architecture.
- **Major operational invariants:**
  - All data resides entirely on the client.
  - No `.innerHTML` allowed (Sentinel Mode security rule).
  - Snapshot imports use robust checksum (fnv1a-32) verification and schema migration.

## Repository Map
```
.
├── .nojekyll
├── 404.html
├── app.js
├── build-info.json
├── index.html
├── llms.txt
├── manifest.webmanifest
├── migration-notes.md
├── quality.json
├── README.md
├── release-checklist.md
├── styles.css
├── sw.js
└── tasks.json
```

## Authoritative Review Summary
- **Core flows:** Case creation -> Evidence/Assumption gathering -> Review/Decision finalization. Includes manual local data snapshot exports (JSON/ZIP) and checksum-validated staged imports.
- **Important interfaces:** IndexedDB (`ledger-suite` v3, schema v2). Export payload `entities` containing strictly validated object arrays.
- **Key configs:** CSP in `index.html`, constants `VERSION = "0.1.46"`, `CACHE_NAME = "ledger-suite-shell-v0.1.46"` in `sw.js` and `app.js`.
- **Major invariants:** `app.js` and `sw.js` version strings must be synchronously updated. Data is always deterministic on export.
- **Principal risks:** IndexedDB persistence is subject to browser storage clearance; explicit manual exports are the only secure backup. Schema updates carry migration risks.

## File Inventory

| Path | Role | Priority | Inclusion | Reason |
| --- | --- | --- | --- | --- |
| `app.js` | Main Application Logic | Critical | Excerpt | Defines IndexedDB structure, UI event wiring, export/import, and migration logic. |
| `sw.js` | Service Worker | Critical | Full | Vital for offline-first invariant and PWA cache-busting. |
| `index.html` | UI Layout / Entry | Important | Excerpt | Contains the primary CSP, DOM structure, and security surface. |
| `styles.css` | Styling | Context | Summary | Glassmorphism UI, layout, and responsive CSS rules. |
| `manifest.webmanifest` | PWA Manifest | Context | Summary | Provides app installation configuration. |
| `README.md` | Documentation | Context | Summary | Usage instructions and architectural summary. |
| `404.html` | Routing Fallback | Context | Summary | Catch-all routing for GitHub Pages to support the SPA. |
| `llms.txt` | AI Context | Context | Summary | Directives for LLMs reading the repository. |
| `tasks.json` | Task Tracking | Context | Excluded | Project management ledger. |
| `quality.json` | Quality Gates | Context | Excluded | Project testing/QA state ledger. |
| `build-info.json` | Build Info | Context | Excluded | Timestamp and target info. |
| `migration-notes.md` | Changelog | Context | Excluded | Versioning history log. |
| `release-checklist.md`| Checklist | Context | Excluded | Development release check steps. |

## Embedded Critical Files

### `sw.js`
- **Role:** Service Worker
- **Why it matters:** Enforces offline-first functionality, caches shell assets, handles PWA skipWaiting logic.
- **Inclusion mode:** Full
```javascript
const CACHE_NAME = "ledger-suite-shell-v0.1.46";
const APP_SHELL = [
  "./",
  "./index.html",
  "./styles.css",
  "./app.js",
  "./manifest.webmanifest",
  "./.resources/Ledger_16.png",
  "./.resources/Ledger_32.png",
  "./.resources/Ledger_48.png",
  "./.resources/Ledger_64.png",
  "./.resources/Ledger_128.png",
  "./.resources/Ledger_256.png"
];

self.addEventListener("install", (event) => {
  event.waitUntil(caches.open(CACHE_NAME).then((cache) => cache.addAll(APP_SHELL)));
});

self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches
      .keys()
      .then((keys) => Promise.all(keys.filter((key) => key !== CACHE_NAME).map((key) => caches.delete(key))))
      .then(() => self.clients.claim())
      .then(() => self.clients.matchAll({ includeUncontrolled: true }))
      .then((clients) => {
        for (const client of clients) {
          client.postMessage({ type: "SW_ACTIVATED", cacheName: CACHE_NAME });
        }
      })
  );
});

self.addEventListener("message", (event) => {
  if (event.data && event.data.type === "SKIP_WAITING") {
    self.skipWaiting();
  }
});

self.addEventListener("fetch", (event) => {
  if (event.request.method !== "GET") {
    return;
  }

  event.respondWith(
    caches.match(event.request).then((cached) => {
      if (cached) {
        return cached;
      }
      return fetch(event.request)
        .then((response) => {
          const copy = response.clone();
          caches.open(CACHE_NAME).then((cache) => cache.put(event.request, copy)).catch(() => {});
          return response;
        })
        .catch(() => caches.match("./index.html"));
    })
  );
});

```

### `app.js`
- **Role:** Main Application Logic
- **Why it matters:** Contains DB models, migration rules, snapshot integrity checksum validation, and file processing logic.
- **Inclusion mode:** Excerpt
- **Covered region:** Constants, DB initialization, migration routines, and startup execution flow. The 1000+ lines of DOM interaction and data mapping are omitted for brevity.
```javascript
// Excerpt of app.js covering constants, DB init, and data bounds
const VERSION = "0.1.46";
const DB_NAME = "ledger-suite";
const DB_VERSION = 3;
const SCHEMA_VERSION = 2;
const SUPPORTED_IMPORT_SCHEMAS = [1, 2];
const MAX_IMPORT_BYTES = 5 * 1024 * 1024;

const STORE_NAMES = [
  "meta", "workspaces", "cases", "evidenceItems", "assumptions", "optionSets", "reviewMatrices", "decisionRecords", "outcomeReviews", "governanceReviews", "packHooks", "recoveryLogs"
];

// ... [UI elements and util functions omitted] ...

function openDatabase() {
  return new Promise((resolve, reject) => {
    const req = indexedDB.open(DB_NAME, DB_VERSION);
    req.onupgradeneeded = () => {
      const db = req.result;
      if (!db.objectStoreNames.contains("meta")) { db.createObjectStore("meta", { keyPath: "key" }); }
      if (!db.objectStoreNames.contains("workspaces")) { db.createObjectStore("workspaces", { keyPath: "id" }); }
      if (!db.objectStoreNames.contains("cases")) {
        const store = db.createObjectStore("cases", { keyPath: "id" });
        store.createIndex("workspaceId", "workspaceId", { unique: false });
      }
      // ... [other stores created similarly] ...
    };
    req.onsuccess = () => resolve(req.result);
    req.onerror = () => reject(req.error);
  });
}

// ... [CRUD ops omitted] ...

async function ensureMetaAndMigrate() {
  const meta = await getRecord("meta", "schema");
  if (!meta) {
    await putRecord("meta", { key: "schema", schemaVersion: SCHEMA_VERSION, appVersion: VERSION, updatedAt: nowIso() });
    return;
  }
  if (Number(meta.schemaVersion) < SCHEMA_VERSION) {
    await putRecord("meta", { ...meta, schemaVersion: SCHEMA_VERSION, appVersion: VERSION, updatedAt: nowIso() });
    await logRecovery("schema-migrated", `Migrated schema from ${meta.schemaVersion} to ${SCHEMA_VERSION}.`);
  }
}

// ... [Snapshot, Zip generation, and Import processing functions omitted] ...

async function boot() {
  try {
    appState.db = await openDatabase();
    await ensureMetaAndMigrate();
    await ensureDefaultData();
    const repaired = await runIntegrityRepair();
    if (repaired > 0) announce(`Integrity auto-repair removed ${repaired} invalid record(s).`);

    wireEvents();
    await refreshSelectorsAndCase();
    await registerServiceWorker();
    announce("Ledger Suite RC candidate is ready for offline decision analysis.");
  } catch (error) {
    await logRecovery("startup-failure", String(error?.message || error));
  }
}

boot();
```

### `index.html`
- **Role:** UI Layout / Entry
- **Why it matters:** Security policy (CSP) definition and functional entry point mapping to `app.js`.
- **Inclusion mode:** Excerpt
- **Covered region:** Meta headers (CSP) and the critical recovery/export HTML structures.
```html
<!-- Excerpt of index.html showing CSP and basic structure -->
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src 'self' data:; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com data:; script-src 'self'; connect-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'" />
    <title>Ledger Suite</title>
    <link rel="manifest" href="manifest.webmanifest" />
    <link rel="stylesheet" href="styles.css" />
  </head>
  <body>
    <!-- ... [Header and Workspace Forms omitted] ... -->
    <main id="main-content">
      <!-- ... [Data Entry Panels omitted] ... -->
      <section class="panel">
        <h2>Step 5: Export / Import / Print</h2>
        <!-- ... [Export/Import Controls omitted] ... -->
      </section>
      <details class="panel expandable-panel">
        <summary><h2>Recovery and Release Checks</h2><span class="chevron"></span></summary>
        <div class="panel-content">
          <div class="actions">
            <button type="button" id="run-repair-btn" class="secondary">Run Integrity Repair</button>
            <button type="button" id="reset-local-btn" class="secondary">Reset Local Data</button>
            <button type="button" id="clear-cache-btn" class="secondary">Clear App Cache</button>
            <button type="button" id="apply-update-btn" disabled>Apply Update</button>
          </div>
        </div>
      </details>
    </main>
    <script src="app.js" defer></script>
  </body>
</html>
```

## Summarized Files

- **`styles.css`:** Defines the responsive layout, generic glassmorphism styling, theme variables (light/dark adaptive), and `touch-action: manipulation` overrides for mobile friendliness. *Omitted for brevity as it contains no behavioral logic.*
- **`manifest.webmanifest`:** Standard PWA configuration declaring app name, icons, and standalone display mode. *Omitted due to boilerplate nature.*
- **`README.md`:** Describes the privacy-first nature, backup best practices, and the 4-step analysis workflow. *Omitted as the logic is fully captured in the embedded `app.js`.*
- **`404.html`:** Identical to `index.html`, used exclusively to prevent 404 errors on GitHub Pages due to SPA URL behaviors. *Omitted to avoid duplication.*
- **`llms.txt`:** Provides context that the app is an offline-first PWA with no backend and specific local models. *Omitted as summary covers its contents.*

## Cross-File Relationships
- **Startup wiring:** `index.html` defers to `app.js`. `app.js` bootstraps IndexedDB, runs integrity repairs, then registers `sw.js`.
- **Module relationships:** Monolithic architecture; `app.js` acts as the sole controller managing state, view, and storage.
- **API/data flow:** No external APIs. Data flows exclusively between memory (`appState`), DOM elements (`textContent`), and IndexedDB. File API is used for Export/Import.
- **Config/env flow:** Version configuration is hardcoded in `app.js` and `sw.js`. `index.html` CSP acts as the security boundary.
- **Test-to-implementation mapping:** Codebase has no automated test runner. Quality and tasks ledgers map manual QA steps.

## Review Hotspots
- **Correctness risks:** Schema v2 migrations (`ensureMetaAndMigrate`) and data integrity repairs (`runIntegrityRepair`) if an unexpected data shape is encountered.
- **Security risks:** Relies heavily on the `index.html` CSP and the invariant of using `textContent`/`createTextNode` instead of `.innerHTML` to prevent XSS.
- **Performance risks:** Serializing all DB data to JSON in `snapshot()` and writing to OPFS can block the main thread for very large datasets (mitigated by `MAX_IMPORT_BYTES`).
- **State/concurrency risks:** Simple transaction model used in IndexedDB operations, but no cross-tab state synchronization exists. Concurrent tab edits may overwrite data.
- **Maintainability smells:** Monolithic `app.js` containing all storage, business, and UI logic (1000+ lines) makes isolated testing difficult.

## Packaging Notes
- **Exclusions:** Project management files (`tasks.json`, `quality.json`, `migration-notes.md`, `release-checklist.md`, `build-info.json`) and image assets (`.resources/`) were excluded as they have no behavioral significance.
- **Compression decisions:** `app.js` and `index.html` were excerpted. Repetitive boilerplate forms, DOM selectors, and specific list-rendering utility functions were summarized out to maintain a dense, high-signal review artifact.
- **Fidelity limits:** The complete logic of the `validateImportShape` and ZIP parsing mechanisms are hidden in the `app.js` excerpt, reducing the downstream reviewer's ability to spot specific parser bugs.
