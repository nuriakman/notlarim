# ADIM 3: INERTIA TARAFI İŞLEMLERİ

## Inertia.js Kurulumu

### Laravel Tarafı Kurulum

```bash
composer require inertiajs/inertia-laravel
```

### JavaScript Tarafı Kurulum

```bash
npm install @inertiajs/vue3
```

## Laravel Yapılandırması

### Middleware Oluşturma

```bash
php artisan inertia:middleware
```

### Middleware'i Kaydetme

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'web' => [
        // ...
        \App\Http\Middleware\HandleInertiaRequests::class,
    ],
];
```

### Root Template Oluşturma

```php
// resources/views/app.blade.php
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
    @vite(['resources/js/app.js', 'resources/css/app.css'])
    @inertiaHead
  </head>
  <body>
    @inertia
  </body>
</html>
```

## Vue ve Inertia Entegrasyonu

### app.js Dosyasını Güncelleme

```javascript
// resources/js/app.js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) =>
    resolvePageComponent(
      `./Pages/${name}.vue`,
      import.meta.glob('./Pages/**/*.vue'),
    ),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el);
  },
});
```

## Inertia Bileşenleri Oluşturma

### Users/Index.vue

```vue
<!-- resources/js/Pages/Users/Index.vue -->
<template>
  <div>
    <h1>Kullanıcılar</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        <Link :href="`/user/${user.id}`">{{ user.name }}</Link>
      </li>
    </ul>
  </div>
</template>

<script>
import { Link } from '@inertiajs/vue3';

export default {
  components: {
    Link,
  },
  props: {
    users: Array,
  },
};
</script>
```

### Users/Show.vue

```vue
<!-- resources/js/Pages/Users/Show.vue -->
<template>
  <div>
    <h1>Kullanıcı Detayı</h1>
    <div v-if="user">
      <h2>{{ user.name }}</h2>
      <p>Email: {{ user.email }}</p>
      <Link href="/">Geri Dön</Link>
    </div>
  </div>
</template>

<script>
import { Link } from '@inertiajs/vue3';

export default {
  components: {
    Link,
  },
  props: {
    user: Object,
  },
};
</script>
```

## Laravel Controller'ları Güncelleme

```php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;

class UserController extends Controller
{
    public function index()
    {
        return Inertia::render('Users/Index', [
            'users' => User::all()
        ]);
    }

    public function show($id)
    {
        return Inertia::render('Users/Show', [
            'user' => User::find($id)
        ]);
    }
}
```

## Route'ları Güncelleme

```php
// routes/web.php
use App\Http\Controllers\UserController;

Route::get('/', [UserController::class, 'index']);
Route::get('/user/{id}', [UserController::class, 'show']);
```

## Vite Yapılandırması

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/css/app.css', 'resources/js/app.js'],
      refresh: true,
    }),
    vue({
      template: {
        transformAssetUrls: {
          base: null,
          includeAbsolute: false,
        },
      },
    }),
  ],
});
```

## Uygulamayı Çalıştırma

```bash
# Terminal 1: Laravel sunucusu
php artisan serve

# Terminal 2: Vite geliştirme sunucusu
npm run dev
```

Artık uygulamanız Inertia.js ile çalışır durumda! Laravel backend'i ile Vue.js frontend'i arasında sorunsuz bir entegrasyon sağlanmış oldu. Sayfalar arasında geçiş yaparken tam sayfa yenileme olmadan, SPA (Single Page Application) deneyimi yaşayabilirsiniz.
