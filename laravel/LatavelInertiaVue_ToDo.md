# Laravel Inertia Vue Todo Uygulaması Ders Notları

## 1. Proje Kurulumu

### 1.1 Laravel Projesi Kurulumu

```bash
composer create-project laravel/laravel todo-app
cd todo-app
```

### 1.2 Inertia.js Entegrasyonu

```bash
composer require inertiajs/inertia-laravel
php artisan inertia:middleware
```

`app/Http/Kernel.php` dosyasına middleware eklenmesi:

```php
'web' => [
    // ...
    \App\Http\Middleware\HandleInertiaRequests::class,
],
```

### 1.3 Vue.js Kurulumu

```bash
npm install @inertiajs/vue3
npm install vue@next
```

### 1.4 Breeze Authentication Kurulumu

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install
npm run dev
```

## 2. Veritabanı Yapılandırması

### 2.1 Todos Migration Oluşturma

```bash
php artisan make:migration create_todos_table
```

Migration içeriği:

```php
public function up()
{
    Schema::create('todos', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->string('title');
        $table->text('description')->nullable();
        $table->enum('status', ['pending', 'completed'])->default('pending');
        $table->enum('priority', ['low', 'medium', 'high'])->default('medium');
        $table->timestamp('due_date')->nullable();
        $table->timestamps();
    });
}
```

### 2.2 Todo Model Oluşturma

```bash
php artisan make:model Todo
```

Model içeriği:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Todo extends Model
{
    protected $fillable = [
        'title',
        'description',
        'status',
        'priority',
        'due_date',
        'user_id'
    ];

    protected $casts = [
        'due_date' => 'datetime'
    ];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

## 3. Backend İşlemleri

### 3.1 TodoController Oluşturma

```bash
php artisan make:controller TodoController
```

Controller içeriği:

```php
namespace App\Http\Controllers;

use App\Models\Todo;
use Illuminate\Http\Request;
use Inertia\Inertia;

class TodoController extends Controller
{
    public function index()
    {
        $todos = auth()->user()->todos()
            ->orderBy('created_at', 'desc')
            ->get();

        return Inertia::render('Todos/Index', [
            'todos' => $todos
        ]);
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'priority' => 'required|in:low,medium,high',
            'due_date' => 'nullable|date'
        ]);

        auth()->user()->todos()->create($validated);

        return redirect()->back();
    }

    public function update(Request $request, Todo $todo)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'description' => 'nullable|string',
            'status' => 'required|in:pending,completed',
            'priority' => 'required|in:low,medium,high',
            'due_date' => 'nullable|date'
        ]);

        $todo->update($validated);

        return redirect()->back();
    }

    public function destroy(Todo $todo)
    {
        $todo->delete();

        return redirect()->back();
    }
}
```

### 3.2 Route Tanımlamaları

`routes/web.php` içeriği:

```php
use App\Http\Controllers\TodoController;

Route::middleware(['auth'])->group(function () {
    Route::resource('todos', TodoController::class);
});
```

## 4. Frontend İşlemleri

### 4.1 Todo Listesi Komponenti

`resources/js/Pages/Todos/Index.vue`:

```vue
<template>
  <div class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
    <div class="px-4 py-6 sm:px-0">
      <!-- Todo Ekleme Formu -->
      <form @submit.prevent="submit" class="mb-8">
        <div class="grid grid-cols-1 gap-4">
          <input
            v-model="form.title"
            type="text"
            placeholder="Yapılacak iş..."
            class="rounded-md"
          />
          <textarea
            v-model="form.description"
            placeholder="Açıklama..."
            class="rounded-md"
          ></textarea>
          <select v-model="form.priority" class="rounded-md">
            <option value="low">Düşük</option>
            <option value="medium">Orta</option>
            <option value="high">Yüksek</option>
          </select>
          <button
            type="submit"
            class="bg-blue-500 text-white rounded-md px-4 py-2"
          >
            Ekle
          </button>
        </div>
      </form>

      <!-- Todo Listesi -->
      <div class="space-y-4">
        <div
          v-for="todo in todos"
          :key="todo.id"
          class="bg-white p-4 rounded-md shadow"
        >
          <div class="flex items-center justify-between">
            <div>
              <h3 class="text-lg font-medium">{{ todo.title }}</h3>
              <p class="text-gray-500">{{ todo.description }}</p>
            </div>
            <div class="flex space-x-2">
              <button
                @click="toggleStatus(todo)"
                :class="
                  todo.status === 'completed' ? 'bg-green-500' : 'bg-gray-500'
                "
                class="text-white rounded-md px-3 py-1"
              >
                {{ todo.status === 'completed' ? 'Tamamlandı' : 'Beklemede' }}
              </button>
              <button
                @click="deleteTodo(todo)"
                class="bg-red-500 text-white rounded-md px-3 py-1"
              >
                Sil
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useForm } from '@inertiajs/vue3';

