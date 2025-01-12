# JWT Token Yaşam Döngüsü Diyagramları

Bu doküman, JWT (JSON Web Token) yaşam döngüsünün farklı aşamalarını görsel olarak açıklar.

## İçindekiler

1. Login Flow
2. Request Flow
3. Token Refresh Flow
4. Error Handling Flow
5. Rate Limiting Flow

## Login Flow

Bu diyagram, kullanıcı girişinden JWT token oluşturulmasına kadar olan süreci gösterir.

```mermaid
flowchart TD
    A[Client] -->|1. Login Request| B(Login Endpoint)
    B -->|2. Validate| C{Credentials Valid?}
    C -->|No| D[Return Error]
    C -->|Yes| E[Generate JWT]
    E -->|3. Create Token| F[Add Claims]
    F -->|4. Sign Token| G[Return Response]
    G -->|5. Store Token| A
```

## Request Flow

Bu sequence diyagramı, JWT token ile yapılan API isteklerinin nasıl işlendiğini gösterir.

```mermaid
sequenceDiagram
    participant C as Client
    participant M as Middleware
    participant A as Auth Service
    participant D as Database

    C->>M: API Request + JWT
    M->>A: Validate Token
    A->>A: Check Signature
    A->>A: Verify Expiry
    A->>D: Load User
    D-->>A: User Data
    A-->>M: User Authenticated
    M->>C: Response
```

## Token Refresh Flow

Bu diyagram, token yenileme sürecinin nasıl işlediğini gösterir.

```mermaid
flowchart TB
    A[Old Token] -->|1. Refresh Request| B{Valid Token?}
    B -->|No| C[Return Error]
    B -->|Yes| D[Blacklist Old Token]
    D -->|2. Generate| E[New Token]
    E -->|3. Return| F[Response]
    F -->|4. Store| G[Client Storage]
```

## Error Handling Flow

Bu diyagram, JWT ile ilgili hata durumlarının nasıl yönetildiğini gösterir.

```mermaid
flowchart TD
    A[JWT Error] -->|Catch| B{Error Type}
    B -->|Invalid| C[TokenInvalidException]
    B -->|Expired| D[TokenExpiredException]
    B -->|Blacklisted| E[TokenBlacklistedException]
    C -->F[Return 401]
    D -->F
    E -->F
    F -->|Redirect| G[Login Page]
```

## Rate Limiting Flow

Bu diyagram, API isteklerinin nasıl sınırlandırıldığını gösterir.

```mermaid
flowchart TD
    A[Request] -->|Check| B{Rate Limit}
    B -->|Under Limit| C[Process Request]
    B -->|Over Limit| D[429 Too Many Requests]
    C -->E[Update Counter]
    D -->F[Add Retry-After Header]
    E -->G[Return Response]
    F -->G
```

## Diyagramların Açıklamaları

### Login Flow Detayları

- Client'tan gelen login isteği
- Kimlik bilgilerinin doğrulanması
- JWT token oluşturma ve imzalama
- Response ile token'ın dönülmesi

### Request Flow Detayları

- Client'tan gelen API isteği
- Middleware'de token kontrolü
- Auth service'te token doğrulama
- Kullanıcı bilgilerinin yüklenmesi

### Token Refresh Detayları

- Eski token kontrolü
- Blacklist işlemleri
- Yeni token oluşturma
- Client'a yeni token'ın dönülmesi

### Error Handling Detayları

- Token hatalarının yakalanması
- Hata türüne göre işlem
- Uygun HTTP status code'ları
- Kullanıcı yönlendirme

### Rate Limiting Detayları

- İstek sayısı kontrolü
- Limit aşımı durumu
- Header bilgileri
- Response hazırlama
