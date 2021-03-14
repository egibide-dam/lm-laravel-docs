# Generar un CRUD completo

## Modelo

### Entrada

1. Generar el modelo y sus recursos asociados:

    ```bash
    php artisan make:model Entrada -a
    ```

    Si ya tenemos creada la migración y el controlador con `-mcr`, podemos crear el _factory_ y el _seeder_ por separado:

    ```bash
    php artisan make:factory EntradaFactory
    php artisan make:seeder EntradaSeeder
    ```

2. Editar la migración:

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

    ```php
    // app/Models/Entrada.php
    
    class Entrada extends Model
    {
        use HasFactory;
    
        protected $fillable = [
            'titulo', 'texto', 'fecha', 'visible'
        ];
    
        protected $casts = [
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
    // database/migrations/2020_..._create_comentarios_table.php
    
    public function up()
    {
        Schema::create('comentarios', function (Blueprint $table) {
            $table->bigIncrements('id');
    
            $table->string('email');
            $table->text('texto')->nullable();
            $table->dateTimeTz('fecha')->nullable();
            $table->boolean('publicado')->nullable()->default(false);
    
            $table->timestamps();
        });
    }
    ```

3. Factory:

    ```php
    // database/factories/ComentarioFactory.php
    
    $factory->define(Comentario::class, function (Faker $faker) {
        return [
            'email' => $faker->email,
            'texto' => $faker->text(200),
            'fecha' => now(),
            'publicado' => $faker->boolean(30),
        ];
    });
    ```

4. Seeder:

    ```php
    // database/seeds/ComentarioSeeder.php
    
    class ComentarioSeeder extends Seeder
    {
        public function run()
        {
            $entrada = Entrada::find(1);
    
            for ($i = 0; $i < 5; $i++) {
                factory(Comentario::class)->create([
                    'entrada_id' => $entrada->id
                ]);
            }
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
            $this->call(EntradaSeeder::class);
            $this->call(ComentarioSeeder::class);
        }
    }
    ```

6. Añadir los campos _fillable_ y _dates_:

    ```php
    // app/Comentario.php
    
    class Comentario extends Model
    {
        protected $fillable = [
            'email', 'texto', 'fecha', 'publicado'
        ];
    
        protected $dates = [
            'created_at', 'updated_at', 'fecha'
        ];
    }
    ```

### Relación 1→N

1. Añadir las _foreign keys_ a las tablas:

    ```php
    // database/migrations/2020_..._create_comentarios_table.php
    
    public function up()
    {
        Schema::create('comentarios', function (Blueprint $table) {
            $table->bigIncrements('id');
    
            $table->string('email');
            $table->text('texto')->nullable();
            $table->dateTimeTz('fecha')->nullable();
            $table->boolean('publicado')->nullable()->default(false);
    
            $table->bigInteger('entrada_id')->unsigned()->index();
            $table->foreign('entrada_id')->references('id')->on('entradas')->onDelete('cascade');
    
            $table->timestamps();
        });
    }
    ```

2. Definir las relaciones entre entidades:

   Lado 1:

    ```php
    // app/Entrada.php
    
    class Entrada extends Model
    {
        ...
    
        public function comentarios()
        {
            return $this->hasMany(Comentario::class);
        }
    }
    ```

   Lado N:

    ```php
    // app/Comentario.php
    
    class Comentario extends Model
    {
        ...
    
        public function entrada()
        {
            return $this->belongsTo(Entrada::class);
        }
    } 
    ```

## Rutas

1. Definir las rutas:

    ```php
    // routes/web.php
    
    Route::resource('entradas', 'EntradaController');
    Route::resource('comentarios', 'ComentarioController');
    ```

2. Ver las rutas:

    ```bash
    php artisan route:list
    ```

## Controlador

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
            'publicada' => $request->has('publicada'),
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
            'publicada' => $request->has('publicada'),
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

```blade
{{-- resources/views/layouts/app.blade.php --}}

<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <title>Blog</title>
</head>
<body>

@yield('content')

</body>
</html>
```

```blade
{{-- resources/views/entradas/index.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Entradas</h1>

    <table border="1">
        <thead>
        <tr>
            <th>Título</th>
            <th>Fecha</th>
            <th>Publicada</th>
            <th colspan="2">Acciones</th>
        </tr>
        </thead>
        <tbody>
        @foreach($entradas as $entrada)
            <tr>
                <td><a href="{{ route('entradas.show', $entrada->id) }}">{{ $entrada->titulo }}</a></td>
                <td>{{ $entrada->fecha }}</td>
                <td>{{ $entrada->publicada ? 'Sí' : 'No' }}</td>
                <td><a href="{{ route('entradas.edit', $entrada->id) }}">Editar</a></td>
                <td>
                    <form action="{{ route('entradas.destroy', $entrada->id) }}" method="POST">
                        @csrf
                        @method('DELETE')
                        <input type="submit" name="borrar" value="Borrar"/>
                    </form>
                </td>
            </tr>
        @endforeach
        </tbody>
    </table>

    <p>
        <a href="{{ route('entradas.create') }}">Nueva entrada</a>
    </p>

@endsection
```

