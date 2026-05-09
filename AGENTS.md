# Oikos — Agent Quick Reference

> Self-hosted family planner. Node.js 22+, vanilla JS, zero build tools, zero frontend frameworks.

---

## Build / Run / Test

| Command | Purpose |
|---|---|
| `npm install` | Install dependencies |
| `npm run dev` | Start server with `--watch` (auto-restart) |
| `npm start` | Start production server |
| `npm test` | Run **all** test suites sequentially |
| `node --experimental-sqlite test-<module>.js` | Run a **single** test file (preferred during dev) |
| `npm run test:db` | Run DB schema tests |
| `npm run test:tasks` | Run tasks module tests |
| `npm run test:shopping` | Run shopping module tests |
| `npm run test:meals` | Run meals module tests |
| `npm run test:calendar` | Run calendar module tests |
| `npm run test:ncb` | Run notes/contacts/budget tests |
| `npm run test:dashboard` | Run dashboard tests |
| `npm run test:api` | Run API client tests |
| `npm run test:reminders` | Run reminders tests |
| `npm run test:ics-parser` | Run ICS parser tests |
| `npm run test:ics-sub` | Run ICS subscription tests |
| `npm run test:modal-utils` | Run modal utility tests (uses `--loader ./test-browser-loader.mjs`) |
| `npm run test:ux-utils` | Run UX utility tests |
| `npm run test:kitchen-tabs` | Run kitchen tabs tests |
| `npm run test:setup` | Run setup tests |
| `npm run test:backup-scheduler` | Run backup scheduler tests |
| `npm run test:caldav` | Run CalDAV sync tests |
| `npm run test:carddav` | Run CardDAV tests |

> Tests use Node.js built-in `node:sqlite` (in-memory) and import route handlers directly. No running server or DB file required.

---

## Code Style

### General
- **ES modules only** (`import`/`export`, never `require`)
- **Semicolons: yes**
- `try/catch` in every route handler — no unhandled promise rejections
- No dynamic code execution (`eval`, `new Function`, etc.)
- Never write user data directly into HTML strings — use `esc()` from `public/utils/html.js` or DOM APIs (`createElement`, `textContent`)
- Use `insertAdjacentHTML` to append HTML fragments, `replaceChildren()` to replace content
- Direct `innerHTML` writes are blocked by a pre-commit hook

### File Header
Every file starts with a JSDoc block:
```js
/**
 * Modul: <Name>
 * Zweck: <One-line purpose>
 * Abhängigkeiten: <comma-separated list>
 */
```

### Imports
- Group: built-in → external → internal
- Use `node:` prefix for built-ins (`node:fs`, `node:crypto`, `node:sqlite`)

### Naming
- `camelCase` for variables, functions, methods
- `PascalCase` for classes, Web Components
- `UPPER_SNAKE_CASE` for constants and enums
- `kebab-case` for file names and CSS custom properties
- Web Components prefix: `oikos-*` (one component per file)

### Types
- JSDoc types for function signatures and parameters
- No TypeScript compiler — types are for documentation only

### Error Handling
- Backend API responses: `{ data: ... }` on success, `{ error: string, code: number }` on failure
- Log errors via `createLogger('ModuleName')` from `server/logger.js`
- In production: JSON logs; in dev: human-readable prefixed logs

### Database
- Every table: `id INTEGER PRIMARY KEY`, `created_at TEXT`, `updated_at TEXT` (ISO 8601)
- Migrations: **append-only** to the `migrations` array in `server/db.js` — never modify existing entries
- Foreign keys: `PRAGMA foreign_keys = ON`
- Journal mode: `WAL`

### Frontend
- **No frameworks** (no React, Vue, Svelte, etc.)
- **No bundlers** (no Webpack, Vite, Rollup, esbuild)
- **No CSS libraries** (no Tailwind, Bootstrap)
- All UI text via i18n keys (`t('key')`) — never hardcode text. German (`de`) is the reference locale
- Date format: `DD.MM.YYYY` — Time format: `HH:MM` (24h)
- CSS uses design tokens from `public/styles/tokens.css` — never hardcode colors, radii, or shadows
- Pages export a `render()` function, no side effects on import
- Lucide Icons only (self-hosted SVG sprite)

### Backend
- One route file per module in `server/routes/`
- Express router pattern: `const router = express.Router(); export default router;`
- Validation via `server/middleware/validate.js` helpers
- CSRF protection on state-changing endpoints (after `requireAuth`)

### Testing
- One test file per module in project root: `test-<module>.js`
- Custom lightweight test runner (`test(name, fn)` + `assert(cond, msg)`)
- In-memory SQLite via `--experimental-sqlite`
- Import route handlers directly — no HTTP calls

---

## Project Structure

```
server/
  index.js          # Express entry point, middleware chain
  db.js             # SQLite connection + migration runner (append-only)
  auth.js           # Session auth + user management
  logger.js         # Level-based structured logging
  middleware/       # CSRF, validation, etc.
  routes/           # API route handlers — one file per module
  services/         # Business logic (calendar sync, recurrence, etc.)
public/
  index.html        # SPA shell (single entry point)
  router.js         # Client-side History API router
  api.js            # Fetch wrapper (auth, CSRF, error handling)
  i18n.js           # Internationalization module
  sw.js             # Service worker
  styles/
    tokens.css      # Design tokens — all colors, radii, shadows, fonts
  components/       # Reusable Web Components (oikos-* prefix)
  pages/            # Page modules — each exports render()
  utils/            # Shared utilities (html.js, ux.js, etc.)
test-*.js           # One test file per module (project root)
```

---

## Environment

- Node.js >= 22 (required for `--experimental-sqlite` in tests)
- Copy `.env.example` to `.env` and set `SESSION_SECRET`
- Leave `DB_ENCRYPTION_KEY` empty for local dev (no SQLCipher needed)
- `npm run dev` starts with `--watch` for auto-restart

---

## Commit Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>(<scope>): <description>
```

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `style`
Scopes: `tasks`, `shopping`, `meals`, `calendar`, `budget`, `notes`, `contacts`, `auth`, `db`, `ui`, `pwa`

Rules: imperative mood, lowercase, no period, max 72 chars. One logical change per commit.

---

## Service Worker Cache Versioning

The app uses a Service Worker (`public/sw.js`) with precached caches. **When you modify any file listed in the precache arrays, you MUST bump the corresponding cache version** — otherwise browsers will continue serving the stale cached version.

| Cache Constant | Files Covered | Bump When Modifying |
|---|---|---|
| `SHELL_CACHE` | `index.html`, `api.js`, `router.js`, `i18n.js`, `styles/*.css`, etc. | Any shell file |
| `PAGES_CACHE` | `pages/*.js` | Any page module |
| `LOCALES_CACHE` | `locales/*.json` | Any locale file |
| `ASSETS_CACHE` | Images, icons, fonts | Any asset |

Example: if you edit `public/pages/settings.js`, increment `PAGES_CACHE`:
```js
const PAGES_CACHE = 'oikos-pages-v68'; // was v67
```

> The browser only detects a new SW version when `sw.js` itself changes. Bumping the cache constant ensures the install event re-fetches precached assets into a new cache, and the activate event deletes the old one.

---

## Hard Constraints (Non-Negotiable)

- No frontend frameworks or bundlers
- No external frontend dependencies (except Lucide Icons as self-hosted SVG sprite)
- Backend dependencies evaluated case-by-case, must remain minimal
- No dynamic code execution
- No direct `innerHTML` writes with user data
