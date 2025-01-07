# Seeder - Faker - Factory

## Soru

Seeder'ı sadece özeelikle kullanacağımız çoğunlukla az sayıdaki veriler için (Örnek: Süperadmin tanımı) gibi kullanmalı, Factory'i ise çok sayıda faker ihtiyacımız olduğunda mı kullanmalıyız?

### CEVAP: **Seeder Kullanımı: Az ve Belirli Veriler İçin**

Seeder, belirli ve genellikle manuel olarak kontrol edilen verileri veritabanına eklemek için idealdir. Bu genellikle uygulamanın başlangıç durumunda ihtiyaç duyduğu, sabit veya az sayıda özel verilerdir. Örnekler:

1. **Süper Admin Kullanıcısı Tanımı:**
   - Uygulamayı ilk kurduğunuzda bir süper admin hesabı oluşturmak.
2. **Sabit Kategoriler:**
   - Ürün kategorileri, roller veya izinler gibi sabit veriler.
3. **Konfigürasyon Tabloları:**
   - Döviz türleri, ülkeler veya diller gibi sabit listeleme tabloları.

#### Örnek Seeder

```php
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run()
    {
        DB::table('users')->insert([
            'name' => 'Super Admin',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
            'role' => 'superadmin',
        ]);
    }
}
```

---

### **Factory Kullanımı: Çok Sayıda ve Dinamik Veriler İçin**

Factory, genellikle test veya geliştirme sırasında, rastgele verilerle tabloları doldurmak için kullanılır. Faker ile birleştirildiğinde, her seferinde rastgele ve gerçekçi görünüme sahip çok sayıda veri oluşturabilir.

#### Örnek Senaryolar

1. **Test Verileri:**
   - 100 kullanıcı oluşturmak.
   - 500 sipariş veya 1000 blog yazısı eklemek.
2. **Geliştirme Ortamı:**
   - Geliştiriciler için dinamik içerikler oluşturmak.
3. **İlişkisel Veriler:**
   - Örneğin, her kullanıcıya 5 yorum atamak.

#### Örnek Factory

```php
use App\Models\User;

User::factory(100)->create(); // 100 kullanıcı oluşturur
```

---

### **Seeder ve Factory Birlikte Kullanımı**

Bazı durumlarda Seeder ve Factory birlikte kullanılabilir. Örneğin:

- Seeder ile bir süper admin oluştururken, diğer test kullanıcılarını Factory ile dinamik olarak ekleyebilirsiniz.

#### Örnek

```php
use Illuminate\Database\Seeder;
use App\Models\User;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        // Süper Admin
        User::create([
            'name' => 'Super Admin',
            'email' => 'admin@example.com',
            'password' => bcrypt('password'),
            'role' => 'superadmin',
        ]);

        // Rastgele Kullanıcılar
        User::factory(50)->create(); // 50 rastgele kullanıcı
    }
}
```

---

### **Özet**

- **Seeder:** Az sayıda, sabit ve özel veriler için uygundur.
- **Factory:** Çok sayıda, rastgele ve dinamik veriler için uygundur.

Seeder ve Factory'nin bu şekilde kullanılması, veritabanı hazırlığı sürecini daha düzenli ve verimli hale getirir.

### Seeder Çalıştırma

```bash
# Tüm Seeder'ları Çalıştırmak
php artisan db:seed

# Belirli Seeder'ları Çalıştırmak
php artisan db:seed --class=DatabaseSeeder
php artisan db:seed --class=UsersTableSeeder

# Veritabanını Sıfırlayıp Seeder ve Factory Çalıştırmak
php artisan migrate:fresh --seed

```

### Örnek Kullanımlar

```php
# 1000 adet kayıt oluşturma
for ($i = 0; $i < 1000; $i++) {
    DB::table('orders')->insert([
        'order_number' => Str::random(10),
        'amount' => rand(100, 1000),
        'created_at' => now(),
        'updated_at' => now(),
    ]);
}

# Statik kayıt oluşturma
DB::table('posts')->insert([
    'title' => 'Post 1',
    'body' => 'Lorem ipsum',
    'user_id' => 1,
]);

DB::table('posts')->insert([
    'title' => 'Post 2',
    'body' => 'Lorem ipsum',
    'user_id' => 2,
]);

# Faker kullanarak 10 adet kayıt oluşturma
\App\Models\Category::factory()->count(10)->create([
    'name' => fake()->word,
    'city' => fake()->city,
    'created_at' => now(),
    'updated_at' => now(),
]);

```

## BÜYÜK ÖRNEK

Şehirler tablosunda 10 adet farklı şehir var. Kullanıcılar tablosunda 1000 kullanıcı var. Fakat, Kullanıcı ve Şehir tablosu birbirine bağlı. Bunun için örnek veri üretimi yapar mısın?

