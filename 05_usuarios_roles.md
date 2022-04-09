# Usuarios y roles

> :book: [Laravel Spatie Roles and Permissions Tutorial from Scratch](https://www.codecheef.org/article/laravel-spatie-roles-and-permissions-tutorial-from-scratch)

## Crear un proyecto nuevo

```shell
composer create-project laravel/laravel usuarios_roles
```

> [Configurar el `AppServiceProvider` para que genere direcciones HTTPS](https://github.com/egibide-dam/lm-laravel-docs/blob/master/01_nuevo_proyecto_laravel.md#generar-un-proyecto-nuevo-en-la-carpeta-sites) y crear el usuario de base de datos para el proyecto.

## Instalar los paquetes necesarios

> :book: [Laravel-permission](https://spatie.be/docs/laravel-permission/v5/introduction)

```shell
composer require spatie/laravel-permission
```

Paquete para crear formularios HTML:

```shell
composer require laravelcollective/html
```

Publicar el fichero de configuración:

```shell
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

Migrar la base de datos:

```shell
php artisan migrate
```

## Crear el modelo de ejemplo, `Product`, y su migración

```shell
php artisan make:model Product -m
```

```php
// database/migrations/2022_..._create_products_table.php

return new class extends Migration {

    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();

            $table->string('name');
            $table->text('detail');

            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('products');
    }
};
```

Migrar la base de datos:

```shell
php artisan migrate
```

## Actualizar los modelos

```php
// app/Models/User.php

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable, HasRoles;

...
```

```php
// app/Models/Product.php

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name', 'detail'
    ];
}
```

## Registrar el middleware

```php
// app/Http/Kernel.php

protected $routeMiddleware = [
    ...
    
    'role' => \Spatie\Permission\Middlewares\RoleMiddleware::class,
    'permission' => \Spatie\Permission\Middlewares\PermissionMiddleware::class,
    'role_or_permission' => \Spatie\Permission\Middlewares\RoleOrPermissionMiddleware::class,
]
```

## Crear el sistema de autenticación de Laravel

> :book: [Authentication](https://laravel.com/docs/9.x/authentication)

Instalar el paquete de soporte:

```shell
composer require laravel/ui
```

Crear el [_scaffolding_](https://es.wikipedia.org/wiki/Andamiaje_(programación)) de autenticación con Boostrap:

```shell
php artisan ui bootstrap --auth
```

Instalar los paquete de Node.js y compilar Bootstrap 5 y otros extras
mediante [Laravel Mix](https://laravel-mix.com/docs/6.0/what-is-mix):

```shell
npm install
npm run dev
```

## Crear los controladores

```shell
php artisan make:controller UserController
php artisan make:controller RoleController
php artisan make:controller ProductController
```

## Añadir las rutas con autenticación

```php
// routes/web.php

Route::group(['middleware' => ['auth']], function () {
    Route::resource('roles', RoleController::class);
    Route::resource('users', UserController::class);
    Route::resource('products', ProductController::class);
});
```

## Codificar los controladores

```php
// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Arr;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Spatie\Permission\Models\Role;

class UserController extends Controller
{
    function __construct()
    {
        $this->middleware('permission:user-list|user-create|user-edit|user-delete', ['only' => ['index', 'store']]);
        $this->middleware('permission:user-create', ['only' => ['create', 'store']]);
        $this->middleware('permission:user-edit', ['only' => ['edit', 'update']]);
        $this->middleware('permission:user-delete', ['only' => ['destroy']]);
    }

    public function index(Request $request)
    {
        $data = User::orderBy('id', 'DESC')->paginate(5);
        return view('users.index', compact('data'));
    }

