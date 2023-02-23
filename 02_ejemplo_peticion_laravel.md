# ¿Cómo procesa Laravel una petición?

1. Vamos a mostrar todas las entradas del blog en la página principal: https://blog.dockerbox.test/

2. Generar un modelo junto su la migración y el controlador de recursos asociado:

    ```bash
    php artisan make:model Entrada -mcr
    ```

3. Editar la migración y añadir las columnas:

    ```php
    // database/migrations/2023_..._create_entradas_table.php

    return new class extends Migration
    {
        public function up(): void
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
    
        public function down(): void
        {
            Schema::dropIfExists('entradas');
        }
    };
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

    ```php
    $entrada = new Entrada();
    $entrada->titulo = 'Segunda entrada';
    $entrada->texto = 'Esta es la segunda entrada del blog.';
    $entrada->fecha = now();
    $entrada->save();
    ```

   > Para salir de Tinker pulsa Ctrl+D.

6. Rutas:

    ```php
    // routes/web.php
    
    Route::get('/', [EntradaController::class, 'index']);
    ```

    ```bash
    php artisan route:list
    ```

   > Recuerda importar la clase EntradaController para que funcione correctamente.

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
    
        <h1 class="mt-3">Entradas</h1>
    
        <table class="table table-striped">
            <thead>
            <tr>
                <th>Título</th>
                <th>Texto</th>
                <th>Fecha</th>
            </tr>
            </thead>
            <tbody>
            @foreach($entradas as $entrada)
                <tr>
                    <td>{{ $entrada->titulo }}</td>
                    <td>{{ $entrada->texto }}</td>
                    <td>{{ $entrada->fecha }}</td>
                </tr>
            @endforeach
            </tbody>
            <tfoot></tfoot>
        </table>
    @endsection
    ```

9. Layout general de la página:

    ```blade
    {{-- resources/views/layouts/app.blade.php --}}
    
    <!doctype html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css" rel="stylesheet"
              integrity="sha384-rbsA2VBKQhggwzxH7pPCaAqO46MgnOM80zW1RWuH61DGLwZJEdK2Kadq2F9CUG65" crossorigin="anonymous">
        <title>Mi blog</title>
    </head>
    <body>
    <div class="container">
        @yield('content')
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/js/bootstrap.bundle.min.js"
            integrity="sha384-kenU1KFdBIe4zVF0s0G1M5b4hcpxyD9F7jL+jjXkk+Q2h455rYXK/7HAuoJl+0I4"
            crossorigin="anonymous"></script>
    </body>
    </html>
    ```
