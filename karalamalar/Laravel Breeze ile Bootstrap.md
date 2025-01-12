# Notlar

## YAPILACAKLAR

- Laravel ile API hazırla
  - Örnek veri çek
  - Resource oluştur
  - Postman ile test et
  - Postman Collection oluştur
- Quasar ile Örnek proje hazırla
  - API çağrılarını buradan yap
- Authentication İşlemleri
  - Giriş
  - Kayıt
  - Profil
  - Sifre yenileme
- API Router'ını Guard ile koru

```bash
composer create-project laravel/laravel example-app
php artisan make:migration products

## seeder ekle
php artisan migrate
php artisan db:seed

php artisan make:controller Api/ProductController --api
php artisan make:model Product
# api.php hazırla
php artisan serve
# GET /api/products
# GET /api/products/{id}
# POST /api/products
# PUT /api/products/{id}
# DELETE /api/products/{id}
```

## Laravel Başlatma

```bash
## Local sunucuya Yerel Ağ'dan IP ile bağlanmak için
php artisan serve --host=192.168.85.138 --port=8000
```

## Laravel Breeze ile Bootstrap kullanımı

```bash
# 1. Yeni Laravel projesi oluştur
composer create-project laravel/laravel example-app

# 2. Proje dizinine git
cd example-app

# 3. Laravel UI paketini yükle
composer require laravel/ui

# 4. Bootstrap auth scaffolding'i oluştur
php artisan ui bootstrap --auth

# 5. NPM paketlerini yükle
npm install

# 6. Assets'leri derle
npm run dev
```

Bu adımlardan sonra:

Projeniz hazır olacak
Bootstrap temalı authentication sistemi kurulmuş olacak
Giriş, kayıt ve diğer auth sayfaları otomatik olarak oluşturulacak
Ayrıca, projeyi çalıştırmak için:

`php artisan serve`

komutunu kullanabilirsiniz.

Veritabanı ayarlarını da .env dosyasında yapılandırdıktan sonra:

`php artisan migrate`

komutu ile veritabanı tablolarını oluşturabilirsiniz.

## Laravel'de Neden `npm` kullanıyoruz?

### Geliştirme (Development) Sürecinde

- `npm run dev` komutu ile CSS ve JavaScript dosyalarını derleyip geliştirme ortamı için hazırlarız
- Bu komut, değişiklikleri anlık olarak izler ve otomatik günceller
- Kaynak haritaları (source maps) oluşturur, böylece hata ayıklama kolaylaşır

### Canlı Ortam (Production) için

- `npm run build` komutu kullanılır
- CSS ve JavaScript dosyaları sıkıştırılır (minify)
- Gereksiz boşluklar ve yorumlar kaldırılır
- Dosya boyutları küçültülür
- Tarayıcı önbellekleme için versiyonlama yapılır

### Paket Yönetimi

`npm install` ile:

- Bootstrap gibi CSS framework'leri
- JavaScript kütüphaneleri
- Font dosyaları
- İkon setleri
  gibi frontend kaynaklarını projemize dahil ederiz

Yani npm, modern web uygulamalarında frontend kaynaklarının:

- Yüklenmesi
- Derlenmesi
- Optimize edilmesi
- Paketlenmesi
  işlemlerini yönetmemizi sağlar.

## API Test İşlemleri

HTTPie harika bir seçim! Önce HTTPie'yi yükleyelim:

```bash
apt install httpie -y
```

### API Endpoint'leri ve Test Komutları

#### 1. Kayıt Olma (Register)

```bash
http POST http://localhost/api/register \
    name="Test Kullanıcı" \
    email=test@example.com \
    password=password123 \
    password_confirmation=password123
```

#### 2. Giriş Yapma (Login)

```bash
http POST http://localhost/api/login \
    email=test@example.com \
    password=password123
```

#### 3. Token ile İstek Yapma

```bash
# Login'den aldığınız token'ı kullanarak
export API_TOKEN="buraya_login_den_aldiginiz_token"

http GET http://localhost/api/user \
    "Authorization: Bearer $API_TOKEN"
```

#### 4. Ana Sayfa Verilerini Alma

```bash
http GET http://localhost/api/home \
    "Authorization: Bearer $API_TOKEN"
```

#### 5. Çıkış Yapma

```bash
http POST http://localhost/api/logout \
    "Authorization: Bearer $API_TOKEN"
```

### Alternatif Test Araçları

Postman veya Insomnia gibi GUI araçlarını da kullanabilirsiniz. İsterseniz size bir Postman Collection hazırlayabilirim.

### Development Server'ı Başlatma

```bash
php artisan serve
```

> Server <http://127.0.0.1:8000> adresinde çalışmaya başladı.

Artık yukarıdaki komutları kullanarak API'nizi test edebilirsiniz. İsterseniz Postman Collection oluşturabilirim veya başka bir test aracı önerebilirim.