    public function create()
    {
        $roles = Role::pluck('name', 'name')->all();
        return view('users.create', compact('roles'));
    }

    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|same:confirm-password',
            'roles' => 'required'
        ]);

        $input = $request->all();
        $input['password'] = Hash::make($input['password']);

        $user = User::create($input);
        $user->assignRole($request->input('roles'));

        return redirect()->route('users.index')
            ->with('success', 'User created successfully');
    }

    public function show($id)
    {
        $user = User::find($id);
        return view('users.show', compact('user'));
    }

    public function edit($id)
    {
        $user = User::find($id);
        $roles = Role::pluck('name', 'name')->all();
        $userRole = $user->roles->pluck('name', 'name')->all();

        return view('users.edit', compact('user', 'roles', 'userRole'));
    }

    public function update(Request $request, $id)
    {
        $this->validate($request, [
            'name' => 'required',
            'email' => 'required|email|unique:users,email,' . $id,
            'password' => 'same:confirm-password',
            'roles' => 'required'
        ]);

        $input = $request->all();
        if (!empty($input['password'])) {
            $input['password'] = Hash::make($input['password']);
        } else {
            $input = Arr::except($input, array('password'));
        }

        $user = User::find($id);
        $user->update($input);
        DB::table('model_has_roles')->where('model_id', $id)->delete();

        $user->assignRole($request->input('roles'));

        return redirect()->route('users.index')
            ->with('success', 'User updated successfully');
    }

    public function destroy($id)
    {
        User::find($id)->delete();
        return redirect()->route('users.index')
            ->with('success', 'User deleted successfully');
    }
}
```

```php
// app/Http/Controllers/RoleController.php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

class RoleController extends Controller
{
    function __construct()
    {
        $this->middleware('permission:role-list|role-create|role-edit|role-delete', ['only' => ['index', 'store']]);
        $this->middleware('permission:role-create', ['only' => ['create', 'store']]);
        $this->middleware('permission:role-edit', ['only' => ['edit', 'update']]);
        $this->middleware('permission:role-delete', ['only' => ['destroy']]);
    }

    public function index(Request $request)
    {
        $roles = Role::orderBy('id', 'DESC')->paginate(5);
        return view('roles.index', compact('roles'));
    }

    public function create()
    {
        $permission = Permission::get();
        return view('roles.create', compact('permission'));
    }

    public function store(Request $request)
    {
        $this->validate($request, [
            'name' => 'required|unique:roles,name',
            'permission' => 'required',
        ]);

        $role = Role::create(['name' => $request->input('name')]);
        $role->syncPermissions($request->input('permission'));

        return redirect()->route('roles.index')
            ->with('success', 'Role created successfully');
    }

    public function show($id)
    {
        $role = Role::find($id);
        $rolePermissions = Permission::join("role_has_permissions", "role_has_permissions.permission_id", "=", "permissions.id")
            ->where("role_has_permissions.role_id", $id)
            ->get();

        return view('roles.show', compact('role', 'rolePermissions'));
    }

    public function edit($id)
    {
        $role = Role::find($id);
        $permission = Permission::get();
        $rolePermissions = DB::table("role_has_permissions")->where("role_has_permissions.role_id", $id)
            ->pluck('role_has_permissions.permission_id', 'role_has_permissions.permission_id')
            ->all();

        return view('roles.edit', compact('role', 'permission', 'rolePermissions'));
    }

    public function update(Request $request, $id)
    {
        $this->validate($request, [
            'name' => 'required',
            'permission' => 'required',
        ]);

        $role = Role::find($id);
        $role->name = $request->input('name');
        $role->save();

        $role->syncPermissions($request->input('permission'));

        return redirect()->route('roles.index')
            ->with('success', 'Role updated successfully');
    }

    public function destroy($id)
    {
        DB::table("roles")->where('id', $id)->delete();
        return redirect()->route('roles.index')
            ->with('success', 'Role deleted successfully');
    }
}
```

```php
// app/Http/Controllers/ProductController.php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    function __construct()
    {
        $this->middleware('permission:product-list|product-create|product-edit|product-delete', ['only' => ['index', 'show']]);
        $this->middleware('permission:product-create', ['only' => ['create', 'store']]);
        $this->middleware('permission:product-edit', ['only' => ['edit', 'update']]);
        $this->middleware('permission:product-delete', ['only' => ['destroy']]);
    }

    public function index()
    {
        $products = Product::latest()->paginate(5);
        return view('products.index', compact('products'));
    }

    public function create()
    {
        return view('products.create');
    }

    public function store(Request $request)
    {
        request()->validate([
            'name' => 'required',
            'detail' => 'required',
        ]);

        Product::create($request->all());

        return redirect()->route('products.index')
            ->with('success', 'Product created successfully.');
    }

    public function show(Product $product)
    {
        return view('products.show', compact('product'));
    }

    public function edit(Product $product)
    {
        return view('products.edit', compact('product'));
    }

    public function update(Request $request, Product $product)
    {
        request()->validate([
            'name' => 'required',
            'detail' => 'required',
        ]);

        $product->update($request->all());

        return redirect()->route('products.index')
            ->with('success', 'Product updated successfully');
    }

    public function destroy(Product $product)
    {
        $product->delete();

        return redirect()->route('products.index')
            ->with('success', 'Product deleted successfully');
    }
}
```

## Crear las vistas

### Layout

```blade
{{-- resources/views/layouts/app.blade.php --}}

