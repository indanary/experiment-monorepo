Sure — here is a **simpler, cleaner README**, and I’ve added a short section about **what to do when you change something in `packages/`**.

You can copy–paste this directly into `README.md`.

---

```md
# Experiment Monorepo (Quasar + npm Workspaces)

This is a learning monorepo using **npm workspaces** with:

- Two Quasar apps:
    - `apps-web`
    - `apps-admin`
- Two shared internal packages:
    - `@experiment-monorepo/shared`
    - `@experiment-monorepo/ui`

---

## Structure (high level)
```

experiment-monorepo/ ├── node_modules/ # all dependencies are hoisted here ├── package.json # root workspace config │ ├── apps/ │ ├── web/ # Quasar app │ └── admin/ # Quasar app │ └── packages/ ├── shared/ # shared utilities (TypeScript) └── ui/ # shared UI logic (TypeScript)

````

---

## Getting started

### 1) Install dependencies (from root)

```bash
npm install
````

### 2) Build shared packages

You **must build packages before running the apps**:

```bash
npm run build --workspace=@experiment-monorepo/shared
npm run build --workspace=@experiment-monorepo/ui
```

This generates:

```
packages/shared/dist/index.js
packages/ui/dist/index.js
```

### 3) Run the apps

Web app:

```bash
npm run dev --workspace=apps-web
```

Admin app:

```bash
npm run dev --workspace=apps-admin
```

---

## How apps use shared packages

Both apps import from the shared packages like this:

```ts
import {formatDate} from "@experiment-monorepo/shared"
import {Button} from "@experiment-monorepo/ui"
```

Vite resolves these via aliases in `quasar.config.js`, which point to:

```
packages/shared/dist/index.js
packages/ui/dist/index.js
```

---

## What to do when you change something in `packages/`

If you modify anything inside:

- `packages/shared/src/`
- `packages/ui/src/`

You **must rebuild the packages**, otherwise the apps will still use the old compiled files.

Run again from the root:

```bash
npm run build --workspace=@experiment-monorepo/shared
npm run build --workspace=@experiment-monorepo/ui
```

Then restart the apps:

```bash
npm run dev --workspace=apps-web
npm run dev --workspace=apps-admin
```

---

## Workspace behavior (important)

This repo uses npm workspaces, so there should be **only one `node_modules/` at the root**.

If you ever see `node_modules/` inside `apps/web` or `apps/admin`, clean them up:

```bash
rm -rf apps/web/node_modules apps/admin/node_modules
rm -rf node_modules package-lock.json
npm install
```

---

## Dependency idea (for later tools)

Conceptually, your monorepo looks like this:

```
shared → ui → web
shared → ui → admin
```

This structure will be useful when you later add Turborepo, Nx, or Lerna.

```

```
