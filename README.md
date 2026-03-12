# Лабораторная работа №2: Установка и настройка LAMP с WordPress и PhpMyAdmin

## Студент
Имя Фамилия: Иордатий Игнат  
Группа: IA2403 
Дата выполнения: 12.03.2026 

---

## Цель работы
Научиться работать с виртуальной машиной Debian, устанавливать и настраивать LAMP (Linux, Apache, MySQL/MariaDB, PHP), настраивать виртуальные хосты и управлять CMS WordPress и PhpMyAdmin.

---

## Задачи
1. Установить Debian Server на QEMU.  
2. Установить и настроить LAMP (Apache2, PHP, MariaDB).  
3. Скачивать и распаковывать PhpMyAdmin и WordPress.  
4. Создать базы данных и пользователей для WordPress.  
5. Настроить виртуальные хосты Apache.  
6. Проверить работу сайтов через браузер.  

---

## Выполнение работы

### 1. Установка Debian
- Загружен ISO-дистрибутив Debian Server (x64, без графического интерфейса).  
- Установщик прошёл без Desktop Environment.  
- Основные настройки:  
  - Hostname: `debian`  
  - Domain: `debian.localhost`  
  - Пользователь: `user` / пароль: `password`  
  - Root: `root` / пароль: `password`  
  - Диск: один раздел, весь диск, стандартная файловая система.
---

### 2. Установка LAMP

```bash
su
apt update -y
apt install -y apache2 php libapache2-mod-php php-mysql mariadb-server mariadb-client unzip
````

**Назначение пакетов:**

| Пакет              | Назначение                             |
| ------------------ | -------------------------------------- |
| apache2            | Веб-сервер для обслуживания сайтов     |
| php                | Язык программирования для сайтов       |
| libapache2-mod-php | Интеграция PHP с Apache                |
| php-mysql          | Позволяет PHP работать с MariaDB/MySQL |
| mariadb-server     | СУБД для сайтов                        |
| mariadb-client     | Клиент для работы с MariaDB            |
| unzip              | Распаковка ZIP-файлов                  |

---

### 3. Скачивание и распаковка PhpMyAdmin и WordPress

```bash
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.2/phpMyAdmin-5.2.2-all-languages.zip
wget https://wordpress.org/latest.zip

mkdir -p /var/www/phpmyadmin
mkdir -p /var/www/wordpress

unzip phpMyAdmin-5.2.2-all-languages.zip
mv phpMyAdmin-5.2.2-all-languages /var/www/phpmyadmin

unzip latest.zip
mv wordpress /var/www/wordpress
```

**Скриншот папки /var/www с PhpMyAdmin и WordPress:**
<img width="853" height="539" alt="image" src="https://github.com/user-attachments/assets/d066b7c9-6e7f-4c3f-8172-8c72172ba57d" />

---

### 4. Создание базы данных WordPress

```sql
mysql -u root
CREATE DATABASE wordpress_db;
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Скриншот консоли MySQL:**
<img width="852" height="541" alt="image" src="https://github.com/user-attachments/assets/bfacc66c-266b-4fcc-99dd-8ad5ae680a4d" />

<img width="848" height="510" alt="image" src="https://github.com/user-attachments/assets/5e80370e-1ef7-49a3-b2db-5d9cb6209397" />



---

### 5. Настройка виртуальных хостов Apache

**01-phpmyadmin.conf**:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/phpmyadmin"
    ServerName phpmyadmin.localhost
    ServerAlias www.phpmyadmin.localhost
    ErrorLog "/var/log/apache2/phpmyadmin.localhost-error.log"
    CustomLog "/var/log/apache2/phpmyadmin.localhost-access.log" common
</VirtualHost>
```

**02-wordpress.conf**:

```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot "/var/www/wordpress"
    ServerName wordpress.localhost
    ServerAlias www.wordpress.localhost
    ErrorLog "/var/log/apache2/wordpress.localhost-error.log"
    CustomLog "/var/log/apache2/wordpress.localhost-access.log" common
</VirtualHost>
```

Активируем сайты и обновляем hosts:

```bash
/usr/sbin/a2ensite 01-phpmyadmin
/usr/sbin/a2ensite 02-wordpress

echo "127.0.0.1 phpmyadmin.localhost" >> /etc/hosts
echo "127.0.0.1 wordpress.localhost" >> /etc/hosts

systemctl restart apache2
```

**Скриншот настроенных сайтов:**
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/4e7412bd-5cdd-4df4-9529-e3cdfe954162" />

<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/535ec829-07e9-4629-baff-0bed305cb43c" />


---

### 6. Проверка работы сайтов

* WordPress: `http://wordpress.localhost:1080`
* PhpMyAdmin: `http://phpmyadmin.localhost:1080`

**Скриншоты сайтов в браузере:**
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/2d225283-4980-40f3-bec1-e30c4169a868" />

<img width="1919" height="1079" alt="Снимок экрана 2026-03-12 043223" src="https://github.com/user-attachments/assets/b95cb0f3-2e3a-4115-8678-6ea5db501102" />

---

## Ответы на вопросы

1. **Как скачать файл в консоли при помощи wget?**

   ```bash
   wget <URL>
   ```

2. **Зачем необходимо создавать для каждого сайта свою базу и пользователя?**
   Для безопасности и изоляции данных. Если один сайт будет взломан, другие останутся защищены.

3. **Как поменять доступ к системе управления БД на порт 1234?**
   В файле `/etc/mysql/mariadb.conf.d/50-server.cnf` изменить строку `port = 3306` на `port = 1234` и перезапустить MariaDB:

   ```bash
   systemctl restart mariadb
   ```

4. **Какие преимущества даёт виртуализация?**

   * Изоляция сервисов
   * Возможность создавать несколько тестовых серверов на одном хосте
   * Простое резервное копирование и восстановление

5. **Для чего необходимо устанавливать время / временную зону на сервере?**
   Чтобы корректно работали логи, задачи cron и временные метки баз данных.

6. **Сколько места занимает установленная ОС (виртуальный диск)?**
   Зависит от выбранного диска (`debian.qcow2` размером 8 ГБ).

7. **Рекомендации по разбиению диска для серверов?**

   * Отдельные разделы для `/var`, `/home`, `/tmp`
   * Позволяет защитить систему от переполнения отдельных разделов и облегчает резервное копирование.

---

## Выводы

В ходе работы я:

* Установил Debian Server без GUI.
* Настроил LAMP (Apache2, PHP, MariaDB).
* Установил и настроил PhpMyAdmin и WordPress.
* Создал виртуальные хосты Apache и базы данных.
* Проверил работу сайтов в браузере.
