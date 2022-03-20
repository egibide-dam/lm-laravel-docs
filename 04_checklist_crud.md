# Checklist para crear un CRUD completo

1. Crear el modelo:

    ```shell
    php artisan make:model Alumno -a
    ```

   > Nombre de la clase en singular y con notación `UpperCamelCase`.

2. Editar la migración y añadir las columnas a la tabla.

   > Nombres de las columnas en minúsculas con notación `snake_case`.

3. Editar las relaciones en el modelo.

4. Añadir los campos al `$fillable` del modelo.

    ```php
    protected $fillable = [
        'nombre', 'apellidos', 'email'
    ];
    ```

5. Editar el factory.

6. Editar el seeder.

7. Añadir el seeder al fichero `DatabaseSeeder`.

    ```php
    $this->call([
        AlumnoSeeder::class,
    ]);
    ```

8. Lanzar la migración:

    ```shell
    php artisan migrate
    ```

9. Añadir las rutas al fichero `routes/web.php`:

    ```php
    Route::resource('alumnos', AlumnoController::class);
    ```

10. Completar los métodos del controlador.

11. Editar la autorización y la validación en los ficheros request para los métodos store y update.

12. Crear las vistas (create, edit, index y show).

13. Crear el test:

    ```shell
    php artisan make:test AlumnoCRUDTest
    ```

14. Editar los test.

15. Lanzar los test:

    ```shell
    vendor/bin/phpunit tests/Feature/AlumnoCRUDTest.php
    ```
