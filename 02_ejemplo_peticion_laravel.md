# ¿Cómo procesa Laravel una petición?

1. Vamos a mostrar todas las entradas del blog en la página principal: https://blog.dockerbox.test/

2. Generar un modelo junto su la migración y el controlador de recursos asociado:

    ```bash
    php artisan make:model Entrada -mcr
    ```

3. Editar la migración y añadir las columnas:

    ```php
    // database/migrations/2021_..._create_entradas_table.php

    class CreateEntradasTable extends Migration
    {
        public function up()
        {
            Schema::create('entradas', function (Blueprint $table) {
                $table->id();
                
                $table->string('titulo');
                $table->text('texto')->nullable();
                $table->dateTimeTz('fecha')->nullable();
                $table->boolean('visible')->nullable()->default(false);

                $table->timestamps();
            });
        }
    
        public function down()
        {
            Schema::dropIfExists('entradas');
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
    $entrada = new Entrada();
    $entrada->titulo = '¡Hola mundo!';
    $entrada->texto = 'Esta es la primera entrada del blog.';
    $entrada->fecha = now();
    $entrada->save();
    ```

6. Rutas:

    ```php
    // routes/web.php
    
    Route::get('/', [EntradaController::class, 'index']);
    ```

    ```bash
    php artisan route:list
    ```

7. Acción en el controlador:

    ```php
    // app/Http/Controllers/EntradaController.php
    
    public function index()
    {
        $entradas = Entrada::all();

        return view('entradas.index', compact('entradas'));
    }
    ```

8. Vista:

    ```blade
    {{-- resources/views/entradas/index.blade.php --}}
    
    @extends('layouts.app')
    
    @section('content')
    
        <h1>Entradas</h1>
    
        <table border="1">
            <tr>
                <th>Título</th>
                <th>Texto</th>
                <th>Fecha</th>
            </tr>
            @foreach($entradas as $entrada)
                <tr>
                    <td>{{ $entrada->titulo }}</td>
                    <td>{{ $entrada->texto }}</td>
                    <td>{{ $entrada->fecha }}</td>
                </tr>
            @endforeach
        </table>
    
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
    
        <title>Mi blog</title>
    </head>
    <body>
    
    @yield('content')
    
    </body>
    </html>
    ```
