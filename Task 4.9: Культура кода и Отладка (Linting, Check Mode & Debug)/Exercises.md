# Checkpoints

**Задание** `9.1`: **Чистка кода** (`Ansible Lint`):

- Установи `ansible-lint` (через `pip install ansible-lint` или пакетный менеджер).
- Запусти проверку на всем своем проекте: `ansible-lint site.yml`.
- Action: Ты увидишь кучу предупреждений (`warnings`) и ошибок.
 - Исправь их все!
 - Примеры частых ошибок:
  - Отсутствие name у таска.
  - Использование `command` вместо специализированного модуля (например `git` или `service`).
  - Использование `chmod 777`.
  - Отсутствие `changed_when` для `shell/command` (`Ansible` не знает, изменил ли скрипт что-то, и всегда пишет `changed`. Ты должен явно указать условие).
- Добейся пустого вывода линтера.

```bash
pip install ansible-lint
```

Покажем в термминале:

<img width="1453" height="645" alt="image" src="https://github.com/user-attachments/assets/0965bbdc-6503-462f-a816-54faf94e3155" />
<img width="1533" height="664" alt="image" src="https://github.com/user-attachments/assets/ec35bebb-fb40-42aa-a631-c60493dc510f" />

```bash
sudo ansible-lint site.yml
```

Покажем в терминале:

<img width="1495" height="18" alt="image" src="https://github.com/user-attachments/assets/dd5d410f-011a-4895-a74e-8d222a0d225b" />
<img width="1132" height="452" alt="image" src="https://github.com/user-attachments/assets/2e17b856-da1a-45fa-aa58-f31d37fb282f" />
<img width="1524" height="378" alt="image" src="https://github.com/user-attachments/assets/ba1b08e2-f559-44a0-ae0f-fedd975a45c8" />
