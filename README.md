# Полное руководство: Установка Nextcloud на RedOS с Nginx и MySQL

## Шаг 1: Предварительная подготовка системы

1. **Обновите систему**:
    ```bash
    sudo dnf update -y
    ```

2. **Установите необходимые пакеты**:
    ```bash
    sudo dnf install -y nginx mariadb-server mariadb php php-fpm php-mysqlnd php-xml php-gd php-mbstring php-intl php-curl php-zip wget policycoreutils-python-utils
    ```

3. **Запустите службы и добавьте их в автозагрузку**:
    ```bash
    sudo systemctl enable --now nginx mariadb php-fpm
    ```
В файле:
```bash
nano /etc/hosts 
```
Добавить:
```bash
server_name 172.16.4.2;
```
nano /etc/hosts 

---

## Шаг 2: Настройка MySQL

1. **Инициализация MariaDB**:
    ```bash
    sudo mysql_secure_installation
    ```
    В процессе настройки:
    - Установите пароль root.
    - Удалите анонимных пользователей.
    - Отключите удаленный доступ для root.
    - Удалите тестовую базу данных.
    - Примите изменения.

2. **Создайте базу данных и пользователя**:
    ```bash
    sudo mysql -u root -p
    ```

    В MySQL выполните следующие команды:
    ```sql
    CREATE DATABASE nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
    CREATE USER 'nextclouduser'@'localhost' IDENTIFIED BY 'student';
    GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextclouduser'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;
    ```

---

## Шаг 3: Настройка PHP

1. **Откройте файл конфигурации PHP-FPM**:
    ```bash
    sudo nano /etc/php-fpm.d/www.conf
    ```

2. **Убедитесь в правильных настройках**:
    - Задайте пользователя и группу:
        ```ini
        user = nginx
        group = nginx
        ```
    - Настройте сокет:
        ```ini
        listen = /run/php-fpm/www.sock
        listen.owner = nginx
        listen.group = nginx
        listen.mode = 0660
        ```

3. **Сохраните изменения и перезапустите PHP-FPM**:
    ```bash
    sudo systemctl restart php-fpm
    ```

---

## Шаг 4: Настройка Nginx

1. **Создайте конфигурацию для Nextcloud**:
    ```bash
    sudo nano /etc/nginx/conf.d/nextcloud.conf
    ```

2. **Вставьте следующее содержимое**:
    ```nginx
    server {
        listen 80;
        server_name 172.16.4.2;

        root /var/www/nextcloud;
        index index.php index.html;

        client_max_body_size 512M;
        fastcgi_buffers 64 4K;

        location / {
            try_files $uri $uri/ /index.php;
        }

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/run/php-fpm/www.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
            deny all;
        }
    }
    ```

3. **Перезапустите Nginx**:
    ```bash
    sudo systemctl restart nginx
    ```

---

## Шаг 5: Установка Nextcloud

1. **Скачайте и распакуйте Nextcloud**:
    ```bash
    wget https://download.nextcloud.com/server/releases/latest.tar.bz2
    tar -xjf latest.tar.bz2
    sudo mv nextcloud /var/www/
    ```

2. **Настройте права доступа**:
    ```bash
    sudo chown -R nginx:nginx /var/www/nextcloud
    sudo chmod -R 755 /var/www/nextcloud
    ```

---

## Шаг 6: Настройка безопасности

1. **Настройте SELinux**:
    ```bash
    sudo semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/nextcloud(/.*)?'
    sudo restorecon -R /var/www/nextcloud
    ```

2. **Настройте файрвол**:
    ```bash
    sudo firewall-cmd --permanent --add-service=http
    sudo firewall-cmd --permanent --add-service=https
    sudo firewall-cmd --reload
    ```

---

## Шаг 7: Оптимизация PHP

1. **Включите opcache**:
    - Откройте файл конфигурации opcache:
        ```bash
        sudo nano /etc/php.d/10-opcache.ini
        ```
    - Добавьте или измените параметры:
        ```ini
        opcache.enable=1
        opcache.enable_cli=1
        opcache.memory_consumption=128
        opcache.interned_strings_buffer=8
        opcache.max_accelerated_files=10000
        opcache.revalidate_freq=1
        opcache.save_comments=1
        ```

2. **Перезапустите PHP-FPM**:
    ```bash
    sudo systemctl restart php-fpm
    ```

---

## Шаг 8: Завершение настройки Nextcloud

1. **Перейдите в браузере на адрес сервера**:
    ```
    http://<ваш_IP_или_домен>
    ```

2. **Заполните форму**:
    - **Данные для базы данных**:
        - Имя базы данных: `nextcloud`
        - Имя пользователя: `nextclouduser`
        - Пароль: `student`
    - **Путь к данным**: `/var/www/nextcloud/data`.
 ```bash
sudo chown -R root:root /var/lib/php/session
sudo chown -R nginx:nginx /var/lib/php/session
sudo chmod 700 /var/lib/php/session
```

3. **Завершите установку**.

---



















# Удаление установленных компонентов и конфигураций Nextcloud

## 1. Удаление установленных компонентов

1. **Удалите установленные пакеты**:
    ```bash
    sudo dnf remove -y nginx mariadb-server mariadb php php-fpm php-mysqlnd php-xml php-gd php-mbstring php-intl php-curl php-zip
    ```

2. **Удалите все зависимости, которые больше не используются**:
    ```bash
    sudo dnf autoremove -y
    ```

---

## 2. Удаление файлов конфигурации

Удалите оставшиеся конфигурационные файлы и каталоги:
```bash
sudo rm -rf /etc/nginx /etc/php-fpm.d /var/www/nextcloud /var/lib/mysql
sudo rm -rf /var/lib/mysql/
sudo rm -rf /var/log/mariadb/
sudo rm -rf /etc/my.cnf*
 ```
