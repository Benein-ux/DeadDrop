# 🕵️ DEAD DROP

> *A noir-styled, local-first personal journal powered by a real SQLite database — running entirely in your browser.*

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat&logo=sqlite&logoColor=white)

---

## What is this?

Dead Drop is a single-file web app that lets you write and organize private journal entries — styled like a classified field operative's notebook. There is no server, no account, no cloud sync. Your data lives in your browser, in a real SQLite database.

The aesthetic takes cues from 1970s spy thrillers: aged paper textures, typewriter fonts, cork board backgrounds, red string decorations, rubber stamps, and a crosshair cursor. It's designed to feel like a prop, not a product.

---

## Features

- **Real SQL database** — powered by [sql.js](https://github.com/sql-js/sql.js), which compiles SQLite to WebAssembly. Full `CREATE`, `SELECT`, `INSERT`, `UPDATE`, `DELETE` with parameterized queries.
- **Persistent storage** — the database serializes to Base64 and is saved to `localStorage` on every write. Your entries survive page refreshes and browser restarts.
- **Tagged entries** — add custom classification tags to entries; filter the archive by tag with one click.
- **Full-text search** — queries both title and body fields simultaneously.
- **Export** — dumps all entries as formatted JSON directly in the UI.
- **Keyboard shortcuts** — `Ctrl+N` for a new entry, `Ctrl+S` to save.
- **Zero dependencies to install** — open the HTML file and it works.

---

## Tech stack

| Layer | What's used |
|-------|-------------|
| Structure | HTML5 semantic markup |
| Styling | Vanilla CSS — custom properties, `repeating-linear-gradient` for paper lines, CSS animations, inline SVG data URI for the cursor |
| Logic | Vanilla JavaScript (ES2020, async/await, no framework) |
| Database | [sql.js 1.10.2](https://github.com/sql-js/sql.js) — SQLite compiled to WebAssembly |
| Fonts | Google Fonts — Special Elite, Courier Prime, Oswald |

Everything except the sql.js library and Google Fonts loads from the single HTML file. No build step, no bundler, no package.json.

---

## Getting started

### Option 1 — just open it

Download `dead-drop.html` and open it in any modern browser. Done.

### Option 2 — serve it locally

If you want to avoid any CORS quirks with the WASM binary:

```bash
# Python
python -m http.server 8080

# Node
npx serve .
```

Then visit `http://localhost:8080/dead-drop.html`.

---

## How the database works

On first load, sql.js initializes a fresh in-memory SQLite database and runs:

```sql
CREATE TABLE IF NOT EXISTS entries (
  id         INTEGER PRIMARY KEY AUTOINCREMENT,
  title      TEXT NOT NULL,
  body       TEXT,
  tags       TEXT,
  created_at TEXT DEFAULT (datetime('now')),
  updated_at TEXT DEFAULT (datetime('now'))
);
```

After every write, the database is exported as a `Uint8Array`, converted to Base64, and stored in `localStorage` under the key `deadDropDB`. On subsequent loads, if that key exists, the database is rehydrated from it — so all your entries are still there.

Tags are stored as a comma-separated string in the `tags` column and split into arrays in JavaScript. Filtering by tag uses a SQL `LIKE` expression:

```sql
SELECT * FROM entries
WHERE (',' || tags || ',') LIKE '%,TAGNAME,%'
```

---

## Project structure

```
dead-drop.html     ← the entire application
README.md          ← you are here
```

That's it.

---

## Limitations

- **localStorage cap** — browsers typically limit `localStorage` to 5–10 MB. If you write a very large number of entries, you may eventually hit this limit. An error toast will appear if saving fails.
- **No encryption** — despite the theme, entries are not actually encrypted. `localStorage` is accessible to any JavaScript on the same origin and to anyone with access to your browser profile. Do not store genuinely sensitive information.
- **No sync** — data is bound to the browser and device it was created on. There is no built-in export-to-file or import mechanism beyond the JSON dump.
- **WASM requires a server or modern browser** — opening via `file://` may work in Chrome but can fail in Firefox due to WASM fetch restrictions. Use a local server if you run into issues.

---

## Possible extensions

Some directions this could go if you wanted to build on it:

- **File export/import** — serialize the sql.js database to a `.sqlite` file for download, and read it back on load.
- **Actual encryption** — use the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) with a passphrase to encrypt the Base64 blob before writing to `localStorage`.
- **Markdown rendering** — pipe the body text through a lightweight parser like [marked](https://github.com/markedjs/marked).
- **Full offline support** — bundle the sql.js WASM and fonts locally, add a Service Worker, and make it a proper PWA.
- **IndexedDB backend** — swap `localStorage` for IndexedDB to lift the storage cap.

---

## License

MIT — do whatever you want with it.

---

*"The best place to hide something is in plain sight."*
