# syntax=docker/dockerfile:1
#
# Multi-stage Dockerfile for Node.js Demo App
# Optimized for small image size and security
#

ARG ARCH=
ARG IMAGE_BASE=20-alpine

# === Builder stage ===
FROM ${ARCH}node:${IMAGE_BASE} AS builder

WORKDIR /app

# Copy package files first (better layer caching)
COPY src/package*.json ./

# Install ALL dependencies (including dev) for building
RUN npm ci --include=dev

# Copy source and build (linting happens here)
COPY src/ ./
RUN npm run lint || true

# Prune to production dependencies only
RUN npm ci --omit=dev --silent

# === Final stage (minimal, secure) ===
FROM ${ARCH}node:${IMAGE_BASE}

LABEL Name="Node.js Demo App" \
      Version="4.9.9" \
      org.opencontainers.image.source="https://github.com/benc-uk/nodejs-demoapp"

ENV NODE_ENV=production
WORKDIR /app

# Create non-root user for security
RUN addgroup -g 1001 appgroup && \
    adduser -u 1001 -G appgroup -D appuser

# Copy only production node_modules from builder
COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules

# Copy application source
COPY --chown=appuser:appgroup src/ ./src

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

ENTRYPOINT ["node", "src/server.mjs"]