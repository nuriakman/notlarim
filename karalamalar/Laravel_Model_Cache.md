# Laravel Model Cache Kullanımı

## İçindekiler

1. Genel Bakış
2. Kurulum ve Konfigürasyon
3. Cache Trait'i Oluşturma
4. Model'e Entegrasyon
5. Kullanım Örnekleri
6. Best Practices
7. Örnek Proje

## 1. Genel Bakış

Model Cache, Laravel uygulamalarında veritabanı sorgularını optimize etmek için kullanılan bir yöntemdir. Bu yöntem ile:

- Database yükü azaltılır
- Response süreleri kısalır
- Otomatik cache invalidation sağlanır
- Model bazlı cache yönetimi yapılır

## 2. Kurulum ve Konfigürasyon

### 2.1 Redis Kurulumu

```bash
composer require predis/predis
```

### 2.2 Cache Konfigürasyonu

```php
// config/cache.php
return [
    'default' => env('CACHE_DRIVER', 'redis'),

    'stores' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'cache',
            'lock_connection' => 'default',
        ],
    ],
];
```

### 2.3 Redis Konfigürasyonu

```php
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

## 3. Cache Trait'i Oluşturma

```php
// app/Traits/Cacheable.php
namespace App\Traits;

use Illuminate\Support\Facades\Cache;

trait Cacheable
{
    /**
     * Boot the trait
     */
    public static function bootCacheable()
    {
        // Model kaydedildiğinde
        static::saved(function ($model) {
            $model->forgetCache();
        });

        // Model silindiğinde
        static::deleted(function ($model) {
            $model->forgetCache();
        });

        // Model geri yüklendiğinde (soft delete)
        if (method_exists(static::class, 'restored')) {
            static::restored(function ($model) {
                $model->forgetCache();
            });
        }
    }

    /**
     * Cache key oluştur
     */
    public function getCacheKey()
    {
        return sprintf(
            '%s:%s:v1:%s',
            $this->getTable(),
            $this->getKey(),
            $this->updated_at ? $this->updated_at->timestamp : ''
        );
    }

    /**
     * Cache tag ismi al
     */
    public function getCacheTagName()
    {
        return $this->getTable();
    }

    /**
     * Cache'i temizle
     */
    public function forgetCache()
    {
        Cache::tags($this->getCacheTagName())->flush();
    }

    /**
     * Tüm kayıtları cache'den al
     */
    public static function getCached()
    {
        $instance = new static;

        return Cache::tags($instance->getCacheTagName())
            ->remember('all', now()->addDay(), function () {
                return static::all();
            });
    }

    /**
     * ID ile cache'den kayıt al
     */
    public static function findCached($id)
    {
        $instance = new static;

        return Cache::tags($instance->getCacheTagName())
            ->remember($id, now()->addDay(), function () use ($id) {
                return static::find($id);
            });
    }

    /**
     * Koşullu sorgu ile cache'den kayıt al
     */
    public static function whereCached($column, $value)
    {
        $instance = new static;
        $key = sprintf('%s:%s:%s', $instance->getTable(), $column, $value);

        return Cache::tags($instance->getCacheTagName())
            ->remember($key, now()->addDay(), function () use ($column, $value) {
                return static::where($column, $value)->get();
            });
    }

    /**
     * Sayfalama ile cache'den kayıt al
     */
    public static function paginateCached($perPage = 15)
    {
        $instance = new static;
        $page = request()->get('page', 1);
        $key = sprintf('%s:page:%s', $instance->getTable(), $page);

        return Cache::tags($instance->getCacheTagName())
            ->remember($key, now()->addDay(), function () use ($perPage) {
                return static::paginate($perPage);
            });
    }
}
```

## 4. Model'e Entegrasyon

```php
// app/Models/User.php
namespace App\Models;

