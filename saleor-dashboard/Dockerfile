# Step 1: Specify the Builder Stage
FROM node:18-alpine AS builder

# Add bash to the image
RUN apk add --no-cache bash git

# Step 2: Set Up the Working Directory
WORKDIR /app

# Step 3: Manage Dependencies
COPY package.json ./
COPY package-lock.json ./
ARG CI=1
RUN npm ci --legacy-peer-deps

# Step 4: Copy Application Files
COPY nginx/ ./nginx/
COPY assets/ ./assets/
COPY locale/ ./locale/
COPY scripts/ ./scripts/
COPY src/ ./src/
COPY vite.config.js ./
COPY tsconfig.json ./
COPY *.d.ts ./
COPY schema.graphql ./
COPY introspection.json ./
COPY introspection*.json ./
COPY .featureFlags/ ./.featureFlags/

# Step 5: Configure Environment Variables
ARG API_URL
ENV API_URL=${API_URL:-http://localhost:8000/graphql/}
ARG APP_MOUNT_URI
ENV APP_MOUNT_URI=${APP_MOUNT_URI:-/dashboard/}
ARG APPS_MARKETPLACE_API_URL
ENV APPS_MARKETPLACE_API_URL=${APPS_MARKETPLACE_API_URL:-https://apps.saleor.io/api/v2/saleor-apps}
ARG APPS_TUNNEL_URL_KEYWORDS
ENV APPS_TUNNEL_URL_KEYWORDS=${APPS_TUNNEL_URL_KEYWORDS}
ARG STATIC_URL
ENV STATIC_URL=${STATIC_URL:-/dashboard/}
ARG SKIP_SOURCEMAPS
ENV SKIP_SOURCEMAPS=${SKIP_SOURCEMAPS:-true}
ARG LOCALE_CODE
ENV LOCALE_CODE=${LOCALE_CODE:-EN}

# Step 6: Build the Application
RUN npm run build

# Step 7: Specify the Runner Stage
FROM nginx:stable-alpine AS runner

# Step 8: Configure the Runner Stage
WORKDIR /app
COPY --from=builder /app/build/ /app/
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf
COPY ./nginx/replace-env-vars.sh /docker-entrypoint.d/50-replace-env-vars.sh

# Expose port
EXPOSE 80

# Start Nginx server
CMD ["nginx", "-g", "daemon off;"]
