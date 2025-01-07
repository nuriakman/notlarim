# Laravel Breeze

## Kurulum

```bash
# Kaynak: https://f24aalam.medium.com/integrating-vue-js-components-in-laravel-a-beginners-guide-cf8414a27d6b
mkdir breeze-app
cd breeze-app
composer create-project laravel/laravel .
composer require laravel/breeze --dev
php artisan breeze:install
# Select:
## [x] Vue with inertia
## [x] Dark Mode
## [x] Inertia SSR
## [x] TypeScript
## [x] ESLint with Prettier
## [x] PHP Unit

php artisan migrate
npm install

npm run dev
php artisan serve
# Yukarıdaki ikili yerine:
composer run dev

# ADIM 7'ye geldim
```
