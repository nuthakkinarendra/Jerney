# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app

COPY package.json package-lock.json* ./

RUN npm ci --omit=dev && npm cache clean --force


# ---- Production stage ----
FROM node:20-alpine AS production

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Security: install dumb-init with version pinning
RUN apk add --no-cache dumb-init=1.2.5-r3

WORKDIR /app

# Copy only production dependencies
COPY --from=build /app/node_modules ./node_modules
COPY src/ ./src/

# Security: set file ownership
RUN chown -R appuser:appgroup /app

USER appuser

EXPOSE 5000

# Proper signal handling
ENTRYPOINT ["dumb-init", "--"]

CMD ["node", "src/index.js"]
