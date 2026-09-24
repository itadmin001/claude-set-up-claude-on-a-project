# CLAUDE.md

A minimal Express API with an in-memory data store, one route file per resource, and a sample test suite.

## Commands

- `npm install` — install dependencies
- `npm run dev` — start the API with auto-restart on file changes (http://localhost:3000)
- `npm start` — start the API without auto-restart
- `npm test` — run all tests (`node --test`, uses supertest against the exported `app`)
- `node --test tests/users.test.js` — run a single test file
- `npm run lint` — run ESLint (`eslint:recommended`, `no-unused-vars` as a warning)

Server port is configurable via the `PORT` env var (defaults to 3000); copy `.env.example` to `.env` for local overrides.

## Conventions

- Routes are defined with `express.Router()`, one file per resource in `routes/`, mounted in `server.js` (e.g. `app.use("/users", usersRoutes)`). Add a new resource by creating `routes/<resource>.js` and mounting it there.
- All data access goes through `db/store.js` — route handlers never touch the `users` array directly.
- `server.js` exports the Express `app` and only calls `app.listen` when run directly (`require.main === module`), so tests import `app` without opening a real port.
- ESLint's `no-unused-vars` ignores unused `req`/`res`/`next`/`_` parameters (common in Express handlers); other unused variables still warn.

## Architecture

- `server.js` — entry point; wires up JSON body parsing and mounts route modules.
- `routes/` — one file per resource (`users.js`, `health.js`); handlers call into `db/store.js` for data.
- `db/store.js` — in-memory data store standing in for a real database; state resets on every restart.
- `tests/` — `node:test` + `supertest`, one file per resource, importing the exported `app`.
