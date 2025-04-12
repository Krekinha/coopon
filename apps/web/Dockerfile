FROM node:20-alpine AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

FROM base AS builder

RUN apk update
RUN apk add --no-cache libc6-compat

WORKDIR /app

# Instala o turbo globalmente antes de usar
RUN pnpm add --global turbo
WORKDIR /app
COPY . .
RUN turbo prune web --docker

FROM base AS installer
RUN apk update
RUN apk add --no-cache libc6-compat

# Instala o turbo novamente no estágio installer
RUN pnpm add --global turbo

WORKDIR /app

# Primeiro instale as dependências (elas mudam menos frequentemente)
COPY --from=builder /app/out/json/ .

RUN pnpm install

# Build the project
COPY --from=builder /app/out/full/ .

RUN turbo build

FROM base AS runner
WORKDIR /app

# Don't run production as root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

# Automatically leverage output traces to reduce image size
# https://nextjs.org/docs/advanced-features/output-file-tracing
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

# Adicione este comando para verificar a estrutura de arquivos
USER root
RUN ls -la /

USER nextjs

EXPOSE 3020

ENV PORT 3020

# Garanta que o Next.js escute em todas as interfaces
ENV HOSTNAME "0.0.0.0"

# Ajuste o caminho para o server.js
CMD ["sh", "-c", "ls -la && node server.js"]
