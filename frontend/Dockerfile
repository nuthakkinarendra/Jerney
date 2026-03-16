# ---- Build stage ----
FROM node:20-alpine AS build
WORKDIR /app

# Copy only package files first (cache npm install)
COPY package.json package-lock.json* ./

# Install dependencies (fallback to npm install if lock missing)
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi \
    && npm cache clean --force

# Copy source code
COPY . .

# Build frontend
RUN npm run build

# ---- Production stage ----
FROM nginx:1.27-alpine AS production

# Remove default nginx config
RUN rm -rf /etc/nginx/conf.d/default.conf /usr/share/nginx/html/*

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built frontend
COPY --from=build /app/dist /usr/share/nginx/html

# Run nginx as non-root
RUN chown -R nginx:nginx /usr/share/nginx/html \
    /var/cache/nginx /var/log/nginx \
    && touch /var/run/nginx.pid \
    && chown -R nginx:nginx /var/run/nginx.pid

USER nginx

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
