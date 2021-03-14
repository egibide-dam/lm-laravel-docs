# Cómo procesa Laravel una petición

1. Ruta: https://dockerbox.test/tareas

2. Generar un modelo y sus recursos asociados:

    ```bash
    php artisan make:model Tarea -mcr
    ```

3. Editar la migración y añadir las columnas:

    ```php
    // database/migrations/2020_..._create_tareas_table.php
    
    class CreateTareasTable extends Migration
    {
        public function up()
        {
            Schema::create('tareas', function (Blueprint $table) {
                $table->bigIncrements('id');
                $table->string('titulo');
                $table->boolean('completada')->default(false);
                $table->timestamps();
            });
        }
    
        public function down()
        {
            Schema::dropIfExists('tareas');
        }
    }
    ```

4. Ejecutar las migraciones (o revertirlas):

    ```bash
    php artisan migrate
    ```

    ```bash
    php artisan migrate:rollback
    ```

5. Creando objetos con Tinker:

    ```bash
    php artisan tinker
    ```

    ```php
    $tarea = new Tarea();
    $tarea->titulo = '¡Hola mundo!';
    $tarea->save();
    ```

6. Rutas:

    ```php
    // routes/web.php
    
    Route::get('/tareas', 'TareaController@index');
    ```

    ```bash
    php artisan route:list
    ```

7. Acción en el controlador:

    ```php
    // app/Http/Controllers/TareaController.php
    
    public function index()
    {
        $tareas = Tarea::all();
    
        return view('tareas.index', compact('tareas'));
    }
    ```

8. Vista:

    ```blade
    {{-- resources/views/tareas/index.blade.php --}}
    
    @extends('layouts.app')
    
    @section('content')

        <h1>Tareas</h1>    
        <ul>
            @foreach($tareas as $tarea)
                <li>{{ $tarea->titulo }}</li>
            @endforeach
        </ul>
    
    @endsection 
    ```

8. Layout general de la página:

    ```blade
    {{-- resources/views/layouts/app.blade.php --}}
    
    <!doctype html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
    
        <title>Tareas</title>
    </head>
    <body>
    
    @yield('content')
    
    </body>
    </html>
    ```
