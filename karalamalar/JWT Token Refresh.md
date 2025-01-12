# JWT Token Refresh ve Proaktif Yenileme Stratejisi

## Token Refresh UI İmplementasyonu

1. **Refresh İhtiyaç Durumları**

   - Token süresinin dolmasına yakın (örn: son 5 dakika)
   - 401 Unauthorized hatası alındığında
   - Kullanıcı aktif olarak uygulamayı kullanıyorken
   - Sayfa yenilendiğinde (SPA uygulamalarda)

2. **Frontend İmplementasyonu**

```javascript
// Axios interceptor örneği
axios.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Token süresi dolmuşsa ve daha önce refresh denenmemişse
    if (error.response.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Refresh token ile yeni token alma
        const response = await axios.post('/api/auth/refresh');
        const newToken = response.data.access_token;

        // Yeni token'ı localStorage/cookie'ye kaydetme
        localStorage.setItem('token', newToken);

        // Orijinal isteği yeni token ile tekrar deneme
        originalRequest.headers['Authorization'] = `Bearer ${newToken}`;
        return axios(originalRequest);
      } catch (refreshError) {
        // Refresh başarısız olursa login sayfasına yönlendir
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  },
);
```

## **Proaktif Yenileme Stratejisi**

```javascript
// Token süresini takip eden ve proaktif yenileyen servis
class TokenService {
  constructor() {
    this.refreshThreshold = 5 * 60 * 1000; // 5 dakika
    this.checkInterval = null;
  }

  startTokenRefreshTimer() {
    const token = localStorage.getItem('token');
    const tokenData = this.parseJwt(token);

    if (!tokenData) return;

    const expiresIn = tokenData.exp * 1000 - Date.now();
    const timeUntilRefresh = expiresIn - this.refreshThreshold;

    if (timeUntilRefresh <= 0) {
      this.refreshToken();
    } else {
      setTimeout(() => this.refreshToken(), timeUntilRefresh);
    }
  }

  async refreshToken() {
    try {
      const response = await axios.post('/api/auth/refresh');
      const newToken = response.data.access_token;
      localStorage.setItem('token', newToken);

      // Yeni token için timer'ı yeniden başlat
      this.startTokenRefreshTimer();
    } catch (error) {
      console.error('Token yenileme hatası:', error);
    }
  }

  parseJwt(token) {
    try {
      return JSON.parse(atob(token.split('.')[1]));
    } catch (e) {
      return null;
    }
  }
}
```

## SORU: refreshToken yapıldığında Toker'ın geçerlilik süresi uzatılmış yeni bir token ile değiştirilmesi mi sağlanır?

Evet, doğru. Token refresh işlemi tam olarak bunu sağlar. İşleyişi şöyle açıklayabilirim:

**Token Değişimi:**

- Eski token sunucuya gönderilir
- Sunucu eski token'ı blacklist'e ekler
- Aynı kullanıcı bilgileriyle yeni bir token oluşturulur
- Yeni token daha uzun bir geçerlilik süresiyle döner
  **Süreç:**
  Eski Token(5 dk kaldı) -> Refresh İsteği -> Yeni Token(60 dk geçerli)

**Backend Tarafında:**

```php
public function refresh()
{
    try {
        // Eski token'ı blacklist'e ekler
        $oldToken = auth()->getToken();
        auth()->invalidate($oldToken);

        // Yeni token oluşturur
        $newToken = auth()->refresh();

        return response()->json([
            'access_token' => $newToken,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60, // Yeni süre (örn: 60 dakika)
            'user' => auth()->user()
        ]);
    } catch (JWTException $e) {
        return response()->json(['error' => 'Token yenilenemedi'], 401);
    }
}
```

**Önemli Noktalar:**

- Yeni token tamamen yeni bir JWT'dir
- Eski token artık geçersizdir (blacklist'e eklenir)
- Yeni token'ın süresi config'de belirlenen TTL kadar olur
- Kullanıcı bilgileri aynı kalır, sadece token yenilenir

**Bu sayede:**

- Kullanıcı oturumu kesintisiz devam eder
- Güvenlik riski minimize edilir
- Token çalınsa bile kısa sürede geçersiz olur
- Uzun süreli oturumlar güvenli şekilde yönetilir
