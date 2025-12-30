# Checkpoints

Склонируем репозиторий `https://gitlab.com/andrew1test-group/counter`:

```bash
git clone https://gitlab.com/andrew1test-group/counter
```

Напишим для него `Dockerfile` с планом сборки, указанным в описании как универсальная структура:

- Определение базового образа
- Определение переменных окружений — опционально
- Установка системных зависимостей — через пакетные менеджеры `ОС`, `curl` и прочие утилиты
- Определение рабочей директории — опционально
- Копирование списка зависимостей кода из репозиториев
- Установка зависимостей кода
- Копирование исходного кода
- Сборка исходного кода — если требуется
- Определение директивы запуска приложения

В роли базового образа используем `python:3.9-slim-bullseye`, так как `python:3.7-buster` уже не поддерживает системные зависимости.

Напишем следующий `Dockerfile`:

```bash
FROM python:3.9-slim-bullseye
RUN apt-get update \ 
    && apt-get install -y --no-install-recommends \
    gcc \ 
    python3-dev \
    && pip install --no-cache-dir pipenv \
    && rm -rf /var/lib/apt/lists/* 
WORKDIR /app
COPY Pipfile ./
RUN pipenv lock && pipenv install --system --deploy
COPY server.py settings.py storage.py ./
CMD ["python", "server.py"]
```

Покажем в терминале: 

<img width="574" height="289" alt="image" src="https://github.com/user-attachments/assets/d83b740d-2a24-484c-a976-352aaa55080b" />

Также не забудем поменять в файле `Pipfile` версию `python`:

<img width="597" height="317" alt="image" src="https://github.com/user-attachments/assets/6c29459c-4fc2-4460-b40c-962915a0a181" />

Соберём образ с тегом `normal`, замерив время, чтобы определить, сколько времени займет сборка:

```bash
time docker build -t counter-app:normal .
```

Покажем в терминале: 

<img width="866" height="621" alt="image" src="https://github.com/user-attachments/assets/e92257a8-fd22-48b5-9809-545f83fb7ea8" />

Внесём в исходный код приложения любое изменение, не препятствующее его работоспособности. Выведем строку `Hello World! Counter API is starting ...`. Сделаем это в файле `server.py`:

```bash
import json
import settings

from klein import Klein
from storage import Storage

app = Klein()
storage = Storage(settings.REDIS_HOST, settings.REDIS_PORT, settings.REDIS_PWD)

# Our line
print("Hello World! Counter API is starting...")

@app.route('/')
def counter(request):
    """
    Base endpoint to get current counter value
    :param request:
    :return: JSON representation of response
    """
    return json.dumps({"counter": int(storage.read())})


@app.route('/increment', methods=['POST'])
def increment(request):
    """
    Endpoint to increment the counter
    :param request:
    :return: JSON representation of response
    """
    return json.dumps({"counter": int(storage.incr())})


@app.route('/increment', methods=['DELETE'])
def reset(request):
    """
    Endpoint to reset the counter
    :param request:
    :return: JSON representation of response
    """
    return json.dumps({"counter": int(storage.reset())})


if __name__ == "__main__":
    app.run(settings.SERVER_HOST, settings.SERVER_PORT)
```

Пересоберём образ с замером времени, чтобы определить, сколько времени займет сборка с учетом кэширования:

```bash
time docker build -t counter-app:normalv2.0 .
```

Покажем в терминале:

<img width="980" height="643" alt="image" src="https://github.com/user-attachments/assets/c15d5db2-2e46-4292-8d52-dd05eb78ed18" />

Создадим копию `Dockerfile` с именем `Dockerfile.mini` и внесём в него изменения, очищающие образ от всего, что может увеличивать его объем (после сборки можно удалить `pipfile*` и сам менеджер `pip`, но нужно делать это в одной директиве, иначе файлы останутся в слоях образа и итоговый объем не уменьшится):

