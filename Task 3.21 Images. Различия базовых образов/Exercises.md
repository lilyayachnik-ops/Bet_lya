# Checkpoints

Склонируем содержимое из официального репозитория `zookeeper`:

```bash
git clone https://github.com/apache/zookeeper
```

Покажем в терминале:

<img width="1032" height="180" alt="image" src="https://github.com/user-attachments/assets/14d11be1-0bc3-4ecd-929d-51ed2bf7230e" />

Создадим в репозитории `/home/user/zookeeper/Dockerfile.alpine`, где в качестве базового образа используется версия `openjdk`, в котором собирается приложение в `JAR`-файл (Используем образ `openjdk:14-ea-8-jdk-alpine3.10` вместо `openjdk:8-jdk-alpine`, так как этот устарел). Для сборки будет необходим `mvn`, нам необходимо добавить его в контейнер (попробуем установить его не менеджером пакетов, а скачав через `curl` и распаковав его). Команда сборки приложения присутствует в `README`, нам необходимо найти флаг, который позволит пропустить этап тестирования:

```bash
FROM openjdk:14-ea-8-jdk-alpine3.10

ARG MAVEN_VERSION=3.8.9
ARG USER_HOME_DIR="/root"

# Install Maven  through curl
RUN apk add --no-cache \ 
    curl \
    tar \
    bash \
    && mkdir -p /usr/share/maven \
    && curl -fsSL https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz -o /tmp/maven.tar.gz \
    && tar -xzf /tmp/maven.tar.gz -C /usr/share/maven --strip-components=1 \
    && rm /tmp/maven.tar.gz \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn  
ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

# Speed up Maven JVM
ENV MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1"

# Copying and assembling
WORKDIR /app
COPY . /app
RUN mvn clean install -DskipTests

# Launching the JAR
CMD ["java", "-jar", "/app/zookeeper-server/target/zookeeper-server-*.jar"]
```

Покажем в терминале:

<img width="1456" height="695" alt="image" src="https://github.com/user-attachments/assets/4149710e-8f8a-4540-bee7-953783861260" />

Соберём этот образ c тегом `zookeeper-alpine:latest`:

```bash
docker build -f Dockerfile.alpine -t zookeeper-alpine:latest .
```

Покажем в терминале:

<img width="1549" height="603" alt="image" src="https://github.com/user-attachments/assets/942d9864-9040-4a88-af5e-3b3af0c3670e" />

Создадим в репозитории `/home/user/zookeeper/Dockerfile.bookworm`, где в качестве базового образа используется версия `openjdk`, в котором собирается приложение в `JAR`-файл (Используем образ `openjdk:22-jdk-bookworm` вместо `openjdk:8-jdk-buster`, так как этот устарел). Для сборки будет необходим `mvn`, нам необходимо добавить его в контейнер (попробуем установить его не менеджером пакетов, а скачав через `curl` и распаковав его). Команда сборки приложения присутствует в `README`, нам необходимо найти флаг, который позволит пропустить этап тестирования:

```bash
FROM openjdk:22-jdk-bookworm

ARG MAVEN_VERSION=3.8.9
ARG USER_HOME_DIR="/root"

# Install Maven  through curl
RUN apt-get update \ 
    && apt-get install --no-install-recommends -y \ 
    curl \
    tar \
    bash \
    && rm -rf /var/lib/apt/lists/* \
    && mkdir -p /usr/share/maven \
    && curl -fsSL https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz -o /tmp/maven.tar.gz \
    && tar -xzf /tmp/maven.tar.gz -C /usr/share/maven --strip-components=1 \
    && rm /tmp/maven.tar.gz \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn  
ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

# Speed up Maven JVM
ENV MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1"

# Copying and assembling
WORKDIR /app
COPY . /app
RUN mvn clean install -DskipTests

# Launching the JAR
CMD ["java", "-jar", "/app/zookeeper-server/target/zookeeper-server-*.jar"]
```

Покажем в терминале:

<img width="1464" height="736" alt="image" src="https://github.com/user-attachments/assets/b381cf69-f5b9-44b8-a644-bfef16e421c6" />

Соберём этот образ c тегом `zookeeper-bookworm:latest`:

```bash
docker build -f Dockerfile.bookworm -t zookeeper-bookworm:latest .
```

Покажем в терминале:

<img width="1600" height="621" alt="image" src="https://github.com/user-attachments/assets/655304da-bced-4d53-9585-0366ecea2a6b" />

Выведим список образов и сравним итоговые размеры. Изучим `docker history` каждого из образов, обратим внимание на размеры слоёв:

```bash
docker images
docker history zookeeper-alpine:latest
docker history zookeeper-bookworm:latest
```

