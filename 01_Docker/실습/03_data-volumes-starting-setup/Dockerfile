FROM node:14

WORKDIR /app

COPY package.json .

# 이미지 내부에 node_modules 생성
RUN npm install 

# 로컬에 있는 모든 것을 복사(node_modules가 있다면 그것까지도 but .dockerignore에 node_modules가 있다면 복사 X)
COPY . .

# PORT 환경 변수를 사용하는데, 기본값은 80임
ENV PORT 80 

EXPOSE $PORT

# EXPOSE 80

# VOLUME [ "/app/feedback" ]

# CMD ["node", "server.js"]
CMD ["npm", "start"]
# npm을 하더라도 nodemon을 사용함

ARG DEFAULT_PORT=80

ENV PORT $DEFAULT_PORT

