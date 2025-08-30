# Сборка через self-hosted overleaf

Для компиляции можно использовать локальный инстанс Overleaf сервера.
Из плюсов - возможность организации совместной работы нескольких человек.
Из минусов - танцы с бубном, чтобы правильно сбилдить контейнер.

Репозиторий с контейнером overleaf - [https://github.com/overleaf/overleaf](https://github.com/overleaf/overleaf)
Инструкция сделана на основе [гиста](https://shihabkhan1.github.io/overleaf/stepbystep.html).
Тут будет краткое его содержание с дополнением.

## Инструкция по развертыванию

1. развертываем `compose` как обычно. Нужно удостовериться, что `*.js` файл инициализации монги корректно передается в `compose`. Также лучше перепроверить, что `volumes` ссылаются на текущую директорию, а не на `~`.
2. по умолчанию `sharelatex` image содержит голый латех, без пакетов и Times New Roman. Вариант 2: доставить пакеты в контейнер вручную, или воспользоваться готовым [image](https://hub.docker.com/r/prokudintema/sharelatex).
3. для использования через reverse proxy, необходимо настроить работу через веб-сокеты. Пример рабочей конфигурации:

```conf
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;

    server_name www.overleaf.compprehension.ru overleaf.compprehension.ru;

    ssl_certificate /etc/letsencrypt/live/compprehension.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/compprehension.ru/privkey.pem;
    keepalive_timeout 75;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # config to enable HSTS(HTTP Strict Transport Security) https://developer.mozilla.org/en-US/docs/Security/HTTP_Strict_Transport_Security
    # to avoid ssl stripping https://en.wikipedia.org/wiki/SSL_stripping#SSL_stripping	
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains;";

    server_tokens off;

    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    
    # gzip config
    gzip on;
    gzip_disable "msie6";
    gzip_comp_level 4;
    gzip_min_length 1100;
    gzip_buffers 16 8k;
    gzip_proxied any;
    gzip_types
        text/plain
        text/css
        text/js
        text/xml
        text/javascript
        application/javascript
        application/json
        application/xml
        application/rss+xml
        image/svg+xml;
        
    client_max_body_size 100M;
        
    location / {
        proxy_pass http://sharelatex:80;
        
        # Для работы веб-сокетов
        proxy_http_version 1.1;        
        proxy_read_timeout 86400;   
        proxy_max_temp_file_size 0;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        
        # reverse proxy
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Will add the user's ip to the request, some apps need this
        proxy_set_header X-Forwarded-Proto $scheme; # will forward the protocole i.e. http, https
        proxy_set_header X-Forwarded-Port $server_port; # Will forward the port 
        proxy_set_header X-Forwarded-Host $host;                    # !Important will forward the host address
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;                    # !Important will forward the host address
    }
}
```

4. для создания учётки администратора перейти на адрес `http://localhost:<port>/launchpad`

## Установка пакетов в образ sharelatex

1. запускаем `compose`

2. устанавливаем пакеты

```bash
# заходим в консоль sharelatex
docker exec -it sharelatex bash

# устанавливаем все латех пакеты
tlmgr install scheme-full
# тут может выскочить ошибка, что tlmgr: Local TeX Live (2024) is older than remote repository (2025) для фикса прогнать строчки ниже:
# wget https://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
# sh update-tlmgr-latest.sh -- --update
# tlmgr option repository https://mirror.ctan.org/systems/texlive/tlnet
# tlmgr update --self
# tlmgr update --all
# tlmgr install scheme-full
```

3. дополнительно нужно установить шрифты CMU + Times New Roman

```bash
# find CMU font paths
find /usr/local/texlive/ -name "cmuntt.otf"

# add & verify symlink for CMU
mkdir -p /usr/share/fonts/opentype/texlive
ln -s /usr/local/texlive/2024/texmf-dist/fonts/opentype/public/cm-unicode /usr/share/fonts/opentype/texlive/
ls -l /usr/share/fonts/opentype/texlive/cm-unicode

# install MS fonts
apt-get update; apt-get install -y ttf-mscorefonts-installer fontconfig
apt-get clean; rm -rf /var/lib/apt/lists/*

# refresh font cache
fc-cache -fv

# ensure fonts exist
fc-list | grep "CMU Typewriter"
fc-list | grep "Times"
```

4. дополнительно необходимо установить biber (он уже уходит в состав `tlmgr install scheme-full`, нужно просто прописать его в PATH)

```bash
# заходим в контейнер
docker exec -it sharelatex bash

# добавляем symlink
ln -s /usr/local/texlive/2024/bin/x86_64-linux/biber /usr/local/bin/biber

# проверка что все ок
biber --version
```

5. все изменения в контейнере необходимо закоммитить в image, чтобы корректно работало после пересоздания сервиса

```bash
# выходим из контейнера и коммитим изменения
docker commit sharelatex sharelatex/sharelatex:<commit-message>

# заходим в docker-compose и проставляем тэг <commit-message> в образ
# ...
# services:
#    sharelatex:
#        image: sharelatex/sharelatex:<commit-message>
# ...

# можно запушить готовый собранный образ в свой репоизиторий
docker login --username prokudintema
docker tag sharelatex/sharelatex:<commit-message> prokudintema/sharelatex:latest prokudintema/sharelatex:<YYYY-MM-DD>
docker push prokudintema/sharelatex:latest prokudintema/sharelatex:<YYYY-MM-DD>
```

## Про бэкапы

В официальной документации довольно хорошо [описан](https://github.com/overleaf/overleaf/wiki/Data-and-Backups) процесс бэкапов.
Там советуют бэкапить вообще все: монгу, редис и файлы с диска. Лично я для себя выбрал комбинацию бэкапов монги + фс, т.к. это минимальный набор, который гарантирует целостность данных после восстановления (пусть и возможно не самых свежих, т.к. в редисе хранится кэш, который периодически сбрасывается в монгу). Но для прям суперсерьезного сервиса с гарантиями нужно все же бэкапить и редиску.

Горячих бэкапов нет, поэтому нужно останавливать сервис перед началом процесса бэкапа. Это вызвано тем, что возможен рассинхрон между монгой и фс, если вдруг пользователь в процессе полезет что-то менять. Думаю, что в теории можно ограничиться переводом баз и фс в режим readonly, перед стартом бэкапов, и этого должно хватить. Но если первоочередной задачей бэкапов является сохранность содержимого проектов, а полноценная работоспособность сайта после отката не столь важна, то я не вижу никаких причин не использовать горячие бэкапы, даже с риском нарушения целостности. Лично я вижу, что оптимальным является комбинация частых горячих бэкапов (частые, но не гарантирующие 100% целостность), с холодными (редкие, но с гарантией целостности). В общем, тут все зависит от того, какие обязательства сайт на себя берет.

Откат из бэкапа довольно простой - перезатираем монгу её бэкапом, перезатираем файлы фс соответствующим бэкапом фс.
После восстановления фс, не забыть проставить `chown -R www-data:www-data <path_to_sharelatex_folder>` на восстановленную папку, иначе будут вылезать на сайте всякие прикольные ошибки странного характера.

## Пример docker-compose.yaml

Полное содержимое `docker-compose.yaml` (без `nginx`):

```yaml
services:
    sharelatex:
        restart: always
        # Server Pro users:
        # image: quay.io/sharelatex/sharelatex-pro
        image: prokudintema/sharelatex:latest
        container_name: sharelatex
        depends_on:
            mongo:
                condition: service_healthy
            redis:
                condition: service_started
        #ports:
        #    - 5080:80
        stop_grace_period: 60s
        networks:
            - internal-network
            - overleaf-network
        volumes:
            - ./data/sharelatex_data:/var/lib/overleaf
            ########################################################################
            ####  Server Pro: Uncomment the following line to mount the docker  ####
            ####             socket, required for Sibling Containers to work    ####
            ########################################################################
            # - /var/run/docker.sock:/var/run/docker.sock
        environment:

            OVERLEAF_APP_NAME: Overleaf Community Edition

            OVERLEAF_MONGO_URL: mongodb://mongo/sharelatex

            # Same property, unfortunately with different names in
            # different locations
            OVERLEAF_REDIS_HOST: redis
            REDIS_HOST: redis

            ENABLED_LINKED_FILE_TYPES: 'project_file,project_output_file'

            # Enables Thumbnail generation using ImageMagick
            ENABLE_CONVERSIONS: 'true'

            # Disables email confirmation requirement
            EMAIL_CONFIRMATION_DISABLED: 'true'

            ## Set for SSL via nginx-proxy
            #VIRTUAL_HOST: 103.112.212.22

            OVERLEAF_SITE_URL: https://overleaf.compprehension.ru
            # OVERLEAF_NAV_TITLE: Overleaf Community Edition
            # OVERLEAF_HEADER_IMAGE_URL: http://example.com/mylogo.png
            OVERLEAF_ADMIN_EMAIL: prokudintema@yandex.ru

            # OVERLEAF_LEFT_FOOTER: '[{"text": "Another page I want to link to can be found <a href=\"here\">here</a>"} ]'
            # OVERLEAF_RIGHT_FOOTER: '[{"text": "Hello I am on the Right"} ]'

            # OVERLEAF_EMAIL_FROM_ADDRESS: "hello@example.com"

            # OVERLEAF_EMAIL_AWS_SES_ACCESS_KEY_ID:
            # OVERLEAF_EMAIL_AWS_SES_SECRET_KEY:

            # OVERLEAF_EMAIL_SMTP_HOST: smtp.example.com
            # OVERLEAF_EMAIL_SMTP_PORT: 587
            # OVERLEAF_EMAIL_SMTP_SECURE: false
            # OVERLEAF_EMAIL_SMTP_USER:
            # OVERLEAF_EMAIL_SMTP_PASS:
            # OVERLEAF_EMAIL_SMTP_TLS_REJECT_UNAUTH: true
            # OVERLEAF_EMAIL_SMTP_IGNORE_TLS: false
            # OVERLEAF_EMAIL_SMTP_NAME: '127.0.0.1'
            # OVERLEAF_EMAIL_SMTP_LOGGER: true
            # OVERLEAF_CUSTOM_EMAIL_FOOTER: "This system is run by department x"

            # ENABLE_CRON_RESOURCE_DELETION: true

            ################
            ## Server Pro ##
            ################

            ## Sandboxed Compiles: https://github.com/overleaf/overleaf/wiki/Server-Pro:-Sandboxed-Compiles
            SANDBOXED_COMPILES: 'true'
            SANDBOXED_COMPILES_SIBLING_CONTAINERS: 'true'
            ### Bind-mount source for /var/lib/overleaf/data/compiles inside the container.
            SANDBOXED_COMPILES_HOST_DIR: '/home/user/sharelatex_data/data/compiles'

            ## Works with test LDAP server shown at bottom of docker compose
            # OVERLEAF_LDAP_URL: 'ldap://ldap:389'
            # OVERLEAF_LDAP_SEARCH_BASE: 'ou=people,dc=planetexpress,dc=com'
            # OVERLEAF_LDAP_SEARCH_FILTER: '(uid={{username}})'
            # OVERLEAF_LDAP_BIND_DN: 'cn=admin,dc=planetexpress,dc=com'
            # OVERLEAF_LDAP_BIND_CREDENTIALS: 'GoodNewsEveryone'
            # OVERLEAF_LDAP_EMAIL_ATT: 'mail'
            # OVERLEAF_LDAP_NAME_ATT: 'cn'
            # OVERLEAF_LDAP_LAST_NAME_ATT: 'sn'
            # OVERLEAF_LDAP_UPDATE_USER_DETAILS_ON_LOGIN: 'true'

            # OVERLEAF_TEMPLATES_USER_ID: "578773160210479700917ee5"
            # OVERLEAF_NEW_PROJECT_TEMPLATE_LINKS: '[ {"name":"All Templates","url":"/templates/all"}]'


            # OVERLEAF_PROXY_LEARN: "true"
            OVERLEAF_SECURE_COOKIE: true
            OVERLEAF_BEHIND_PROXY: true
        deploy:
          resources:
            limits:
              cpus: 4
              memory: 8G
            reservations:
              cpus: '0.1'
              memory: 100M

    mongo:
        restart: always
        image: mongo:6.0
        container_name: mongo
        command: '--replSet overleaf'
        networks:
            - internal-network
        volumes:
            - ./data/mongo_data:/data/db
            - ./server-ce/mongodb-init-replica-set.js:/docker-entrypoint-initdb.d/mongodb-init-replica-set.js
        environment:
          MONGO_INITDB_DATABASE: sharelatex
        extra_hosts:
          # Required when using the automatic database setup for initializing the replica set.
          # This override is not needed when running the setup after starting up mongo.
          - mongo:127.0.0.1
        healthcheck:
            test: echo 'db.stats().ok' | mongosh localhost:27017/test --quiet
            interval: 10s
            timeout: 10s
            retries: 5

    redis:
        restart: always
        image: redis:6.2
        container_name: redis
        volumes:
            - ./data/redis_data:/data
        networks:
            - internal-network

    # See https://github.com/jwilder/nginx-proxy for documentation on how to configure the nginx-proxy container,
    # and https://github.com/overleaf/overleaf/wiki/HTTPS-reverse-proxy-using-Nginx for an example of some recommended
    # settings. We recommend using a properly managed nginx instance outside of the Overleaf Server Pro setup,
    # but the example here can be used if you'd prefer to run everything with docker-compose

    # nginx-proxy:
    #     image: jwilder/nginx-proxy
    #     container_name: nginx-proxy
    #     ports:
    #       - "80:80"
    #       - "443:443"
    #     volumes:
    #       - /var/run/docker.sock:/tmp/docker.sock:ro
    #       - /home/overleaf/tmp:/etc/nginx/certs
networks:
    internal-network:
        driver: bridge
    overleaf-network:
        external: true
```