Покажем в терминале:

<img width="1046" height="530" alt="image" src="https://github.com/user-attachments/assets/0f2f18f2-026b-4593-a9b4-e38d46c31082" />
<img width="1348" height="464" alt="image" src="https://github.com/user-attachments/assets/42b44354-6003-4b46-9ca2-63ee6eed4026" />
<img width="1159" height="509" alt="image" src="https://github.com/user-attachments/assets/197a03b9-f950-49e2-bb39-d63f4378ab15" />



**Опционально**: Объединим команду скачки/распаковки `mvn`, команду сборки приложения и команду удаления `mvn` (после сборки приложения менеджер пакетов не нужен в образе) в одну директиву `RUN`. Соберём образ заново и оценим новые размеры образов и слоёв:

1) Содержимое `Dockerfile.alpine`:

```bash
FROM openjdk:14-ea-8-jdk-alpine3.10

ARG MAVEN_VERSION=3.8.9
ARG USER_HOME_DIR="/root"

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

# Speed up Maven JVM
ENV MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1"

WORKDIR /app

# Copying and assembling
COPY . /app


# Install Maven  through curl
RUN apk add --no-cache \ 
    curl \
    tar \
    bash \
    && mkdir -p /usr/share/maven \
    && curl -fsSL https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz -o /tmp/maven.tar.gz \
    && tar -xzf /tmp/maven.tar.gz -C /usr/share/maven --strip-components=1 \
    && rm /tmp/maven.tar.gz \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
    && mvn clean install -DskipTests \
    && rm -rf /usr/share/maven \
    && rm -rf ~/.m2/repository/* \
    && apk del \
       curl \
       tar \
       bash \
    && rm -rf /var/cache/apk/* 

# Launching the JAR
CMD ["java", "-jar", "/app/zookeeper-server/target/zookeeper-server-*.jar"]
```

Соберём его:

```bash
docker build -f Dockerfile.alpine -t zookeeper-alpine:latest .
```

Покажем в термминале:

<img width="1244" height="600" alt="image" src="https://github.com/user-attachments/assets/a28d4579-42bb-46ed-9b78-a82c9430cd15" />

2) Содержимое `Dockerfile.bookworm`:

```bash
FROM openjdk:22-jdk-bookworm

ARG MAVEN_VERSION=3.8.9
ARG USER_HOME_DIR="/root"

ENV MAVEN_HOME /usr/share/maven
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"

# Speed up Maven JVM
ENV MAVEN_OPTS="-XX:+TieredCompilation -XX:TieredStopAtLevel=1"

WORKDIR /app

# Copying and assembling
COPY . /app

# Single RUN directive: Install Maven, build application, and cleanup
RUN apt-get update \ 
    && apt-get install --no-install-recommends -y \ 
    curl \
    bash \
    && mkdir -p /usr/share/maven \
    && curl -fsSL https://dlcdn.apache.org/maven/maven-3/${MAVEN_VERSION}/binaries/apache-maven-${MAVEN_VERSION}-bin.tar.gz -o /tmp/maven.tar.gz \
    && tar -xzf /tmp/maven.tar.gz -C /usr/share/maven --strip-components=1 \
    && rm /tmp/maven.tar.gz \
    && ln -s /usr/share/maven/bin/mvn /usr/bin/mvn \
    && mvn clean install -DskipTests \
    && rm -rf /usr/share/maven \
    && rm -rf ~/.m2/repository/* \
    && apt-get purge -y --auto-remove  \
       curl \
    && rm -rf /var/lib/apt/lists/* 

# Launching the JAR
CMD ["java", "-jar", "/app/zookeeper-server/target/zookeeper-server-*.jar"]
```

Соберём его:

```bash
docker build -f Dockerfile.bookworm -t zookeeper-bookworm:latest .
```

Покажем в термминале:

<img width="1407" height="601" alt="image" src="https://github.com/user-attachments/assets/3ce54dbe-2311-4ef6-b572-0aab848f2ae2" />

Посмотрим слои и оценим размеры:

```bash
docker images
docker history zookeeper-alpine:latest
docker history zookeeper-bookworm:latest
```

Покажем в терминале:

<img width="1089" height="531" alt="image" src="https://github.com/user-attachments/assets/738f3576-e1d1-41bd-b6f8-1e1f32d838a9" />
<img width="1346" height="442" alt="image" src="https://github.com/user-attachments/assets/064c9e16-cc8c-4935-9208-d3fc018bbb34" />
<img width="1402" height="492" alt="image" src="https://github.com/user-attachments/assets/3241598c-58e2-4263-9876-27c533c01332" />

Размер значительно уменьшился.

