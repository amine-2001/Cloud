# Use Node.js base image
FROM node:18-alpine as build

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Install Vite locally and set permissions for the node_modules binaries
RUN npm install vite && chmod +x ./node_modules/.bin/*

# Copy the rest of the application
COPY . .

# Build the application
RUN ./node_modules/.bin/vite build

# Use NGINX for serving the built app
FROM nginx:alpine

# Copy the build output to the nginx folder
COPY --from=build /app/dist /usr/share/nginx/html
