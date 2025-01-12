# Model Oluşturma

## JWT ve API Kullanırken Model Oluşturma

JWT API için sadece gerekli olanları oluşturmak için şu komutu kullanabiliriz:

```bash
php artisan make:model Product -mrc --api --requests
```

Bu komutun açıklaması:

- `-m` veya `--migration`: Database migration
- `-r` veya `--resource`: Resource/API controller
- `-c` veya `--controller`: Controller oluşturur
- `--api`: API controller (5 metod)
- `--requests`: Form Request sınıfları

Bu komut şunları oluşturur:

- Model: `app/Models/Product.php`
- Migration: `database/migrations/xxx_create_products_table.php`
- Controller: `app/Http/Controllers/Api/ProductController.php` (API metodları ile)
- Request'ler:
  - `app/Http/Requests/StoreProductRequest.php`
  - `app/Http/Requests/UpdateProductRequest.php`

Factory ve Seeder'a ihtiyaç yoksa (`-f`, `-s`) eklemeye gerek yok çünkü:

- Test verisi üretmeyeceğiz
- Database seed yapmayacağız
- Sadece API endpoint'leri oluşturacağız

Eğer API Resource (JSON dönüşüm) de istiyorsanız:

```bash
php artisan make:model Product -mrc --api --requests --resource
```

Bu komut ekstra olarak:

- `app/Http/Resources/ProductResource.php`
- `app/Http/Resources/ProductCollection.php`

dosyalarını da oluşturur.