const form = useForm({
  title: '',
  description: '',
  priority: 'medium',
});

const props = defineProps({
  todos: Array,
});

const submit = () => {
  form.post(route('todos.store'), {
    onSuccess: () => form.reset(),
  });
};

const toggleStatus = (todo) => {
  useForm({
    ...todo,
    status: todo.status === 'completed' ? 'pending' : 'completed',
  }).put(route('todos.update', todo.id));
};

const deleteTodo = (todo) => {
  if (confirm('Bu görevi silmek istediğinizden emin misiniz?')) {
    useForm().delete(route('todos.destroy', todo.id));
  }
};
</script>
```

## 5. Özelleştirmeler ve İleri Seviye Özellikler

### 5.1 Etiketleme Sistemi

Yeni bir migration oluşturma:

```bash
php artisan make:migration create_tags_table
php artisan make:migration create_todo_tag_table
```

Tags migration:

```php
Schema::create('tags', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('color');
    $table->timestamps();
});
```

Todo-Tag pivot tablosu:

```php
Schema::create('todo_tag', function (Blueprint $table) {
    $table->id();
    $table->foreignId('todo_id')->constrained()->onDelete('cascade');
    $table->foreignId('tag_id')->constrained()->onDelete('cascade');
    $table->timestamps();
});
```

### 5.2 Bildirim Sistemi

```php
php artisan make:notification TodoDueNotification
```

Notification içeriği:

```php
namespace App\Notifications;

use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Messages\MailMessage;

class TodoDueNotification extends Notification
{
    public function via($notifiable)
    {
        return ['mail', 'database'];
    }

    public function toMail($notifiable)
    {
        return (new MailMessage)
            ->line('Bir görevinizin son teslim tarihi yaklaşıyor!')
            ->action('Görevi Görüntüle', url('/todos'))
            ->line('Bu bildirimi görev yönetim sisteminizden alıyorsunuz.');
    }
}
```

## 6. Performans İyileştirmeleri

### 6.1 Cache Kullanımı

```php
use Illuminate\Support\Facades\Cache;

public function index()
{
    $todos = Cache::remember('user.' . auth()->id() . '.todos', 60, function () {
        return auth()->user()->todos()
            ->with('tags')
            ->orderBy('created_at', 'desc')
            ->get();
    });

    return Inertia::render('Todos/Index', [
        'todos' => $todos
    ]);
}
```

### 6.2 API Rate Limiting

`routes/web.php` içinde:

```php
Route::middleware(['auth', 'throttle:60,1'])->group(function () {
    Route::resource('todos', TodoController::class);
});
```

## 7. Test Yazımı

```bash
php artisan make:test TodoTest
```

Test içeriği:

```php
namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use App\Models\Todo;

class TodoTest extends TestCase
{
    public function test_can_create_todo()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)->post('/todos', [
            'title' => 'Test Todo',
            'description' => 'Test Description',
            'priority' => 'medium'
        ]);

        $response->assertRedirect();
        $this->assertDatabaseHas('todos', [
            'title' => 'Test Todo'
        ]);
    }
}
```
