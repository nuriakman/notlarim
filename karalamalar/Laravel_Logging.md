# Laravel'de Loglama (Logging)

## İçindekiler

1. Genel Bakış
2. Log Konfigürasyonu
3. Temel Log Kullanımı
4. Custom Log Channels
5. Log Seviyeleri
6. Context ve Extra Data
7. Custom Logger Oluşturma
8. Best Practices
9. Örnek Senaryolar

## 1. Genel Bakış

Laravel'de loglama, Monolog kütüphanesi üzerine kurulmuştur ve çeşitli log kanalları destekler:

- single (tek dosya)
- daily (günlük dosyalar)
- slack
- syslog
- errorlog
- monolog
- custom

## 2. Log Konfigürasyonu

```php
// config/logging.php
return [
    'default' => env('LOG_CHANNEL', 'stack'),

    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['daily', 'slack'],
        ],

        'single' => [
            'driver' => 'single',
            'path' => storage_path('logs/laravel.log'),
            'level' => 'debug',
        ],

        'daily' => [
            'driver' => 'daily',
            'path' => storage_path('logs/laravel.log'),
            'level' => 'debug',
            'days' => 14,
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],
];
```

## 3. Temel Log Kullanımı

### 3.1 Log Seviyelerine Göre Kullanım

```php
// Temel kullanım
Log::info('Bu bir bilgi mesajı');
Log::warning('Bu bir uyarı mesajı');
Log::error('Bu bir hata mesajı');

// Context ile kullanım
Log::info('Kullanıcı giriş yaptı', ['id' => $user->id, 'email' => $user->email]);

// Exception logları
try {
    // ... kod
} catch (\Exception $e) {
    Log::error('Bir hata oluştu', [
        'exception' => $e->getMessage(),
        'file' => $e->getFile(),
        'line' => $e->getLine()
    ]);
}
```

### 3.2 Channel Seçerek Kullanım

```php
// Belirli bir kanala yazma
Log::channel('slack')->critical('Acil durum!');

// Birden fazla kanala yazma
Log::stack(['daily', 'slack'])->info('Önemli bir işlem gerçekleşti');
```

## 4. Custom Log Channels

```php
// config/logging.php
'channels' => [
    'custom' => [
        'driver' => 'custom',
        'via' => App\Logging\CustomLogger::class,
    ],

    'api' => [
        'driver' => 'daily',
        'path' => storage_path('logs/api.log'),
        'level' => 'debug',
        'days' => 7,
    ],
],

// App\Logging\CustomLogger.php
namespace App\Logging;

use Monolog\Logger;

class CustomLogger
{
    public function __invoke(array $config)
    {
        return new Logger('custom');
    }
}
```

## 5. Log Seviyeleri

```php
Log::emergency($message);  // Sistem kullanılamaz
Log::alert($message);      // Hemen aksiyon alınmalı
Log::critical($message);   // Kritik durumlar
Log::error($message);      // Çalışma hataları
Log::warning($message);    // İstisnai durumlar
Log::notice($message);     // Normal ama önemli
Log::info($message);       // Bilgilendirme
Log::debug($message);      // Debug bilgisi
```

## 6. Context ve Extra Data

### 6.1 Context Ekleme

```php
Log::info('Ödeme işlemi', [
    'user_id' => $user->id,
    'amount' => $payment->amount,
    'status' => $payment->status,
    'transaction_id' => $payment->transaction_id
]);
```

### 6.2 Global Context

```php
// App\Providers\AppServiceProvider.php
public function boot()
{
    Log::withContext([
        'environment' => app()->environment(),
        'server' => request()->server('SERVER_NAME')
    ]);
}
```

## 7. Custom Logger Oluşturma

### 7.1 Custom Logger Sınıfı

```php
// app/Logging/ApiLogger.php
namespace App\Logging;

use Monolog\Logger;
use Monolog\Handler\RotatingFileHandler;
use Monolog\Formatter\LineFormatter;

class ApiLogger
{
    public function __invoke(array $config)
    {
        $logger = new Logger('api');

        $handler = new RotatingFileHandler(
            storage_path('logs/api.log'),
            7,
            Logger::DEBUG
        );

        $formatter = new LineFormatter(
            "[%datetime%] %channel%.%level_name%: %message% %context% %extra%\n",
            "Y-m-d H:i:s"
        );

        $handler->setFormatter($formatter);
        $logger->pushHandler($handler);

        return $logger;
    }
}
```

