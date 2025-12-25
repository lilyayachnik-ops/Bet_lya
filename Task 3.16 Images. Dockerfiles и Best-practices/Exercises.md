# Checkpoints

Создадим репозиторий на `gitlab` с именем `dkr-17-alonetone`:

<img width="910" height="120" alt="image" src="https://github.com/user-attachments/assets/d8dad457-18cc-4a0c-a6a2-e8dea4311363" />

Скопируем в созданный нами репозиторий `dkr-17-alonetone` содержимое из репозитория проекта `alonetone`:

```bash
mkdir -p dkr-17-alonetone
cd ./dkr-17-alonetone
git init
git pull https://github.com/sudara/alonetone.git
```

Покажем в терминале:

<img width="1008" height="314" alt="image" src="https://github.com/user-attachments/assets/32d9974d-e737-424d-8ed4-1e7c4cb22053" />
<img width="1103" height="201" alt="image" src="https://github.com/user-attachments/assets/fc077b18-e929-4631-8905-d050bbd49a42" />

Напиши и добавь в репозиторий `/home/user/dkr-17-alonetone/Dockerfile.dev` — с установкой `development` зависимостей (внутри `Dockerfile` необходимо выставить переменную `RAILS_ENV`): 

```bash
# Use Ubuntu 24.04 as the base image
FROM ubuntu:24.04

# Set environment variables for Rails development environment
ENV RAILS_ENV=development

# Install system dependencies and required packages
RUN apt-get update \
    && apt-get install -y \
    software-properties-common \
    && add-apt-repository -y ppa:chris-needham/ppa \
    && apt-get update \
    && apt-get install -y \
    libvips \
    libvips-dev \
    audiowaveform \
    rustc \
    ruby-full \
    bundler \
    curl \
    default-libmysqlclient-dev \
    pkg-config \
    build-essential \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 18.x and Yarn for JavaScript dependencies
RUN apt-get update \
    && apt-get install -y \
    && curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g yarn \
    && rm -rf /var/lib/apt/lists/*
    
# Set the working directory inside the container
WORKDIR /app

# Copy dependency files first for better caching
COPY package.json yarn.lock Gemfile Gemfile.lock ./

# Install JavaScript dependencies using Yarn
RUN yarn install

# Install Ruby dependencies using Bundler
RUN bundle install

# Copy the rest of the application code
COPY . .

# Start the Rails development server on all network interfaces
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```

Покажем в терминале:

<img width="820" height="167" alt="image" src="https://github.com/user-attachments/assets/9ca15eab-b53e-4786-b8a6-c9dbb6eec834" />
<img width="733" height="432" alt="image" src="https://github.com/user-attachments/assets/6ec2b3a9-bd4a-4dc7-94b0-bed8ef2a5df5" />
<img width="927" height="330" alt="image" src="https://github.com/user-attachments/assets/77f80168-85c1-4605-89ee-c1dea2c4e29f" />
<img width="831" height="271" alt="image" src="https://github.com/user-attachments/assets/00877865-f6bb-4fb2-84da-7208cc94bf26" />

Второй `/home/user/dkr-17-alonetone/Dockerfile.prod` без таковых (изучим флаги `bundler`) (внутри `Dockerfile` необходимо выставить переменную `RAILS_ENV`):

```bash
# Use Ubuntu 24.04 as the base image for production
FROM ubuntu:24.04

# Set environment variables for Rails production environment
ENV RAILS_ENV=production

# Install system dependencies optimized for production
RUN apt-get update \
    && apt-get install -y \
    software-properties-common \
    && add-apt-repository -y ppa:chris-needham/ppa \
    && apt-get update \
    && apt-get install -y \
    libvips \
    audiowaveform \
    ruby-full \
    bundler \
    curl \
    default-libmysqlclient-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 18.x and Yarn package manager
RUN apt-get update \
    && apt-get install -y \
    && curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get update \
    && apt-get install -y nodejs \
    && npm install -g yarn \
    && rm -rf /var/lib/apt/lists/*
    
# Set the working directory inside the container
WORKDIR /app

# Copy dependency definition files first (optimizes Docker layer caching)
COPY package.json yarn.lock Gemfile Gemfile.lock ./

# Install JavaScript dependencies in production mode (no development dependencies)
RUN yarn install --production

# Install Ruby dependencies, excluding development and test gems for production
RUN bundle install --without development test

# Copy the rest of the application code into the container
COPY . .

# Start the Rails production server, binding to all network interfaces
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
```

Покажем в терминале:

<img width="1031" height="166" alt="image" src="https://github.com/user-attachments/assets/8655b29d-b9bf-4bea-ab7d-8f60f325002d" />
<img width="932" height="371" alt="image" src="https://github.com/user-attachments/assets/c34e01a0-6921-4486-85bd-1e3f4bbbf2ae" />
<img width="1045" height="406" alt="image" src="https://github.com/user-attachments/assets/030f7193-3844-405a-8f33-cdb8113ceeaa" />
<img width="1190" height="191" alt="image" src="https://github.com/user-attachments/assets/9d6833f5-8771-42e7-b285-1bf65a37bd7c" />

Собери `2` образа, выставив им теги `alonetone:dev` для образа с `development` зависимостями, и `alonetone:prod` для образа без завистимостей: 

```bash
docker build -f ./Dockerfile.dev -t alonetone:dev .
docker build -f ./Dockerfile.prod -t alonetone:prod .
```

Покажем в терминале:

<img width="1593" height="597" alt="image" src="https://github.com/user-attachments/assets/5fb7b5e9-592e-4c9c-8e66-4165c6c4ede3" />
<img width="1593" height="577" alt="image" src="https://github.com/user-attachments/assets/570428bc-2c64-48f0-999f-c7dcee6cd1b8" />

Выведи список образов:

```bash
docker images
```

Покажем в терминале:

<img width="1607" height="112" alt="image" src="https://github.com/user-attachments/assets/da835ea2-c97f-4911-b96a-5414808e13f5" />

**Подсказка**: отправной точкой для тебя может послужить файл `specs.yml` в репозитории. БД внутри собираемого контейнера запускать не надо. Если есть желание запустить полностью рабочее приложение — запустите БД как отдельный контейнер. В данном файле нас интересуют этапы:

- Установка системных зависимостей
  
- Установка `yarn`-зависимостей
  
- Установка `ruby`-зависимостей при помощи `bundler`
  
- Для сборки используйте `ruby` не ниже `2.6`
