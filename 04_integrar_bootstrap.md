# Integrar una plantilla de Bootstrap en Laravel

Vamos a integrar la plantilla de ejemplo [starter template](https://getbootstrap.com/docs/4.4/examples/starter-template/).

## Layout

1. Partimos del [layout básico](https://getbootstrap.com/docs/4.4/getting-started/introduction/) que nos propone Bootstrap en su documentación y lo adaptamos con las directivas de Laravel:

    ```blade
    {{-- resources/views/layouts/app.blade.php --}}
    
    <!doctype html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        <!-- Required meta tags -->
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    
        <!-- Bootstrap CSS -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css"
            integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
    
        <title>Blog</title>
    </head>
    <body>
    
    @yield('content')
    
    <!-- Optional JavaScript -->
    <!-- jQuery first, then Popper.js, then Bootstrap JS -->
    <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js"
            integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n"
            crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js"
            integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo"
            crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js"
            integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6"
            crossorigin="anonymous"></script>
    </body>
    </html>
    ```

2. Creamos y enlazamos la hoja de estilos del tema:

    ```css
    /* public/css/starter-template.css */
    
    body {
        padding-top: 5rem;
    }
    
    .starter-template {
        padding: 3rem 1.5rem;
        /* text-align: center; */
    }
    ```

    ```blade
    {{-- resources/views/layouts/app.blade.php --}}
    
    <!doctype html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    <head>
        ...
    
        <title>Blog</title>
    
        <link href="{{ asset('css/starter-template.css') }}" rel="stylesheet">
    </head>
    ...
    ```

3. Añadimos el resto del layout de la plantilla:

    ```blade
    {{-- resources/views/layouts/app.blade.php --}}
    
    ...
    
    <body>
    
    <nav class="navbar navbar-expand-md navbar-dark bg-dark fixed-top">
        <a class="navbar-brand" href="#">Navbar</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarsExampleDefault"
                aria-controls="navbarsExampleDefault" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
    
        <div class="collapse navbar-collapse" id="navbarsExampleDefault">
            <ul class="navbar-nav mr-auto">
                <li class="nav-item active">
                    <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="#">Link</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link disabled" href="#" tabindex="-1" aria-disabled="true">Disabled</a>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="dropdown01" data-toggle="dropdown" aria-haspopup="true"
                    aria-expanded="false">Dropdown</a>
                    <div class="dropdown-menu" aria-labelledby="dropdown01">
                        <a class="dropdown-item" href="#">Action</a>
                        <a class="dropdown-item" href="#">Another action</a>
                        <a class="dropdown-item" href="#">Something else here</a>
                    </div>
                </li>
            </ul>
            <form class="form-inline my-2 my-lg-0">
                <input class="form-control mr-sm-2" type="text" placeholder="Search" aria-label="Search">
                <button class="btn btn-secondary my-2 my-sm-0" type="submit">Search</button>
            </form>
        </div>
    </nav>
    
    <main role="main" class="container">
    
        <div class="starter-template">
            @yield('content')
        </div>
    
    </main><!-- /.container -->
    
    <!-- Optional JavaScript -->
    
    ...
    
    </body>
    </html>
    ```

4. Refactorizamos y extraemos la barra de navegación a un _partial_:

    ```blade
    {{-- resources/views/layouts/menu.blade.php --}}
    
    <nav class="navbar navbar-expand-md navbar-dark bg-dark fixed-top">
        <a class="navbar-brand" href="#">Navbar</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarsExampleDefault"
                aria-controls="navbarsExampleDefault" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
        </button>
    
        <div class="collapse navbar-collapse" id="navbarsExampleDefault">
            <ul class="navbar-nav mr-auto">
                <li class="nav-item active">
                    <a class="nav-link" href="#">Home <span class="sr-only">(current)</span></a>
                </li>
                <li class="nav-item">
                    <a class="nav-link" href="#">Link</a>
                </li>
                <li class="nav-item">
                    <a class="nav-link disabled" href="#" tabindex="-1" aria-disabled="true">Disabled</a>
                </li>
                <li class="nav-item dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="dropdown01" data-toggle="dropdown" aria-haspopup="true"
                    aria-expanded="false">Dropdown</a>
                    <div class="dropdown-menu" aria-labelledby="dropdown01">
                        <a class="dropdown-item" href="#">Action</a>
                        <a class="dropdown-item" href="#">Another action</a>
                        <a class="dropdown-item" href="#">Something else here</a>
                    </div>
                </li>
            </ul>
            <form class="form-inline my-2 my-lg-0">
                <input class="form-control mr-sm-2" type="text" placeholder="Search" aria-label="Search">
                <button class="btn btn-secondary my-2 my-sm-0" type="submit">Search</button>
            </form>
        </div>
    </nav>
    ```
    
    ```blade
    {{-- resources/views/layouts/app.blade.php --}}
    
    ...
    
    <body>
    
    @include('layouts.menu')
    
    <main role="main" class="container">
    
        <div class="starter-template">
            @yield('content')
        </div>
    
    </main><!-- /.container -->
    
    <!-- Optional JavaScript -->
    
    ...
    
    </body>
    </html>
    ```

5. Modificamos las vistas para que utilicen las clases de Bootstrap:

    ```blade
    {{-- resources/views/entradas/index.blade.php --}}
    
    @extends('layouts.app')
    
    @section('content')
    
        <h1>Entradas</h1>
    
        <table class="table table-hover">
            <thead class="thead-dark">
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
                    <td class="align-middle">
                        <a href="{{ route('entradas.show', $entrada->id) }}">{{ $entrada->titulo }}</a>
                    </td>
                    <td class="align-middle">{{ $entrada->fecha }}</td>
                    <td class="align-middle">{{ $entrada->publicada ? 'Sí' : 'No' }}</td>
                    <td><a class="btn btn-secondary" href="{{ route('entradas.edit', $entrada->id) }}">Editar</a></td>
                    <td>
                        <form action="{{ route('entradas.destroy', $entrada->id) }}" method="POST">
                            @csrf
                            @method('DELETE')
                            <input class="btn btn-danger" type="submit" name="borrar" value="Borrar"/>
                        </form>
                    </td>
                </tr>
            @endforeach
            </tbody>
        </table>
    
        <p>
            <a class="btn btn-primary" href="{{ route('entradas.create') }}">Nueva entrada</a>
        </p>
    
    @endsection
    ```
    
    ```blade
    {{-- resources/views/entradas/show.blade.php --}}
    
    @extends('layouts.app')
    
    @section('content')
    
        <h1>{{ $entrada->titulo }}</h1>
    
        <table class="table table-bordered">
            <tbody class="thead-light">
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
            <a class="btn btn-primary" href="{{ route('entradas.index') }}">Volver</a>
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
            <div class="form-group">
                <label>Título: </label>
                <input class="form-control {{ $errors->first('titulo') ? 'is-invalid' : '' }}" type="text" name="titulo"/>
                <div class="invalid-feedback">{{ $errors->first('titulo') }}</div>
            </div>
            <div class="form-group">
                <label>Texto: </label>
                <textarea class="form-control" name="texto"></textarea>
            </div>
            <div class="form-group">
                <label>Fecha: </label>
                <input class="form-control" type="datetime-local" name="fecha" value="{{ now() }}"/>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox" name="publicada" checked/>
                <label class="form-check-label">Publicada</label>
            </div>
            <div class="form-group mt-3">
                <input class="btn btn-primary" type="submit" name="guardar" value="Guardar"/>
                <a class="btn btn-link text-secondary" href="{{ route('entradas.index') }}">Cancelar</a>
            </div>
        </form>
    
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
            <div class="form-group">
                <label>Título: </label>
                <input class="form-control {{ $errors->first('titulo') ? 'is-invalid' : '' }}" type="text" name="titulo"
                    value="{{ $entrada->titulo }}"/>
                <div class="invalid-feedback">{{ $errors->first('titulo') }}</div>
            </div>
            <div class="form-group">
                <label>Texto: </label>
                <textarea class="form-control" name="texto">{{ $entrada->texto }}</textarea>
            </div>
            <div class="form-group">
                <label>Fecha: </label>
                <input class="form-control" type="datetime-local" name="fecha" value="{{ $entrada->fecha ?: now() }}"/>
            </div>
            <div class="form-check">
                <input class="form-check-input" type="checkbox"
                    name="publicada" {{ $entrada->publicada ? 'checked' : '' }}/>
                <label class="form-check-label">Publicada</label>
            </div>
            <div class="form-group mt-3">
                <input class="btn btn-primary" type="submit" name="guardar" value="Guardar"/>
                <a class="btn btn-link text-secondary" href="{{ route('entradas.index') }}">Cancelar</a>
            </div>
        </form>
    
    @endsection
    ```
