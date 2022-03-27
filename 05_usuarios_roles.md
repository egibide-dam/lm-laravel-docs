# Usuarios y roles

> :book: [Laravel Spatie Roles and Permissions Tutorial from Scratch](https://www.codecheef.org/article/laravel-spatie-roles-and-permissions-tutorial-from-scratch)

## Crear un proyecto nuevo

```shell
composer create-project laravel/laravel usuarios_roles
```

> Configurar el `AppServiceProvider` para que genere direcciones HTTPS y crear el usuario de base de datos para el proyecto.

## Instalar los paquetes necesarios

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

## Crear el modelo de ejemplo, `Producto`, y su migración

```shell
php artisan make:model Producto -m
```

```php
// database/migrations/2022_..._create_productos_table.php

return new class extends Migration {

    public function up()
    {
        Schema::create('productos', function (Blueprint $table) {
            $table->id();

            $table->string('nombre');
            $table->text('detalle');

            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('productos');
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
// app/Models/Producto.php

class Producto extends Model
{
    use HasFactory;

    protected $fillable = [
        'nombre', 'detalle'
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
php artisan make:controller ProductoController
```

## Añadir las rutas con autenticación

```php
// routes/web.php

Route::group(['middleware' => ['auth']], function () {
    Route::resource('roles', RoleController::class);
    Route::resource('users', UserController::class);
    Route::resource('productos', ProductoController::class);
});
```

## Codificar los controladores

```php

```

```php

```

```php

```

## Crear las vistas

### Layout

```php

```

### User

```php

```

```php

```

```php

```

```php

```

### Role

```php

```

```php

```

```php

```

```php

```

### Producto

```php

```

```php

```

```php

```

```php

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
            'producto-list',
            'producto-create',
            'producto-edit',
            'producto-delete'
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
Auth::routes(['verify' => true]);
```

Exigir que la cuenta esté verificada en alguna ruta:

```php
// routes/web.php

Route::group(['middleware' => ['auth', 'verified']], function () {
    Route::resource('roles', RoleController::class);
    Route::resource('users', UserController::class);
    Route::resource('productos', ProductoController::class);
});
```

Configurar el servidor de correo saliente en el fichero `.env`:

```dotenv
MAIL_DRIVER=smtp
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
