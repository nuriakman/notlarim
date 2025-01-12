# Routers

## `routes/api.php` dosyasına Route'lar Eklemek

```php
// Her resource için ayrı ayrı
Route::apiResource('products', ProductController::class);
Route::apiResource('categories', CategoryController::class);

// Birden fazla resource için
Route::apiResources([
    'products' => ProductController::class,
    'categories' => CategoryController::class
]);

// Auth gerektiren route'lar için
Route::middleware('auth:api')->group(function () {
    Route::apiResource('products', ProductController::class);
});
```

**`Route::apiResource()` şu route'ları oluşturur**

```text
GET    /api/products          - index()   -> Listeleme
POST   /api/products          - store()   -> Oluşturma
GET    /api/products/{id}     - show()    -> Görüntüleme
PUT    /api/products/{id}     - update()  -> Güncelleme
DELETE /api/products/{id}     - destroy() -> Silme
```

**Route'ları görmek için:**

```php
php artisan route:list
```
