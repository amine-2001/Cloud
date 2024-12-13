# Stage 1: Build the application
FROM node:18-alpine as build

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the application source code
COPY . .

# Ensure proper permissions for binaries
RUN chmod +x ./node_modules/.bin/*

# Build the application
RUN npm run build

# Stage 2: Serve the application using NGINX
FROM nginx:alpine

# Copy the build output from the previous stage
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80

# Start NGINX
CMD ["nginx", "-g", "daemon off;"]
