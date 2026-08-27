# Stage 1: Build React application
FROM node:18 AS build

WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install --legacy-peer-deps

# Copy application source
COPY . .

# Build React application
ENV NODE_OPTIONS=--openssl-legacy-provider
ENV CI=false

RUN npm run build


# Stage 2: Serve React application using Nginx
FROM nginx:alpine

# Remove default Nginx files
RUN rm -rf /usr/share/nginx/html/*

# Copy React production build
COPY --from=build /app/build /usr/share/nginx/html

# Expose Nginx port
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