**Şehirler ve Kullanıcılar tablolarını birbirine bağlamak için ilişkiyi simüle eden örnek veri üretimi** yapabiliriz. Laravel'de bunu yapmak için Seeder ve Factory'leri kullanabilirsiniz.

Aşağıdaki örnek senaryo üzerinden ilerleyelim:

- **Şehirler tablosu**: 10 adet farklı şehir (örneğin: `id`, `name` alanları).
- **Kullanıcılar tablosu**: 1000 kullanıcı, her bir kullanıcı `city_id` alanı ile bir şehre bağlı.

---

### **Adım 1: Şehir Factory Tanımı**

`database/factories/CityFactory.php`:

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class CityFactory extends Factory
{
    public function definition()
    {
        return [
            'name' => $this->faker->city, // Rastgele şehir isimleri
        ];
    }
}
```

---

### **Adım 2: Kullanıcı Factory Tanımı**

`database/factories/UserFactory.php`:

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    public function definition()
    {
        return [
            'name' => $this->faker->name,
            'email' => $this->faker->unique()->safeEmail,
            'password' => bcrypt('password'), // Şifre varsayılan olarak "password"
            'city_id' => \App\Models\City::inRandomOrder()->first()->id, // Rastgele bir şehir atanır
        ];
    }
}
```

---

### **Adım 3: Şehir Seeder'ı**

`database/seeders/CitySeeder.php`:

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\City;

class CitySeeder extends Seeder
{
    public function run()
    {
        City::factory()->count(10)->create(); // 10 şehir oluştur
    }
}
```

---

### **Adım 4: Kullanıcı Seeder'ı**

`database/seeders/UserSeeder.php`:

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\User;

class UserSeeder extends Seeder
{
    public function run()
    {
        User::factory()->count(1000)->create(); // 1000 kullanıcı oluştur
    }
}
```

---

### **Adım 5: DatabaseSeeder Tanımı**

`database/seeders/DatabaseSeeder.php`:

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            CitySeeder::class,  // Şehir seeder'ı çalıştırılır
            UserSeeder::class,  // Kullanıcı seeder'ı çalıştırılır
        ]);
    }
}
```

---

### **Adım 6: Veri Tabanını Sıfırla ve Seed Et**

Aşağıdaki komut ile veritabanını sıfırlayıp seeder'ları çalıştırabilirsiniz:

```bash
php artisan migrate:fresh --seed
```

---

### **Sonuç**

- **City tablosunda 10 şehir** olacak.
- **User tablosunda 1000 kullanıcı** olacak.
- Her kullanıcı `city_id` alanı ile bir şehre bağlanacak ve rastgele bir şehir seçilecek.

Bu yapı, şehir ve kullanıcı tablolarını birbirine bağlayan ilişkisel bir veri modelini simüle eder.

## Laravel'de Sınıf Çağırma

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\City;

class CitySeeder extends Seeder
{
    public function __construct()
    {
        // Constructor burada çalışır ve başlangıç işlemleri yapılabilir
        echo "CitySeeder Constructor Çalıştı!\n";
    }

    public function run()
    {
        // Burada veri ekleme işlemi yapılır
        echo "CitySeeder run() Çalıştı!\n";
        City::factory()->count(10)->create();
    }
}

```

**Çalışma Akışı:**

### ÖRNEK ÇAĞIRMA 1: **`php artisan db:seed` komutunu çalıştırdığınızda**

- Laravel **önce** `CitySeeder` sınıfının örneğini oluşturur (constructor tetiklenir).
- Daha sonra **`run()` metodu tetiklenir** ve veritabanına veri eklenir.

**ÖRNEK Çıktı:**

```bash
CitySeeder Constructor Çalıştı!
CitySeeder run() Çalıştı!
```

### ÖRNEK ÇAĞIRMA 2: **$this->call** Kullanarak çağırma

```php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $this->call([
            CitySeeder::class,  // Şehir seeder'ı çalıştırılır
            UserSeeder::class,  // Kullanıcı seeder'ı çalıştırılır
        ]);
    }
}
```

**ÖRNEK Çıktı:**

```bash
CitySeeder Constructor Çalıştı!
CitySeeder run() Çalıştı!
UserSeeder Constructor Çalıştı!
UserSeeder run() Çalıştı!
```

**Özet**:

- **`__construct()`** metodunun çalışması Laravel'in **sınıfı örneklediği** anda tetiklenir
- **`run()` metodu**, **`php artisan db:seed` komutu ile** otomatik tetiklenir
- AYRICA, **`CitySeeder::class` komutu da**, seeder'in **`run()`** metodunu otomatik tetikler
