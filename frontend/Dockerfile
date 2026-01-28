# ---------- Stage 1: Build ----------
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build


# ---------- Stage 2: Runtime ----------
FROM nginx:alpine

# Create non-root user for nginx workers
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Remove default nginx config
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built files
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/dist /usr/share/nginx/html

# Fix permissions (important)
RUN chown -R appuser:appgroup /usr/share/nginx/html \
    /var/cache/nginx \
    /var/run \
    /var/log/nginx

# Expose standard HTTP port
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

