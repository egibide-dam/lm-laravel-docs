# Nuevo proyecto Laravel

## Prerrequisitos

Descargar, instalar y arrancar [Dockerbox](https://github.com/ijaureguialzo/dockerbox).

## Nuevo proyecto

> :warning: Reemplazar `mi_aplicacion` por el nombre del proyecto a partir de aquí.

### Generar un proyecto nuevo en la carpeta `sites`

1. Abrir un terminal en la carpeta `dockerbox` y entrar en el workspace:

    ```bash
    make workspace
    ```

2. Crear el proyecto:

    ```bash
    composer create-project --prefer-dist laravel/laravel mi_aplicacion
    ```

3. Salir del `workspace`:

    ```bash
    exit
    ```

4. Editar el fichero `app/Providers/AppServiceProvider.php` y añadir al método `boot()`:

    ```php
    public function boot()
    {
        \URL::forceScheme('https');
    }    
    ```

5. (Opcional) Poner el proyecto bajo control de versiones:

    ```bash
    cd sites/mi_aplicacion && git init && git add . && git commit -m "Initial commit" && cd ../..
    ```

### Añadir el nuevo sitio web a Dockerbox

1. Editar como root el fichero `/etc/hosts` (en macOS y Linux) o
   en [Windows](https://www.adslzone.net/esenciales/windows-10/editar-archivo-host/) y añadir una nueva línea:

   ```text
   127.0.0.1    mi_aplicacion.dockerbox.test
   ```

2. Reiniciar los contenedores:

    ```bash
    make restart
    ```

3. Acceder al [nuevo sitio](https://mi_aplicacion.dockerbox.test).

## Crear la base de datos

1. Acceder a [phpMyAdmin](https://phpmyadmin.dockerbox.test) y crear el usuario `mi_aplicacion/12345Abcde` y marcar la
   opción para generar la base de datos asociada, que se llamará como el usuario.

2. Editar el `.env` de la aplicación:

    ```dotenv
    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=mi_aplicacion
    DB_USERNAME=mi_aplicacion
    DB_PASSWORD=12345Abcde
    ```

## (Opcional) Abrir ficheros en PhpStorm desde las excepciones de Ignition

Editar el `.env` de la aplicación y establecer estas dos variables:

```dotenv
IGNITION_REMOTE_SITES_PATH=/var/www/html/mi_aplicacion/
IGNITION_LOCAL_SITES_PATH=/Users/.../dockerbox/sites/mi_aplicacion/
```

Las dos rutas tienen que apuntar a la carpeta raiz del proyecto Laravel, la primera dentro del workspace y la segunda en
el sistema de archivos local.

Más información en la documentación de [Flare](https://flareapp.io/docs/ignition-for-laravel/configuration).

## Utilidades

### Lanzar comandos en el proyecto (composer, artisan, npm...)

```bash
make workspace

cd mi_aplicacion
```

Y después el comando que necesitemos. Por ejemplo:

```bash
php artisan tinker
```

o

```bash
php artisan make:model Tarea -mcr
```
