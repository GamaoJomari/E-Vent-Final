# AGENTS.md — E-Vent

Event service booking marketplace. Expo/React Native frontend, Express/MySQL backend. Three roles: user, provider, admin.

## Quick start

```bash
npm install        # frontend deps
cd server && npm install && cd ..
# .env from .env.example — needs DB_HOST, DB_USER, DB_NAME, DB_PASSWORD
npm run server     # Express on :3001
npm start          # Expo on auto-detected LAN IP
```

## Commands

| Purpose | Command |
|---|---|
| Expo dev server (auto LAN) | `npm start` |
| Fast start (no cache clear) | `npm run start:fast` |
| Web | `npm run web` |
| Android | `npm run android` |
| Server | `npm run server` |
| TypeScript check | `npm run type-check` |
| Lint | `npm run lint` / `npm run lint:fix` |
| Format (prettier) | `npm run format` / `npm run format:check` |
| Frontend tests | `npm test` (Jest) |
| Backend tests | `cd server && npm test` |
| E2E (Playwright) | `npm run test:e2e` |
| DB migrations | `cd server && npm run migrate:up` |
| DB seed | `npm run db:seed` |
| EAS build | `npm run build:android` |

Pre-commit hook runs `lint-staged` → lint + format on staged `mvc/` and `app/` files.

## Architecture

- **Expo Router** (file-based routing in `app/`), *not* manual state navigation
- **MVC** lives in `mvc/`: controllers, models, views (role-split: `user/`, `provider/`, `admin/`), services, components
- **Auth**: Firebase Auth (email/password + Google OAuth) via `mvc/services/firebase.ts`; role/profile stored in MySQL, synced via `mvc/contexts/AuthContext.tsx`
- **Backend**: Express on port 3001. Routes in `server/routes/*.js` (14 modules), controllers in `server/controllers/*.js`, middleware in `server/middleware/`. Knex migrations in `server/migrations/`. Socket.io attached on same port.
- **API Base URL** (`mvc/services/api.ts`): env `EXPO_PUBLIC_API_BASE_URL` > Metro host auto-detect > platform defaults (`10.0.2.2:3001` for Android emulator, `localhost:3001` for iOS simulator and web). Physical devices need LAN IP — `scripts/start-expo-with-ip.js` sets it automatically.
- **Payments**: PayMongo (GCash, InstaPay, cards) via `server/services/`
- **API prefix**: `app.use('/api', routes)` — all business endpoints under `/api/`

## Conventions

- TypeScript everywhere in `mvc/` and `app/`; backend is plain JS
- Prettier: single quotes, trailing commas, 100 print width
- ESLint: `@typescript-eslint/recommended`; no `console.log` (warn), no `require` in TS, no unused vars (prefix `_` to ignore)
- Tailwind (NativeWind v4) for styling
- `@/` path alias maps to root — `@/mvc/...` works
- Backend `.env` loaded from root (parent of `server/`) via `dotenv`
- `app/index.tsx` redirects by auth + role: unauthenticated → `/landing` (desktop) or `/login` (mobile); authenticated → `/{role}/dashboard`
- Server tests use `DB_NAME=event_test`, run with `--runInBand`, `forceExit: true`
