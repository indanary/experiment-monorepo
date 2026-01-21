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

# 🚀 Deployment

## Option A — Docker (recommended)

Each app has its own Dockerfile:

```

apps/web/Dockerfile
apps/admin/Dockerfile

```

### Build and run locally with Docker

From the **root**:

```bash
docker compose build
docker compose up
```

Then open:

- Web app → [http://localhost:9000](http://localhost:9000)
- Admin app → [http://localhost:9001](http://localhost:9001)

These containers:

- install workspace dependencies,
- build `@experiment-monorepo/shared`,
- build `@experiment-monorepo/ui`,
- build the specific Quasar app,
- and serve the built static files with **Nginx**.

---

## Option B — CI/CD (GitHub Actions)

This repo includes a GitHub Actions pipeline:

```
.github/workflows/deploy.yml
```

On every push to **main**, the pipeline:

1. Installs all workspace dependencies
2. Builds:
    - `@experiment-monorepo/shared`
    - `@experiment-monorepo/ui`

3. Builds each app (`apps-web`, `apps-admin`)
4. Builds a Docker image for each app
5. Pushes images to **GitHub Container Registry (GHCR)**:

```
ghcr.io/<your-username>/experiment-monorepo/web:latest
ghcr.io/<your-username>/experiment-monorepo/admin:latest
```

You can view them here after your first successful run:

[https://github.com/](https://github.com/)<your-username>?tab=packages

---

## Deploying the Docker images (examples)

### VPS / Server (e.g. DigitalOcean, EC2, etc.)

```bash
# Web
docker pull ghcr.io/<your-username>/experiment-monorepo/web:latest
docker run -d -p 80:80 ghcr.io/<your-username>/experiment-monorepo/web:latest

# Admin (different port or subdomain)
docker pull ghcr.io/<your-username>/experiment-monorepo/admin:latest
docker run -d -p 8080:80 ghcr.io/<your-username>/experiment-monorepo/admin:latest
```

### Render / Railway / Fly.io

Point your service to:

- Image: `ghcr.io/<your-username>/experiment-monorepo/web:latest`
- Port: `80`

---

```md
# Turborepo in this repository

## How `npx turbo build` and `npx turbo dev` know what to do

Turborepo does **not invent tasks by itself** — it runs your existing npm scripts across all workspaces in a smart way.

### 1) Your existing npm scripts (what to run)

Each workspace already defines its own scripts in `package.json`, for example:
```

packages/shared/package.json → "build": "tsc"

packages/ui/package.json → "build": "tsc"

apps/web/package.json → "build": "quasar build", "dev": "quasar dev"

apps/admin/package.json → "build": "quasar build", "dev": "quasar dev"

```

Because of this:

- `npx turbo build` → runs the **`build` script in every workspace**
- `npx turbo dev` → runs the **`dev` script in every workspace**

Rule of thumb:
> Turborepo can only run tasks that already exist in your `package.json` files.

---

### 2) npm Workspaces (where to run tasks)

Your root `package.json` defines:

```

"workspaces": ["apps/*", "packages/*"]

````

This tells Turborepo which projects exist:

- `@experiment-monorepo/shared`
- `@experiment-monorepo/ui`
- `apps-web`
- `apps-admin`

Turborepo automatically discovers these and knows it can run tasks inside them.

---

### 3) `turbo.json` (how to run tasks)

While npm scripts define **what** to run and workspaces define **where**,
`turbo.json` defines **in what order, in parallel, and with caching**.

In one sentence:

> **Your npm scripts define *what* to run, npm workspaces define *where*, and `turbo.json` defines *how* Turborepo runs them.**

---

# Understanding `turbo.json`

This file is the “brain” of Turborepo — it controls task ordering, parallelism, and caching.

### Current configuration

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".quasar/**", "build/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
````

### What each part means

#### `$schema`

Helps editors (like VS Code) understand the file and provide autocomplete and validation.

---

## `tasks.build`

### `dependsOn: ["^build"]`

This means:

> “Before building this package, build all of its dependencies first.”

Because your dependencies look like this:

```
shared → ui → web
shared → ui → admin
```

Turborepo automatically runs tasks in this order:

1. `@experiment-monorepo/shared` → build
2. `@experiment-monorepo/ui` → build
3. `apps-web` and `apps-admin` → build (in parallel)

You did **not** manually specify this order — Turborepo inferred it from `package.json` dependencies.

---

### `outputs: ["dist/**", ".quasar/**", "build/**"]`

These folders are treated as **important build artifacts**.

Turborepo caches them so that if nothing changes, future runs of:

```
npx turbo build
```

will be mostly instant.

---

## `tasks.dev`

```json
"dev": {
  "cache": false,
  "persistent": true
}
```

This means:

- `cache: false` → do not cache dev servers
- `persistent: true` → this is a long-running process (like `quasar dev`)

This is what enables:

```
npx turbo dev
```

to:

- start both `apps-web` and `apps-admin` at the same time,
- keep them running,
- and stream their logs together in one terminal.

---

### In short

- `build` → smart, ordered, cached builds
- `dev` → parallel, long-running development servers

```

---
```
