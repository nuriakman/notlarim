# Laravel Vue ve Inertia ile öğrenci projesi

```bash
laravel new ogrenci-vue-inertia
cd ogrenci-vue-inertia
php artisan migrate
npm install
# npm run dev
composer run dev
# http://127.0.0.1:8000

## Model'ler tekil tanımlanır
## Model ve bağlı migration dosyasını oluştur
php artisan make:model Student -m
php artisan make:model Classes -m
php artisan make:model Section -m
## Model dosyası içinde bağlı tablolara hasMany ve belongsTo tanımları olmalı

## Factory'ler tekil tanımlanır
php artisan make:factory StudentFactory
php artisan make:factory ClassesFactory
php artisan make:factory SectionFactory
## Factory dosyası içinde defination() fonksiyonu tanımlanır
## Factory içinde fake() tanımları yer alır
## foreignId'ler \App\Models\Classes::factory() gibi kullanılır

## Seeder'ler çoğul tanımlanır
php artisan make:seeder StudentsTableSeeder
php artisan make:seeder ClassesTableSeeder
php artisan make:seeder SectionsTableSeeder

## Migration İşlemleri, önce tüm tabloları drop et
php artisan migrate:rollback
## Tüm tabloları tanımla
php artisan migrate
## Tüm seederleri tanımla
php artisan db:seed

## Tek tek yapacak olursak
php artisan db:seed --class=UsersTableSeeder
php artisan db:seed --class=ClassesTableSeeder
php artisan db:seed --class=SectionsTableSeeder
php artisan db:seed --class=StudentsTableSeeder


```

### Migration

```php
public function up(): void
{
    Schema::create('students', function (Blueprint $table) {
        $table->id();
        $table->foreignId('class_id')->constrained();
        $table->foreignId('section_id')->constrained();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamps();
    });
}
```

### Model

```php
class Section extends Model
{
    protected $fillable = [
        'name',
        'class_id',
    ];

    public function class()
    {
        return $this->belongsTo(Classes::class, 'class_id'); // Tekli İlişki
    }

    public function students()
    {
        return $this->hasMany(Student::class); // Çoklu İlişki
    }
}

```

### Factory

```php
class StudentFactory extends Factory
{
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            // 'class_id' => \App\Models\Classes::factory(),
            // 'section_id' => \App\Models\Sections::factory(),
        ];
    }
}

```

### Seeder

```php

```
