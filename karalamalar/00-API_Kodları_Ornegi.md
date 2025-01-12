# API Örneği

## Kurulum

**Dosya İçerikleri (Sadece Ana Bölümler):**

```php
// File: app/Models/Product.php
    protected $fillable = [
        'name',
        'description',
        'price',
        'stock'
    ];

// File: 2025_01_11_165041_create_products_table.php
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->text('description');
        $table->decimal('price', 10, 2);
        $table->integer('stock')->default(0);
        $table->timestamps();
    });

// File: database/factories/ProductFactory.php
    public function definition(): array
    {
        return [
            'name' => fake()->words(3, true),
            'description' => fake()->paragraph(),
            'price' => fake()->randomFloat(2, 10, 1000),
            'stock' => fake()->numberBetween(0, 100),
            'created_at' => fake()->dateTimeBetween('-1 year'),
            'updated_at' => fake()->dateTimeBetween('-6 months'),
        ];
    }

// File: database/seeders/ProductSeeder.php
    public function run(): void
    {
        // Product::factory(100)->create();
        // veya
        for ($i = 0; $i < 100; $i++) {
            Product::factory()->create();
        }
    }

// File: database/seeders/DatabaseSeeder.php
    public function run(): void
    {
        $this->call([
            ProductSeeder::class
        ]);
    }

// File: app/Providers/RouteServiceProvider.php
    public function boot(): void
    {
      $this->routes(function () {
        Route::middleware('api')
          ->prefix('api')
          ->group(base_path('routes/api.php'));

        Route::middleware('web')
          ->group(base_path('routes/web.php'));
      });
    }

```

```bash
# UI bileşenleri olmadan, sadece API geliştirmesi için gereken temel yapıyla projeye başlayalım
composer create-project laravel/laravel api-example --prefer-dist
cd api-example

# Product için model, migration ve controller hazırla
php artisan make:model Product -mc --api

# Product için Model dosyası içindeki tanımlar
# (Dosya içeriği yukarıda)

# Product için migration dosyası içini düzenle
# (Dosya içeriği yukarıda)

# api için provider eklenmesi
# Bu komut ayrıca `bootstrap/providers.php` dosyasına da otomatik ekleme yapar
php artisan make:provider RouteServiceProvider
# (Dosya içeriği yukarıda)

# Product için factory ekle
php artisan make:factory ProductFactory
# (Dosya içeriği yukarıda)

# Product için seeder ekle
php artisan make:seeder ProductSeeder
# (Dosya içeriği yukarıda)

# Seedlerları çalıştır
php artisan db:seed

# api.php hazırla
# (Dosya içeriği yukarıda)

###########################################
########################################### Yardımcı Komutlar
###########################################

#  composer dump-autoload   //// ????

# Önce çalışan portları kontrol edelim:
lsof -i :8000-8010
# Varsa tüm sunucu servisleri kapat
for port in {8000..8010}; do lsof -i:$port; done
for port in {8000..8010}; do kill -9 $port; done

php artisan route:clear
php artisan cache:clear

php artisan route:list
php artisan route:list --path=api

php artisan serve

# GET /api/products
# GET /api/products/{id}
# POST /api/products
# PUT /api/products/{id}
# DELETE /api/products/{id}
```

## Notlar

---

## Faker / Sahte Veri Oluşturma

### Faker İçin Gerekli Dosyalar ve İçeriği

#### `/Database/Seeders/DatabaseSeeder.php` Dosyası İçeriği (Factory)

```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Database\Seeders\ProductSeeder;
// use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        $this->call([
            ProductSeeder::class,
        ]);
    }
}
$product = New Product();

$product->name = 'Laravel';
$product->description = 'The web framework for PHP';
$product->price = 10.99;
$product->stock = 100;
```

#### `/Database/Seeders/ProductSeeder.php` Dosyası İçeriği (Seeder)

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Faker\Factory as Faker;

class ProductSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $faker = Faker::create('tr_TR');

        for ($i = 0; $i < 100; $i++) {
            DB::table('products')->insert([
                'name' => $faker->word(3),
                'description' => $faker->sentence(3),
                'price' => $faker->randomFloat(2, 10, 1000),
                'stock' => $faker->numberBetween(0, 100),
                'created_at' => now(),
                'updated_at' => now()
            ]);
        }
    }
}

```

---

## API için Gerekli Dosyalar ve İçeriği

### `/app/Models/Product.php` Dosyası İçeriği (Model)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'description',
        'price',
        'stock'
    ];
}

```

---

