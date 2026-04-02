# TravelTejas

E-commerce storefront built with **Laravel 12** and **Aimeos 2025.10**, deployed at [traveltejas.com](https://traveltejas.com).

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Laravel 12 |
| E-commerce | Aimeos 2025.10 |
| Auth scaffolding | Laravel Breeze |
| Database | MariaDB (production), SQLite (local dev) |
| PHP | 8.2 |
| Web server | Apache 2.4 + PHP-FPM |
| CSS | Tailwind CSS (CDN) |
| Mail | Sendmail via local Postfix |

## Routes

| Prefix | Handled by | Purpose |
|---|---|---|
| `/` | Aimeos | Shop homepage |
| `/shop/*` | Aimeos | Catalog, basket, checkout |
| `/p/{slug}` | Aimeos | CMS pages |
| `/account/*` | Aimeos (auth) | Orders, favorites, subscriptions |
| `/brand/*` | Aimeos | Supplier pages |
| `/login`, `/register` | Breeze | Authentication |
| `/profile` | Breeze (auth) | Edit name, email, password |
| `/dashboard` | Breeze (auth) | User dashboard |
| `/admin` | Aimeos (auth + admin Gate) | Admin panel |

## Local Development

### Requirements

- PHP 8.2
- Composer
- Node.js + npm
- MySQL or SQLite

### Setup

```bash
git clone git@github.com:ETXSVC/traveltejas.git
cd traveltejas

composer install
cp .env.example .env
php artisan key:generate

# Edit .env — set DB_* credentials
php artisan migrate
php artisan aimeos:setup

npm install && npm run build
php artisan serve
```

> **Note:** The production server uses `php8.2` explicitly — always use `php8.2 artisan ...` on the server (system default is PHP 8.1).

### Running Tests

```bash
php artisan test
# or a single file:
php artisan test tests/Feature/ExampleTest.php
```

## Admin Access

Admin access is controlled by the `admin` Gate in `app/Providers/AppServiceProvider.php`:

- Users with `superuser = 1` in the `users` table bypass all group checks.
- Other users need to belong to an `admin` or `editor` Aimeos group.

To create or promote an admin account:

```bash
php8.2 artisan aimeos:account --super --password='...' user@example.com
```

## CMS Pages

Footer and static pages live in `mshop_cms`. The URL field must include a leading slash (e.g., `/terms`). Pages are served at `/p/{slug}`.

## Deployment Notes

- **DocumentRoot**: `/home/ddavis/laravel/public/`
- **PHP-FPM socket**: `/run/php/php82-ddavis.sock`
- **Apache vhost**: `/etc/apache2/sites-enabled/traveltejas.com.conf`
- **Session driver**: `file` (not database)
- **Cache driver**: `database`
- Vite is **not** built in production — layouts use the Tailwind CDN.

After any config or code change:

```bash
php8.2 artisan config:clear && php8.2 artisan config:cache
php8.2 artisan route:clear && php8.2 artisan route:cache
php8.2 artisan view:clear
```

## License

Proprietary. All rights reserved.
