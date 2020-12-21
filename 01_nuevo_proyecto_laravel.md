# Nuevo proyecto Laravel

## Prerrequisitos

Descargar, instalar y arrancar [Dockerbox](https://github.com/egibide/dockerbox).

## Nuevo proyecto

> :warning: Reemplazar `app` por el nombre del proyecto a partir de aquí.

### Generar un proyecto nuevo en la carpeta `www`

1. Abrir un terminal en la carpeta `dockerbox` y entrar en el workspace:

    ```bash
    make workspace
    ```

2. Crear el proyecto:

    ```bash
    composer create-project --prefer-dist laravel/laravel app
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
    cd www/app && git init && git add . && git commit -m "Initial commit" && cd ../..
    ```

### Configurar el nuevo sitio web como predeterminado

1. Ir a la carpeta `dockerbox/nginx` y editar el fichero `dockerbox.conf`:

    ```
    root /var/www/html/app/public;
    ```

2. Reiniciar los contenedores:

    ```bash
    make restart
    ```

3. Acceder al [sitio web](https://dockerbox.test).

## Crear la base de datos

1. Acceder a [phpMyAdmin](https://phpmyadmin.dockerbox.test) y crear el usuario `app/12345Abcde` y marcar la opción para generar la base de datos asociada, que se llamará como el usuario.

2. Editar el `.env` de la aplicación:

    ```
    DB_CONNECTION=mysql
    DB_HOST=mariadb
    DB_PORT=3306
    DB_DATABASE=app
    DB_USERNAME=app
    DB_PASSWORD=12345Abcde
    ```

## (Opcional) Abrir ficheros en PhpStorm desde las excepciones de Ignition

Editar el `.env` de la aplicación y establecer estas dos variables:

```
IGNITION_REMOTE_SITES_PATH=/var/www/html/app/
IGNITION_LOCAL_SITES_PATH=/Users/.../dockerbox/www/app/
```
    
Las dos rutas tienen que apuntar a la carpeta raiz del proyecto Laravel, la primera dentro del workspace y la segunda en el sistema de archivos local.

Más información en la documentación de [Flare](https://flareapp.io/docs/ignition-for-laravel/configuration).
    
## Utilidades

### Lanzar comandos en el proyecto (composer, artisan, npm...)

```
make workspace

cd app
```

Y después el comando que necesitemos. Por ejemplo:

```
php artisan tinker
```

o

```
php artisan make:model Tarea -mcr
```