### `/routes/api.php` Dosyası İçeriği (Rooter)

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\ProductController;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider and all of them will
| be assigned to the "api" middleware group. Make something great!
|
*/

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});


Route::prefix('/products')->group(function () {
    Route::get('test', [ProductController::class, 'test']);
    Route::put('bulk-update', [ProductController::class, 'bulkUpdate']);
    Route::apiResource('/', ProductController::class);
});

Route::fallback(function () {
    return response()->json([
        'status' => 'error',
        'message' => 'Rota bulunamadı'
    ], 400); //400: Bad Request
});

```

---

### `/app/Http/Controllers/Api/ProductController.php` Dosyası İçeriği (Controller)

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Validator;


class ProductController extends Controller
{
    /**
     * Display a listing of the resource.
     */
    public function index()
    {
        $products = Product::all();
        return response()->json([
            'status' => 'success',
            'data' => $products
        ]);
    }

    /**
     * Store a newly created resource in storage.
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required|string|max:255',
            'description' => 'required|string',
            'price' => 'required|numeric|min:0',
            'stock' => 'required|integer|min:0'
        ]);

        if ($validator->fails()) {
            return response()->json([
                'status' => 'error',
                'message' => 'Doğrulama hatası',
                'errors' => $validator->errors()
            ], 422);
        }

        $product = Product::create($request->all());

        return response()->json([
            'status' => 'success',
            'message' => 'Ürün başarıyla oluşturuldu',
            'data' => $product
        ], 201);
    }

    /**
     * Display the specified resource.
     */
    public function show($id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'status' => 'error',
                'message' => 'Ürün bulunamadı'
            ], 404);
        }

        return response()->json([
            'status' => 'success',
            'data' => $product
        ]);
    }

    /**
     * Update the specified resource in storage.
     */
    public function update(Request $request, $id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'status' => 'error',
                'message' => 'Ürün bulunamadı'
            ], 404);
        }

        $validator = Validator::make($request->all(), [
            'name' => 'string|max:255',
            'description' => 'string',
            'price' => 'numeric|min:0',
            'stock' => 'integer|min:0'
        ]);

        if ($validator->fails()) {
            return response()->json([
                'status' => 'error',
                'message' => 'Doğrulama hatası',
                'errors' => $validator->errors()
            ], 422);
        }

        $product->update($request->all());

        return response()->json([
            'status' => 'success',
            'message' => 'Ürün başarıyla güncellendi',
            'data' => $product
        ]);
    }

    /**
     * Remove the specified resource from storage.
     */
    public function destroy($id)
    {
        $product = Product::find($id);

        if (!$product) {
            return response()->json([
                'status' => 'error',
                'message' => 'Ürün bulunamadı'
            ], 404);
        }

        $product->delete();

        return response()->json([
            'status' => 'success',
            'message' => 'Ürün başarıyla silindi'
        ]);
    }

    public function bulkUpdate(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'products' => 'required|array',
            'products.*.id' => 'required|exists:products,id',
            'products.*.name' => 'string|max:255',
            'products.*.description' => 'string',
            'products.*.price' => 'numeric|min:0',
            'products.*.stock' => 'integer|min:0'
        ]);

        if ($validator->fails()) {
            return response()->json([
                'status' => 'error',
                'message' => 'Doğrulama hatası',
                'errors' => $validator->errors()
            ], 422);
        }

        $updatedProducts = [];

        DB::beginTransaction();
        try {
            foreach ($request->products as $productData) {
                $product = Product::find($productData['id']);
                $product->update($productData);
                $updatedProducts[] = $product;
            }
            DB::commit();

            return response()->json([
                'status' => 'success',
                'message' => count($updatedProducts) . ' adet ürün başarıyla güncellendi',
                'data' => $updatedProducts
            ]);
        } catch (\Exception $e) {
            DB::rollback();
            return response()->json([
                'status' => 'error',
                'message' => 'Güncelleme işlemi sırasında bir hata oluştu',
                'error' => $e->getMessage()
            ], 500);
        }
    }
}

```

---

### '/app/Providers/RouteServiceProvider.php' Dosyası İçeriği (API Service Provider)

```php
<?php

namespace App\Providers;

use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Foundation\Support\Providers\RouteServiceProvider as ServiceProvider;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Support\Facades\Route;

class RouteServiceProvider extends ServiceProvider
{
  /**
   * The path to your application's "home" route.
   *
   * Typically, users are redirected here after authentication.
   *
   * @var string
   */
  // public const HOME = '/home';

  /**
   * Define your route model bindings, pattern filters, and other route configuration.
   */
  public function boot(): void
  {
    // RateLimiter::for('api', function (Request $request) {
    //     return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    // });

    $this->routes(function () {
      Route::middleware('api')
        ->prefix('api')
        ->group(base_path('routes/api.php'));

      Route::middleware('web')
        ->group(base_path('routes/web.php'));
    });
  }
}

```

