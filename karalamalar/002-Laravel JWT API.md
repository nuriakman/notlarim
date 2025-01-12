# Laravel JWT API Rehberi

## TLDR

```bash
# Projenin oluşturulması
laravel new jwt-api

# Proje klasörüne giriş
cd jwt-api

# 1. JWT paketini yükle
composer require php-open-source-saver/jwt-auth

# 2. Provider'ı yayınla
php artisan vendor:publish --provider="PHPOpenSourceSaver\JWTAuth\Providers\LaravelServiceProvider"

# 3. JWT secret oluştur
php artisan jwt:secret

# 4. Auth controller oluştur
php artisan make:controller Api/AuthController

```

- JWT Kurulumu
- User Model Hazırlığı
- Auth Config Ayarları
- Provider'ın Yayınlanması
- Middleware'ın Yayınlanması
- Route'ların Tanımlanması
- Auth Controller Oluşturma
- API Route'ların Tanımlanması
- Token Kullanımı İpuçları

## 1. JWT Kurulumu

**JWT Nedir?**

JWT (JSON Web Token), Web'de güvenli bir şekilde verilerin aktarılması için
kullanılan bir token tabanlı kimlik doğrulama sistemidir.

**Neden JWT'ye İhtiyaç Duyarız?**

- Geleneksel kimlik doğrulama yöntemlerinde, her istekte sunucu ile istemci
  arasında kimlik doğrulama bilgileri gönderilir. Bu durum, sunucuda kimlik
  doğrulama bilgisinin saklanması gerektiğinden, sunucunun yükünü artırmaktadır.
- JWT ile kimlik doğrulama bilgileri, sunucuda saklanmaz. Onun yerine, sunucu
  tarafında imzalanmış bir token oluşturulur. Bu token, istemci tarafında
  saklanır ve her istekte sunucu tarafına gönderilir.
- Sunucu tarafında, tokenın imzasını kontrol edilir. Eğer imza geçerli ise,
  tokenın içindeki kimlik doğrulama bilgileri kullanılır.

## **JWT vs Sanctum: Hangisini Seçmeliyim?**

**Sanctum:**

- Laravel'in resmi authentication sistemidir
- İki tür authentication sağlar:
  1. SPA'lar için cookie tabanlı session authentication
  2. Mobile/API uygulamaları için token tabanlı authentication
- Session tabanlı authentication kullandığında:
  - Oturum bilgisi sunucuda saklanır
  - CSRF koruması otomatik olarak sağlanır
  - Session'lar veritabanında veya Redis'te saklanabilir
- Token tabanlı authentication kullandığında:
  - Stateless API'ler oluşturabilirsiniz
  - Rate limiting özelliği built-in gelir
  - Token'lar veritabanında saklanır (güvenlik için)

**JWT:**

- Tamamen stateless çalışır (sunucuda hiçbir şey saklanmaz)
- Token'ın kendisi kimlik bilgilerini içerir
- Microservice mimarileri için daha uygundur
- Cross-domain authentication daha kolaydır
- Token içinde veri (claims) taşıyabilirsiniz

**Ne Zaman Sanctum Kullanmalısınız?**

1. Laravel + SPA (Vue, React vb.) uygulaması geliştiriyorsanız
2. Session bazlı güvenlik istiyorsanız
3. CSRF koruması önemliyse
4. Rate limiting özelliği istiyorsanız
5. Token'ların veritabanında saklanmasını istiyorsanız

**Ne Zaman JWT Kullanmalısınız?**

1. Tamamen stateless bir API istiyorsanız
2. Microservice mimarisi kullanıyorsanız
3. Token içinde veri taşımanız gerekiyorsa
4. Farklı domainler/servisler arası authentication gerekiyorsa
5. Yüksek ölçeklenebilirlik gerekiyorsa

Her iki yaklaşımın da güçlü yanları var. Seçiminizi projenizin gereksinimlerine
göre yapmalısınız.

**Öneriler:**

- Microservice mimarisi için -> JWT
- Monolitik Laravel + Vue.js SPA için -> Sanctum
- Mobile API + Web için -> İkisi de olur, ama Sanctum daha kolay
- Çok yüksek trafik ve ölçeklenebilirlik -> JWT

### JWT Paketinin Yüklenmesi

```bash
composer require php-open-source-saver/jwt-auth
```

#### Neden en yaygın paket olan `firebase/php-jwt` kullanılmadı?

