FROM node:18-alpine

WORKDIR /app

# Dependency files copy karo
COPY server/package*.json ./

# Production dependencies install karo
RUN npm install --production

# Pura backend ka code copy karo
COPY server/ .

# Backend standard port expose karo
EXPOSE 5000

CMD ["npm", "start"]
