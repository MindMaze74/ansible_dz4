# lighthouse-role

Роль устанавливает веб-интерфейс LightHouse для работы с ClickHouse.

## Переменные

| Переменная | По умолчанию | Описание |
|---|---|---|
| `lighthouse_dir` | `/usr/share/nginx/lighthouse` | Директория со статикой |

## Пример использования

```yaml
- hosts: lighthouse
  roles:
    - lighthouse-role

    
Что делает роль
Устанавливает EPEL-репозиторий.

Устанавливает git и nginx.

Клонирует LightHouse в указанную директорию.

Деплоит конфиг Nginx.

Запускает Nginx.
