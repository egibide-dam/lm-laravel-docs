# Usuarios y roles

## Laravel Breeze

> :book: [Starter Kits](https://laravel.com/docs/8.x/starter-kits)

## Verificación de emails

> :book: [Email Verification](https://laravel.com/docs/8.x/verification)

Activar la verificación:

```php
// app/Models/User.php

class User extends Authenticatable implements MustVerifyEmail
```

> :book: [HTTP Requests](https://laravel.com/docs/8.x/requests#configuring-trusted-proxies)

Problema con el proxy inverso:

```php
// app/Http/Middleware/TrustProxies.php

protected $proxies = '*';
```

Configurar el servidor de correo saliente en el fichero `.env`:

```dotenv
MAIL_DRIVER=smtp
MAIL_HOST=maildev
MAIL_PORT=25
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=noreply@dockerbox.test
MAIL_FROM_NAME="${APP_NAME}"
```

Exigir que la cuenta esté verificada en alguna ruta:

```php
// routes/web.php

Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```

## Agrupar rutas

```php
// routes/web.php

// Sesión iniciada y cuenta verificada
Route::middleware(['auth', 'verified'])->group(function () {
    Route::resource('entradas', EntradaController::class);
    Route::resource('comentarios', ComentarioController::class);
});
```

## Roles y permisos

> :book: [Implementación de un Sistema basado en Roles y permisos con laravel-permission](https://blog.pleets.org/article/sistema-basado-en-roles-con-laravel-permission)
