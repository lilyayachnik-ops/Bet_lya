# Checkpoints

Мы изучили различные способы уменьшения образов, которые будут постоянно встречаться в реальной работе. 
В качестве задания предлагаю ознакомиться на примере репозитория `awesome-compose` с `best-practice` по сборке различных языков программирования.
Склонируем репозиторий на исходную машину:

```bash
git init
git pull https://github.com/docker/awesome-compose
```

Покажем в терминале:

<img width="946" height="422" alt="image" src="https://github.com/user-attachments/assets/ac924041-5c4d-497d-8408-74b027d0464e" />


Рассмотрим `Dockefile` в `django/app` и соберём, образ с тегом `django:latest`:

```bash
# syntax=docker/dockerfile:1.4

FROM --platform=$BUILDPLATFORM python:3.7-alpine AS builder
EXPOSE 8000
WORKDIR /app 
COPY requirements.txt /app
RUN pip3 install -r requirements.txt --no-cache-dir
COPY . /app 
ENTRYPOINT ["python3"] 
CMD ["manage.py", "runserver", "0.0.0.0:8000"]

FROM builder as dev-envs
RUN <<EOF
apk update
apk add git
EOF

RUN <<EOF
addgroup -S docker
adduser -S --shell /bin/bash --ingroup docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /
CMD ["manage.py", "runserver", "0.0.0.0:8000"]
```

Соберём образ:

```bash
docker build -t django:latest .
```

Покажем в терминале:

<img width="1108" height="881" alt="image" src="https://github.com/user-attachments/assets/fb1fd13b-0d35-4ed4-96e0-957b5e1e228d" />
<img width="867" height="487" alt="image" src="https://github.com/user-attachments/assets/aa133abe-aa0b-42cc-9c3e-ca4c030e8144" />


Рассмотрим `Dockefile` в `vuejs/vuejs` и собери, образ с тегом `vuejs:latest`:

```bash
# syntax=docker/dockerfile:1.4
FROM --platform=$BUILDPLATFORM node:14.4.0-alpine AS development

RUN mkdir /project
WORKDIR /project

COPY . .

RUN yarn global add @vue/cli@4.5.0 --ignore-engines 
RUN yarn install
ENV HOST=0.0.0.0
CMD ["yarn", "run", "serve"]

FROM development as dev-envs
RUN <<EOF
apk update
apk add git
EOF

RUN <<EOF
addgroup -S docker
adduser -S --shell /bin/bash --ingroup docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /
CMD ["yarn", "run", "serve"]
```

Соберём образ:

```bash
docker build -t vuejs:latest .
```

Покажем в терминале:

<img width="1608" height="863" alt="image" src="https://github.com/user-attachments/assets/a1820afb-d1eb-4c58-a161-088303e9892d" />
<img width="1580" height="243" alt="image" src="https://github.com/user-attachments/assets/dc017c8f-c0d6-4ea1-85c0-f9865d0c9327" />

Рассмотрим `Dockefile` в `react-java-mysql/backend` и собери, образ с тегом `java:backend`:

```bash
# syntax=docker/dockerfile:1.4

FROM --platform=$BUILDPLATFORM maven:3.8.5-eclipse-temurin-17 AS builder
WORKDIR /workdir/server
COPY pom.xml /workdir/server/pom.xml
RUN mvn dependency:go-offline

COPY src /workdir/server/src
RUN mvn install

FROM builder as dev-envs

RUN <<EOF
apt-get update
apt-get install -y git
EOF

RUN <<EOF
useradd -s /bin/bash -m vscode
groupadd docker
usermod -aG docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /
CMD ["mvn", "spring-boot:run"]

FROM builder as prepare-production
RUN mkdir -p target/dependency
WORKDIR /workdir/server/target/dependency
RUN jar -xf ../*.jar

FROM eclipse-temurin:17-jre-focal

EXPOSE 8080
VOLUME /tmp
ARG DEPENDENCY=/workdir/server/target/dependency
COPY --from=prepare-production ${DEPENDENCY}/BOOT-INF/lib /app/lib
COPY --from=prepare-production ${DEPENDENCY}/META-INF /app/META-INF
COPY --from=prepare-production ${DEPENDENCY}/BOOT-INF/classes /app
ENTRYPOINT ["java","-cp","app:app/lib/*","com.company.project.Application"]
```

Соберём образ:

```bash
docker build -t java:backend .
```

Покажем в терминале:

<img width="1037" height="904" alt="image" src="https://github.com/user-attachments/assets/01e3b3fa-a977-4af2-b81c-60b13fbcefb8" />
<img width="993" height="664" alt="image" src="https://github.com/user-attachments/assets/1d9a1890-f4fe-4bd7-b661-81dd17c5a47e" />

Рассмотрим `Dockefile` в `react-java-mysql/frontend` и собери, образ с тегом `react:frontend`:

```bash
# syntax=docker/dockerfile:1.4

FROM --platform=$BUILDPLATFORM node:lts AS development

WORKDIR /code
COPY package.json /code/package.json
COPY package-lock.json /code/package-lock.json

RUN npm ci
COPY . /code

ENV CI=true
ENV PORT=3000

CMD [ "npm", "start" ]

FROM development AS dev-envs
RUN <<EOF
apt-get update
apt-get install -y git
EOF

RUN <<EOF
useradd -s /bin/bash -m vscode
groupadd docker
usermod -aG docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /
CMD [ "npm", "start" ]

FROM development AS build

RUN ["npm", "run", "build"]

FROM nginx:1.13-alpine

COPY --from=build /code/build /usr/share/nginx/html
```

Соберём образ:

```bash
docker build -t react:frontend .
```

Покажем в терминале:

<img width="1116" height="751" alt="image" src="https://github.com/user-attachments/assets/9d2db029-f474-4aed-8431-33d75f6583b7" />
<img width="820" height="551" alt="image" src="https://github.com/user-attachments/assets/b9382977-1fc7-4c6f-b222-516b33189402" />
