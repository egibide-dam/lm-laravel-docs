# Generar un CRUD completo

## Modelo

### Entrada

1. Generar el modelo y sus recursos asociados:

   > :book: [Eloquent: Getting Started](https://laravel.com/docs/8.x/eloquent)

    ```bash
    php artisan make:model Entrada -a
    ```

   Si ya tenemos creada la migración y el controlador con `-mcr`, podemos crear el _factory_ y el _seeder_ por separado:

    ```bash
    php artisan make:factory EntradaFactory
    php artisan make:seeder EntradaSeeder
    ```

2. Editar la migración:

   > :book: [Database: Migrations](https://laravel.com/docs/8.x/migrations)

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

3. Lanzar la migración:

    ```bash
    php artisan migrate
    ```

4. Editar el _factory_:

   > :book: [Defining Model Factories](https://laravel.com/docs/8.x/database-testing#defining-model-factories)
   > :book: [Faker](https://fakerphp.github.io)

    ```php
    // database/factories/EntradaFactory.php

    class EntradaFactory extends Factory
    {
        protected $model = Entrada::class;
    
        public function definition()
        {
            return [
                'titulo' => $this->faker->sentence(3),
                'texto' => $this->faker->text(100),
                'fecha' => $this->faker->dateTime(),
                'visible' => $this->faker->boolean(),
            ];
        }
    }
    ```

5. Editar el _seeder_:

   > :book: [Database: Seeding](https://laravel.com/docs/8.x/seeding)

    ```php
    // database/seeds/EntradaSeeder.php
    
    class EntradaSeeder extends Seeder
    {
        public function run()
        {
            Entrada::factory(10)->create();
        }
    }
    ```

6. Activar el _seeder_:

    ```php
    // database/seeds/DatabaseSeeder.php
    
    class DatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->call([
                EntradaSeeder::class,
            ]);
        }
    }
    ```

7. Lanzar el _seeder_:

    ```bash
    php artisan db:seed
    ```

   Si no reconoce el _seeder_:

    ```bash
    composer dump-autoload
    ```

8. Recrear la base de datos completa:

   > :warning: Atención, borra la base de datos y la vuelve a crear.

    ```bash
    php artisan migrate:fresh --seed
    ```

9. Añadir los campos _fillable_ y _casts_:

   > :book: [Mass Assignment](https://laravel.com/docs/8.x/eloquent#mass-assignment)
   > :book: [Eloquent: Mutators & Casting](https://laravel.com/docs/8.x/eloquent-mutators)

    ```php
    // app/Models/Entrada.php
    
    class Entrada extends Model
    {
        use HasFactory;
    
        protected $fillable = [
            'titulo', 'texto', 'fecha', 'visible'
        ];

        protected $casts = [
            'created_at' => 'datetime',
            'updated_at' => 'datetime',
            'fecha' => 'datetime',
        ];   
    }
    ```

### Comentario

1. Crear el modelo:

    ```bash
    php artisan make:model Comentario -a
    ```

2. Migración:

    ```php
    // database/migrations/2021_..._create_comentarios_table.php
    
    public function up()
    {
        Schema::create('comentarios', function (Blueprint $table) {
            $table->id();

            $table->string('email');
            $table->text('texto')->nullable();
            $table->dateTimeTz('fecha')->nullable();
            $table->boolean('visible')->nullable()->default(false);

            $table->timestamps();
        });
    }
    ```

3. Factory:

    ```php
    // database/factories/ComentarioFactory.php

    public function definition()
    {
        return [
            'email' => $this->faker->email(),
            'texto' => $this->faker->text(100),
            'fecha' => $this->faker->dateTime(),
            'visible' => $this->faker->boolean(),
        ];
    }
    ```

4. Seeder:

    ```php
    // database/seeds/ComentarioSeeder.php
    
    class ComentarioSeeder extends Seeder
    {
        public function run()
        {
            Comentario::factory(10)->create();
        }
    }
    ```

5. Activar el seeder:

    ```php
    // database/seeds/DatabaseSeeder.php
    
    class DatabaseSeeder extends Seeder
    {
        public function run()
        {
            $this->call([
                EntradaSeeder::class,
                ComentarioSeeder::class,
            ]);
        }
    }
    ```

6. Añadir los campos _fillable_ y _casts_ al modelo:

    ```php
    // app/Models/Comentario.php
    
    class Comentario extends Model
    {
        use HasFactory;
    
        protected $fillable = [
            'email', 'texto', 'fecha', 'publicado'
        ];
    
        protected $casts = [
            'created_at' => 'datetime',
            'updated_at' => 'datetime',
            'fecha' => 'datetime',
        ];
    }
    ```

### Relación 1→N

> :book: [Eloquent: Relationships](https://laravel.com/docs/8.x/eloquent-relationships)
> :book: [Eloquent Relationships Cheat Sheet](https://hackernoon.com/eloquent-relationships-cheat-sheet-5155498c209)
> :book: [Creating Columns](https://laravel.com/docs/8.x/migrations#creating-columns)

1. Añadir las _foreign keys_ a las tablas:

    ```bash
    php artisan make:migration add_entrada_id_to_comentarios_table
    ```

    ```php
    // database/migrations/2021_..._add_entrada_id_to_comentarios_table.php
    
    public function up()
    {
        Schema::table('comentarios', function (Blueprint $table) {

            // Sintaxis abreviada, utilizando las convenciones del ORM
            $table->foreignId('entrada_id')->constrained()->onDelete('cascade');

            // Sintaxis completa, personalizable
            //$table->unsignedBigInteger('entrada_id');
            //$table->foreign('entrada_id')->references('id')->on('entradas')->onDelete('cascade');
        });
    }
    ```

    ```bash
    php artisan migrate
    ```

2. Definir las relaciones entre entidades:

   Lado 1:

    ```php
    // app/Models/Entrada.php
    
    class Entrada extends Model
    {
        // ...
    
        public function comentarios()
        {
            return $this->hasMany(Comentario::class);
        }
    }
    ```

   Lado N:

    ```php
    // app/Models/Comentario.php
    
    class Comentario extends Model
    {
        // ...
    
        public function entrada()
        {
            return $this->belongsTo(Entrada::class);
        }
    } 
    ```

3. Modificar el seeder:

    ```php
    // database/seeds/ComentarioSeeder.php
    
    class ComentarioSeeder extends Seeder
    {
        public function run()
        {
            Comentario::factory(10)->create([
                'entrada_id' => 1
            ]);
        }
    }
    ```

## Rutas

> :book: [Routing](https://laravel.com/docs/8.x/routing)

1. Definir las rutas:

    ```php
    // routes/web.php
    
    Route::resource('entradas', EntradaController::class);
    Route::resource('comentarios', ComentarioController::class);
    ```

2. Ver las rutas:

    ```bash
    php artisan route:list
    ```

## Controlador

> :book: [Controllers](https://laravel.com/docs/8.x/controllers)
> :book: [Validation](https://laravel.com/docs/8.x/validation)

```php
// app/Http/Controllers/EntradaController.php

class EntradaController extends Controller
{
    public function index()
    {
        $entradas = Entrada::all();

        return view('entradas.index', compact('entradas'));
    }

    public function create()
    {
        return view('entradas.create');
    }

    public function store(Request $request)
    {
        $this->validate($request, [
            'titulo' => 'required',
        ]);

        Entrada::create([
            'titulo' => request('titulo'),
            'texto' => request('texto'),
            'fecha' => request('fecha'),
            'visible' => $request->has('visible'),
        ]);

        return redirect(route('entradas.index'));
    }

    public function show(Entrada $entrada)
    {
        return view('entradas.show', compact('entrada'));
    }

    public function edit(Entrada $entrada)
    {
        return view('entradas.edit', compact('entrada'));
    }

    public function update(Request $request, Entrada $entrada)
    {
        $this->validate($request, [
            'titulo' => 'required',
        ]);

        $entrada->update([
            'titulo' => request('titulo'),
            'texto' => request('texto'),
            'fecha' => request('fecha'),
            'visible' => $request->has('visible'),
        ]);

        return redirect(route('entradas.index'));
    }

    public function destroy(Entrada $entrada)
    {
        $entrada->delete();

        return back();
    }
}
```

## Vistas

> :book: [Views](https://laravel.com/docs/8.x/views)

```blade
{{-- resources/views/layouts/app.blade.php --}}

<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/css/bootstrap.min.css" rel="stylesheet"
          integrity="sha384-EVSTQN3/azprG1Anm3QDgpJLIm9Nao0Yz1ztcQTwFspd3yD65VohhpuuCOmLASjC" crossorigin="anonymous">
    <title>Mi blog</title>
</head>
<body>
<div class="container">
    @yield('content')
</div>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.2/dist/js/bootstrap.bundle.min.js"
        integrity="sha384-MrcW6ZMFYlzcLA8Nl+NtUVF0sA7MsXsP1UyJoMp4YLEuNSfAP+JcXn/tWtIaxVXM"
        crossorigin="anonymous"></script>
</body>
</html>
```

> :book: [CSRF Protection](https://laravel.com/docs/8.x/csrf)

```blade
{{-- resources/views/entradas/index.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1 class="mt-3">Entradas</h1>

    <table class="table table-striped">
        <thead>
        <tr>
            <th>Título</th>
            <th>Fecha</th>
            <th>Visible</th>
            <th colspan="2">Acciones</th>
        </tr>
        </thead>
        <tbody class="align-middle">
        @foreach($entradas as $entrada)
            <tr>
                <td><a href="{{ route('entradas.show', $entrada->id) }}">{{ $entrada->titulo }}</a></td>
                <td>{{ $entrada->fecha }}</td>
                <td>{{ $entrada->visible ? 'Sí' : 'No' }}</td>
                <td><a class="btn btn-secondary" href="{{ route('entradas.edit', $entrada->id) }}">Editar</a></td>
                <td>
                    <form action="{{ route('entradas.destroy', $entrada->id) }}" method="POST"
                          onsubmit="return confirm('¿Estás seguro?');">
                        @csrf
                        @method('DELETE')
                        <input class="btn btn-danger" type="submit" name="borrar" value="Borrar"/>
                    </form>
                </td>
            </tr>
        @endforeach
        </tbody>
        <tfoot></tfoot>
    </table>

    <a class="btn btn-primary mt-3" href="{{ route('entradas.create') }}">Nueva entrada</a>

@endsection
```

```blade
{{-- resources/views/entradas/show.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1 class="mt-3">{{ $entrada->titulo }}</h1>

    <table class="table table-striped">
        <tbody class="align-middle">
        <tr>
            <th>Texto</th>
            <td>{{ $entrada->texto }}</td>
        </tr>
        <tr>
            <th>Fecha</th>
            <td>{{ $entrada->fecha }}</td>
        </tr>
        <tr>
            <th>Visible</th>
            <td>{{ $entrada->visible ? 'Sí' : 'No' }}</td>
        </tr>
        </tbody>
    </table>

    <a class="btn btn-secondary mt-3" href="{{ route('entradas.index') }}">Volver</a>

@endsection
```

```blade
{{-- resources/views/entradas/create.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1 class="mt-3">Nueva entrada</h1>

    <form action="{{ route('entradas.store') }}" method="POST">
        @csrf
        <div class="row mb-3">
            <label class="col-2 form-label">Título: </label>
            <div class="col-10">
                <input class="form-control" type="text" name="titulo"/>
                <span class="text-danger">{{ $errors->first('titulo') }}</span>
            </div>
        </div>
        <div class="row mb-3">
            <label class="col-2 form-label">Texto: </label>
            <div class="col-10">
                <textarea class="form-control" name="texto"></textarea>
            </div>
        </div>
        <div class="row mb-3">
            <label class="col-2 form-label">Fecha: </label>
            <div class="col-10">
                <input class="form-control" type="datetime-local" name="fecha" value="{{ now() }}"/>
            </div>
        </div>
        <div class="row mb-3">
            <label class="col-2 form-label">Visible: </label>
            <div class="col-10">
                <input class="form-check-input" type="checkbox" name="visible" checked/>
            </div>
        </div>
        <input class="btn btn-primary" type="submit" name="guardar" value="Guardar"/>
        <a class="link-secondary ms-2" href="{{ route('entradas.index') }}">Cancelar</a>
    </form>

@endsection
```

```blade
{{-- resources/views/entradas/edit.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1 class="mt-3">Editar entrada</h1>

    <form action="{{ route('entradas.update', $entrada->id) }}" method="POST">
        @csrf
        @method('PUT')
        <div class="row mb-3">
            <label class="col-2 form-label">Título: </label>
            <div class="col-10">
                <input class="form-control" type="text" name="titulo" value="{{ $entrada->titulo }}"/>
                <span class="text-danger">{{ $errors->first('titulo') }}</span>
            </div>
        </div>
        <div class="row mb-3">
            <label class="col-2 form-label">Texto: </label>
            <div class="col-10">
                <textarea class="form-control" name="texto">{{ $entrada->texto }}</textarea>
            </div>
        </div>
        <div class="row mb-3">
            <label class="col-2 form-label">Fecha: </label>
            <div class="col-10">
                <input class="form-control" type="datetime-local" name="fecha" value="{{ $entrada->fecha ?: now() }}"/>
            </div>
        </div>
        <div class="row mb-3">
            <label class="col-2 form-label">Visible: </label>
            <div class="col-10">
                <input class="form-check-input" type="checkbox"
                       name="visible" {{ $entrada->visible ? 'checked' : '' }}/>
            </div>
        </div>
        <input class="btn btn-primary" type="submit" name="guardar" value="Guardar"/>
        <a class="link-secondary ms-2" href="{{ route('entradas.index') }}">Cancelar</a>
    </form>

@endsection
```

## Tests

> :book: [Testing: Getting Started](https://laravel.com/docs/8.x/testing)
> :book: [Database Testing](https://laravel.com/docs/8.x/database-testing)

1. Crear la clase de test:

    ```bash
    php artisan make:test EntradaTest
    ```

2. Escribir los tests:

   Al escribir los tests seguiremos el orden "Given→When→Then".

    ```php
    // tests/Feature/EntradaTest.php
    
    class EntradaTest extends TestCase
    {
        use RefreshDatabase;
    
        public function testIndex()
        {
            // Given
            $entrada = Entrada::factory()->create();
    
            // When
            $response = $this->get(route('entradas.index'));
    
            // Then
            $response->assertSee($entrada->titulo);
        }
    
        /*
        public function testNotAuthNotIndex()
        {
            // Given
            // When
            // Then
            $this->get(route('entradas.index'))
                ->assertRedirect(route('login'));
        }
        */
    
        public function testCreate()
        {
            // Given
            // When
            $response = $this->get(route('entradas.create'));
    
            // Then
            $response->assertSeeInOrder(['Nueva entrada', 'Guardar']);
        }
    
        public function testStore()
        {
            // Given
            $entrada = Entrada::factory()->make();
    
            // When
            $this->post(route('entradas.store'), $entrada->toArray());
    
            // Then
            $this->assertEquals(1, Entrada::all()->count());
        }
    
        public function testStoreRequiresTitulo()
        {
            // Given
            $entrada = Entrada::factory()->make(['titulo' => null]);
    
            // When
            // Then
            $this->post(route('entradas.store'), $entrada->toArray())
                ->assertSessionHasErrors('titulo');
        }
    
        public function testShow()
        {
            // Given
            $entrada = Entrada::factory()->create();
    
            // When
            $response = $this->get(route('entradas.show', $entrada));
    
            // Then
            $response->assertSee($entrada->titulo);
        }
    
        public function testEdit()
        {
            // Given
            $entrada = Entrada::factory()->create();
    
            // When
            $response = $this->get(route('entradas.edit', $entrada), $entrada->toArray());
    
            // Then
            $response->assertSeeInOrder([$entrada->titulo, 'Guardar']);
        }
    
        public function testUpdate()
        {
            // Given
            $entrada = Entrada::factory()->create();
            $entrada->titulo = "Actualizada";
    
            // When
            $this->put(route('entradas.update', $entrada), $entrada->toArray());
    
            // Then
            $this->assertDatabaseHas('entradas', ['id' => $entrada->id, 'titulo' => $entrada->titulo]);
        }
    
        public function testDelete()
        {
            // Given
            $entrada = Entrada::factory()->create();
    
            // When
            $this->delete(route('entradas.destroy', $entrada));
    
            // Then
            $this->assertDatabaseMissing('entradas', $entrada->toArray());
        }
    } 
    ```

3. Lanzar los tests:

    ```bash
    vendor/bin/phpunit tests/Feature/EntradaTest.php
    ```
