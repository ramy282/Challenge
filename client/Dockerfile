FROM node:18

WORKDIR /app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install --legacy-peer-deps 

COPY . .

RUN npm run build

EXPOSE 3000
CMD ["npm", "run", "start"]
