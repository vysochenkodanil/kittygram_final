# Kittygram: Контейнеризация и CI/CD

## Технологический стек

- Django (бэкенд)
- PostgreSQL (база данных)
- React (фронтенд)
- Nginx (обратный прокси)
- Docker (контейнеризация)
- GitHub Actions (CI/CD)

## Настройка сервера

### Установка Docker и Docker Compose

Выполните на сервере следующие команды для установки:

```bash
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt install docker-compose-plugin
```

### Копирование конфигурационных файлов

1. Скопируйте файл `docker-compose.production.yml` на сервер:

```bash
scp -i путь_до_SSH/имя_SSH docker-compose.production.yml \
    username@server_ip:/home/username/kittygram/docker-compose.production.yml
```

2. Скопируйте файл `.env` с переменными окружения:

```bash
scp -i путь_до_SSH/имя_SSH .env \
    username@server_ip:/home/username/kittygram/.env
```

### Настройка Nginx

1. Откройте конфигурационный файл Nginx:
```bash
sudo nano /etc/nginx/sites-enabled/default
```

2. Замените все настройки `location` на следующие:
```nginx
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:8000;
}
```

3. Перезагрузите конфигурацию Nginx:
```bash
sudo service nginx reload
```

## Workflow для автоматического деплоя

GitHub Actions workflow автоматически выполняет следующие шаги при пуше в ветку `main`:

1. Запуск тестов:
   - Проверка кода flake8
   - Тесты Django (бэкенд)
   - Тесты React (фронтенд)

2. Сборка и публикация Docker-образов:
   - Бэкенд (Django)
   - Фронтенд (React)
   - Nginx (gateway)

3. Деплой на сервер:
   - Копирование docker-compose.production.yml
   - Обновление контейнеров (`docker compose pull`)
   - Перезапуск сервисов
   - Применение миграций
   - Сбор статических файлов

4. Отправка уведомления в Telegram об успешном деплое

## Требуемые секреты (Secrets)

В настройках репозитория (Settings → Secrets and variables → Actions) необходимо добавить:

1. `DOCKER_USERNAME` - логин Docker Hub
2. `DOCKER_PASSWORD` - пароль Docker Hub
3. `HOST` - IP сервера
4. `USER` - пользователь сервера
5. `SSH_KEY` - SSH-ключ
6. `SSH_PASSPHRASE` - пароль SSH-ключа
7. `TELEGRAM_TOKEN` - токен бота Telegram
8. `TELEGRAM_TO` - ID чата для уведомлений

## Как работает автоматический деплой

При каждом push в ветку `main`:
1. GitHub Actions запускает тесты
2. После успешного прохождения тестов собираются Docker-образы
3. Образы публикуются в Docker Hub
4. Происходит подключение к серверу и обновление контейнеров
5. Применяются миграции и собирается статика
6. Отправляется уведомление в Telegram