<!doctype html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Scripts -->
    <script src="{{ asset('js/app.js') }}" defer></script>

    <!-- Fonts -->
    <link rel="dns-prefetch" href="//fonts.gstatic.com">
    <link href="https://fonts.googleapis.com/css?family=Nunito" rel="stylesheet">

    <!-- Styles -->
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">
</head>
<body>
<div id="app">
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="{{ url('/') }}">
                {{ config('app.name', 'Laravel') }}
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse"
                    data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent"
                    aria-expanded="false" aria-label="{{ __('Toggle navigation') }}">
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="collapse navbar-collapse" id="navbarSupportedContent">
                <!-- Left Side Of Navbar -->
                <ul class="navbar-nav me-auto">

                </ul>

                <!-- Right Side Of Navbar -->
                <ul class="navbar-nav ms-auto">
                    <!-- Authentication Links -->
                    @guest
                        @if (Route::has('login'))
                            <li class="nav-item">
                                <a class="nav-link" href="{{ route('login') }}">{{ __('Login') }}</a>
                            </li>
                        @endif

                        @if (Route::has('register'))
                            <li class="nav-item">
                                <a class="nav-link" href="{{ route('register') }}">{{ __('Register') }}</a>
                            </li>
                        @endif
                    @else
                        <li><a class="nav-link" href="{{ route('home') }}">Dashboard</a></li>
                        <li><a class="nav-link" href="{{ route('users.index') }}">Users</a></li>
                        <li><a class="nav-link" href="{{ route('roles.index') }}">Roles</a></li>
                        <li><a class="nav-link" href="{{ route('products.index') }}">Products</a></li>
                        <li class="nav-item dropdown">
                            <a id="navbarDropdown" class="nav-link dropdown-toggle" href="#" role="button"
                               data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
                                {{ Auth::user()->name }}
                            </a>

                            <div class="dropdown-menu dropdown-menu-end" aria-labelledby="navbarDropdown">
                                <a class="dropdown-item" href="{{ route('logout') }}"
                                   onclick="event.preventDefault();
                                                     document.getElementById('logout-form').submit();">
                                    {{ __('Logout') }}
                                </a>

                                <form id="logout-form" action="{{ route('logout') }}" method="POST" class="d-none">
                                    @csrf
                                </form>
                            </div>
                        </li>
                    @endguest
                </ul>
            </div>
        </div>
    </nav>

    <main class="py-4">
        <div class="container">
            @yield('content')
        </div>
    </main>
</div>
</body>
</html>
```

### Inicio

```blade
{{-- resources/views/welcome.blade.php --}}

@extends('layouts.app')

@section('content')
    <div class="row">
        <p>Bienvenidos a esta aplicación de ejemplo.</p>
    </div>