`firebase/php-jwt` yerine `php-open-source-saver/jwt-auth` kullanmamın birkaç
önemli nedeni var:

- Laravel Entegrasyonu:
  - jwt-auth özellikle Laravel için tasarlanmıştır
  - Laravel'in auth sistemine doğrudan entegre olur
  - Guard, middleware gibi Laravel'in authentication özelliklerini hazır sunar
- Kullanım Kolaylığı:
  - firebase/php-jwt sadece JWT token oluşturma ve doğrulama yapar
  - Her şeyi manuel olarak yapmanız gerekir (token yönetimi, refresh token, auth
    sistemi vb.)
  - jwt-auth ise bunların hepsini hazır sunar
- Özellikler:
  - Otomatik token yenileme
  - Blacklist mekanizması
  - Token süresi yönetimi
  - Claims yönetimi
  - Laravel session entegrasyonu

Ancak haklısınız, `firebase/php-jwt` de iyi bir alternatif. `firebase/php-jwt`
kullanmak daha basit, ama daha fazla manuel iş gerektirecek bir yaklaşım
olacaktır.

### Provider'ın Yayınlanması

```bash
php artisan vendor:publish --provider="PHPOpenSourceSaver\JWTAuth\Providers\LaravelServiceProvider"
```

**Açıklama:**

Bu komut, JWT paketinin yapılandırma dosyasını yayınlar (publish). Detaylı
açıklayayım:

- Paketi yüklediğinizde (`composer require`) paket `vendor` klasörüne gelir
- Provider yayınla (Publish) işlemi ile paketin ayar dosyası (`config/jwt.php`)
  projenizin içine kopyalanır

- Ne Yapar?

  - `config/jwt.php` adında bir yapılandırma dosyası oluşturur
  - Bu dosya JWT ile ilgili tüm ayarları içerir

- Yapılandırma Dosyasında Neler Var?

  ```php
  return [
      'secret' => env('JWT_SECRET'),  // JWT imzalama anahtarı
      'ttl' => env('JWT_TTL', 60),    // Token geçerlilik süresi (dakika)
      'refresh_ttl' => 20160,         // Refresh token süresi (dakika)
      'algo' => 'HS256',              // İmzalama algoritması
      'blacklist_enabled' => true,    // Token karalistesi aktif/pasif
      // ... diğer ayarlar
  ];
  ```

- Neden Gerekli?
  - JWT'nin nasıl çalışacağını özelleştirebilmenizi sağlar
  - Token süreleri, algoritma seçimi gibi önemli ayarları yapabilirsiniz
  - Güvenlik ayarlarını kontrol edebilirsiniz
- Sonraki Adım
  - Bu dosya oluştuktan sonra `php artisan jwt:secret` komutu ile güvenli bir
    JWT anahtarı oluşturmanız gerekir
  - Bu anahtar `.env` dosyasına JWT_SECRET olarak kaydedilir
  - Bu adım, JWT entegrasyonunun temel yapılandırma aşamasıdır ve güvenli bir
    JWT implementasyonu için gereklidir.

### JWT Secret Key Oluşturma

```bash
php artisan jwt:secret
```

**Açıklama:**

- Bu komut, guvenli JWT anahtarı oluşturur ve `.env` dosyasına kaydeder
- `config/jwt.php` dosyasındaki `secret` ayarını otomatik olarak oluşturur
- Bu anahtar `.env` dosyasına JWT_SECRET olarak kaydedilir
- Bu adım, JWT entegrasyonunun temel yapılandırma aşamasıdır ve güvenli bir JWT
  implementasyonu için gereklidir.

## 2. User Model Hazırlığı

**Neden User Model'inde JWT için hazırlık yapmak gerekiyor?**

Kimlik bilgilerini JWT tarafında kullanmak için gerekiyor

Bu hazırlıkların amacı:

