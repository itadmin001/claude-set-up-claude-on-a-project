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

- Add a new resource as an `express.Router()` file in `routes/`, mounted in `server.js` (e.g. `app.use("/users", usersRoutes)`) — not as inline routes added directly to `server.js`.
- Read and write data through `db/store.js`'s exported functions, not by importing or mutating the `users` array directly.
- Guard `app.listen` behind `require.main === module`, not called unconditionally at module load, so `tests/` can `require("../server")` without binding a real port.
- Leave unused `req`/`res`/`next`/`_` parameters as-is, not renamed or removed to satisfy lint — ESLint already exempts those names; only rename or remove other unused variables.

## Architecture

- `server.js` — entry point; wires up JSON body parsing and mounts route modules.
- `routes/` — one file per resource (`users.js`, `health.js`); handlers call into `db/store.js` for data.
- `db/store.js` — in-memory data store standing in for a real database; state resets on every restart.
- `tests/` — `node:test` + `supertest`, one file per resource, importing the exported `app`.
