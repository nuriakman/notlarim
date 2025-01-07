# Örnek Kodlar

## Model1

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Student extends Model
{
    protected $fillable = [
        'name',
        'email',
        'class_id',
        'section_id',
    ];

    public function class()
    {
        return $this->belongsTo(Classes::class, 'class_id');
    }

    public function section()
    {
        return $this->belongsTo(Section::class, 'section_id');
    }
}

```

## Model2

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

use App\Models\Classes;
use App\Models\Student;

class Section extends Model
{
    protected $fillable = [
        'name',
        'class_id',
    ];

    public function class()
    {
        return $this->belongsTo(Classes::class, 'class_id');
    }

    public function students()
    {
        return $this->hasMany(Student::class);
    }
}

```

## Seeder1

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class UsersTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $users = [
            [
                'name' => 'Admin',
                'email' => 'admin@example.com',
                'password' => bcrypt('12345678'),
            ]
        ];

        foreach ($users as $user) {
            \App\Models\User::create($user);
        }
    }
}

```

## Seeder2

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class StudentsTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $faker = \Faker\Factory::create('tr_TR'); // faker nesnesinin Türkçe verilerle oluşturulmasını istiyoruz
        $classes = \App\Models\Classes::all(); // Tüm sınıfları getir
        $sections = \App\Models\Section::all(); // Tüm şubeleri getir

        for ($i = 1; $i <= 100; $i++) { // 100 tane öğrenci oluştur
            $randomClass = $classes->random(); // Rastgele bir Sınıf seç
            $randomSection = $sections->random(); // Rastgele bir şube seç

            $firstName = $faker->firstName;
            $lastName = $faker->lastName;

            \App\Models\Student::create([
                'name' => $firstName . ' ' . $lastName,
                'email' => strtolower($firstName) . '.' . strtolower($lastName) . $i . '@ogrenci.edu.tr',
                'class_id' => $randomClass->id,
                'section_id' => $randomSection->id
            ]);
        }
    }
}

```

## Seeder3

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class SectionsTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $classes = \App\Models\Classes::all(); // Tüm sınıfları getir
        $sections = ['A', 'B', 'C']; // Oluşturulacak şubeleri tanımla

        foreach ($classes as $class) { // Her bir sınıf için dön
            foreach ($sections as $section) { // Her sınıfın A,B,C şubeleri için dön
                \App\Models\Section::create([
                    'name' => $section . ' Şubesi',
                    'class_id' => $class->id,
                ]);
            }
        }
    }
}

```

## Seeder4

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use App\Models\Classes;
use App\Models\Section;
use App\Models\Student;
use Illuminate\Database\Eloquent\Factories\Sequence;

class ClassesTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        for ($i = 1; $i <= 12; $i++) { // 1..12 arası 12 tane sınıf oluştur
            \App\Models\Classes::create([
                'name' => $i . '. Sınıf',
            ]);
        }

        // Alternatif yöntem:
        // \App\Models\Classes::factory()
        //     ->count(12)
        //     ->sequence(fn ($sequence) => ['name' => ($sequence->index + 1) . '. Sınıf'])
        //     ->create();


    }
}

```

## DatabaseSeader

```php
<?php

namespace Database\Seeders;

use App\Models\User;
// use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     */
    public function run(): void
    {
        // User::factory(10)->create();
        $this->call([
            UsersTableSeeder::class,
            ClassesTableSeeder::class,
            SectionsTableSeeder::class,
            StudentsTableSeeder::class,
        ]);
    }
}

```

## Model

```php

```
