# ---------- SHARED DEPS + BUILD STAGE ----------
FROM node:20-alpine AS deps

WORKDIR /app

# Copy package files first (better cache)
COPY package.json package-lock.json ./
COPY apps/web/package.json ./apps/web/package.json
COPY apps/admin/package.json ./apps/admin/package.json
COPY packages/shared/package.json ./packages/shared/package.json
COPY packages/ui/package.json ./packages/ui/package.json

# Install workspace deps at root
RUN npm install --ignore-scripts

# Copy full source
COPY . .

# Build shared packages (used by both apps)
RUN npm run build --workspace=@experiment-monorepo/shared
RUN npm run build --workspace=@experiment-monorepo/ui

# ---------- BUILD WEB APP ----------
FROM deps AS builder

WORKDIR /app
RUN npm run build --workspace=apps-web

# ---------- SERVE WITH NGINX ----------
FROM nginx:alpine AS runner

RUN rm -rf /usr/share/nginx/html/*

COPY --from=builder /app/apps/web/dist/spa /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