### 7.2 Custom Processor Ekleme

```php
// app/Logging/RequestProcessor.php
namespace App\Logging;

class RequestProcessor
{
    public function __invoke(array $record)
    {
        $record['extra']['ip'] = request()->ip();
        $record['extra']['user_agent'] = request()->userAgent();
        $record['extra']['url'] = request()->fullUrl();

        return $record;
    }
}

// Kullanımı
$logger->pushProcessor(new RequestProcessor());
```

## 8. Best Practices

### 8.1 Log Formatı Standardizasyonu

```php
// app/Logging/StandardFormatter.php
namespace App\Logging;

use Monolog\Formatter\LineFormatter;

class StandardFormatter
{
    public function __invoke($logger)
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                "[%datetime%] %channel%.%level_name%: %message% %context% %extra%\n"
            ));
        }
    }
}
```

### 8.2 Log Helper Trait

```php
// app/Traits/Loggable.php
namespace App\Traits;

use Illuminate\Support\Facades\Log;

trait Loggable
{
    protected function logInfo($message, array $context = [])
    {
        Log::info(class_basename($this) . ": {$message}", $context);
    }

    protected function logError($message, \Throwable $exception = null)
    {
        $context = [];

        if ($exception) {
            $context = [
                'message' => $exception->getMessage(),
                'file' => $exception->getFile(),
                'line' => $exception->getLine(),
                'trace' => $exception->getTraceAsString()
            ];
        }

        Log::error(class_basename($this) . ": {$message}", $context);
    }
}
```

## 9. Örnek Senaryolar

### 9.1 Payment İşlemlerini Loglama

```php
// app/Services/PaymentService.php
namespace App\Services;

use App\Traits\Loggable;

class PaymentService
{
    use Loggable;

    public function processPayment($order)
    {
        try {
            $this->logInfo('Ödeme başlatıldı', [
                'order_id' => $order->id,
                'amount' => $order->amount
            ]);

            // Ödeme işlemleri...

            $this->logInfo('Ödeme tamamlandı', [
                'order_id' => $order->id,
                'transaction_id' => $response->transaction_id
            ]);

        } catch (\Exception $e) {
            $this->logError('Ödeme hatası', $e);
            throw $e;
        }
    }
}
```

### 9.2 API İsteklerini Loglama

```php
// app/Http/Middleware/ApiLogger.php
namespace App\Http\Middleware;

use Illuminate\Support\Facades\Log;

class ApiLogger
{
    public function handle($request, \Closure $next)
    {
        // İstek başlangıcını logla
        Log::channel('api')->info('API İsteği Başladı', [
            'url' => $request->fullUrl(),
            'method' => $request->method(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'headers' => $request->headers->all(),
            'body' => $request->all()
        ]);

        $response = $next($request);

        // İstek sonucunu logla
        Log::channel('api')->info('API İsteği Tamamlandı', [
            'status' => $response->status(),
            'duration' => microtime(true) - LARAVEL_START
        ]);

        return $response;
    }
}
```

### 9.3 Kullanıcı İşlemlerini Loglama

```php
// app/Observers/UserObserver.php
namespace App\Observers;

use App\Models\User;
use Illuminate\Support\Facades\Log;

class UserObserver
{
    public function created(User $user)
    {
        Log::info('Yeni kullanıcı oluşturuldu', [
            'id' => $user->id,
            'email' => $user->email
        ]);
    }

    public function updated(User $user)
    {
        Log::info('Kullanıcı güncellendi', [
            'id' => $user->id,
            'changes' => $user->getDirty()
        ]);
    }

    public function deleted(User $user)
    {
        Log::warning('Kullanıcı silindi', [
            'id' => $user->id,
            'email' => $user->email
        ]);
    }
}
```

Bu yapılandırmalar ve örnekler sayesinde:

- Uygulama içindeki tüm önemli işlemleri takip edebilir
- Hataları hızlıca tespit edebilir
- Güvenlik olaylarını izleyebilir
- Performans sorunlarını analiz edebilirsiniz