## API Çağrıları (API Calls)

### Tüm ürünleri listele (READ - List)

```bash
GET /api/products
```

### Yeni ürün oluştur (CREATE)

```bash
POST /api/products
Content-Type: application/json

{
    "name": "Örnek Ürün",
    "description": "Ürün açıklaması",
    "price": 99.99,
    "stock": 50
}
```

### Tek bir ürünü getir (READ - Single)

```bash
GET /api/products/{id}
```

### Ürün güncelle (UPDATE)

```bash
PUT /api/products/{id}
Content-Type: application/json

{
    "name": "Güncellenmiş Ürün",
    "price": 149.99
}
```

### Ürün sil (DELETE)

```bash
DELETE /api/products/{id}
```

### Birden fazla ürünü aynı anda güncellenmesi

```bash
PUT /api/products/bulk-update
Content-Type: application/json

{
    "products": [
        {
            "id": 1,
            "name": "Güncellenmiş Ürün 1",
            "price": 149.99
        },
        {
            "id": 2,
            "stock": 75
        },
        {
            "id": 3,
            "name": "Güncellenmiş Ürün 3",
            "description": "Yeni açıklama"
        }
    ]
}
```

## API Test

### POSTMAN ile çağırma Örneği

Postman'de bu API çağrısını yapmak için aşağıdaki adımları takip edebilirsiniz:

1. Postman'i açın ve yeni bir istek (request) oluşturun
2. İsteğin detaylarını şu şekilde ayarlayın:

   - HTTP Metodu: `POST` (dropdown menüden seçin)
   - URL: `http://localhost:8000/api/products`
   - Headers sekmesinde:
     - Key: `Content-Type`
     - Value: `application/json`
   - Body sekmesinde:
     - `raw` seçeneğini seçin
     - Sağ taraftaki dropdown menüden `JSON` formatını seçin
     - Aşağıdaki JSON verisini girin:

   ```json
   {
     "name": "Test Ürün",
     "description": "Test açıklama",
     "price": 199.99,
     "stock": 25
   }
   ```

3. "Send" butonuna tıklayarak isteği gönderebilirsiniz.

Başarılı bir istek sonucunda aşağıdaki gibi bir yanıt almalısınız:

```json
{
  "status": "success",
  "message": "Ürün başarıyla oluşturuldu",
  "data": {
    "name": "Test Ürün",
    "description": "Test açıklama",
    "price": 199.99,
    "stock": 25,
    "updated_at": "...",
    "created_at": "...",
    "id": 1
  }
}
```

### CURL ile çağırma Örneği - 1

```bash
curl -X POST http://localhost:8000/api/products \
     -H "Content-Type: application/json" \
     -d '{"name":"Test Ürün","description":"Test açıklama","price":199.99,"stock":25}'
```

### CURL ile çağırma Örneği - 2

```bash
curl -X PUT http://localhost:8000/api/products/bulk-update \
     -H "Content-Type: application/json" \
     -d '{
         "products": [
             {
                 "id": 1,
                 "name": "Güncellenmiş Ürün 1",
                 "price": 149.99
             },
             {
                 "id": 2,
                 "stock": 75
             }
         ]
     }'
```

## Test İçin Sahte API Sayfası

```php
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| API Routes
|--------------------------------------------------------------------------
|
| Here is where you can register API routes for your application. These
| routes are loaded by the RouteServiceProvider within a group which
| is assigned the "api" middleware group. Enjoy building your API!
|
*/

// API Root Test Endpoint
Route::get('/', function () {
    return response()->json([
        'message' => 'Welcome to the API root!',
        'status' => 'success',
    ]);
});

// Example GET Endpoint
Route::get('/example', function () {
    return response()->json([
        'message' => 'This is an example GET endpoint',
    ]);
});

// Example POST Endpoint
Route::post('/example', function (Request $request) {
    return response()->json([
        'message' => 'This is an example POST endpoint',
        'data' => $request->all(),
    ]);
});

// Example Route with Parameters
Route::get('/user/{id}', function ($id) {
    return response()->json([
        'message' => 'User data retrieved successfully',
        'user_id' => $id,
    ]);
});

// Example Route with Controller
Route::get('/products', [\App\Http\Controllers\Api\ProductController::class, 'index']);
Route::post('/products', [\App\Http\Controllers\Api\ProductController::class, 'store']);

```
