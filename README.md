# Лабораторная работа №6: Создание многоконтейнерного приложения

## Цель работы
Ознакомиться с работой многоконтейнерного приложения на базе `docker-compose`.

## Задание
Создать PHP-приложение на базе трёх контейнеров: `nginx`, `php-fpm`, `mariadb`, используя `docker-compose`.

## Описание выполнения работы

### 1. Подготовка
- Работа основана на лабораторной работе №5.
- Создан репозиторий `containers07` и склонирован на локальную машину.

### 3. Конфигурация

#### .gitignore
```gitignore
# Ignore files and directories
mounts/site/*
```

#### nginx/default.conf
```nginx
server {
    listen 80;
    server_name _;
    root /var/www/html;
    index index.php;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }
    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### docker-compose.yml
```yaml
version: '3.9'

services:
  frontend:
    image: nginx:1.19
    volumes:
      - ./mounts/site:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    ports:
      - "80:80"
    networks:
      - internal
    env_file:
      - mysql.env
      - app.env

  backend:
    image: php:7.4-fpm
    volumes:
      - ./mounts/site:/var/www/html
    networks:
      - internal
    env_file:
      - mysql.env
      - app.env

  database:
    image: mysql:8.0
    env_file:
      - mysql.env
    networks:
      - internal
    volumes:
      - db_data:/var/lib/mysql

networks:
  internal: {}

volumes:
  db_data: {}
```

#### mysql.env
```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=app
MYSQL_USER=user
MYSQL_PASSWORD=secret
```

#### app.env
```env
APP_VERSION=1.0.0
```

### 4. Запуск
Запуск контейнеров выполняется командой:
```bash
docker-compose up -d
```

### 5. Проверка работы
- Перейти в браузере по адресу [http://localhost](http://localhost).
- Если загружается базовая страница nginx — обновить страницу.
- Убедиться, что PHP-сайт работает.

---

## Ответы на вопросы

**1. В каком порядке запускаются контейнеры?**  
Контейнеры запускаются в порядке, указанном в `docker-compose.yml`, однако `docker-compose` не гарантирует последовательность запуска. Взаимодействие между контейнерами (например, `nginx` и `php-fpm`) реализуется через общую сеть, и сервисы могут дождаться друг друга при необходимости. Чтобы гарантировать порядок, можно использовать параметр `depends_on`.

**2. Где хранятся данные базы данных?**  
Данные MySQL сохраняются в Docker-томе `db_data`, который монтируется в `/var/lib/mysql`.

**3. Как называются контейнеры проекта?**  
Названия контейнеров формируются по шаблону `<имя_проекта>_<имя_сервиса>_1`. Например:
- `containers07_frontend_1`
- `containers07_backend_1`
- `containers07_database_1`

(Название проекта — это имя каталога проекта, если не задано явно через параметр `-p`.)

**4. Как добавить переменную окружения `APP_VERSION` для сервисов backend и frontend?**  
Создать файл `app.env` со строкой:
```env
APP_VERSION=1.0.0
```
Далее в секциях `frontend` и `backend` файла `docker-compose.yml` добавлен параметр:
```yaml
env_file:
  - mysql.env
  - app.env
```
Теперь переменная `APP_VERSION` будет доступна внутри контейнеров `frontend` и `backend`.

---

## Выводы
В рамках лабораторной работы:
- Изучена структура многоконтейнерного приложения.
- Настроено взаимодействие между `nginx`, `php-fpm` и `mysql`.
- Получен опыт работы с `docker-compose`, сетями и томами Docker.
- На практике применены принципы модульной архитектуры контейнеров.
```
