# 2/2 Project setup
- Docker configureren
  - php
  - nginx
  - mariadb
  - phpmyadmin
- Environment instellen
- Project opstarten
- Commit in git

## Docker configureren
Wij werken weer met docker containers voor de web based applicatie.
In het document `docker-compose.yml` gaan wij de structuur aanmaken van dit project.
1. Maak een onderdeel `services` aan
2. Binnen de services gaan wij verschillende services toevoegen. _Let hierbij op het inspringen van de code._

### php
Maak een php service aan:
```yaml
  # PHP FPM Service
  php:
    image: wodby/php:latest
    volumes:
      - './:/var/www/html'
    depends_on:
      - mariadb
```

### nginx
Maak een nginx proxy service aan:
```yaml
  # Nginx Service
  nginx:
    image: nginx:latest
    ports:
      - 88:80
    links:
      - php
    volumes:
      - './:/var/www/html'
      - './docker/nginx:/etc/nginx/conf.d'
    depends_on:
      - php
```

### mariadb
Maak een MariaDB database service aan:
```yaml
  # MariaDB Service
  mariadb:
    image: mariadb:latest
    environment:
      MYSQL_DATABASE: '${DB_NAME}'
      MYSQL_USER: '${DB_USERNAME}'
      MYSQL_PASSWORD: '${DB_PASSWORD}'
      MYSQL_ROOT_PASSWORD: '${DB_ROOT_PASSWORD}'
    ports:
      - 3308:3306
```

### PhpMyAdmin
Om eenvoudig de database te kunnen beheren maken wij gebruiken van een phpmyadmin service. Maar je kunt ook via poort 3308 verbinden met een externe tool zoals heidisql of sequelAce.<br>
Gebruik de root ( admin user ) gegevens om in te loggen zodat je straks meer rechten hebt
```yaml
  # phpmyadmin in an external image
  phpmyadmin:
    image: phpmyadmin:latest
    environment:
      PMA_HOST: '${DB_HOST}'
      PMA_USER: '${DB_USERNAME}'
      PMA_PASSWORD: '${DB_PASSWORD}'
    ports:
      - "1088:80"
```


## nginx conf
Nu je de docker klaar hebt staan kun je de nginx.conf instellen. Plaats daarvoor deze code in de /docker/nginx/nginx.conf
```apacheconf
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    
    server_name localhost;

    root /var/www/html/public;
    index index.php index.html;

    # Support Yii2 pretty URL routing
    location / {
            try_files $uri $uri/ =404;
            if (!-e $request_filename){
                    rewrite ^/(.*) /index.php?r=$1 last;
            }
    }

    location ~* \.php$ {
        fastcgi_pass php:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    }

    # Prevent additional headers like TRACE, DELETE, PUSH
    if ($request_method !~ ^(GET|HEAD|POST)$ )
        {
            return 405;
        }
}
```

## Environment instellen via een .env file
1. Maak een voorbeeld environments file aan met de naam: `.env.example`  
2. Plaats hierin de variabele die je straks gaat gebruiken:  
```dotenv
DB_HOST=mariadb
DB_PORT=3306
DB_SCHEMA=
DB_USERNAME=
DB_PASSWORD=
DB_ROOT_PASSWORD=

HOSTNAME=http://localhost:88/
SOURCE_ROOT=/var/www/html/source/
```
3. Kopieer _(Dupliceer)_ dit bestand en noem het: `.env`
4. Vul de ontbrekende waardes in deze `.env` file

## Start het project
Nu gaan wij het project initialiseren.
1. Zorg dat DockerDesktop aan staat
2. Open de terminal of commandline en zorg dat je in de project folder zit waar ook het docker-compose.yml bestand staat.
3. Start docker: `docker-compose up -d`
4. Open je project in de browser en zie een wit scherm omdat de /public/index.php leeg is: <br>
[http://localhost:88](http://localhost:88)
5. Je hebt nu een werkend en gestructureerd project

## Commit
Zorg ervoor dat je alle wijzigingen commit
