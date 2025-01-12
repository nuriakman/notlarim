# Laravel JWT ile API Yetkilendirme

## TLDR

- Laravel'de JWT (JSON Web Token) kullanarak API yetkilendirmesi yapılır.
- Kurulum için composer paketi yüklenecek ve config dosyası yayınlacaktır.
- Model'a yetkilendirme özelliği eklenir.
- Auth konfigürasyonu yapılacak ve controller yapısında gerekli ayarlamalar gerçekleştirilecektir.
- Middleware ile koruma sağlanacak ve API kullanım örnekleri verilecektir.

## İçindekiler

Bu notta şu konulara yer verilmiştir:

- JWT'nin ne olduğu
- Kurulum adımları
- Model yapılandırması
- Auth konfigürasyonu
- Controller yapısı
- Route tanımlamaları
- Middleware kullanımı
- API kullanım örnekleri
- Güvenlik ipuçları
- Hata yönetimi

## JWT Nedir?

JWT (JSON Web Token), kullanıcı kimlik doğrulaması için kullanılan, içerisinde kullanıcıya ait bilgileri taşıyan ve güvenli bir şekilde şifrelenmiş token yapısıdır.

## Kurulum Adımları

**1. JWT paketini yükleme:**

```bash
composer require php-open-source-saver/jwt-auth
```

**2. Config dosyasını yayınlama:**

```bash
php artisan vendor:publish --provider="PHPOpenSourceSaver\JWTAuth\Providers\LaravelServiceProvider"
```

**3. JWT secret key oluşturma:**

```bash
php artisan jwt:secret
```

## Model Hazırlığı

`User` modelinde `JWTSubject` interface'ini implement etmemiz gerekir:

```php
use PHPOpenSourceSaver\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

## Auth Config Ayarları

`config/auth.php` dosyasında guard ayarlarını yapılandırma:

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

## Controller Yapısı

```php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class AuthController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:api', ['except' => ['login', 'register']]);
    }

    // Kullanıcı Girişi
    public function login(Request $request)
    {
        $credentials = $request->validate([
            'email' => 'required|email',
            'password' => 'required|string',
        ]);

        if (!$token = auth()->attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    // Token ile Kullanıcı Bilgisi
    public function me()
    {
        return response()->json(auth()->user());
    }

    // Çıkış Yapma
    public function logout()
    {
        auth()->logout();
        return response()->json(['message' => 'Successfully logged out']);
    }

    // Token Yenileme
    public function refresh()
    {
        return $this->respondWithToken(auth()->refresh());
    }

    // Token Response
    protected function respondWithToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
}
```

## Route Tanımlamaları

`routes/api.php` dosyasında:

```php
Route::group(['prefix' => 'auth'], function () {
    Route::post('login', [AuthController::class, 'login']);
    Route::post('logout', [AuthController::class, 'logout']);
    Route::post('refresh', [AuthController::class, 'refresh']);
    Route::get('me', [AuthController::class, 'me']);
});

// Korumalı rotalar
Route::group(['middleware' => 'auth:api'], function () {
    // Sadece giriş yapmış kullanıcıların erişebileceği rotalar
});

// Örnek Yetki Kontrolleri
Route::group(['middleware' => ['auth:api', 'user.level:1']], function () {
    Route::get('/test', function () {
        return response()->json(['message' => 'Level 1 yetkisi ile erişildi']);
    });
});

Route::group(['middleware' => ['auth:api', 'user.level:2']], function () {
    Route::get('/yetki', function () {
        return response()->json(['message' => 'Level 2 yetkisi ile erişildi']);
    });
});
```

## Middleware Kullanımı

1. `Authenticate` middleware'i token kontrolü yapar
2. Özel yetki kontrolleri için yeni middleware'ler oluşturabilirsiniz:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckUserLevel
{
    public function handle(Request $request, Closure $next, int $requiredLevel = 1)
    {
        if (auth()->check() && auth()->user()->level >= $requiredLevel) {
            return $next($request);
        }

        return response()->json(['error' => 'Yetersiz yetki seviyesi'], 403);
    }
}
```

Middleware'i Kernel'a kaydetme:

```php
protected $middlewareAliases = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth' => \App\Http\Middleware\Authenticate::class,
    'user.level' => \App\Http\Middleware\CheckUserLevel::class,
];
```

## API Kullanımı

**1. Login isteği:**

```bash
POST /api/auth/login
{
    "email": "user@example.com",
    "password": "password"
}
```

**2. Token kullanımı:**

```bash
GET /api/auth/me
Authorization: Bearer {token}
```

## Güvenlik İpuçları

1. Token süresini kısa tutun (`config/jwt.php` içinde `ttl` değeri)
2. HTTPS kullanın
3. Hassas bilgileri token içinde saklamayın
4. Token'ı güvenli bir şekilde saklayın (örn: HttpOnly cookie)
5. CORS ayarlarını doğru yapılandırın

## Hata Yönetimi

```php
try {
    // JWT işlemleri
} catch(\PHPOpenSourceSaver\JWTAuth\Exceptions\TokenExpiredException $e) {
    return response()->json(['error' => 'Token süresi doldu'], 401);
} catch(\PHPOpenSourceSaver\JWTAuth\Exceptions\TokenInvalidException $e) {
    return response()->json(['error' => 'Geçersiz token'], 401);
} catch(\PHPOpenSourceSaver\JWTAuth\Exceptions\JWTException $e) {
    return response()->json(['error' => 'Token bulunamadı'], 401);
}
```

### Login İşlemi

1. **Controller Hazırlığı**

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Models\User;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        // Validasyon
        $request->validate([
            'email' => 'required|email',
            'password' => 'required|string',
        ]);

        // Kimlik doğrulama
        if (!$token = auth()->attempt($request->only('email', 'password'))) {
            return response()->json([
                'status' => 'error',
                'message' => 'Geçersiz kimlik bilgileri'
            ], 401);
        }

        // Token oluşturma ve dönüş
        return $this->createNewToken($token);
    }

    protected function createNewToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60,
            'user' => auth()->user()
        ]);
    }
}
```

2. **Login Endpoint Kullanımı**

```bash
POST /api/auth/login
Content-Type: application/json

{
    "email": "user@example.com",
    "password": "123456"
}
```

3. **Başarılı Login Yanıtı**

```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "user@example.com",
    "level": 2
  }
}
```

### Logout İşlemi

1. **Controller Metodu**

```php
public function logout()
{
    try {
        auth()->logout();
        return response()->json([
            'status' => 'success',
            'message' => 'Başarıyla çıkış yapıldı'
        ]);
    } catch (\Exception $e) {
        return response()->json([
            'status' => 'error',
            'message' => 'Çıkış yapılırken bir hata oluştu'
        ], 500);
    }
}
```

2. **Logout Endpoint Kullanımı**

```bash
POST /api/auth/logout
Authorization: Bearer {token}
```

3. **Token Yenileme**

```php
public function refresh()
{
    try {
        return $this->createNewToken(auth()->refresh());
    } catch (\Exception $e) {
        return response()->json([
            'status' => 'error',
            'message' => 'Token yenilenirken bir hata oluştu'
        ], 401);
    }
}
```

### Güvenlik Notları

1. **Token Güvenliği**

   - Token'ları güvenli bir şekilde saklayın (localStorage yerine httpOnly cookie tercih edin)
   - Token süresini (TTL) uygulamanızın ihtiyacına göre ayarlayın
   - Hassas bilgileri token içinde saklamaktan kaçının

2. **Rate Limiting**
   - Login denemelerini sınırlandırın
   - Brute force saldırılarına karşı önlem alın

---

---

---
