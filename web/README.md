# SeaPass — Web App & API

The **Laravel 12** backend for SeaPass: a RESTful JSON API plus the web admin dashboard for the maritime passenger ticketing system.

## Stack
- Laravel 12, PHP 8.2+, MySQL 8+ / MariaDB 10.5+
- Vite for frontend asset bundling
- Sanctum token authentication for the mobile API
- PayMongo payments, QR ticket verification, email notifications (OTP, bookings, advisories, reschedules)

## Setup

```powershell
cd web
composer install
npm install
npm run build
Copy-Item .env.example .env   # then edit DB credentials
php artisan key:generate
php artisan migrate --seed
php artisan serve --host=0.0.0.0 --port=8000
```

Bind with `--host=0.0.0.0` so Android phones on the same Wi-Fi can reach the API on port 8000.

See the [root README](../README.md) for the full setup, firewall, and device-connection guide.

## Structure

- `app/Http/Controllers/` — API + admin controllers
- `app/Services/` — booking, PayMongo, recurring-schedule, and trip-status services
- `app/Models/` — Eloquent models
- `database/migrations/` — schema migrations
- `resources/views/` — admin dashboard Blade views + emails
- `routes/api.php` — mobile API endpoints; `routes/web.php` — admin panel routes