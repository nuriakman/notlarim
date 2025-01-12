# Notlar

## Test

### Tüm testleri calistirmak icin

php artisan test

### Bir testi calistirmak icin

php artisan test tests/Feature/Api/ProductApiTest.php

### Validasyona takılan update

422 durum kodu "Unprocessable Entity" anlamına gelir ve validasyon hatalarında kullanılır

### 204 (Silme) durum kodunun özelliği

- "No Content" (İçerik Yok) anlamına gelir
- Yanıt gövdesinde hiçbir içerik OLMAMALIDIR
- Sunucu yanıt gövdesi göndermeye çalışsa bile, client bu içeriği görmezden gelmelidir
- Aşağıdaki iki örnek de doğrudur. Ancak, ilk versiyon (return response()->json(null, 204);) daha doğru ve temiz bir yaklaşımdır

  - ÖRNEK 1:
    return response()->json(null, 204);

  - ÖRNEK 2:
    return response()->json([
    'status' => 'success',
    'message' => 'Yorum başarıyla silindi'
    ], 204);
