# Stage 1: Build the frontend
FROM node:26 AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN npm run build

# Stage 2: DHI Alpine runtime (minimal, secure, ~15MB)
FROM dhi.io/nginx:1.31-alpine3.24

WORKDIR /usr/share/nginx/html

# Copy only built dist files - replace default nginx html
COPY --from=builder /app/dist .

EXPOSE 80

# DHI Alpine nginx runs as non-root by default