@endsection
```

### User

```blade
{{-- resources/views/users/index.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>User management</h1>

    @can('user-create')
        <div class="my-3">
            <a class="btn btn-primary" href="{{ route('users.create') }}">Create new user</a>
        </div>
    @endcan

    @if ($message = Session::get('success'))
        <div class="alert alert-success">
            <p>{{ $message }}</p>
        </div>
    @endif

    <table class="table table-striped">
        <thead>
        <tr>
            <th>#</th>
            <th>Name</th>
            <th>Email</th>
            <th>Roles</th>
            <th>Action</th>
        </tr>
        </thead>
        <tbody class="align-middle">
        @foreach ($data as $key => $user)
            <tr>
                <td>{{ $user->id }}</td>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
                <td>
                    @if(!empty($user->getRoleNames()))
                        @foreach($user->getRoleNames() as $v)
                            <span class="badge bg-primary">{{ $v }}</span>
                        @endforeach
                    @endif
                </td>
                <td>
                    <a class="btn btn-success" href="{{ route('users.show',$user->id) }}">Show</a>
                    @can('user-edit')
                        <a class="btn btn-secondary" href="{{ route('users.edit',$user->id) }}">Edit</a>
                    @endcan
                    @can('user-delete')
                        {!! Form::open(['method' => 'DELETE','route' => ['users.destroy', $user->id],'style'=>'display:inline']) !!}
                        {!! Form::submit('Delete', ['class' => 'btn btn-danger', 'onclick' => 'return confirm("Are you sure?")']) !!}
                        {!! Form::close() !!}
                    @endcan
                </td>
            </tr>
        @endforeach
        </tbody>
        <tfoot>
        <tr>
            <th colspan="5" class="border-0">Total: {{ $data->count() }}</th>
        </tr>
        </tfoot>
    </table>
@endsection
```

```blade
{{-- resources/views/users/create.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Create new user</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    {!! Form::open(array('route' => 'users.store','method'=>'POST')) !!}
    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {!! Form::text('name', null, array('placeholder' => 'Name','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Email:</strong>
                {!! Form::text('email', null, array('placeholder' => 'Email','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Password:</strong>
                {!! Form::password('password', array('placeholder' => 'Password','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Confirm password:</strong>
                {!! Form::password('confirm-password', array('placeholder' => 'Confirm Password','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Role:</strong>
                {!! Form::select('roles[]', $roles,[], array('class' => 'form-control','multiple')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <button type="submit" class="btn btn-primary">Submit</button>
            <a class="link-secondary ms-2" href="{{ route('users.index') }}">Cancel</a>
        </div>
    </div>
    {!! Form::close() !!}
@endsection
```

```blade
{{-- resources/views/users/edit.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Edit user</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    {!! Form::model($user, ['method' => 'PATCH','route' => ['users.update', $user->id]]) !!}
    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {!! Form::text('name', null, array('placeholder' => 'Name','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Email:</strong>
                {!! Form::text('email', null, array('placeholder' => 'Email','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Password:</strong>
                {!! Form::password('password', array('placeholder' => 'Password','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Confirm password:</strong>
                {!! Form::password('confirm-password', array('placeholder' => 'Confirm Password','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Role:</strong>
                {!! Form::select('roles[]', $roles,$userRole, array('class' => 'form-control','multiple')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <button type="submit" class="btn btn-primary">Submit</button>
            <a class="link-secondary ms-2" href="{{ route('users.index') }}">Cancel</a>
        </div>
    </div>
    {!! Form::close() !!}
@endsection
```

```blade
{{-- resources/views/users/show.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Show user</h1>

    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {{ $user->name }}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Email:</strong>
                {{ $user->email }}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Roles:</strong>
                @if(!empty($user->getRoleNames()))
                    @foreach($user->getRoleNames() as $v)
                        <span class="badge bg-primary">{{ $v }}</span>
                    @endforeach
                @endif
            </div>
        </div>
    </div>

    <a class="btn btn-secondary" href="{{ route('users.index') }}">Back</a>
@endsection
```

### Role

```blade
{{-- resources/views/roles/index.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Role management</h1>

    @can('role-create')
        <div class="my-3">
            <a class="btn btn-primary" href="{{ route('roles.create') }}">Create new role</a>
        </div>
    @endcan

    @if ($message = Session::get('success'))
        <div class="alert alert-success">
            <p>{{ $message }}</p>
        </div>
    @endif

    <table class="table table-striped">
        <thead>
        <tr>
            <th>#</th>
            <th>Name</th>
            <th>Permissions</th>
            <th>Action</th>
        </tr>
        </thead>
        <tbody class="align-middle">
        @foreach ($roles as $key => $role)
            <tr>
                <td>{{ $role->id }}</td>
                <td>{{ $role->name }}</td>
                <td>{{ $role->permissions()->count() }}</td>
                <td>
                    <a class="btn btn-success" href="{{ route('roles.show',$role->id) }}">Show</a>
                    @can('role-edit')
                        <a class="btn btn-secondary" href="{{ route('roles.edit',$role->id) }}">Edit</a>
                    @endcan
                    @can('role-delete')
                        {!! Form::open(['method' => 'DELETE','route' => ['roles.destroy', $role->id],'style'=>'display:inline']) !!}
                        {!! Form::submit('Delete', ['class' => 'btn btn-danger', 'onclick' => 'return confirm("Are you sure?")']) !!}
                        {!! Form::close() !!}
                    @endcan
                </td>
            </tr>
        @endforeach
        </tbody>
        <tfoot>
        <tr>
            <th colspan="3" class="border-0">Total: {{ $roles->count() }}</th>
        </tr>
        </tfoot>
    </table>
@endsection
```

```blade
{{-- resources/views/roles/create.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Create new role</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    {!! Form::open(array('route' => 'roles.store','method'=>'POST')) !!}
    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {!! Form::text('name', null, array('placeholder' => 'Name','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Permission:</strong>
                <br/>
                @foreach($permission as $value)
                    <label>{{ Form::checkbox('permission[]', $value->id, false, array('class' => 'name')) }}
                        {{ $value->name }}</label>
                    <br/>
                @endforeach
            </div>
        </div>
        <div class="col-12 mb-3">
            <button type="submit" class="btn btn-primary">Submit</button>
            <a class="link-secondary ms-2" href="{{ route('roles.index') }}">Cancel</a>
        </div>
    </div>
    {!! Form::close() !!}
@endsection
```

```blade
{{-- resources/views/roles/edit.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Edit role</h1>

    @if (count($errors) > 0)
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    {!! Form::model($role, ['method' => 'PATCH','route' => ['roles.update', $role->id]]) !!}
    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {!! Form::text('name', null, array('placeholder' => 'Name','class' => 'form-control')) !!}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Permissions:</strong>
                <br/>
                @foreach($permission as $value)
                    <label>{{ Form::checkbox('permission[]', $value->id, in_array($value->id, $rolePermissions) ? true : false, array('class' => 'name')) }}
                        {{ $value->name }}</label>
                    <br/>
                @endforeach
            </div>
        </div>
        <div class="col-12 mb-3">
            <button type="submit" class="btn btn-primary">Submit</button>
            <a class="link-secondary ms-2" href="{{ route('roles.index') }}">Cancel</a>
        </div>
    </div>
    {!! Form::close() !!}
@endsection
```

```blade
{{-- resources/views/roles/show.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Show role</h1>

    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {{ $role->name }}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Permissions:</strong>
                @if(!empty($rolePermissions))
                    @foreach($rolePermissions as $v)
                        <span class="badge bg-primary">{{ $v->name }}</span>
                    @endforeach
                @endif
            </div>
        </div>
    </div>

    <a class="btn btn-secondary" href="{{ route('roles.index') }}">Back</a>
@endsection
```

### Product

```blade
{{-- resources/views/products/index.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Product management</h1>

    @can('product-create')
        <div class="my-3">
            <a class="btn btn-primary" href="{{ route('products.create') }}">Create new product</a>
        </div>
    @endcan

    @if ($message = Session::get('success'))
        <div class="alert alert-success">
            <p>{{ $message }}</p>
        </div>
    @endif

    <table class="table table-striped">
        <thead>
        <tr>
            <th>#</th>
            <th>Name</th>
            <th>Details</th>
            <th>Action</th>
        </tr>
        </thead>
        <tbody class="align-middle">
        @forelse($products as $product)
            <tr>
                <td>{{ $product->id }}</td>
                <td>{{ $product->name }}</td>
                <td>{{ $product->detail }}</td>
                <td>
                    <a class="btn btn-success" href="{{ route('products.show',$product->id) }}">Show</a>
                    @can('product-edit')
                        <a class="btn btn-secondary" href="{{ route('products.edit',$product->id) }}">Edit</a>
                    @endcan
                    @can('product-delete')
                        {!! Form::open(['method' => 'DELETE','route' => ['products.destroy', $product->id],'style'=>'display:inline']) !!}
                        {!! Form::submit('Delete', ['class' => 'btn btn-danger', 'onclick' => 'return confirm("Are you sure?")']) !!}
                        {!! Form::close() !!}
                    @endcan
                </td>
            </tr>
        @empty
            <tr>
                <td colspan="4">No data found.</td>
            </tr>
        @endforelse
        </tbody>
        <tfoot>
        <tr>
            <th colspan="4" class="border-0">Total: {{ $products->count() }}</th>
        </tr>
        </tfoot>
    </table>
@endsection
```

```blade
{{-- resources/views/products/create.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Create new product</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <form action="{{ route('products.store') }}" method="POST">
        @csrf
        <div class="row">
            <div class="col-12 mb-3">
                <div class="form-group">
                    <strong>Name:</strong>
                    <input type="text" name="name" class="form-control" placeholder="Name">
                </div>
            </div>
            <div class="col-12 mb-3">
                <div class="form-group">
                    <strong>Detail:</strong>
                    <textarea class="form-control" style="height:150px" name="detail" placeholder="Detail"></textarea>
                </div>
            </div>
            <div class="col-12 mb-3">
                <button type="submit" class="btn btn-primary">Submit</button>
                <a class="link-secondary ms-2" href="{{ route('products.index') }}">Cancel</a>
            </div>
        </div>
    </form>
@endsection
```

```blade
{{-- resources/views/products/edit.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Edit product</h1>

    @if ($errors->any())
        <div class="alert alert-danger">
            <strong>Whoops!</strong> There were some problems with your input.<br><br>
            <ul>
                @foreach ($errors->all() as $error)
                    <li>{{ $error }}</li>
                @endforeach
            </ul>
        </div>
    @endif

    <form action="{{ route('products.update',$product->id) }}" method="POST">
        @csrf
        @method('PUT')

        <div class="row">
            <div class="col-12 mb-3">
                <div class="form-group">
                    <strong>Name:</strong>
                    <input type="text" name="name" value="{{ $product->name }}" class="form-control"
                           placeholder="Name">
                </div>
            </div>
            <div class="col-12 mb-3">
                <div class="form-group">
                    <strong>Detail:</strong>
                    <textarea class="form-control" style="height:150px" name="detail"
                              placeholder="Detail">{{ $product->detail }}</textarea>
                </div>
            </div>
            <div class="col-12 mb-3">
                <button type="submit" class="btn btn-primary">Submit</button>
                <a class="link-secondary ms-2" href="{{ route('products.index') }}">Cancel</a>
            </div>
        </div>
    </form>
@endsection
```

```blade
{{-- resources/views/products/show.blade.php --}}

@extends('layouts.app')

@section('content')

    <h1>Show product</h1>

    <div class="row">
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Name:</strong>
                {{ $product->name }}
            </div>
        </div>
        <div class="col-12 mb-3">
            <div class="form-group">
                <strong>Details:</strong>
                {{ $product->detail }}
            </div>
        </div>
    </div>

    <a class="btn btn-secondary" href="{{ route('products.index') }}">Back</a>
@endsection
```

## Crear un usuario de ejemplo

### Crear permisos

```shell
php artisan make:seeder PermissionTableSeeder
```

```php
// database/seeders/PermissionTableSeeder.php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;

class PermissionTableSeeder extends Seeder
{
    public function run()
    {
        $permissions = [
            'user-list',
            'user-create',
            'user-edit',
            'user-delete',
            'role-list',
            'role-create',
            'role-edit',
            'role-delete',
            'product-list',
            'product-create',
            'product-edit',
            'product-delete'
        ];

        foreach ($permissions as $permission) {
            Permission::create(['name' => $permission]);
        }
    }
}
```

```shell
php artisan db:seed --class=PermissionTableSeeder
```

### Crear un usuario administrador con todos los permisos

```shell
php artisan make:seeder CreateAdminUserSeeder
```

```php
// database/seeders/CreateAdminUserSeeder.php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\Models\Role;

class CreateAdminUserSeeder extends Seeder
{
    public function run()
    {
        $user = User::create([
            'name' => 'Admin',
            'email' => 'admin@example.org',
            'password' => bcrypt('12345Abcde')
        ]);

        $role = Role::create(['name' => 'Admin']);

        $permissions = Permission::pluck('id', 'id')->all();

        $role->syncPermissions($permissions);

        $user->assignRole([$role->id]);
    }
}
```

```shell
php artisan db:seed --class=CreateAdminUserSeeder
```

## Verificación de emails

> :book: [Email Verification](https://laravel.com/docs/9.x/verification)

Activar la verificación:

```php
// app/Models/User.php

class User extends Authenticatable implements MustVerifyEmail
```

Activar las rutas de verificación:

```php
// routes/web.php

Auth::routes(['verify' => true]);
```

Exigir que la cuenta esté verificada en alguna ruta:

```php
// routes/web.php

Route::group(['middleware' => ['auth', 'verified']], function () {
    Route::resource('roles', RoleController::class);
    Route::resource('users', UserController::class);
    Route::resource('products', ProductController::class);
});
```

Configurar el servidor de correo saliente en el fichero `.env`:

```dotenv
# .env

MAIL_MAILER=smtp
MAIL_HOST=mailcatcher
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=noreply@example.org
MAIL_FROM_NAME="${APP_NAME}"
```

Problema de firma inválida por que estamos usando un proxy inverso:

```php
// app/Http/Middleware/TrustProxies.php

protected $proxies = '*';
```

> :book: [HTTP Requests](https://laravel.com/docs/9.x/requests#configuring-trusted-proxies)
