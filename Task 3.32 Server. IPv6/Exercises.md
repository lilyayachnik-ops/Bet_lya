# Checkpoints

Включим поддержку `IPv6` в `docker` через конфигурационный файл и перезапустим `docker-daemon`:

```bash
sudo mkdir -p /etc/docker
sudo touch /etc/docker/daemon.json
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "ipv6": true,
  "fixed-cidr-v6": "fd00::/80",
  "experimental": true
}
EOF
sudo systemctl restart docker
sudo systemctl status docker
```

Покажем в терминале:

<img width="944" height="748" alt="изображение" src="https://github.com/user-attachments/assets/61ad9c5a-2b95-4a6b-932c-165a72671a55" />


Выведим список сетевых адресов в хостовой системе:

```bash
ip addr show
```

Покажем в терминале:

<img width="944" height="583" alt="изображение" src="https://github.com/user-attachments/assets/470d7010-70dd-4837-9d7b-b12d5e097011" />

Запустим контейнер `nginx:stable`:

```bash
docker run -d -p 8080:80 nginx:stable
```

Покажем в терминал:

<img width="947" height="241" alt="изображение" src="https://github.com/user-attachments/assets/8f558643-1989-45f0-b435-05b8895c39af" />

Выведим список сетевых адресов в хостовой системе. Убедись, что присутствует ipv6 адрес:

```bash
ip -6 addr show
```

Покажем в терминале:

<img width="885" height="349" alt="изображение" src="https://github.com/user-attachments/assets/4181e7a1-9bfd-429c-898e-d3d28524887d" />


Из контейнера `nginx:stable` при помощи команды `curl -i [IPV6_ADDR]` попробуем обратиться к `AAAA` записи `google.com` (адрес можно узнать при помощи `nslookup`):

```bash
docker exec -it pedantic_meitner sh
apt-get update
apt-get install -y curl
apt-get install -y dnsutils
nslookup -type=AAAA google.com
curl -i http://[2a00:1450:4025:807::8a]
```

Покажем в терминале:

<img width="696" height="26" alt="изображение" src="https://github.com/user-attachments/assets/378d5636-4d3f-4747-a413-1edf31ae6cf4" />
<img width="705" height="202" alt="изображение" src="https://github.com/user-attachments/assets/7989984b-f627-4f38-a89e-1bcc18915660" />
<img width="952" height="256" alt="изображение" src="https://github.com/user-attachments/assets/e4da768b-b1ed-41f0-aecd-6488e5aa2e41" />
<img width="951" height="256" alt="изображение" src="https://github.com/user-attachments/assets/44c4c7d5-5295-4b44-b7b3-f2aebcd64be6" />
<img width="951" height="509" alt="изображение" src="https://github.com/user-attachments/assets/b48f060a-9740-4e18-99b0-394b6fb3f16a" />


Произведём рестарт `docker-daemon:

```bash
sudo systemctl restart docker
```

Покажем в терминале:

<img width="688" height="46" alt="изображение" src="https://github.com/user-attachments/assets/b4d9ea5e-d72a-4335-a8b2-cf38073ac09d" />


Выведим список контейнеров и обрати внимание на время и статус:

```bash
docker ps -a
```

Покажем в терминале:

<img width="957" height="96" alt="изображение" src="https://github.com/user-attachments/assets/b452848a-fe4f-4a4c-a273-60949d504b6e" />


Включим в конфигурационном файле опцию live-restore и перезапустим docker-daemon(Запустим контейнер nginx:stable в фоновом режиме):

```bash
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "ipv6": true,
  "fixed-cidr-v6": "fd00::/80",
  "experimental": true,
  "live-restore": true
}
EOF
sudo systemctl restart docker
docker ps
docker run -d nginx:stable
```

Покажем в терминале:

<img width="910" height="351" alt="изображение" src="https://github.com/user-attachments/assets/12413284-ea43-436b-811f-ce4d8b378746" />

Подождём около минуты и произведём перезапуск docker-daemon повторно.

```bash
sudo systemctl restart docker
```

Покажем в терминале:

<img width="915" height="25" alt="изображение" src="https://github.com/user-attachments/assets/08ea8165-3604-46ac-b452-e48b991d06ea" />

Выведим список контейнеров и обратим внимание на время запуска и статус:

```bash
docker ps
```

Покажем в терминале:

<img width="915" height="93" alt="изображение" src="https://github.com/user-attachments/assets/41a07e9a-d415-4b1e-bdc8-20617d67426d" />
