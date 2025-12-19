# Checkpoints

Используем `dockerfile`, приведённый ниже:

```bash
FROM alpine:latest
ARG MYARG
RUN apk update && apk add build-base
```

Покажем в терминале:

<img width="597" height="84" alt="image" src="https://github.com/user-attachments/assets/e58b3028-3f15-4273-8460-f6516595ebec" />

Выполним сборку образа, с именем `cache:1`:

```bash
time docker build -t cache:1 .
```

Покажем в терминале:

<img width="1081" height="489" alt="image" src="https://github.com/user-attachments/assets/28922f78-1ce3-4046-a9c7-97bab0c8c0b8" />

Повторим команду, но для имени образа используем `cache:2`:

```bash
time docker build -t cache:2 .
```

Покажем  в терминале:

<img width="1607" height="486" alt="image" src="https://github.com/user-attachments/assets/88f412f6-8727-48c1-93a2-b29495e236f1" />

Повторим команду, с использованием флага `--no-cache`:

```bash
time docker build --no-cache -t cache:2 .
```

Покажем в терминале:

<img width="1249" height="510" alt="image" src="https://github.com/user-attachments/assets/a521c17e-3be8-47c9-aba6-909d97e4d916" />

Сравним скорость выполнения обеих команд, их выводы и `ID` получившихся слоёв: скорось увеличилась без использования кеша, их `ID` тоже:

<img width="905" height="109" alt="image" src="https://github.com/user-attachments/assets/0f25bc9b-bfda-440a-97f7-57585017dd25" />

Соберём образ с именем `cache:3`. Используем дополнительный флаг `--build-arg` и установим значение аргумента `MYARG=3` (не используем директиву `--no-cache`):

```bash
time docker build --build-arg MYARG=3 -t cache:3 .
```

Покажем в терминале:

<img width="1628" height="491" alt="image" src="https://github.com/user-attachments/assets/762d7e5f-4755-4a2a-84ae-cda9aed61027" />

Повторим команду, убедимся, что наша сборка кэширована. Покажем в терминале:

<img width="1622" height="488" alt="image" src="https://github.com/user-attachments/assets/5685b661-4aa4-4e31-99d0-074fbe9dfb97" />

Соберём образ с именем `cache:4`. Используем дополнительный флаг `--build-arg` и установим значение аргумента `MYARG=4` (не используем директиву `--no-cache`):

```bash
time docker build --build-arg MYARG=4 -t cache:4 .
```

Покажем в терминале:

<img width="1618" height="484" alt="image" src="https://github.com/user-attachments/assets/47734cd4-a31f-49cb-b997-3da11ea225e4" />

Используя команду `docker history cache:4`, найдём свой аргумент. Покажем в терминале:

<img width="1180" height="138" alt="image" src="https://github.com/user-attachments/assets/4c83f2e5-507b-4144-a9c0-e7739ed8604c" />