```blade
{{-- resources/views/entradas/show.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>{{ $entrada->titulo }}</h1>

    <table border="1">
        <tbody>
        <tr>
            <th>Texto</th>
            <td>{{ $entrada->texto }}</td>
        </tr>
        <tr>
            <th>Fecha</th>
            <td>{{ $entrada->fecha }}</td>
        </tr>
        <tr>
            <th>Publicada</th>
            <td>{{ $entrada->publicada ? 'Sí' : 'No' }}</td>
        </tr>
        </tbody>
    </table>

    <p>
        <a href="{{ route('entradas.index') }}">Volver</a>
    </p>

@endsection
```

```blade
{{-- resources/views/entradas/create.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Nueva entrada</h1>

    <form action="{{ route('entradas.store') }}" method="POST">
        @csrf
        <div>
            <label>Título: </label>
            <input type="text" name="titulo"/>
            <span>{{ $errors->first('titulo') }}</span>
        </div>
        <div>
            <label>Texto: </label>
            <textarea name="texto"></textarea>
        </div>
        <div>
            <label>Fecha: </label>
            <input type="datetime-local" name="fecha" value="{{ now() }}"/>
        </div>
        <div>
            <label>Publicada: </label>
            <input type="checkbox" name="publicada" checked/>
        </div>
        <input type="submit" name="guardar" value="Guardar"/>
    </form>

    <p>
        <a href="{{ route('entradas.index') }}">Cancelar</a>
    </p>

@endsection
```

```blade
{{-- resources/views/entradas/edit.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Editar entrada</h1>

    <form action="{{ route('entradas.update', $entrada->id) }}" method="POST">
        @csrf
        @method('PUT')
        <div>
            <label>Título: </label>
            <input type="text" name="titulo" value="{{ $entrada->titulo }}"/>
            <span>{{ $errors->first('titulo') }}</span>
        </div>
        <div>
            <label>Texto: </label>
            <textarea name="texto">{{ $entrada->texto }}</textarea>
        </div>
        <div>
            <label>Fecha: </label>
            <input type="datetime-local" name="fecha" value="{{ $entrada->fecha ?: now() }}"/>
        </div>
        <div>
            <label>Publicada: </label>
            <input type="checkbox" name="publicada" {{ $entrada->publicada ? 'checked' : '' }} />
        </div>
        <input type="submit" name="guardar" value="Guardar"/>
    </form>

    <p>
        <a href="{{ route('entradas.index') }}">Cancelar</a>
    </p>

@endsection
```

## Tests

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
            $entrada = factory(Entrada::class)->create();
    
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
            $entrada = factory(Entrada::class)->make();
    
            // When
            $this->post(route('entradas.store'), $entrada->toArray());
    
            // Then
            $this->assertEquals(1, Entrada::all()->count());
        }
    
        public function testStoreRequiresTitulo()
        {
            // Given
            $entrada = factory(Entrada::class)->make(['titulo' => null]);
    
            // When
            // Then
            $this->post(route('entradas.store'), $entrada->toArray())
                ->assertSessionHasErrors('titulo');
        }
    
        public function testShow()
        {
            // Given
            $entrada = factory(Entrada::class)->create();
    
            // When
            $response = $this->get(route('entradas.show', $entrada));
    
            // Then
            $response->assertSee($entrada->titulo);
        }
    
        public function testEdit()
        {
            // Given
            $entrada = factory(Entrada::class)->create();
    
            // When
            $response = $this->get(route('entradas.edit', $entrada), $entrada->toArray());
    
            // Then
            $response->assertSeeInOrder([$entrada->titulo, 'Guardar']);
        }
    
        public function testUpdate()
        {
            // Given
            $entrada = factory(Entrada::class)->create();
            $entrada->titulo = "Actualizada";
    
            // When
            $this->put(route('entradas.update', $entrada), $entrada->toArray());
    
            // Then
            $this->assertDatabaseHas('entradas', ['id' => $entrada->id, 'titulo' => $entrada->titulo]);
        }
    
        public function testDelete()
        {
            // Given
            $entrada = factory(Entrada::class)->create();
    
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
