# Laravel Route Sorunları ve Çözümleri

## İçindekiler

1. Route Temel Kontroller
2. Route Debug Araçları
3. Sık Karşılaşılan Sorunlar
4. Route Loglama
5. Best Practices
6. Örnek Senaryolar

## 1. Route Temel Kontroller

### 1.1 Route Cache Temizleme

```bash
# Tüm cache'i temizle
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Route cache'i yeniden oluştur
php artisan route:cache
```

### 1.2 Route Listesi Kontrolü

```bash
# Tüm route'ları listele
php artisan route:list

# Belirli bir route'u ara
php artisan route:list --path=products

# API route'larını listele
php artisan route:list --path=api
```

## 2. Route Debug Araçları

### 2.1 Debug Bar Kurulumu

```bash
composer require barryvdh/laravel-debugbar --dev
```

```php
// config/debugbar.php
return [
    'enabled' => env('DEBUGBAR_ENABLED', true),
    'collectors' => [
        'routes' => true,
        'time' => true,
    ],
];
```

### 2.2 Route Debugging Middleware

```php
// app/Http/Middleware/RouteDebugMiddleware.php
namespace App\Http\Middleware;

use Illuminate\Support\Facades\Log;

class RouteDebugMiddleware
{
    public function handle($request, \Closure $next)
    {
        $startTime = microtime(true);

        $response = $next($request);

        $duration = microtime(true) - $startTime;

        Log::channel('routes')->info('Route İsteği', [
            'url' => $request->fullUrl(),
            'method' => $request->method(),
            'duration' => $duration,
            'status' => $response->status(),
            'route' => $request->route()->getName() ?? 'unnamed',
            'action' => $request->route()->getActionName(),
            'parameters' => $request->route()->parameters(),
        ]);

        return $response;
    }
}
```

## 3. Sık Karşılaşılan Sorunlar ve Çözümleri

### 3.1 404 Not Found Hataları

```php
// routes/api.php
// Doğru Kullanım
Route::prefix('api')->group(function () {
    Route::apiResource('products', ProductController::class);
});

// web.php ve api.php Çakışması Kontrolü
Route::fallback(function () {
    return response()->json(['message' => 'Route bulunamadı'], 404);
});
```

### 3.2 Method Not Allowed Hataları

```php
// Route method kontrolü
Route::match(['get', 'post'], 'products', [ProductController::class, 'index']);

// Tüm metodları loglama
Route::middleware(['web'])->group(function () {
    Route::any('products/{id}', function ($id) {
        Log::info('Gelen İstek', [
            'method' => request()->method(),
            'url' => request()->fullUrl()
        ]);
    });
});
```

### 3.3 CORS Sorunları

```php
// config/cors.php
return [
    'paths' => ['api/*'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['*'],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => false,
];
```

## 4. Route Loglama

### 4.1 Route Log Channel

```php
// config/logging.php
'channels' => [
    'routes' => [
        'driver' => 'daily',
        'path' => storage_path('logs/routes.log'),
        'level' => 'debug',
        'days' => 14,
    ],
],
```

### 4.2 Route Event Listener

```php
// app/Providers/RouteServiceProvider.php
public function boot()
{
    Route::matched(function ($route, $request) {
        Log::channel('routes')->info('Route Eşleşti', [
            'uri' => $route->uri(),
            'name' => $route->getName(),
            'action' => $route->getActionName(),
        ]);
    });
}
```

## 5. Best Practices

### 5.1 Route İsimlendirme

```php
// Tutarlı isimlendirme
Route::get('/products', [ProductController::class, 'index'])->name('products.index');
Route::get('/products/{id}', [ProductController::class, 'show'])->name('products.show');
```

### 5.2 Route Gruplandırma

```php
// Prefix ve middleware gruplandırma
Route::prefix('api/v1')->middleware(['auth:api'])->group(function () {
    Route::apiResource('products', ProductController::class);
    Route::apiResource('categories', CategoryController::class);
});
```

### 5.3 Route Parameter Doğrulama

```php
// Route constraint
Route::get('products/{id}', [ProductController::class, 'show'])
    ->where('id', '[0-9]+')
    ->name('products.show');

// Global constraint
public function boot()
{
    Route::pattern('id', '[0-9]+');
}
```

## 6. Örnek Senaryolar

### 6.1 Route Sorun Tespiti

```php
// routes/api.php
Route::middleware(['route.debug'])->group(function () {
    Route::get('products/{id}', function ($id) {
        try {
            $product = Product::findOrFail($id);
            return response()->json($product);
        } catch (\Exception $e) {
            Log::channel('routes')->error('Product Route Hatası', [
                'id' => $id,
                'error' => $e->getMessage(),
                'trace' => $e->getTraceAsString()
            ]);

            return response()->json([
                'error' => 'Ürün bulunamadı',
                'debug_id' => uniqid()
            ], 404);
        }
    });
});
```

### 6.2 Route Performance İzleme

```php
// app/Http/Middleware/RoutePerformanceMiddleware.php
class RoutePerformanceMiddleware
{
    public function handle($request, \Closure $next)
    {
        $startTime = microtime(true);
        $response = $next($request);
        $duration = microtime(true) - $startTime;

        if ($duration > 1.0) { // 1 saniyeden uzun süren istekler
            Log::channel('performance')->warning('Yavaş Route', [
                'url' => $request->fullUrl(),
                'duration' => $duration,
                'route' => $request->route()->getName()
            ]);
        }

        return $response;
    }
}
```

### 6.3 Route Health Check

```php
// routes/api.php
Route::get('health', function () {
    $status = [
        'database' => DB::connection()->getPdo() ? 'connected' : 'failed',
        'cache' => Cache::store()->get('test-key') !== false,
        'routes' => Route::has('api.products.index'),
        'timestamp' => now()->toIso8601String()
    ];

    return response()->json($status);
});
```

## 7. Sorun Giderme Kontrol Listesi

1. Route Cache Kontrolü:

   - `php artisan route:clear`
   - `php artisan route:cache`

2. Route Listesi Kontrolü:

   - `php artisan route:list --path=your-path`

3. Log Dosyaları Kontrolü:

   - `storage/logs/routes.log`
   - `storage/logs/laravel.log`

4. Middleware Kontrolü:

   - `app/Http/Kernel.php` dosyasında middleware sırası
   - Route middleware grupları

5. CORS Kontrolü:

   - `config/cors.php` ayarları
   - OPTIONS istekleri kontrolü

6. Cache Kontrolü:

   - Browser cache
   - Server cache
   - Route cache

7. Server Kontrolü:
   - Apache/Nginx rewrite rules
   - `.htaccess` dosyası
   - Virtual host ayarları

Bu yapılandırmalar ve kontroller sayesinde:

- Route sorunlarını hızlıca tespit edebilir
- Performans sorunlarını izleyebilir
- Hataları loglayabilir
- Sorunları sistematik şekilde çözebilirsiniz
