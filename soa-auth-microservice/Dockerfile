FROM node:12.10-slim

WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

EXPOSE 8081

CMD ["npm", "start"]
