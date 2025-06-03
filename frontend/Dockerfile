# Dockerfile (frontend)
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the files
COPY . .

# Expose Vite's default port
EXPOSE 8080
CMD ["npm", "run", "dev"]