use App\Traits\Cacheable;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use Cacheable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];
}
```

## 5. Kullanım Örnekleri

### 5.1 Controller'da Kullanım

```php
// app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    /**
     * Tüm kullanıcıları listele
     */
    public function index()
    {
        $users = User::getCached();
        return response()->json($users);
    }

    /**
     * Tek kullanıcı getir
     */
    public function show($id)
    {
        $user = User::findCached($id);
        return response()->json($user);
    }

    /**
     * Aktif kullanıcıları getir
     */
    public function active()
    {
        $users = User::whereCached('status', 'active');
        return response()->json($users);
    }

    /**
     * Sayfalı liste
     */
    public function paginated()
    {
        $users = User::paginateCached(20);
        return response()->json($users);
    }

    /**
     * Kullanıcı güncelle
     */
    public function update(Request $request, $id)
    {
        $user = User::find($id);
        $user->update($request->all());
        // Cache otomatik temizlenir
        return response()->json($user);
    }
}
```

### 5.2 API Route Tanımları

```php
// routes/api.php
Route::prefix('users')->group(function () {
    Route::get('/', [UserController::class, 'index']);
    Route::get('/{id}', [UserController::class, 'show']);
    Route::get('/status/active', [UserController::class, 'active']);
    Route::get('/paginated', [UserController::class, 'paginated']);
    Route::put('/{id}', [UserController::class, 'update']);
});
```

## 6. Best Practices

### 6.1 Cache Süreleri

```php
// Sabit süreler tanımlama
class CacheDuration
{
    const SHORT = 60; // 1 dakika
    const MEDIUM = 60 * 60; // 1 saat
    const LONG = 60 * 60 * 24; // 1 gün
    const WEEK = 60 * 60 * 24 * 7; // 1 hafta
}

// Kullanım
public static function getCached()
{
    return Cache::tags(self::getCacheTagName())
        ->remember('all', CacheDuration::MEDIUM, function () {
            return static::all();
        });
}
```

### 6.2 Race Condition Önleme

```php
public static function findCached($id)
{
    $instance = new static;
    $lock = Cache::lock('user:'.$id, 10);

    if ($lock->get()) {
        try {
            return Cache::tags($instance->getCacheTagName())
                ->remember($id, now()->addDay(), function () use ($id) {
                    return static::find($id);
                });
        } finally {
            $lock->release();
        }
    }
}
```

### 6.3 Cache Versiyonlama

```php
public function getCacheKey()
{
    return sprintf(
        '%s:%s:v2:%s', // v2 versiyon numarası
        $this->getTable(),
        $this->getKey(),
        $this->updated_at ? $this->updated_at->timestamp : ''
    );
}
```

## 7. Örnek Proje

### 7.1 Repository Pattern ile Kullanım

```php
// app/Repositories/UserRepository.php
namespace App\Repositories;

use App\Models\User;

class UserRepository
{
    public function getAllUsers()
    {
        return User::getCached();
    }

    public function getUserById($id)
    {
        return User::findCached($id);
    }

    public function getActiveUsers()
    {
        return User::whereCached('status', 'active');
    }

    public function updateUser($id, array $data)
    {
        $user = User::find($id);
        $user->update($data);
        return $user;
    }
}

// Controller'da kullanım
class UserController extends Controller
{
    protected $repository;

    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }

    public function index()
    {
        return response()->json($this->repository->getAllUsers());
    }
}
```

### 7.2 Service Layer ile Kullanım

```php
// app/Services/UserService.php
namespace App\Services;

use App\Repositories\UserRepository;

class UserService
{
    protected $repository;

    public function __construct(UserRepository $repository)
    {
        $this->repository = $repository;
    }

    public function getAllUsers()
    {
        return $this->repository->getAllUsers();
    }

    public function getUserProfile($id)
    {
        $user = $this->repository->getUserById($id);
        // Ek business logic...
        return $user;
    }
}
```

Bu yapı sayesinde:

- Veritabanı sorgularınız otomatik olarak cache'lenir
- Model güncellendiğinde cache otomatik temizlenir
- Performans optimizasyonu sağlanır
- Kod tekrarı önlenir
- Bakımı kolay bir yapı oluşur
