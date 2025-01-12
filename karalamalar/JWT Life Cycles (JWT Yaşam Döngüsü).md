# JWT Life Cycles (JWT Yaşam Döngüsü)

## JWT Altyapısında Request Life Cycle'i (JWT Request Life Cycle)

1. **İstek Başlangıcı**

   - Client, API endpoint'ine bir istek gönderir
   - İstek header'ında `Authorization: Bearer {token}` bulunur

2. **Middleware Katmanı**

   - `auth:api` middleware'i devreye girer
   - JWT token'ı header'dan çıkarılır ve doğrulanır
   - Token geçersizse veya süresi dolmuşsa hata döner

3. **Token Doğrulama**

   - Token'ın imzası kontrol edilir
   - Token'ın süresi kontrol edilir
   - Token'ın blacklist'te olup olmadığı kontrol edilir

4. **Kullanıcı Yükleme**

   - Token geçerliyse, içindeki kullanıcı ID'si çıkarılır
   - Bu ID ile veritabanından kullanıcı bilgileri yüklenir
   - Kullanıcı bilgileri request boyunca saklanır

5. **Yetki Kontrolü**

   - Varsa ek middleware'ler çalışır (örn: user.level)
   - Kullanıcının yetki seviyesi kontrol edilir
   - Yetkisiz erişim durumunda hata döner

6. **Controller İşlemi**

   - Tüm kontroller başarılıysa controller çalışır
   - `auth()->user()` ile kullanıcı bilgilerine erişilir
   - İşlem sonucu client'a döndürülür

7. **Response Dönüşü**
   - İşlem sonucu JSON formatında döner
   - Gerekirse yeni token üretilir (refresh)
   - Response header'larına ek bilgiler eklenir

---

## JWT İle Login Life Cycle'i

1. **Kullanıcı Girişi**

   - Client login endpoint'ine POST isteği yapar
   - Email ve şifre bilgileri gönderilir
   - İstek `LoginController`'a yönlendirilir

2. **Validasyon**

   - Gelen veriler doğrulanır (email formatı, şifre uzunluğu vb.)
   - Eksik veya hatalı veri varsa hata döner
   - Rate limiting kontrolleri yapılır

3. **Kimlik Doğrulama**

   - Email ile kullanıcı veritabanında aranır
   - Şifre hash'i kontrol edilir
   - Kullanıcı aktif mi kontrol edilir

4. **JWT Token Oluşturma**

   - Kullanıcı bilgileri ile claims hazırlanır
   - Token süresi (TTL) belirlenir
   - JWT token'ı imzalanır ve oluşturulur

5. **Response Hazırlama**

   - Token bilgisi response'a eklenir
   - Kullanıcı bilgileri response'a eklenir
   - Token türü ve geçerlilik süresi eklenir

6. **Client Tarafı**
   - Token güvenli şekilde saklanır
   - Sonraki istekler için header'a eklenir
   - Token süresi takip edilir

---

## JWT İle Logout Life Cycle'i

1. **Logout İsteği**

   - Client logout endpoint'ine POST isteği yapar
   - Header'da geçerli JWT token bulunur
   - İstek `LogoutController`'a yönlendirilir

2. **Token Kontrolü**

   - Token varlığı kontrol edilir
   - Token geçerliliği doğrulanır
   - Token'ın blacklist'te olup olmadığı kontrol edilir

3. **Token İptali**

   - Token blacklist'e eklenir
   - Token'ın süresi dolana kadar blacklist'te tutulur
   - Kullanıcı oturumu sonlandırılır

4. **Güvenlik İşlemleri**

   - Aktif oturumlar sonlandırılır
   - Refresh token'lar iptal edilir
   - Önbellekteki kullanıcı bilgileri temizlenir

5. **Response Hazırlama**

   - Başarılı logout mesajı hazırlanır
   - HTTP 200 OK status kodu döner
   - Varsa ek bilgiler eklenir

6. **Client Tarafı**

   - Stored token'lar temizlenir
   - Kullanıcı state'i sıfırlanır
   - Login sayfasına yönlendirilir

7. **Güvenlik Önlemleri**
   - Tüm aktif istekler sonlandırılır
   - Local storage/cookie temizlenir
   - Session bilgileri silinir

---

## JWT Token Refresh Life Cycle'i

1. **Refresh İsteği**

   - Client refresh endpoint'ine POST isteği yapar
   - Mevcut JWT token header'da gönderilir
   - İstek `AuthController@refresh`'e yönlendirilir

2. **Eski Token Kontrolü**

   - Mevcut token'ın geçerliliği kontrol edilir
   - Token'ın refresh window'da olup olmadığı kontrol edilir
   - Blacklist kontrolü yapılır

3. **Yeni Token Oluşturma**

   - Eski token blacklist'e eklenir
   - Kullanıcı bilgileri ile yeni claims hazırlanır
   - Yeni JWT token oluşturulur ve imzalanır

4. **Response Hazırlama**

   - Yeni token bilgisi response'a eklenir
   - Yeni geçerlilik süresi eklenir
   - Varsa ek kullanıcı bilgileri eklenir

