# 09_Dockerizing

> 도커 이미지를 만들고 이미지를 컨테이너화하여 실행 및 배포하는 행동

## 1. FrontEnd Dockerizing

### 올바른 배포 방법

> npm run build를 통해 빌드한 결과물을 nginx 위에서 동작하게 해야 함

`npm run build`를 하게 되면 `dist`나 `build`폴더가 생성되는데, 해당 폴더 안에 있는 `index.html`을 배포해야 한다.

![image](https://user-images.githubusercontent.com/93081720/236376456-ad5d4354-b210-40f3-9fea-83097ee3c203.png)

따라서 올바른 도커 파일 작성 방법은

다음과 같이 멀티스테이지 빌드를 통해 빌드를 진행한 다음 nginx를 통해 프로젝트를 실행하는 방법이 있다.

```dockerfile
FROM node:16.15.0 AS build

WORKDIR /jenkins_home/workspace/deployment/frontend

COPY package*.json ./

RUN ["npm", "install"]

COPY . .

RUN ["npm", "run", "build"]

FROM nginx:alpine

RUN rm /etc/nginx/conf.d/default.conf

RUN mkdir /app

WORKDIR /app

RUN mkdir ./build

COPY --from=build /jenkins_home/workspace/deployment/frontend/build ./build

COPY ./nginx.conf /etc/nginx/conf.d

EXPOSE 3000

CMD ["nginx", "-g", "daemon off;"]
```

또는 이미 빌드한 파일을 바로 nginx에서 실행하는 방법도 있다.

```dockerfile
FROM nginx:stable-alpine

WORKDIR /app

RUN mkdir ./build

ADD ./build ./build

RUN rm /etc/nginx/conf.d/default.conf

COPY ./nginx.conf /etc/nginx/conf.d

EXPOSE 3000

CMD ["nginx", "-g", "daemon off;"]
```

<br>

### 올바르지 않은 배포 방법

> CMD ["npm", "start"]를 통한 프로젝트 실행은 개발자 모드로 실행되기 때문에 프로젝트 올바르지 않음

React 공식 문서에도 나와 있듯이, `npm start`를 통해 실행을 하게 되면 개발자 모드로 실행되기 때문에 배포 단계의 프로젝트에는 적절하지 않다.

- `CMD ["npm", "start"]`를 통해 이미지를 빌드하고 실행하는 도커 파일

```dockerfile
FROM node:16.15.0

COPY package*.json ./

RUN ["npm", "install"]

COPY . .

RUN ["npm", "run", "build"]

EXPOSE 3000

CMD ["npm", "start"] # CMD ["npm", "run", "start"]
```

![image](https://user-images.githubusercontent.com/93081720/236374793-fb25e771-c536-484c-bbb3-bf4d64f6ef71.png)



![image](https://user-images.githubusercontent.com/93081720/236375014-7c5765c5-6cc0-4685-8e02-df6a9f7772b8.png)

#### 문제가 되는 이유

리엑트 공식 문서의 `<StrictMode>`에 대해 살펴보면, 해당 컴포넌트는 개발 단계에서 버그를 찾는데 도움을 준다고 나와 있다.

![image](https://user-images.githubusercontent.com/93081720/236375333-301b5454-88b4-4446-a5a8-75aca76198f9.png)

![image](https://user-images.githubusercontent.com/93081720/236375731-dd1dcfd2-08cf-4796-8e8a-dc28c1da26d8.png)

문제는 `<StrictMode>` 컴포넌트가 `<App>`을 감싸고 있고 개발자 모드로 실행된 경우 버그 fix, side effect 검증을 위해서 `모든 렌더링이 2번씩 발생한다`는 것이다.

![image](https://user-images.githubusercontent.com/93081720/236375807-b2b0a5c8-0805-4d63-853c-277a44c581f2.png)

즉, 프로덕션 단계에서 쓸데 없이 렌더링이 2번 발생한다는 것은 만약에 렌더링할 내용이 많을 경우 렌더링 속도에도 영향을 줄 수 있고, 사이트 속도에 저하를 가져오기 때문에 사용자 경험에서도 좋지 않다.

따라서 `성능 최적화와 올바른 번들링을 위해 반드시 프로덕션 모드로 배포를 진행`해야 한다.