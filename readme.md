# moodle-docker

This repository contains a Docker configuration aimed at Moodle developers. It includes

* poppler-utils
* PHP 8.0
* composer
* PHP Codesniffer
* Default Codesniffer Moodle standard

## Usage

1. Edit the volume mounts for moodle and moodledata in `docker-compose.yml` for the `php-apache` service. 

```
volumes:
    - /path/to/moodle/www:/var/www/html:cached
    - /path/to/moodledata:/var/www/moodledata:cached
```

2. Run the docker strategy.

```shell
docker-compose up -d
```

3. Run `install.php` from the running `php-apache` container. You can run `docker ps` to list your containers. In my case, the `php-apache` container is named `moodle-docker_php-apache_1`. Replace any of the variables as needed.

```shell
docker exec -it moodle-docker_php-apache_1 bash
```

```shell
root@container:/var/www/html# php admin/cli/install.php \
  --agree-license \
  --dataroot=/var/www/moodledata \
  --wwwroot=http://localhost:8080 \
  --dbtype=mariadb --dbhost=localhost \
  --dbname=moodle \
  --dbuser=root --dbpass=moodle \
  --fullname=Moodle \
  --shortname=moodle \
  --adminuser=moodle \
  --adminpass=moodle \
  --non-interactive
```