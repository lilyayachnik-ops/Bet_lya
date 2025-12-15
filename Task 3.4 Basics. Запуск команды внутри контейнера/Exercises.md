# Checkpoints

Скачай конфигурационный файл nginx. Запусти контейнер со следующими параметрами: должен работать в фоне, слушает на хосте `127.0.0.1:8890`,
имеет имя `inno-dkr-03`, должен пробрасывать скачанный вами конфигурационный файл внутрь контейнера как основной конфигурационный файл, образ — `nginx:stable`:

```bash
cp ~/docker_3.2/nginx.conf ~/docker_3.3/
ls -la
cat nginx.conf
docker run -d -p 127.0.0.1:8890:80 --name inno-dkr-03 --mount type=bind,source"$(pwd)/nginx.conf",target=/etc/nginx/nginx.conf nginx:stable
curl http://127.0.0.1:8890
docker ps
cp ~/docker_3.2/nginx-new.conf ~/docker_3.3/nginx.conf


# nginx -s reload, где -s — сигнал, reload — перезагрузить конфигурацию

$ docker exec inno-dkr-03 nginx -s reload
curl http://127.0.0.1:8890
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="880" height="139" alt="image" src="https://github.com/user-attachments/assets/d5b799ee-63f3-43c5-9034-f766bdf4ca51" />
<img width="889" height="355" alt="image" src="https://github.com/user-attachments/assets/c134e9c9-0d8d-45bc-860e-0b2529c9a52b" />
<img width="895" height="197" alt="image" src="https://github.com/user-attachments/assets/c40e9de5-a2e2-4b7c-bdea-7d842d08d5c5" />
<img width="891" height="178" alt="image" src="https://github.com/user-attachments/assets/a48e8bb9-d348-4ab6-b90a-3ccb0f044da1" />
<img width="860" height="44" alt="image" src="https://github.com/user-attachments/assets/dc087cfc-10aa-48c6-a77e-740ff2329a08" />
<img width="1513" height="47" alt="image" src="https://github.com/user-attachments/assets/c252a4b8-496e-4b87-bc36-3e69e9ef5cc7" />


Давайте через редактор `nano`:

```bash
nano nginx.conf
cat nginx.conf
docker exec inno-dkr-03 nginx -s reload
curl http://127.0.0.1:8890
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
```

Покажем в терминале:

<img width="1319" height="27" alt="image" src="https://github.com/user-attachments/assets/d1260e69-42b2-4b6f-a0a2-d4aaae33adfb" />
<img width="1834" height="360" alt="image" src="https://github.com/user-attachments/assets/c40f7f35-8bf5-4217-b6e1-55e39364efc9" />
<img width="1322" height="47" alt="image" src="https://github.com/user-attachments/assets/be64c6ff-2eeb-44f1-bcbb-bf6a29e92a2e" />
<img width="1515" height="67" alt="image" src="https://github.com/user-attachments/assets/291baac8-3ac6-4715-add9-202aa6ad1e01" />


Давайте через редактор `vim`:

```bash
vim nginx.conf
cat nginx.conf
docker exec inno-dkr-03 nginx -s reload
curl http://127.0.0.1:8890
docker exec -ti inno-dkr-03 md5sum /etc/nginx/nginx.conf
docker ps
```

Покажем в терминале:

<img width="1137" height="39" alt="image" src="https://github.com/user-attachments/assets/dfe80f31-990c-40b8-bc2c-565ea801c9d7" />
<img width="1630" height="72" alt="image" src="https://github.com/user-attachments/assets/0b3c8378-e02d-4240-9b40-c55be23e5921" />
<img width="1629" height="72" alt="image" src="https://github.com/user-attachments/assets/b7a2dcc0-2909-458f-981f-aa9ae4775231" />



