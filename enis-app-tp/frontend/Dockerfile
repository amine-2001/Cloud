# Stage 1: Build the frontend app
FROM node:18-alpine AS build

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Build the frontend
COPY . .
RUN npm run build

# Stage 2: Set up nginx to serve the built app
FROM nginx:alpine

# Copy the build artifacts from the build stage to nginx
COPY --from=build /app/dist /usr/share/nginx/html
