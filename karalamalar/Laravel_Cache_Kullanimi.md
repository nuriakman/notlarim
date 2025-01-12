# Laravel'de Cache Kullanımı

## İçindekiler

1. Cache Nedir?
2. Cache Sürücüleri
3. Temel Cache İşlemleri
4. Redis ile Cache
5. Cache Tags
6. Cache Events
7. Best Practices

## 1. Cache Nedir?

Cache (önbellek), sık kullanılan verilerin hızlı erişim için geçici olarak saklandığı bir mekanizmadır. Laravel'de cache kullanarak:

- Database sorgularını azaltabilir
- Uygulama performansını artırabilir
- Sunucu yükünü azaltabilir
- Response sürelerini kısaltabilir

## 2. Cache Sürücüleri

Laravel'de kullanılabilecek cache sürücüleri:

```php
// config/cache.php
'default' => env('CACHE_DRIVER', 'file'),

'stores' => [
    'apc' => [
        'driver' => 'apc',
    ],
    'array' => [
        'driver' => 'array',
        'serialize' => false,
    ],
    'file' => [
        'driver' => 'file',
        'path' => storage_path('framework/cache/data'),
    ],
    'memcached' => [
        'driver' => 'memcached',
        'persistent_id' => env('MEMCACHED_PERSISTENT_ID'),
    ],
    'redis' => [
        'driver' => 'redis',
        'connection' => 'cache',
        'lock_connection' => 'default',
    ],
],
```

## 3. Temel Cache İşlemleri

### 3.1 Veri Kaydetme

```php
// Basit kayıt
Cache::put('key', 'value', $seconds);

// Süresiz kayıt
Cache::forever('key', 'value');

// Yoksa kaydet
Cache::add('key', 'value', $seconds);

// Remember ile kayıt
Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});

// Forever remember
Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

### 3.2 Veri Okuma

```php
// Değer okuma
$value = Cache::get('key');

// Varsayılan değer ile okuma
$value = Cache::get('key', 'default');

// Closure ile varsayılan değer
$value = Cache::get('key', function () {
    return DB::table('users')->get();
});

// Birden fazla değer okuma
$values = Cache::many(['key1', 'key2']);
```

### 3.3 Veri Silme

```php
// Tek değer silme
Cache::forget('key');

// Tüm cache'i temizleme
Cache::flush();

// Süreli silme
Cache::put('key', 'value', now()->addMinutes(10));
```

## 4. Redis ile Cache

### 4.1 Redis Kurulumu

```bash
composer require predis/predis
```

### 4.2 Redis Konfigürasyonu

```php
// .env
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

// config/database.php
'redis' => [
    'client' => env('REDIS_CLIENT', 'predis'),
    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_DB', 0),
    ],
],
```

### 4.3 Redis Kullanımı

```php
// Redis ile veri kaydetme
Redis::set('key', 'value');
Redis::setex('key', $seconds, 'value');

// Redis ile veri okuma
$value = Redis::get('key');

// Redis ile hash işlemleri
Redis::hset('user:1', 'name', 'John');
Redis::hget('user:1', 'name');
```

## 5. Cache Tags

```php
// Tag ile cache kaydetme
Cache::tags(['people', 'artists'])->put('John', $john, $seconds);

// Tag ile cache okuma
$john = Cache::tags(['people', 'artists'])->get('John');

// Tag ile cache silme
Cache::tags(['people', 'artists'])->flush();
```

## 6. Cache Events

```php
// Cache event listener
class CacheEventSubscriber
{
    public function handleCacheHit($event) {}
    public function handleCacheMissed($event) {}
    public function handleKeyForgotten($event) {}
    public function handleKeyWritten($event) {}

    public function subscribe($events)
    {
        $events->listen(
            'cache.hit',
            [CacheEventSubscriber::class, 'handleCacheHit']
        );

        $events->listen(
            'cache.missed',
            [CacheEventSubscriber::class, 'handleCacheMissed']
        );
    }
}
```

## 7. Best Practices

### 7.1 Cache Key İsimlendirme

```php
// Prefix kullanımı
Cache::put('users:' . $userId, $userData, $seconds);

// Versiyon ekleme
Cache::put('users:v1:' . $userId, $userData, $seconds);

// Dinamik key oluşturma
public function getCacheKey($model, $id)
{
    return sprintf('%s:%s:v1', $model, $id);
}
```

### 7.2 Cache Süreleri

```php
// Sabit süreler tanımlama
const CACHE_SHORT = 60; // 1 dakika
const CACHE_MEDIUM = 60 * 60; // 1 saat
const CACHE_LONG = 60 * 60 * 24; // 1 gün

// Süre kullanımı
Cache::put('key', 'value', self::CACHE_MEDIUM);
```

### 7.3 Cache Gruplandırma

```php
// Model bazlı cache
class User extends Model
{
    public function getCacheKey()
    {
        return 'users:' . $this->id . ':v1';
    }

    public function cacheFor()
    {
        return now()->addHours(24);
    }
}
```

### 7.4 Race Condition Önleme

```php
// Atomic locks kullanımı
Cache::lock('processing')->get(function () {
    // İşlemler
});

// Timeout ile lock
$lock = Cache::lock('processing', 10);
if ($lock->get()) {
    // İşlemler
    $lock->release();
}
```

## 8. Örnek Kullanım Senaryoları

### 8.1 API Response Cache

```php
public function index()
{
    return Cache::remember('api.users', 60, function () {
        return User::all();
    });
}
```

### 8.2 View Cache

```php
// Blade view cache
@cache('key', $minutes)
    @include('expensive.view')
@endcache

// Controller'da view cache
public function show()
{
    $view = Cache::remember('view.home', 60, function () {
        return view('home', ['data' => $this->getData()])->render();
    });

    return $view;
}
```

### 8.3 Query Cache

```php
// Query sonuçlarını cache'leme
public function getUsers()
{
    return Cache::remember('users.all', 60, function () {
        return DB::table('users')
            ->select('id', 'name', 'email')
            ->where('active', 1)
            ->get();
    });
}
```
