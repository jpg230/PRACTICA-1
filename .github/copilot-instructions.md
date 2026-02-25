# ✨ DeliverUS Backend — Copilot Instructions

This repo contains a **Node.js/Express backend** for a food-delivery project. The codebase uses **ES modules** (`"type":"module"` in `package.json`) and is organised around an MVC‑like pattern with Sequelize as the ORM.

## 🧠 Big picture

1. **Entry points**
   * `src/backend.js` – bootstraps the server (called by root script `npm run start:backend`).
   * `src/app.js` – builds the `express` app, loads global middleware, dynamic routes and initializes/disconnects the database.

2. **Configuration**
   * Environment variables stored in `.env` (copy from `.env.example`).  `dotenv` is loaded in `app.js` and `config/*`.
   * `src/config/config.js` returns a Sequelize‑style config object keyed by `NODE_ENV` (`development`, `test`, `production`).
   * `src/config/sequelize.js` contains low‑level `initSequelize`/`disconnectSequelize` used by the app and tests.
   * Import aliases (`#root/` and `#src/`) are defined in `DeliverUS-Backend/package.json`.

3. **Models & database**
   * All model definitions in `src/models/*.js`.  Each file exports a function that receives `(sequelize, DataTypes)` and returns a class extending `Model`.
   * Associations are defined via a static `associate(models)` method.  `models/models.js` imports every model, calls `associate()` and exports them along with the `sequelizeSession`.
   * **Restaurant.js** currently has a TODO to add its properties – look at migrations for the expected fields.
   * Migrations live under `src/database/migrations` (one per entity). Seeders are in `src/database/seeders`.
   * `package.json` script `migrate` runs `node src/database/db-setup.js`, a robust helper that creates the database, undoes old migrations, runs new ones and applies seeders.  You can also use the standard `npx sequelize-cli` commands manually.

4. **Controllers & routes**
   * `src/controllers/*Controller.js` exports plain async functions.  Example: `RestaurantController.index` returns all restaurants with their category.
   * Routes are in `src/routes/*Routes.js`.  `routes/index.js` dynamically imports every `.js` file in the directory and invokes its default export with the Express app.
   * Global middleware (JSON parser, CORS, Helmet) is wired in `middlewares/GlobalMiddlewaresLoader.js`.

5. **Tests**
   * E2E tests under `DeliverUS-Backend/tests/e2e`.  They use Jest + SuperTest.
   * `tests/e2e/utils/testApp.js` starts/shuts the server and disconnects the DB using exports from `src/app.js`.
   * Tests set `NODE_ENV=production` via `cross-env` (see root script `test:backend`).  They assume migrations have run.
   * Utility modules (`auth.js`, `date.js`, etc.) live in the helper folder.

6. **Miscellaneous patterns**
   * ESLint with `eslint-config-standard` – files are auto‑formatted on save if the extension is installed; the template encourages fixing with `npm run lint` if added.
   * The `#src/` import alias is used in tests and other places to avoid long relative paths.
   * `postInitializeDatabase(app)` in `app.js` is a placeholder for future hooks; many files guard against `app.connection` being `undefined`.
   * The `initializeServer` function accepts an `enableConsoleLog` flag; tests call it with the default (silent) behaviour.

## 🚀 Common workflows (run from workspace root)

| Task | Command |
|------|---------|
| Install dependencies | `npm run install:backend` (installs root deps and backend deps) |
| Start dev server | `npm run start:backend` (runs `node --watch ./src/backend.js`) |
| Run migrations + seeders | `npm run migrate:backend` or `cd DeliverUS-Backend && npm run migrate` |
| Reset DB manually | `npx sequelize-cli db:migrate:undo:all` then `npm run migrate:backend` |
| Run backend tests | `npm run test:backend` (invokes migrations then `npm test` inside backend) |
| Debug in VS Code | Use "Debug Backend" configuration; breakpoints in `src/app.js` work. |

> ⚠️ Always populate `.env` before attempting to connect to MariaDB.  The `.env` file is ignored by git.

## 🔗 External dependencies and integration points

* MariaDB is the only external service; the connection parameters come from environment variables.
* Passport is imported but not yet used (future auth flows).  There is an empty `src/controllers/validation` directory reserved for `express-validator` schemas.
* Static assets (avatars, restaurant images) are stored under `public/*` and served via CORS/Helmet settings.

## 🧾 Project‑specific conventions

* `src/backend.js` is the entry point but seldom edited – most logic lives in `src/app.js`.
* Every route module should export a `loadFileRoutes(app)` function that attaches routes to the Express app.
* Models should keep property definitions in sync with corresponding migration files; look at `Product.js` or `User.js` for examples.
* When you add a new model, update `models/models.js` and create a migration + seeder.
* Tests expect the server to listen on whatever port `process.env.APP_PORT` or `3000` provides; they call the route directly via SuperTest.

## 👁️ Review & feedback

Read through the `README.md` for developer onboarding steps – Copilot can leverage that when generating code or tests.

> If any part of this description feels incomplete or outdated, drop a comment or ask for clarification so we can iterate.