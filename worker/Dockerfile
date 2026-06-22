# ============================================
# STAGE 1: Build stage
# ============================================
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --omit=dev

# ============================================
# STAGE 2: Runtime stage
# ============================================
FROM node:20-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/
COPY package.json ./
USER appuser

# Worker NEMA EXPOSE jer ne prima vanjski promet.
# On samo čita poruke iz Redis queuea i piše u PostgreSQL.
CMD ["node", "src/worker.js"]