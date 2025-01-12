# Controller Oluşturma

```bash
php artisan make:controller Api/AuthController

# Temel parametreler
--resource    # 7 CRUD metodu ile oluşturur
--api         # 5 API CRUD metodu ile oluşturur
--invokable   # Tek bir __invoke metodu ile oluşturur
--model=Post  # Model ile birlikte oluşturur
--singleton   # Tekil resource controller oluşturur
--creatable   # store() ve create() metodlarını ekler
--requests    # Form Request sınıfları ile oluşturur

# Kombinasyonlar
--resource --model=Post           # Model bağlantılı resource controller
--api --model=Post               # Model bağlantılı API controller
--resource --requests --model=Post # Form Request'li ve Model'li resource controller

# Örnekler
php artisan make:controller ProductController --resource --model=Product
php artisan make:controller ProductController --resource --requests --model=Product

```

```bash
php artisan make:controller Api/AuthController
php artisan make:controller Api/AuthController --api
php artisan make:controller Api/AuthController --resource
```

Normal Controller (`php artisan make:controller Api/AuthController`):

- Boş bir controller oluşturur
- Sadece ihtiyacımız olan metodları ekleyeceğiz:
  - login()
  - register()
  - logout()
  - refresh()
  - me()
- Resource Controller (`--resource` ile):
- CRUD işlemleri için 7 metod oluşturur:
  - index()
  - create()
  - store()
  - show()
  - edit()
  - update()
  - destroy()

## API Resource Controller'da Neden 5 Metod Var?

`create()` ve `edit()` metodları geleneksel web uygulamaları için tasarlanmıştır:

1. **create()** - `GET /products/create`

   - Web'de: Yeni ürün ekleme formunu gösterir
   - API'de: Gereksiz, çünkü:
     - API'ler form HTML'i döndürmez
     - Frontend kendi form yapısını oluşturur
     - Direkt `store()` metodunu kullanır

2. **edit()** - `GET /products/{id}/edit`
   - Web'de: Ürün düzenleme formunu gösterir
   - API'de: Gereksiz, çünkü:
     - API'ler form HTML'i döndürmez
     - Frontend `show()` ile veriyi alır
     - Kendi form yapısında gösterir
     - Direkt `update()` metodunu kullanır

**Yani API'lerde kullanılan 5 temel metod**:

- `index()` - Listeleme
- `store()` - Oluşturma
- `show()` - Görüntüleme
- `update()` - Güncelleme
- `destroy()` - Silme

Bu yüzden API Resource Controller oluştururken genelde:

```bash
php artisan make:controller Api/ProductController --api
```

komutunu kullanırız. Bu komut sadece API için gerekli olan 5 metodu oluşturur.

---

## Controller Oluşturma komutlarının kullanımı

```bash
php artisan make:controller Api/AuthController
php artisan make:controller Api/AuthController --resource
php artisan make:controller Api/AuthController --api
```

**Basic Controller** - `php artisan make:controller Api/AuthController`

- Boş bir controller oluşturur
- Hiçbir hazır metod içermez
- Ne zaman kullanılır?
  - Özel metodlar yazacağımızda
  - Auth işlemleri gibi spesifik işlevler için
  - Resource/API yapısına uymayan özel durumlar için

**Resource Controller** - `php artisan make:controller Api/AuthController --resource`

- 7 CRUD metodunu içerir (index, create, store, show, edit, update, destroy)
- Ne zaman kullanılır?
  - Web uygulamaları için
  - Form sayfaları gerektiğinde (create, edit)
  - Blade view'lar ile çalışırken

**API Controller** - `php artisan make:controller Api/AuthController --api`

- 5 CRUD metodunu içerir (index, store, show, update, destroy)
- Form metodlarını içermez (create, edit)
- Ne zaman kullanılır?
  - API geliştirirken
  - Frontend ayrı olduğunda
  - JSON response döndüren endpoint'ler için

Bizim `JWT auth` için `php artisan make:controller Api/AuthController` (basic) kullanmamız yeterli çünkü:

- Sadece auth metodlarını içerecek (login, register, logout, refresh, me)
- CRUD yapısına uymuyor
- Özel metodlar kullanacağız

## Örnek Controller

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Exception;

class ProductController extends Controller
{
    /**
     * Tüm ürünleri listele
     */
    public function index()
    {
        try {
            $products = [
                ['id' => 1, 'name' => 'Ürün 1'],
                ['id' => 2, 'name' => 'Ürün 2']
            ];

            return response()->json($products, Response::HTTP_OK); // 200
        } catch (Exception $e) {
            return response()->json(
                ['error' => 'Bir hata oluştu'],
                Response::HTTP_INTERNAL_SERVER_ERROR // 500
            );
        }
    }

    /**
     * Yeni ürün kaydet
     */
    public function store(Request $request)
    {
        try {
            // Validasyon
            $validated = $request->validate([
                'name' => 'required|min:3'
            ]);

            $product = ['id' => 3, 'name' => $request->name];

            return response()->json($product, Response::HTTP_CREATED); // 201

        } catch (ValidationException $e) {
            return response()->json(
                ['errors' => $e->errors()],
                Response::HTTP_UNPROCESSABLE_ENTITY // 422
            );
        } catch (Exception $e) {
            return response()->json(
                ['error' => 'Kayıt oluşturulamadı'],
                Response::HTTP_INTERNAL_SERVER_ERROR // 500
            );
        }
    }

    /**
     * Belirli bir ürünü göster
     */
    public function show($id)
    {
        try {
            // Dummy ürün kontrolü
            if ($id > 2) {
                return response()->json(
                    ['error' => 'Ürün bulunamadı'],
                    Response::HTTP_NOT_FOUND // 404
                );
            }

            $product = ['id' => $id, 'name' => 'Ürün ' . $id];

            return response()->json($product, Response::HTTP_OK); // 200

        } catch (Exception $e) {
            return response()->json(
                ['error' => 'Bir hata oluştu'],
                Response::HTTP_INTERNAL_SERVER_ERROR // 500
            );
        }
    }

    /**
     * Ürün güncelle
     */
    public function update(Request $request, $id)
    {
        try {
            // Dummy ürün kontrolü
            if ($id > 2) {
                return response()->json(
                    ['error' => 'Ürün bulunamadı'],
                    Response::HTTP_NOT_FOUND // 404
                );
            }

            // Validasyon
            $validated = $request->validate([
                'name' => 'required|min:3'
            ]);

            $product = ['id' => $id, 'name' => $request->name];

            return response()->json($product, Response::HTTP_OK); // 200

        } catch (ValidationException $e) {
            return response()->json(
                ['errors' => $e->errors()],
                Response::HTTP_UNPROCESSABLE_ENTITY // 422
            );
        } catch (Exception $e) {
            return response()->json(
                ['error' => 'Güncelleme yapılamadı'],
                Response::HTTP_INTERNAL_SERVER_ERROR // 500
            );
        }
    }

    /**
     * Ürün sil
     */
    public function destroy($id)
    {
        try {
            // Dummy ürün kontrolü
            if ($id > 2) {
                return response()->json(
                    ['error' => 'Ürün bulunamadı'],
                    Response::HTTP_NOT_FOUND // 404
                );
            }

            return response()->json(null, Response::HTTP_NO_CONTENT); // 204

        } catch (Exception $e) {
            return response()->json(
                ['error' => 'Silme işlemi başarısız'],
                Response::HTTP_INTERNAL_SERVER_ERROR // 500
            );
        }
    }
}

```