- User modelinin JWT ile çalışabilmesini sağlamak
- Token oluşturulurken hangi kullanıcı bilgilerinin kullanılacağını belirlemek
- Token içinde taşınacak ekstra bilgileri tanımlamak
- Örnek olarak, eğer token içinde ekstra bilgiler taşımak isterseniz:

  ```php
  /**
   * JWT için gerekli metodlar
   */
  public function getJWTIdentifier()
  {
    /*
      getJWTIdentifier() metodu, JWT token'ın subject'ini (sub) belirler.
      Bu metod return $this->getKey(); ile kullanıcının primary key'ini (genellikle id) döndürür.

      Bu identifier şunlar için kullanılır:
      - Token'ı belirli bir kullanıcıyla ilişkilendirmek
      - Token'dan kullanıcıyı bulmak
      - Token'ın kime ait olduğunu belirlemek
    */

    // return $this->getKey();
    return $this->id;
    // genelde id kullanılır çünkü: Benzersizdir, Değişmez, Integer olduğu için token boyutunu çok artırmaz

    /*
      Örnek JWT token decode edildiğinde:
      {
        "sub": "1",           // getJWTIdentifier()'dan gelen id
        "iat": 1704901631,    // Token oluşturma zamanı
        "exp": 1704905231,    // Token son kullanma zamanı
        "nbf": 1704901631,    // Token geçerlilik başlangıç zamanı
        "jti": "abc123...",   // Benzersiz token ID'si
        ...getJWTCustomClaims() // Ekstra claims buraya eklenir
      }
    */
  }

  public function getJWTCustomClaims()
  {
    return [];
    /*
      ÖRNEK KULLANIM:
        return [
          'name' => $this->name,
          'email' => $this->email,
          'role' => $this->role,         // Kullanıcı rolü
          'permissions' => $this->permissions, // Yetkiler
          'team_id' => $this->team_id,   // Takım bilgisi
          'is_admin' => $this->is_admin, // Admin mi?
          'created_at' => $this->created_at
        ];

    */
  }
  ```

`app/Models/User.php` dosyasında yapılacak değişiklikler:

```php
use PHPOpenSourceSaver\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    // ... diğer kodlar ...

    /**
     * JWT için gerekli metodlar
     */
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

## 3. Auth Config Ayarları

`config/auth.php` dosyasında:

```php
'defaults' => [
    'guard' => 'api',        // Varsayılan kimlik doğrulama yöntemi
    'passwords' => 'users',  // Şifre sıfırlama için kullanılacak provider
],

'guards' => [
    'api' => [
        'driver' => 'jwt',      // Kimlik doğrulama sürücüsü olarak JWT kullan
        'provider' => 'users',  // Kullanıcı bilgileri için users tablosunu kullan
    ],
],
```

Bunun yerine, `.env` dosyasında şu ayarlar eklenebilir:

```php
AUTH_GUARD=api
AUTH_PASSWORD_BROKER=users
```

**Bu durumda `config/auth.php` dosyası içeriği:**

```php
  'defaults' => [
    'guard' => env('AUTH_GUARD', 'web'),
    'passwords' => env('AUTH_PASSWORD_BROKER', 'users'),
  ],

  'guards' => [
    'api' => [
      'driver' => 'jwt',      // Kimlik doğrulama sürücüsü olarak JWT kullan
      'provider' => 'users',  // Kullanıcı bilgileri için users tablosunu kullan
    ],
  ],
```

**Açıklama:**

Bu ayarların pratikteki anlamı:

- API'ye gelen her istek JWT token ile doğrulanacak
- Token geçerliyse, kullanıcı bilgileri `users` tablosundan alınacak
- `Auth::user()` çağrıldığında _JWT token'daki kullanıcı bilgileri dönecek_

**Örnek Kullanım:**

```php
public function profile()
{
    $user = Auth::user(); // JWT token'dan kullanıcı bilgisini alır
    return response()->json($user);
}
```

## 4. Auth Controller Oluşturma

```bash
php artisan make:controller Api/AuthController
```

Controller içeriği:

```php
namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use App\Models\User;

