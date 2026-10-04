# Домашнее задание №4: Роли в Ansible - Старцев Данила Антонович

## Что сделано

Playbook для развёртывания стека **ClickHouse + Vector + LightHouse** разбит на три роли:
- **ClickHouse** — подключена готовая роль из публичного репозитория через `requirements.yml`;
- **Vector** — создана своя роль и опубликована в отдельном репозитории;
- **LightHouse** — создана своя роль и опубликована в отдельном репозитории.

## Ссылки на репозитории

| Что | Ссылка |
|---|---|
| Основной репозиторий с playbook | https://github.com/MindMaze74/ansible_dz4 |
| Роль Vector | https://github.com/MindMaze74/vector-role |
| Роль LightHouse | https://github.com/MindMaze74/lighthouse-role |
| Роль ClickHouse (внешняя) | https://github.com/AlexeySetevoi/ansible-clickhouse |

Все свои роли опубликованы с тегом **v1.0.0**.

## Структура проекта

```
.
├── group_vars/
│   ├── all/vars.yml              # общие переменные
│   └── clickhouse/vars.yml       # переопределения переменных роли ClickHouse
├── inventory/
│   └── prod.yml                  # inventory с тремя группами хостов
├── roles/
│   ├── clickhouse/               # внешняя роль (v1.13)
│   ├── vector-role/              # своя роль (v1.0.0)
│   └── lighthouse-role/          # своя роль (v1.0.0)
├── requirements.yml              # зависимости ролей
├── site.yml                      # playbook с использованием roles
└── README.md
```

## Описание ролей

### Роль `clickhouse` (внешняя)
- **Источник:** [AlexeySetevoi/ansible-clickhouse](https://github.com/AlexeySetevoi/ansible-clickhouse), версия 1.13
- **Назначение:** установка ClickHouse, настройка, создание БД.
- **Ключевые переменные:**
  - `clickhouse_version` — версия пакета (`22.3.3.44`).
  - `clickhouse_dbs_custom` — список БД для создания (`logs`).
  - `clickhouse_listen_host` — интерфейсы для прослушивания (`::`).
  - `clickhouse_networks_default` — разрешённые сети для пользователя `default`.

### Роль `vector-role`
- **Источник:** [MindMaze74/vector-role](https://github.com/MindMaze74/vector-role), тег `v1.0.0`
- **Назначение:** установка Vector 0.39.0, настройка отправки логов в ClickHouse через HTTP sink.
- **Ключевые переменные:**
  - `vector_version` — версия Vector (`0.39.0`).
  - `clickhouse_host` — IP-адрес ClickHouse (берётся из `hostvars`).

![скриншот 3](https://github.com/MindMaze74/ansible_dz4/blob/main/img/3.png)

![скриншот 4](https://github.com/MindMaze74/ansible_dz4/blob/main/img/4.png)

### Роль `lighthouse-role`
- **Источник:** [MindMaze74/lighthouse-role](https://github.com/MindMaze74/lighthouse-role), тег `v1.0.0`
- **Назначение:** установка Nginx и веб-интерфейса LightHouse.
- **Ключевые переменные:**
  - `lighthouse_dir` — директория со статикой (`/usr/share/nginx/lighthouse`).

![скриншот 1](https://github.com/MindMaze74/ansible_dz4/blob/main/img/1.png)

![скриншот 2](https://github.com/MindMaze74/ansible_dz4/blob/main/img/2.png)

## Запуск

```bash
# Установка ролей из requirements.yml
ansible-galaxy install -r requirements.yml -p roles

# Полный запуск
ansible-playbook -i inventory/prod.yml site.yml

# Запуск только для конкретной группы
ansible-playbook -i inventory/prod.yml site.yml --limit clickhouse
ansible-playbook -i inventory/prod.yml site.yml --limit vector
ansible-playbook -i inventory/prod.yml site.yml --limit lighthouse

# Проверка синтаксиса
ansible-playbook -i inventory/prod.yml site.yml --syntax-check
```

## Результаты проверки

### Идемпотентность
Повторный запуск playbook даёт `changed=0` у всех трёх хостов:

```
PLAY RECAP
clickhouse-01 : ok=23 changed=0 unreachable=0 failed=0 skipped=10
lighthouse-01 : ok=6  changed=0 unreachable=0 failed=0
vector-01     : ok=5  changed=0 unreachable=0 failed=0
```

![скриншот 9](https://github.com/MindMaze74/ansible_dz4/blob/main/img/9.png)

### Цепочка данных
- **Vector** генерирует события (demo_logs) и отправляет их в ClickHouse через HTTP (JSONEachRow).
- **ClickHouse** хранит данные в БД `logs`, таблица `app_logs`.
- На момент проверки в таблице — **3635 записей**.
- **LightHouse** отображает БД `logs` и таблицу `app_logs` через веб-интерфейс.

### Стек на managed-хостах
- **clickhouse-vm** — ClickHouse 22.3.3.44, слушает порты 9000 и 8123.
- **vector-vm** — Vector 0.39.0, отправляет данные в ClickHouse.
- **lighthouse-vm** — Nginx + LightHouse, открывается по адресу `http://111.88.152.34`.


![скриншот 8](https://github.com/MindMaze74/ansible_dz4/blob/main/img/8.png)

## Итог

Задание выполнено полностью:
- Playbook разбит на три роли.
- Две роли (`vector-role`, `lighthouse-role`) написаны самостоятельно и опубликованы на GitHub с тегом `v1.0.0`.
- Одна внешняя роль (`clickhouse`) подключена через `requirements.yml`.
- `site.yml` использует `roles:` вместо `tasks:`.
- Идемпотентность подтверждена — `changed=0` при повторном запуске.
- Цепочка Vector → ClickHouse → LightHouse работает end-to-end.