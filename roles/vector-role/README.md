# vector-role

Роль устанавливает и настраивает агент Vector для сбора логов и отправки их в ClickHouse.

## Переменные

| Переменная | По умолчанию | Описание |
|---|---|---|
| `vector_version` | `0.39.0` | Версия Vector |
| `clickhouse_host` | `hostvars[groups['clickhouse'][0]]['ansible_host']` | IP-адрес ClickHouse |

## Пример использования

```yaml
- hosts: vector
  roles:
    - vector-role
Что делает роль
Добавляет YUM-репозиторий Vector.

Устанавливает Vector указанной версии.

Деплоит конфиг /etc/vector/vector.yaml.

Запускает сервис vector.
