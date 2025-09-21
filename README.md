# References
https://github.com/the-road-to-learn-react/the-road-to-learn-react




Dockerfile
# --- Build stage ---
FROM node:18 AS build
WORKDIR /app

# Install dependencies first (cache-friendly)
COPY package*.json ./
# Use ci if you have a lockfile; fallback to install otherwise
RUN npm ci || npm install

# Copy source and build
COPY . .
RUN npm run build

# --- Runtime stage (Nginx) ---
FROM nginx:alpine

# Static site
COPY --from=build /app/build /usr/share/nginx/html

# Nginx config with a placeholder port 8080 (we'll replace it at start)
COPY default.conf /etc/nginx/conf.d/default.conf

# Cloud Run injects $PORT at runtime (usually 8080)
ENV PORT=8080
EXPOSE 8080

# Replace the listen port with $PORT, then start nginx in foreground
CMD ["sh","-c","sed -i \"s/listen 8080;/listen ${PORT};/\" /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"]


default.conf
------------
server {
  listen 8080;
  server_name _;

  root /usr/share/nginx/html;
  index index.html;

  # React SPA routing: serve index.html for client-side routes
  location / {
    try_files $uri /index.html;
  }

  # Optional compression
  gzip on;
  gzip_types text/plain text/css application/javascript application/json image/svg+xml;
  gzip_min_length 1024;
}


PROJECT_ID="bowtieinc-448310"
gcloud config set project $PROJECT_ID

# Build (from ~/lab/my-app)
gcloud builds submit --tag gcr.io/$PROJECT_ID/my-react-app .

# Deploy
gcloud run deploy my-react-app \
  --image gcr.io/$PROJECT_ID/my-react-app \
  --region us-central1 \
  --allow-unauthenticated
