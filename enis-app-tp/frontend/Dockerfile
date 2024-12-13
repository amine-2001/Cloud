FROM node:18-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Check vite version (if necessary)
RUN npx vite --version

# Copy source code
COPY . .

# Build the frontend
RUN npm run build

# Set up nginx for production
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
