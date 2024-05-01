# 📶 Unifi Controller - Инструкция по установке.
> [!IMPORTANT]
> Инструкция написана для установки контроллера на Linux Ubuntu 22.04 LTS (Jammy release).

## Шаг 1. Обновление системы.
```
sudo apt update -y
sudo apt upgrade -y
```

## Шаг 2. Установка Java 17.
```
sudo apt install openjdk-17-jre-headless
```

## Шаг 3. Установка MongoDB 7.0.
1. Установите `gnupg` и `curl`, если они не установлены изначально:
```
sudo apt-get install gnupg curl
```

2. Импортируйте открытый GPG-ключ MongoDB:
```
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor
```

3. Добавьте репозиторий MongoDB в список источников APT:
```
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

4. Обновите локальную базу пакетов APT:
```
sudo apt-get update
```

5. Установите последнюю стабильную версию MongoDB:
```
sudo apt-get install -y mongodb-org
```

## Шаг 4. Установка Unifi контроллера.
1. Скачайте deb пакет Unifi контроллера:
```
wget https://dl.ui.com/unifi/7.3.83/unifi_sysvinit_all.deb
```

2. Установите deb пакет:
```
dpkg -i unifi_sysvinit_all.deb
```

> [!NOTE]
> На этом моменте вы можете закончить.
> 
> **Unifi контроллер установлен и доступен** по следующим адресам:
> 
> SSL: https://your-ip-address:8443/
> 
> NoSSL: http://your-ip-address:8080/
> 
> Дальше мы установим веб-сервер **nginx** для настройки доступа по SSL и доменному имени.

## Шаг 5. Установка nginx (Опционально).
1. Скачайте и установите nginx:
```
sudo apt update
sudo apt install nginx -y
```

> [!CAUTION]
> **Обязательно в следующих шагах замените yourdomain.ru на свой домен.**

2. Создайте конфигурационный файл для вашего сайта в `/etc/nginx/sites-available/` и откройте его для редактирования:
```
sudo nano /etc/nginx/sites-available/yourdomain.ru
```

3. Скопируйте настройки в конфигурационный файл вашего сайта:
```
server {
    listen 80;
    server_name yourdomain.ru; # Заменить здесь

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    server_name yourdomain.ru; # Заменить здесь

    location / {
        proxy_pass https://localhost:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

4. Затем активируйте этот сайт, создав символическую ссылку в каталоге `/etc/nginx/sites-enabled`:
```
sudo ln -s /etc/nginx/sites-available/yourdomain.ru /etc/nginx/sites-enabled/
```

5. Выполните тестирование конфигурационных файлов:
```
sudo nginx -t
```

Если **ошибок нет**, тогда **можно идти дальше**.

В **ином случае**, **исправьте их**, чтобы продолжить.

## Шаг 6. Установка Certbot (Опционально).
1. Установите `certbot`, инструмент для автоматической генерации SSL-сертификатов **Let's Encrypt**:
```
sudo apt install certbot python3-certbot-nginx
```

2. Запустите `certbot` для вашего домена, чтобы получить SSL-сертификат и настроить его автоматическое обновление:
```
sudo certbot --nginx -d yourdomain.ru
```

3. Выполните тестирование конфигурационных файлов:
```
sudo nginx -t
```
Если **ошибок нет**, тогда **можно идти дальше**.

В **ином случае**, **исправьте их**, чтобы продолжить.

4. Перезапустите nginx:
```
sudo systemctl restart nginx
```
5. Перейдите по адресу https://yourdomain.ru/
6. Вы должны увидеть настройку Unifi контроллера.
