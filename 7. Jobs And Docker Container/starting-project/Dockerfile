FROM node:16

ENV MONGODB_CONNECTION_PROTOCOL mongodb+srv
ENV MONGODB_DB_NAME gha-demo1
ENV MONGODB_CLUSTER_ADDRESS cluster0.13dgjsx.mongodb.net
ENV MONGODB_USERNAME ziancrlswong_db_user
ENV MONGODB_PASSWORD y2lnSbjneXtwuMdt

WORKDIR /app

COPY package*.json .

RUN npm install

COPY . .

CMD ["npm", "start"]