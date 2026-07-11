# Stage 1: Dependencies
FROM node:22-alpine AS deps

WORKDIR /app

RUN apk add --no-cache openssl openssl-dev
RUN npm install -g pnpm@9

COPY package.json pnpm-lock.yaml ./

RUN pnpm install --frozen-lockfile


# Stage 2: Builder
FROM node:22-alpine AS builder

WORKDIR /app

RUN apk add --no-cache openssl openssl-dev
RUN npm install -g pnpm@9

COPY --from=deps /app/node_modules ./node_modules

COPY . .

ENV NEXT_TELEMETRY_DISABLED=1

RUN pnpm build


# Stage 3: Production runner
FROM node:22-alpine AS runner

WORKDIR /app

RUN apk add --no-cache openssl openssl-dev
RUN npm install -g pnpm@9

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV CONTAINER_RUNTIME=docker

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs && \
    chown -R nextjs:nodejs /app


# Copy standalone build
COPY --from=builder --chown=nextjs:nodejs /app/next.config.js ./
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

COPY --from=builder --chown=nextjs:nodejs /app/package.json ./
COPY --from=builder --chown=nextjs:nodejs /app/pnpm-lock.yaml ./


RUN pnpm install --prod --frozen-lockfile


USER nextjs


# Next.js server configuration
ENV PORT=3000
ENV HOSTNAME=0.0.0.0

EXPOSE 3000


CMD ["node", "server.js"]
