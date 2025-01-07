# ADIM 1: LARAVEL TARAFI İŞLEMLERİ

## İşlemler

### Laravel Kurulumu

```bash
composer create-project laravel/laravel blog
```

### Migration'ın Çalışması

```bash
php artisan migrate
```

### user modelinde index metodu oluştur (Model)

```bash
php artisan make:model User
```

```php
<?php
// Dosya Konumu: app/Models/User.php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
  use HasFactory;
  protected $table = 'users';

  public function index()
  {
    return $this->hasMany(User::class);
  }
  public function getUser($id)
  {
    return $this->where('id', $id)->first();
  }
}
```

### Kullanıcıları listeleyecek bir sayfa hazırla (Controller)

```bash
php artisan make:controller UserController
```

````php
<?php
// Dosya Konumu: app/Http/Controllers/UserController.php
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
  public function index()
  {
    return User::all();
  }
  public function getUser($id)
  {
    return User::where('id', $id)->first();
  }
}
```

### Kullanıcıları listeleyecek bir sayfa hazırla (View)

```bash
php artisan make:view user.index
```

```blade
// Dosya Konumu: resources/views/user/index.blade.php
<div>
  <h1>All Users</h1>
  <ul>
      @foreach ($users as $user)
          <li>
              <a href="/user/{{ $user->id }}">{{ $user->name }}</a>
          </li>
      @endforeach
  </ul>
</div>
```

### user detail için view oluştur (view)

```bash
php artisan make:view user.detail
```

```blade
// Dosya Konumu: resources/views/user/detail.blade.php
<div>
  <h1>{{ $user->name }}</h1>
  <p>{{ $user->email }}</p>
</div>
```

### Laravel için router oluşturuldu (Web Routes)

```php
<?php
// Dosya Konumu: routes/web.php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('user.index', ['users' => User::all()]);
});

Route::get('/user/{id}', function ($id) {
    return view('user.detail', ['user' => User::find($id)]);
});
```

### User tablosuna 20 örnek kayıt ekleyelim (Factory)

```bash
php artisan make:factory UserFactory
```

```php
<?php
// Dosya Konumu: database/factories/UserFactory.php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    protected $model = User::class;
    public function definition()
    {
        return [
            'name' => $this->faker->name(),
            'email' => $this->faker->unique()->email(),
            'password' => bcrypt('password')
        ];
    }
}
```

### User tablosuna 20 örnek kayıt ekleyelim (Seeder)

```bash
php artisan make:seeder UserSeeder
```

```php
<?php
// Dosya Konumu: database/seeders/UserSeeder.php
namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        User::factory(20)->create();
    }
}
```

### Database seeder'ının oluşturulması

```bash
php artisan make:seeder DatabaseSeeder
```

```php
<?php
// Dosya Konumu: database/seeders/DatabaseSeeder.php
namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class
        ]);
    }
}
```


### Seeder'ın Çalışması

```bash
php artisan migrate:rollback
php artisan db:seed
```

### Laravel projetinin başlatılması

Bu noktaya geldiğimizde laravel projesi çalışır durumdadır.

```bash
php artisan serve
```

## ADIM 2: VUE TARAFI İŞLEMLERİ

## ADIM 3: INERTIA TARAFI İŞLEMLERİ


````
