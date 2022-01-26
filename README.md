# Español

Este es un repositorio para crear rápidamente un entorno de trabajo con Moodle (Apache2, PHP-FPM con XDEBUG y Postgres) usando contenedores para cada uno sus principales componentes. El entorno de trabajo se crea y gestiona con Docker Compose.
Basado en el repositorio de [jobcespedes](https://github.com/jobcespedes/docker-compose-moodle#espa%C3%B1ol)

## Pasos rápidos para crear proyecto:
1. Tener Docker. Ver como instalar [Docker](https://docs.docker.com/install/)
2. Tener Docker Compose. Ver como instalar [Docker Compose](https://docs.docker.com/compose/install/)
3. Descargar este repo y acceder a él: ```git clone https://github.com/jobcespedes/docker-compose-moodle.git && cd docker-compose-moodle```
4. Copiar repositorio de código de Moodle: ```git clone --branch MOODLE_35_STABLE --depth 1 git://github.com/moodle/moodle html```
5. Desplegar con: ```docker-compose up -d```

## Variables de entorno
La siguiente tabla contiene las variables utilizadas en el archivo [**.env**](.env) para Docker Compose. Los valores por defecto funcionan para una configuración inicial. Cámbielos de ser necesario.

| Variable | Valor por defecto | Utilidad |
| :--- |:--- |:--- |
| **REPO_FOLDER** | html | Ruta relativa para el código de Moodle |
| **DOCUMENT_ROOT** | /var/www/html | Punto de montaje para **REPO_FOLDER** dentro de contenedores |
| **MY_TZ** | America/Costa_Rica | Zona horaria para los contenedores |
| **PG_LOCALE** | es_CR | Configuración de lugar |
| **PG_PORT** | 5432 | Puerto de base de datos postgres a publicar  |
| **POSTGRES_DB** | moodle | Nombre de la base de datos postgres de Moodle |
| **POSTGRES_USER** | user | Nombre de usuario de la base de datos postgres de Moodle |
| **POSTGRES_PASSWORD** | password | Contraseña de la base de datos postgres de Moodle |
| **PHP_SOCKET** | 9000 | Socket para conectar apache2 con php-fpm |
| **ALIAS_DOMAIN** | localhost | Alias del Dominio |
| **WWW_PORT** | 80 | Puerto web a publicar |
| **MOODLE_DATA** | /var/moodledata | Carpeta de datos de Moodle a montar en los contenedores |
| **WWWROOT** | localhost | Para de nombre de host en la url de config.php de Moodle |

## Estructura de Docker Compose
A continuación se incluye una tabla que resume la estructura del archivo de Docker Compose:

| Componente | Tipo | Responsabilidad | Contenido | Configuración |
| :--- |:--- | :--- | :---| :---|
| **apache2** | Contenedor | Servidor web | Debian9, Apache2 | El mínimo de módulos de [Apache2](http://dockerfile.readthedocs.io/en/latest/content/DockerImages/dockerfiles/php-apache.html#web-environment-variables) |
| **cron** | Contenedor|Tarea de cron de Moodle | Debian9, Cron | Frecuencia de ejecución de tarea [cron de Moodle](https://docs.moodle.org/35/es/Cron) |
| **php-fpm** | Contenedor | Interprete y manejador de procesos para PHP | Debian9, PHP-FPM, XDEBUG | Modulos de php y paquetes adicionales para Moodle  |
| **postgres** | Contenedor | Gestor de base de datos  | Debian9, Postgres | [Usuario y base de datos](https://hub.docker.com/_/postgres/) |
| **db_dumps** | Volumen | Restaurar una base de datos inicial | Archivos de respaldo de base de datos. | Para restaurar al iniciar si se comienza con directorio de datos vacío. El nombre del archivo de respaldo a utilizar debe ser "dump-init.dump" |
| **moodledata** | Volumen | Almacén de datos de Moodle | Archivos generados por Moodle | [Moodle data dir ](https://docs.moodle.org/all/es/Gu%C3%ADa_r%C3%A1pida_de_Instalaci%C3%B3n#Crea_el_directorio_de_datos) |
| ***REPO_FOLDER*** | Volumen | Código de aplicación  | Código de Moodle  | Por defecto es './html' (ver archivo .env) |

## Gestión del proyecto con Docker Compose
> **Dentro de la carpeta del proyecto**
1. Correr proyecto
``` bash
docker-compose up -d
# Nombrar el proyecto diferente a la carpeta:
# docker-compose -p mi-proy up -d
```
2. Detener el proyecto
``` bash
docker-compose stop
# docker-compose stop php-fpm
```
3. Iniciar el proyecto
``` bash
docker-compose start
# docker-compose start php-fpm
```
4. Eliminar proyecto
``` bash
docker-compose down
# Eliminar los volumenes también:
# docker-compose --volumes
# Eliminar con un nombre de proyecto especifico:
# docker-compose -p mi-proy down
```
5. Logs
``` bash
docker-compose logs
# docker-compose logs -f --tail="20" php-fpm
```

### XDEBUG
> Se utiliza el idekey `PHPTEST`

#### PHPStorm
Configuración para depurar con IDE PHPStorm:

1. Agregar server:
    * Settings -> Languages -> PHP -> Servers
    * Name: localhost
    * Host: localhost
    * Port: 80
    * Debugger: Xdebug
    * Use path mapping: checked
    * Absolute path on the server: /var/www/html

![Debug button](docs/img/phpstorm_add_server.gif)

2. Agregar PHP remote debug
    * Run / Debug Configurations -> PHP remote debug
    * Use server created in step #1 and set idekey `PHPTEST`

![Debug button](docs/img/phpstorm_add_remote_debug.gif)

3. Activar botón `Start listening for PHP Debug Connections` ![Debug button](docs/img/phpstorm_enable_debug.png)

### Depurar tareas de cron
Siga los pasos anteriores, establezca una interrupción y ejecuta en el la línea de comandos:
```bash
docker-compose exec php-fpm php admin/cli/cron.php
```
Se pueden ejecutar también [tareas específicas de cron](https://docs.moodle.org/all/es/Administraci%C3%B3n_por_l%C3%ADnea_de_comando#Trabajos_agendados)Por ejemplo:
```bash
docker-compose exec php-fpm php admin/tool/task/cli/schedule_task.php --execute='\core\task\cache_cleanup_task'
# Listar tareas
# docker-compose exec php-fpm php admin/tool/task/cli/schedule_task.php --list
```

### Pgadmin4
Pasos para usar pgadmin4
1. Ingresar a http://localhost:5050
2. En ```File -> Preferences -> Binary paths``` establecer en ```/usr/bin```
3. Agregar nuevo servidor:
    * Pestaña ```General```
        * Name: Un nombre para el servidor
    * Pestaña ```Connection```
        * Host name/address: ```postgres```
        * Host Username: ```user```
        * Host Password: ```password```
    * Guardar

## Respaldar y restaurar la base de datos
Es posible respaldar y restaurar la base de datos de la siguiente manera.
```bash
# Establecer vars de entorno
POSTGRES_USER=user
POSTGRES_DB=moodle
DB_DUMP_NAME=dump-init.$(date +"%Y%m%d%H%M%S").dump

# Respaldar
# -Fc  Formato personalizado para pg_restore
docker-compose exec postgres pg_dump -U ${POSTGRES_USER} ${POSTGRES_DB} -Fc -f /opt/db_dumps/${DB_DUMP_NAME}

# Restaurar
# -c  Limpia los objetos de la base de datos antes de recrearlos
# -C  Crea la base de datos antes de restaurarla
docker-compose exec postgres pg_restore -U ${POSTGRES_USER} -d postgres -c -C -O --role ${POSTGRES_USER} /opt/db_dumps/${DB_DUMP_NAME}
```

Se puede restaurar una base de datos, usando pg_dump (formato personalizado de Posgres), a la caperta 'db_dumps' y nombrando el archivo como dump-init.dump
> **IMPORTANTE**: Dependiendo del tamaño, la ejecución de este sql podría demorar la disponibilidad inicial de la base de datos.
