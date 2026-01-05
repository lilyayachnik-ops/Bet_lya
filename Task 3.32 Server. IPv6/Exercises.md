# Checkpoints

Включим поддержку `IPv6` в `docker` через конфигурационный файл и перезапустим `docker-daemon`:

```bash
 "ipv6": true,
"fixed-cidr-v6": "fd00::/80",
```

Покажем в терминале:

<img width="802" height="401" alt="image" src="https://github.com/user-attachments/assets/a03bd125-074f-457c-bfff-f8a6d89f8154" />

Выведим список сетевых адресов в хостовой системе:

```bash
ip addr show
```

Покажем в терминале:

<img width="694" height="333" alt="image" src="https://github.com/user-attachments/assets/fd2f6bad-e786-441d-bb1a-cb6067a47692" />

Запустим контейнер `nginx:stable`:

```bash
docker run -d -p 8080:80 nginx:stable
```

Покажем в терминал:

<img width="693" height="51" alt="image" src="https://github.com/user-attachments/assets/6bdd1c1e-f05a-45f1-9f39-513aeb031986" />

Выведим список сетевых адресов в хостовой системе. Убедись, что присутствует ipv6 адрес:

```bash
ip -6 addr show
```
Из контейнера nginx:stable при помощи команды curl -i [IPV6_ADDR] попробуй обратиться к AAAA записи google.com (адрес можно узнать при помощи nslookup);
Произведи рестарт docker-daemon;
Выведи список контейнеров и обрати внимание на время и статус;
Включи в конфигурационном файле опцию live-restore и перезапусти docker-daemon(Запусти контейнер nginx:stable в фоновом режиме);
Подожди около минуты и произведи перезапуск docker-daemon повторно;
Выведи список контейнеров и обрати внимание на время запуска и статус.
