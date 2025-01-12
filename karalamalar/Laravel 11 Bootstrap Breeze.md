# Laravel 11 Bootstrap Breeze

[Laravel 11 Bootstrap Breeze Tartışması](https://laracasts.com/discuss/channels/laravel/laravel-11-bootstrap-breeze)

## 1. Laravel Breeze'in Kurulu Olduğundan Emin Olun

Laravel Breeze'in projenizde kurulu olduğundan emin olun. Eğer kurulu değilse, aşağıdaki komutlarla kurabilirsiniz:

```bash
composer require laravel/breeze --dev
php artisan breeze:install
npm install && npm run dev
```

## 2. Bootstrap 5'i Kurun

npm kullanarak Bootstrap 5'i projenize ekleyin:

```bash
npm install bootstrap@latest
npm install @popperjs/core --save
```

## 3. app.css'i Güncelleyin

Varsayılan Laravel CSS'ini Bootstrap stillerini içerecek şekilde değiştirin. `resources/css/app.css` dosyasını açın, içeriğini silin ve şununla değiştirin:

```css
@import 'bootstrap/dist/css/bootstrap.min.css';
```

## 4. Bootstrap JavaScript'i Ekleyin

Bootstrap JavaScript'i içe aktarmak için `resources/js/app.js` dosyasını düzenleyin:

```javascript
import 'bootstrap';
```

## 5. Asset'leri Derleyin

Değişiklikleri yaptıktan sonra, asset'lerinizi derleyin:

```bash
npm run dev
```

Sürekli geliştirme için şunu kullanın:

```bash
npm run watch
```

## 6. Tailwind CSS'i Kaldırın (İsteğe Bağlı)

Eğer artık Tailwind CSS kullanmak istemiyorsanız, kaldırabilirsiniz:

- `resources/css/app.css` dosyasından `@tailwind` direktiflerini kaldırın.
- `package.json`'dan Tailwind CSS ve bağımlılıklarını kaldırın:

```bash
npm uninstall tailwindcss postcss autoprefixer
```

Asset'leri yeniden kurun ve derleyin:

```bash
npm install && npm run dev
```
