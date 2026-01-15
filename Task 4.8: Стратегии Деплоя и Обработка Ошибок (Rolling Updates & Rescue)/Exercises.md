<img width="1533" height="74" alt="image" src="https://github.com/user-attachments/assets/03f9e83f-1ae3-4b22-916b-c7e1c39aeb79" /># Checkpoints

**Задание** `8.1`: Масштабирование (`Scale Out`) `Rolling Update` на одном сервере не имеет смысла (у тебя все равно будет простой):

- **Действие**: Клонируй свою виртуальную машину `appserver` (или создай новую). Назови её `vm-app-2`. Покажем в терминале:

<img width="1533" height="74" alt="image" src="https://github.com/user-attachments/assets/0c6851a6-8c4d-4b14-96c8-c72004531bc7" />

- Добавь её `IP` в инвентарь в группу `appservers`:

```bash
    appservers:
      hosts:
        appserver:
          ansible_host: 34.30.111.203
        appserver1:
          ansible_host: 34.68.78.218
```

Покажем в терминале:

<img width="1005" height="194" alt="image" src="https://github.com/user-attachments/assets/96a31c75-255d-43a6-ad29-fbc65f978a47" />

- Прогони свой старый `site.yml`. Благодаря идемпотентности, первый сервер не изменится, а второй настроится с нуля (пользователи, `Nginx`, ключи):


  
- Убедись, что балансировщик (`Task 5`) видит оба сервера и распределяет трафик (проверь через `curl` на балансер — должен меняться ответ, если ты менял `index.html`):

```bash

```