class AuthController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth:api', ['except' => ['login', 'register']]);
    }

    /**
     * Kullanıcı kaydı
     */
    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:6',
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => Hash::make($request->password),
        ]);

        $token = Auth::login($user);
        return response()->json([
            'status' => 'success',
            'message' => 'Kullanıcı başarıyla oluşturuldu',
            'user' => $user,
            'authorisation' => [
                'token' => $token,
                'type' => 'bearer',
            ]
        ], 201);
    }

    /**
     * Giriş yapma
     */
    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|string|email',
            'password' => 'required|string',
        ]);

        $credentials = $request->only('email', 'password');

        $token = Auth::attempt($credentials);
        if (!$token) {
            return response()->json([
                'status' => 'error',
                'message' => 'Yetkisiz',
            ], 401);
        }

        $user = Auth::user();
        return response()->json([
            'status' => 'success',
            'user' => $user,
            'authorisation' => [
                'token' => $token,
                'type' => 'bearer',
            ]
        ]);
    }

    /**
     * Kullanıcı bilgilerini getir
     */
    public function me()
    {
        return response()->json([
            'status' => 'success',
            'user' => Auth::user(),
        ]);
    }

    /**
     * Çıkış yapma
     */
    public function logout()
    {
        Auth::logout();
        return response()->json([
            'status' => 'success',
            'message' => 'Başarıyla çıkış yapıldı',
        ]);
    }

    /**
     * Token yenileme
     */
    public function refresh()
    {
        return response()->json([
            'status' => 'success',
            'user' => Auth::user(),
            'authorisation' => [
                'token' => Auth::refresh(),
                'type' => 'bearer',
            ]
        ]);
    }
}
```

**AuthController ne işe yarıyor? Neden kullanıyoruz?:**

- Kayıt (Register)
  - Yeni kullanıcı kaydı
  - Şifre hashleme
  - İlk token oluşturma
- Giriş (Login)
  - Kullanıcı bilgilerini doğrulama
  - JWT token oluşturma
  - Token'ı kullanıcıya gönderme
- Token Yenileme (Refresh)
  - Süresi dolmak üzere olan token'ı yenileme
  - Yeni token oluşturma
  - Eski token'ı geçersiz kılma
- Çıkış (Logout)
  - Aktif token'ı geçersiz kılma
  - Token'ı blacklist'e ekleme
  - Oturumu sonlandırma
- Profil (Me)
  - Token'dan kullanıcı bilgilerini okuma
  - Aktif kullanıcı bilgilerini döndürme
- Bu controller sayesinde:
  - Güvenli bir kimlik doğrulama sistemi kurulur
  - Token yönetimi merkezi bir yerden yapılır
  - Kullanıcı işlemleri standart hale gelir

## 5. API Route'larının Tanımlanması

`routes/api.php` dosyasında:

```php
use App\Http\Controllers\Api\AuthController;

// Route::prefix('auth')->group(function () { // Prefix kullanmak istersek bu satır
// Route::group(function () {
Route::middleware('auth:api')->group(function () {
    Route::post('register', [AuthController::class, 'register'])->name('auth.register');
    Route::post('login',    [AuthController::class, 'login']   )->name('auth.login');
    Route::post('logout',   [AuthController::class, 'logout']  )->name('auth.logout');
    Route::post('refresh',  [AuthController::class, 'refresh'] )->name('auth.refresh');
    Route::get('me',        [AuthController::class, 'me']      )->name('auth.me');
});
```

`routes/api.php` dosyasında korumalı ve korumasız route tanımı

**NOT:** Korunmak istenilen route'lar `auth:api` middleware'ine sahip
olmalıdır!!!

```php
// JWT korumalı route'lar
Route::middleware('auth:api')->group(function () {
    Route::post('register', [AuthController::class, 'register'])->name('auth.register');
    Route::post('login',    [AuthController::class, 'login']   )->name('auth.login');
    Route::post('logout',   [AuthController::class, 'logout']  )->name('auth.logout');
    Route::post('refresh',  [AuthController::class, 'refresh'] )->name('auth.refresh');
    Route::get('me',        [AuthController::class, 'me']      )->name('auth.me');
});

// Korumasız route'lar
Route::post('register', [AuthController::class, 'register'])->name('auth.register');
Route::post('login',    [AuthController::class, 'login']   )->name('auth.login');
```

## 6. CURL ile API Endpoint'lerinin Kullanımı

### Kayıt Olma

```bash
curl -X POST http://localhost:8000/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test User",
    "email": "test@example.com",
    "password": "123456"
  }'
```

### Giriş Yapma

```bash
curl -X POST http://localhost:8000/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "123456"
  }'
```

### Kullanıcı Bilgilerini Alma

```bash
curl -X GET http://localhost:8000/api/me \
  -H "Authorization: Bearer {token}"
```

### Token Yenileme

```bash
curl -X POST http://localhost:8000/api/refresh \
  -H "Authorization: Bearer {token}"
```

### Çıkış Yapma

```bash
curl -X POST http://localhost:8000/api/logout \
  -H "Authorization: Bearer {token}"
```

## 7. Korumalı Route Oluşturma

Örnek bir korumalı route:

```php
// routes/api.php
Route::middleware('auth:api')->group(function () {
    Route::get('protected-route', function () {
        return response()->json(['message' => 'Bu route korumalıdır']);
    });
});