```bash
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    gcc \
    python3-dev \
    && pip install --no-cache-dir pipenv \
    && pipenv lock \
    && pipenv install --system --deploy \
    && pip uninstall -y pipenv \
    && apt-get purge -y gcc python3-dev \ # apg-get purge удаляет пакет с его конфигурационными файлами в отличие от команды apt-get remove, которая удаляет только пакет
    && apt-get autoremove -y \ # apt-get autoremove удаляет те пакеты, которые были установлены как зависимости для других пакетов и больше не нужны
    && rm -rf /var/lib/apt/lists/* \
    && rm -f Pipfile Pipfile.lock 
COPY server.py settings.py storage.py ./
CMD ["python", "server.py"]
```

Соберём образ:

```bash
time docker build -f Dockerfile.mini -t couter-app:mini .
```

Покажем в терминале:

<img width="1604" height="619" alt="image" src="https://github.com/user-attachments/assets/6962dae7-531f-44fc-92ba-eb03552788f6" />

Соберём новый образ с тегом `squash`. При сборке используй ключ `--squash` для уменьшения объема образа и с замером времени, чтобы определить, сколько времени займет сборка (для включения `squash` понадобится включение эксперементальных функций). 
Поскольку опция `--squash` не поддерживает в моей версии `Docker`, установим его через `pip`:

```bash
pip install docker-squash
```

Покажем в терминале:

<img width="1513" height="688" alt="image" src="https://github.com/user-attachments/assets/39e7fca4-03fe-435a-baf4-613934a03dab" />

Дальше сожмем наш образ `counter-app:mini`,  используя `docker-squash`

```bash
# docker-squash — утилита для сжатия Docker-images
# -t counter-app:squash, где -t указывает имя нового образа после сжатия
# counter-app:mini — имя исходного образа, который нужно сжать

$ time docker-squash -t counter-app:squash counter-app:mini

docker history counter-app:squash
docker history counter-app:mini
```

Покажем в терминале:

<img width="1602" height="863" alt="image" src="https://github.com/user-attachments/assets/f6031b58-ed06-486e-8181-ce2d88418107" />
<img width="1380" height="441" alt="image" src="https://github.com/user-attachments/assets/6c7a53f6-9ca3-41df-aedc-70cb2f957dae" />

Внесём в исходный код приложения любое изменение, не препятствующее его работоспособности, например, `print(Hello, Ilya!)`:

```bash
import json
import settings

from klein import Klein
from storage import Storage

app = Klein()
storage = Storage(settings.REDIS_HOST, settings.REDIS_PORT, settings.REDIS_PWD)

# Our line
print("Hello World! Counter API is starting...")
# Out new line
print("Hello, Ilya!")

@app.route('/')
def counter(request):
    """
    Base endpoint to get current counter value
    :param request:
    :return: JSON representation of response
    """
    return json.dumps({"counter": int(storage.read())})


@app.route('/increment', methods=['POST'])
def increment(request):
    """
    Endpoint to increment the counter
    :param request:
    :return: JSON representation of response
    """
    return json.dumps({"counter": int(storage.incr())})


@app.route('/increment', methods=['DELETE'])
def reset(request):
    """
    Endpoint to reset the counter
    :param request:
    :return: JSON representation of response
    """
    return json.dumps({"counter": int(storage.reset())})


if __name__ == "__main__":
    app.run(settings.SERVER_HOST, settings.SERVER_PORT)
```

Пересоберём образ с замером времени, чтобы определить, сколько времени займет сборка с учетом кэширования:

```bash
time docker build -f Dockerfile.mini -t counter-app:mini .
time docker-squash -t counter-app:squash counter-app:mini
```

Покажем в терминале:

<img width="1605" height="593" alt="image" src="https://github.com/user-attachments/assets/5da49ebd-6bc4-4246-ba42-564975398303" />
<img width="1595" height="867" alt="image" src="https://github.com/user-attachments/assets/6d5dff29-5706-443d-95e8-1b1b72460f3f" />
