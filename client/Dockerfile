# Use Node.js as the base image
FROM node:18

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install --legacy-peer-deps 

# Copy the rest of the application files
COPY . .

# Build the application for production
RUN npm run build

# Expose the necessary port and start the Nuxt server
EXPOSE 3000
CMD ["npm", "run", "start"]