// Veya
Route::get('me', [AuthController::class, 'me'])
    ->middleware('auth:api')
    ->name('auth.me');
```

## 8. Middleware Kullanımı

Controller'larda auth middleware'ini kullanma:

```php
public function __construct()
{
    $this->middleware('auth:api');
    // veya belirli metodlar için:
    $this->middleware('auth:api', ['except' => ['index', 'show']]);
    /*
      public function login() { ... }     // ✅ Token kontrolü YOK
      public function register() { ... }  // ✅ Token kontrolü YOK
      public function me() { ... }        // ❌ Token kontrolü VAR
      public function logout() { ... }    // ❌ Token kontrolü VAR
    */
}
```

Controller'ın `construct()`'ında auth middleware'ini kullanma yaklaşımının
avantajları:

- Her route'da tek tek middleware tanımlamaya gerek kalmaz
- Güvenlik kontrolü unutulma riski ortadan kalkar
- Kod tekrarı önlenir
- Merkezi yönetim sağlanır

Yani route'larda tekrar middleware('auth:api') yazmaya gerek yok, controller
seviyesinde zaten tanımlı.

## 9. Token Kullanımı İpuçları

1. Token'ı her zaman "Bearer" öneki ile kullanın:

```text
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbG...
```

2. Token'ı güvenli bir şekilde saklayın (örn: localStorage yerine httpOnly
   cookie)

3. Token süresini config/jwt.php'den ayarlayabilirsiniz:

```php
'ttl' => env('JWT_TTL', 60) // dakika cinsinden
```

## 10. Hata Yönetimi

`app/Exceptions/Handler.php` dosyasında JWT hatalarını yönetme:

```php
use PHPOpenSourceSaver\JWTAuth\Exceptions\TokenExpiredException;
use PHPOpenSourceSaver\JWTAuth\Exceptions\TokenInvalidException;
use PHPOpenSourceSaver\JWTAuth\Exceptions\JWTException;

public function render($request, Throwable $exception)
{
    if ($exception instanceof TokenExpiredException) {
        return response()->json(['error' => 'Token süresi doldu'], 401);
    }

    if ($exception instanceof TokenInvalidException) {
        return response()->json(['error' => 'Token geçersiz'], 401);
    }

    if ($exception instanceof JWTException) {
        return response()->json(['error' => 'Token bulunamadı'], 401);
    }

    return parent::render($request, $exception);
}
```

## 11. Postman Collection

API'nizi test etmek için Postman collection örneği:

```json
{
  "info": {
    "name": "Laravel JWT API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Register",
      "request": {
        "method": "POST",
        "url": "http://localhost:8000/api/register",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n    \"name\": \"Test User\",\n    \"email\": \"test@example.com\",\n    \"password\": \"123456\"\n}"
        }
      }
    },
    {
      "name": "Login",
      "request": {
        "method": "POST",
        "url": "http://localhost:8000/api/login",
        "header": [
          {
            "key": "Content-Type",
            "value": "application/json"
          }
        ],
        "body": {
          "mode": "raw",
          "raw": "{\n    \"email\": \"test@example.com\",\n    \"password\": \"123456\"\n}"
        }
      }
    }
  ]
}
```

## JWT İçeriğine Erişim

JWT Token içinde saklanan bilgilere token decode edildiğinde erişilebilir:

```php
// Controller'da kullanımı
$token = auth()->payload();
$name = $token->get('name');
$role = $token->get('role');

// veya
$claims = $token->toArray();
```

**Ancak dikkat edilmesi gerekenler:**

- Token boyutu büyüdükçe her request'te gönderilen veri miktarı artar
- Hassas bilgiler token içine konulmamalı
- Sık değişen bilgiler token içinde tutulmamalı (çünkü token yenilenene kadar
  eski değer kalır)

Bu yüzden genelde:

- Sadece authentication/authorization için gerekli bilgiler eklenir
- Değişmeyen veya az değişen bilgiler tercih edilir
- Token boyutu makul seviyede tutulur

---

---

---

## JWT ile yetkilendirme için örnek bir route tanımlaması

Bunun için önce middleware ve test route'unu oluşturmamız gerekiyor.

İlk olarak CheckUserLevel adında bir middleware oluşturalım:

```bash
php artisan make:middleware CheckUserLevel
```

```php
Route::get('/protected', function () {
    return response()->json(['message' => 'Bu route korumalıdır']);
})->middleware('auth:api');
```
