---
marp: true
author: Alexander Dydychkin
title: Computer and OS architecture
theme: default
paginate: true
math: mathjax
header: '&#9670;'
footer: "Star slides [here](https://github.com/GoAlexander/courses)"

---

# Семинары. 3-ий модуль

---

## Виртуализация (для лекции №14)

```bash
sudo apt install qemu-system
sudo apt install virt-manager
```

```bash
# Примеры работающих iso образов (не ограничивайтесь представленными)
wget https://distro.ibiblio.org/puppylinux/puppy-bookwormpup/BookwormPup64/10.0.9/BookwormPup64_10.0.9.iso
wget https://download.fedoraproject.org/pub/fedora/linux/releases/41/Workstation/x86_64/iso/Fedora-Workstation-Live-x86_64-41-1.4.iso
wget https://dl-cdn.alpinelinux.org/alpine/v3.21/releases/x86_64/alpine-virt-3.21.2-x86_64.iso
```

```bash
sudo virt-manager
```

---

## Лабораторная работа 1 (3-ий модуль)

В группах по 1-2 человека:

1) Создать виртуальную машину и установить там любой дистрибьютив Linux
1) Поэксперементировать с опциями и настройками QEMU. Рассказать про какую-нибудь особенность / подсистему / настройку / умение в virt-manager и/или QEMU.

**Максимальная оценка: 8 баллов**
+1 балл любой non-Ubuntu будет оцениваться выше + объяснить особенности этого дистрибьютива (почему он был создан, где и для чего используется)
+1 демонстрация работающего ssh подключения на машину в виртуалке (к примеру из WSL в VM1)
**Срок: 1 неделя.** Сдача через 2 недели -2 балла, после работа не принимается.

---

## Лабораторная работа 2 (3-ий модуль)
В группах по 1-2 человека:
1) Создать и запустить контейнер с веб-сервером (к примеру nginx), который будет отдавать **кастомную** html-страницу на 80-ом порту хост-системы.

---

2) Создать и запустить контейнер с окружением для сборки [openh264](https://github.com/cisco/openh264):
    - в качестве основы использовать ubuntu 22.04
    - использовать билд систему Meson: https://github.com/cisco/openh264#using-meson
    - пакетны/е зависимости для сборки можно найти анализируя отчет meson или методом внимательного чтения лекционной презентации :smile:
    - строить master head
    - после работы контейнера на хост-системе должны остаться артефакты билда


**Срок: 2 недели.** После работа не принимается.

---

# Справочная информация для лабораторной №2
##### nginx

nginx - один из самых популярных веб-серверов (программ для отдачи контента в веб, к примеру html страницы).

Default configuration location: `/etc/nginx/nginx.conf`
Configuration of default html: `/etc/nginx/sites-enabled/default`
From conf above location of default html: `/var/www/html/index.nginx-debian.html`
Example configuration for sharing files:

---

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}
http {
        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;
        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
        gzip on;
        server {
            listen 80 default_server;
            listen [::]:80 default_server;

            location /{
                root /var/www/html;
                index index.html;
                autoindex on;
            }
        }
}
```

---

## Systemd
**systemd** - подсистема (набор приложений) для инициализации и управления службами в Linux, автор Леннарт Пёттеринг. Пришел на смену подсистеме `init`. Крайне важная часть современных дистрибьютивов. Наиболее важные команды:
```bash
$ systemctl status 
# systemctl - (system control), получить статус по всем запущенным сервисам
$ systemctl status nginx 
# получить статус по сервису nginx

$ systemctl enable nginx 
# включить сервис nginx (nginx запустится во время следующей загрузки OS)
$ systemctl start nginx  
# запустить сервис nginx прямо сейчас (после перезагрузки не перезапустится)
$ systemctl stop nginx   
# остановить сервис nginx прямо сейчас (после перезагрузки перезапустится)
```

---

Пример синтаксиса service-файлов
```bash
cat ~/.config/systemd/user/mpd.service
[Unit]
Description=Music Player Daemon

[Service]
ExecStart=/usr/bin/mpd --no-daemon

[Install]
WantedBy=default.target
```

systemd, video, критика, кратко: [тыц](https://www.youtube.com/watch?v=4RqclqgmwLo)

---

## Лабораторная работа 3 (3-ий модуль)

Вы выступаете в роли хостинг-провайдера. Вы должны предоставить своим клиентам веб-сервис для аренды виртуальных машин и контейнеров. Доступ в виде ssh.

---

Техническая часть:
1) Со стороны пользователя должно быть веб-приложение (сайт), который предоставляет функционал для настройки арендуемых мощностей.
    - тип (виртуалка/контейнер)
    - ОС
    - ресурсы
    - **расширенная кастомизация будет в +**
    - Для простоты можете использовать Streamlit (по запросу могу немного рассказать про него), но не ограничиваю. Предлагаю использовать ЯП Python, но тоже не ограничиваю.
        - *Кратко что такое Streamlit и как им пользоваться описано в небольшой серии постов у меня в телеге.*

--- 

2) Со стороны backend'а, должны быть два объекта, которые предоставляют API (любой тип от REST до функций) для создания виртуалок в QEMU и контейнеров через Docker.
    - с QEMU можно работать на уровне Python-модулей и/или методом вызова консольных команд (если очень хотите можно через virsh)

На 9 баллов:
3) Сделать сервис (механизм), который будет отслеживать запущенные виртуалки/контейнеры и будет их гасить по критерию X, к примеру у клиента закончилось процессорное время или израсходован сетевой лимит и т. д. Совсем шикарно, если будет реализован "монитор" поднятых виртуалок/контейнеров (хотя бы список и состояние).
    - **поддержка множества критериев будет в +**
    - **расширенные действия, к примеру перезаливка будет в +**

---

**Максимальная оценка 8 баллов.** Критерий получения 9 баллов - работа выполнена (около) идеально, группа разобралась в новой (дополнительной) теме и реализована дополнительно любая сложная автоматика. Если коротко: сделали всё + вижу, что команда "реально заморочилась".
**Команда с лучшей работой получит 10 баллов.**
**Защита:** презентация проекта на основе use-case'ов, кратко рассказать про архитектуру, реализованные фичи и killer-фичи (за что горды). Ответить на вопросы по проекту, связанной с этим теории и использованному технологическому стеку.
**В командах по 3-4 человека.**
**Срок 3 недели.**
