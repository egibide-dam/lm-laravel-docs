# Tailwind CSS en Laravel

## Instalar Tailwind en el proyecto Laravel

> :book: [Install Tailwind CSS with Laravel](https://tailwindcss.com/docs/guides/laravel)
> :book: [Compiling Assets (Mix)](https://laravel.com/docs/8.x/mix)

```javascript
// tailwind.config.js

module.exports = {
    purge: [
        './resources/**/*.blade.php',
        './resources/**/*.js',
        './resources/**/*.vue',
    ],
    darkMode: false, // or 'media' or 'class'
    theme: {
        extend: {},
    },
    variants: {
        extend: {},
    },
    plugins: [],
}
```

```javascript
// webpack.mix.js

const mix = require('laravel-mix');

/*
 |--------------------------------------------------------------------------
 | Mix Asset Management
 |--------------------------------------------------------------------------
 |
 | Mix provides a clean, fluent API for defining some Webpack build steps
 | for your Laravel applications. By default, we are compiling the CSS
 | file for the application as well as bundling up all the JS files.
 |
 */

mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css', [
        require("tailwindcss"),
    ]);
```

## Configurar PhpStorm para que indexe archivos grandes

El fichero CSS que genera Tailwind ocupa más de 3MB y PhpStorm no lo indexa, por lo que no tendremos autocompletado de
las clases de Tailwind.

Para que lo haga, vamos a `Help -> Edit Custom Properties...` y añadimos una línea con el nuevo tamaño máximo en KB:

```dotenv
idea.max.intellisense.filesize=256000
```

> :bulb: Hay que reiniciar PhpStorm para que el valor tenga efecto.

## Layout y vistas

```blade
{{-- resources/views/layouts/app.blade.php --}}

<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">

    <title>Mi blog</title>
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

    <div class="container mx-auto">

        <table class="container table-fixed tabla-hover border my-8">
            <thead>
            <tr>
                <th>Título</th>
                <th>Fecha</th>
                <th class="text-center">Visible</th>
                <th colspan="2" class="w-1/6">Acciones</th>
            </tr>
            </thead>
            <tbody>
            @foreach($entradas as $entrada)
                <tr>
                    <td>
                        <a href="{{ route('entradas.show', $entrada->id) }}">{{ $entrada->titulo }}</a>
                    </td>
                    <td>{{ $entrada->fecha }}</td>
                    <td class="text-center">{{ $entrada->visible ? 'Sí' : 'No' }}</td>
                    <td>
                        <a class="btn-secondary" href="{{ route('entradas.edit', $entrada->id) }}">Editar</a>
                    </td>
                    <td class="text-left">
                        <form action="{{ route('entradas.destroy', $entrada->id) }}" method="POST"
                              onsubmit="return confirm('¿Estás seguro?');">
                            @csrf
                            @method('DELETE')
                            <input class="btn-danger" type="submit" name="borrar" value="Borrar"/>
                        </form>
                    </td>
                </tr>
            @endforeach
            </tbody>
        </table>

        <div class="pt-4">
            <a class="btn-primary" href="{{ route('entradas.create') }}">Nueva entrada</a>
        </div>
    </div>
@endsection
```

```blade
{{-- resources/views/entradas/show.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>{{ $entrada->titulo }}</h1>

    <div class="container mx-auto">

        <table class="tabla-alterna border my-8">
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
                <th>Visible</th>
                <td>{{ $entrada->visible ? 'Sí' : 'No' }}</td>
            </tr>
            </tbody>
        </table>

        <div class="pt-4">
            <a class="btn-secondary" href="{{ route('entradas.index') }}">Volver</a>
        </div>
    </div>
@endsection
```

```blade
{{-- resources/views/entradas/create.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Nueva entrada</h1>

    <div class="container mx-auto my-8">

        <form action="{{ route('entradas.store') }}" method="POST">
            @csrf
            <div class="container grid grid-cols-2 w-1/2 gap-4">
                <label class="text-xl font-bold">Título</label>
                <div>
                    <input class="rounded w-full" type="text" name="titulo"/>
                    <span class="font-bold text-sm text-red-500">{{ $errors->first('titulo') }}</span>
                </div>
                <label class="text-xl font-bold">Texto</label>
                <textarea class="rounded h-72" name="texto"></textarea>
                <label class="text-xl font-bold">Fecha</label>
                <input type="datetime-local" name="fecha" value="{{ now() }}"/>
                <label class="text-xl font-bold">Visible</label>
                <input type="checkbox" name="visible"/>
            </div>
            <div class="pt-8">
                <input class="btn-primary" type="submit" name="guardar" value="Guardar"/>
                <a class="text-gray-500 ml-4" href="{{ route('entradas.index') }}">Cancelar</a>
            </div>
        </form>
    </div>
@endsection
```

```blade
{{-- resources/views/entradas/edit.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Editar entrada</h1>

    <div class="container mx-auto my-8">

        <form action="{{ route('entradas.update', $entrada->id) }}" method="POST">
            @csrf
            @method('PUT')
            <div class="container grid grid-cols-2 w-1/2 gap-4">
                <label class="text-xl font-bold">Título</label>
                <div>
                    <input class="rounded w-full" type="text" name="titulo" value="{{ $entrada->titulo }}"/>
                    <span class="font-bold text-sm text-red-500">{{ $errors->first('titulo') }}</span>
                </div>
                <label class="text-xl font-bold">Texto</label>
                <textarea class="rounded h-72" name="texto">{{ $entrada->texto }}</textarea>
                <label class="text-xl font-bold">Fecha</label>
                <input type="datetime-local" name="fecha" value="{{ $entrada->fecha ?: now() }}"/>
                <label class="text-xl font-bold">Visible</label>
                <input type="checkbox" name="visible" {{ $entrada->visible ? 'checked' : '' }} />
            </div>
            <div class="pt-8">
                <input class="btn-primary" type="submit" name="guardar" value="Guardar"/>
                <a class="text-gray-500 ml-4" href="{{ route('entradas.index') }}">Cancelar</a>
            </div>
        </form>
    </div>
@endsection
```

## CSS

```css
/* resources/css/app.css */

@tailwind base;
@tailwind components;
@tailwind utilities;

/* REF: https://tailwindcss.com/docs/adding-base-styles */
@layer base {
    h1 {
        @apply text-6xl p-8 bg-blue-500 text-white;
    }

    th {
        @apply bg-gray-200 text-left text-xl;
    }

    td, th {
        @apply p-2;
    }

    a {
        @apply text-blue-500 hover:underline;
    }
}

/* REF: https://tailwindcss.com/docs/extracting-components#extracting-component-classes-with-apply */
/* REF: REF: https://tailwindcomponents.com/component/button-component */
@layer components {
    .btn-primary {
        @apply inline-block focus:outline-none text-white text-sm py-2.5 px-5 rounded-md bg-blue-500 hover:bg-blue-600 hover:shadow-lg hover:no-underline;
    }

    .btn-secondary {
        @apply inline-block focus:outline-none text-white text-sm py-2.5 px-5 rounded-md bg-gray-500 hover:bg-gray-600 hover:shadow-lg hover:no-underline;
    }

    .btn-danger {
        @apply inline-block focus:outline-none text-white text-sm py-2.5 px-5 rounded-md bg-red-500 hover:bg-red-600 hover:shadow-lg hover:no-underline;
    }

    .tabla-alterna > tbody > tr:nth-of-type(even) {
        @apply bg-gray-100;
    }

    .tabla-hover > tbody > tr {
        @apply hover:bg-gray-100;
    }
}
```

## Plugin para dar aspecto a los formularios

Instalar el plugin [siguiendo las instrucciones](https://github.com/tailwindlabs/tailwindcss-forms).
