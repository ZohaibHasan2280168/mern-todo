# Stage 1: Build Stage
FROM node:18-alpine AS builder
WORKDIR /app

ARG VITE_API_BASE_URL=/api
ENV VITE_API_BASE_URL=$VITE_API_BASE_URL

COPY client/package*.json ./
RUN npm install

# --- APINE LINUX ROLLUP FIX ---
RUN npm install @rollup/rollup-linux-x64-musl
# ------------------------------

COPY client/ .
RUN npm run build

# Stage 2: Production Stage (Nginx)
FROM nginx:1.25-alpine
COPY --from=builder /app/dist /usr/share/nginx/html

RUN rm /etc/nginx/conf.d/default.conf
COPY client/default.conf /etc/nginx/conf.d/

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
