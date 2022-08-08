FROM node:14

WORKDIR /app

COPY package.json .

RUN npm install

COPY . .

EXPOSE 80

# VOLUME [ "/app/feedback" ]

# CMD ["node", "server.js"]
CMD ["npm", "start"]
# npm을 하더라도 nodemon을 사용함

