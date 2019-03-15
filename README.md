# flask 和 react 前后端分离的微服务搭建

<a name="9dadb97f"></a>
## 项目初始目录结构
.<br />├── docker-compose-dev.yml<br />└── services<br />   ├── client<br />    │   ├── Dockerfile-dev<br />   ├── nginx<br />    │   ├── Dockerfile-dev<br />    │   └── dev.conf<br />   └── users
<a name="7707ec61"></a>
## 初始项目代码
<a name="docker-compose-dev.yml"></a>
### docker-compose-dev.yml

```yaml
version: '3.7'

services:

  client:
    build:
      context: ./services/client
      dockerfile: Dockerfile-dev
    volumes:
      - './services/client:/usr/src/app'
      - '/usr/src/app/node_modules'
    ports:
      - 6000:3000
    environment:
      - NODE_ENV=development

  nginx:
    build:
      context: ./services/nginx
      dockerfile: Dockerfile-dev
    restart: always
    ports:
      - 80:80
    depends_on:
      - client
```

<a name="0688e493"></a>
### services/client/Dockerfile-dev

```dockerfile
# base image
FROM node:11.6.0-alpine

# set working directory
WORKDIR /usr/src/app

# add `/usr/src/app/node_modules/.bin` to $PATH
ENV PATH /usr/src/app/node_modules/.bin:$PATH

# install and cache app dependencies
COPY package.json /usr/src/app/package.json
RUN npm install 
RUN npm install react-scripts@2.1.5 -g 

# start app
CMD ["npm", "start"]
```

<a name="54f3d525"></a>
### services/nginx/Dockerfile-dev

```dockerfile
FROM nginx:1.15.8-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY /dev.conf /etc/nginx/conf.d
```

<a name="fb431270"></a>
### services/nginx/dev.conf

```nginx
server {

  listen 80;

  location / {
    proxy_pass http://client:3000;
    proxy_http_version 1.1;
    proxy_redirect    default;
    proxy_set_header  Upgrade $http_upgrade;
    proxy_set_header  Connection "upgrade";
    proxy_set_header  Host $host;
    proxy_set_header  X-Real-IP $remote_addr;
    proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Host $server_name;
  }
}
```


<a name="fc0b5de9"></a>
## 初始项目运行

<a name="4c763bb6"></a>
#### 运行

```shell
docker-compose -f docker-compose-dev.yml up -d --build
```

<a name="0ea78e42"></a>
#### 查看日志

```shell
docker-compose -f docker-compose-dev.yml logs -f
```