5. **Client Tarafı**
   - Eski token silinir
   - Yeni token güvenli şekilde saklanır
   - Sonraki istekler yeni token ile yapılır

---

## JWT Token Validation Life Cycle'i

1. **Token Doğrulama İsteği**

   - Her API isteğinde tetiklenir
   - Token header'dan çıkarılır
   - Middleware zinciri başlar

2. **Format Kontrolü**

   - Token formatı kontrol edilir (Bearer şeması)
   - Token yapısı incelenir (header.payload.signature)
   - Base64 decode işlemleri yapılır

3. **İmza Doğrulama**

   - Token imzası kontrol edilir
   - Secret key ile doğrulama yapılır
   - Manipülasyon kontrolü gerçekleştirilir

4. **Claims Kontrolü**

   - Token süresi (exp) kontrol edilir
   - Başlangıç zamanı (nbf) kontrol edilir
   - Özel claims'ler doğrulanır

5. **Blacklist Kontrolü**

   - Token'ın blacklist'te olup olmadığı kontrol edilir
   - Cache/database sorgusu yapılır
   - Geçersiz token varsa hata döner

6. **Kullanıcı Doğrulama**

   - Token'dan user ID çıkarılır
   - Kullanıcı veritabanında aranır
   - Kullanıcı durumu kontrol edilir

7. **Sonuç**
   - Başarılı ise request devam eder
   - Başarısız ise 401/403 hatası döner
   - Hata detayları response'a eklenir

---

## JWT Error Handling Life Cycle'i

1. **Hata Yakalama**

   - JWT ile ilgili tüm exception'lar yakalanır
   - Hata türü belirlenir
   - Log kaydı oluşturulur

2. **Hata Türleri**

   - TokenInvalidException: Geçersiz token
   - TokenExpiredException: Süresi dolmuş token
   - JWTException: Genel JWT hataları
   - TokenBlacklistedException: Blacklist'teki token

3. **Hata İşleme**

   - Hata mesajı hazırlanır
   - HTTP status kodu belirlenir
   - Response formatı ayarlanır

4. **Client Bilgilendirme**

   - Anlaşılır hata mesajı döner
   - Yönlendirme bilgisi eklenir
   - Çözüm önerisi sunulur

5. **Güvenlik Önlemleri**
   - Hassas hata detayları gizlenir
   - Genel hata mesajları kullanılır
   - Güvenlik logları tutulur

---

## Rate Limiting Life Cycle'i

1. **İstek Alımı**

   - API endpoint'ine istek gelir
   - İstek kaynağı belirlenir (IP, user ID vb.)
   - Rate limiting middleware devreye girer

2. **Limit Kontrolü**

   - Belirli zaman diliminde yapılan istek sayısı kontrol edilir
   - Cache/Redis'te tutulan sayaç güncellenir
   - Kalan istek hakkı hesaplanır

3. **Kısıtlama Kuralları**

   - Endpoint bazlı limitler (örn: login için 5 deneme/dakika)
   - Kullanıcı bazlı limitler (örn: 1000 istek/saat)
   - IP bazlı limitler (örn: 100 istek/dakika)

4. **Limit Aşımı Kontrolü**

   - Limit aşıldıysa 429 Too Many Requests hatası döner
   - Retry-After header'ı eklenir
   - Kullanıcı bilgilendirilir

5. **Header Bilgileri**

   - X-RateLimit-Limit: Toplam izin verilen istek sayısı
   - X-RateLimit-Remaining: Kalan istek sayısı
   - X-RateLimit-Reset: Limitin sıfırlanacağı zaman

6. **Güvenlik Önlemleri**
   - Brute force saldırılarına karşı koruma
   - DDoS önleme
   - Servis stabilitesini koruma

---

## JWT Token Refresh Kullanım Nedenleri

1. **Güvenlik**

   - Kısa süreli token kullanımı güvenliği artırır
   - Token çalınsa bile sınırlı süre kullanılabilir
   - Refresh token ile güvenli yenileme sağlanır

2. **Kullanıcı Deneyimi**

   - Kullanıcının sürekli login olması engellenir
   - Arka planda otomatik token yenileme yapılır
   - Kesintisiz hizmet sağlanır

3. **Token Yönetimi**

   - Access token süresi kısa tutulur (örn: 15dk)
   - Refresh token süresi uzun tutulur (örn: 7gün)
   - İhtiyaç halinde tüm tokenlar iptal edilebilir

4. **Avantajları**

   - Daha iyi güvenlik/kullanılabilirlik dengesi
   - Oturum kontrolü ve yönetimi kolaylaşır
   - Token rotasyonu sağlanır

5. **Kullanım Senaryoları**

   - Mobil uygulamalar
   - Single Page Applications (SPA)
   - Uzun süreli oturumlar

6. **Best Practices**

   - Refresh token'ı güvenli şekilde saklayın
   - Her refresh işleminde yeni token üretin
   - Şüpheli durumlarda tüm tokenları iptal edin
