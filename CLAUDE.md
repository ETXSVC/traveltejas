# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Environment

- **Server**: Virtualmin-managed Ubuntu, Apache 2.4
- **PHP**: 8.2 (via `php8.2`). The system default `php` is 8.1 — always use `php8.2 artisan ...`
- **Database**: MariaDB, database name `laravel`, user `ddavis`
- **Domain**: traveltejas.com → DocumentRoot `/home/ddavis/laravel/public/`
- **PHP-FPM socket**: `/run/php/php82-ddavis.sock` (custom pool at `/etc/php/8.2/fpm/pool.d/177450423371763.conf`)
- **Apache vhost**: `/etc/apache2/sites-enabled/traveltejas.com.conf`

## Common Commands

All artisan commands must be run from `/home/ddavis/laravel/`:

```bash
# After any code or config change
php8.2 artisan config:clear && php8.2 artisan config:cache
php8.2 artisan route:clear && php8.2 artisan route:cache
php8.2 artisan view:clear && php8.2 artisan view:cache

# Run migrations
php8.2 artisan migrate --force

# Aimeos: rebuild search index
php8.2 artisan aimeos:jobs "index/rebuild"

# Aimeos: run all scheduled jobs
php8.2 artisan aimeos:jobs

# Create/update an Aimeos admin account
php8.2 artisan aimeos:account --super --password='...' user@example.com

# Aimeos setup (re-run migrations + seed)
php8.2 artisan aimeos:setup

# Clear application cache
php8.2 artisan cache:clear

# Run tests
php8.2 artisan test
# Run a single test file
php8.2 artisan test tests/Feature/ExampleTest.php
```

After changes to Apache config or PHP-FPM:
```bash
apache2ctl configtest && systemctl reload apache2
systemctl restart php8.2-fpm
```

## Architecture

This is a **Laravel 12 + Aimeos 2025.10** e-commerce application.

### Routing layers

There are two parallel route systems:

1. **Laravel/Breeze routes** (`routes/web.php`, `routes/auth.php`) — handle `/login`, `/register`, `/profile`, `/dashboard`
2. **Aimeos routes** (auto-registered by `ShopServiceProvider`) — handle everything else:
   - `/` → shop homepage (`aimeos_home`)
   - `/shop/*` → catalog, basket, checkout
   - `/p/{path}` → CMS pages (`aimeos_page`)
   - `/admin` → Aimeos admin redirect
   - `/admin/{site}/jqadm/*` → admin panel (requires auth + admin Gate)

### Authorization

The `admin` Gate in `app/Providers/AppServiceProvider.php` controls admin access. Superusers (`users.superuser = 1`) bypass group checks. Non-superusers need to be in an `admin` or `editor` Aimeos group.

### Frontend assets

Vite is **not built** — Breeze layouts (`resources/views/layouts/guest.blade.php` and `app.blade.php`) use the Tailwind CDN instead of `@vite(...)`. Do not reintroduce `@vite()` calls without first running `npm install && npm run build`.

### Aimeos data model

All Aimeos data lives in `mshop_*` and `madmin_*` tables. The site code is `default` (siteid `1.`). Key tables:
- `mshop_cms` — CMS pages (url field must include leading `/`, e.g. `/terms`)
- `mshop_product`, `mshop_catalog` — products and categories
- `mshop_text` — all translatable text, linked via `mshop_*_list` tables
- `mshop_locale_site` — multi-site config

### Session & cache

- `SESSION_DRIVER=file` (database driver requires a `sessions` table not present)
- `CACHE_STORE=database` — uses the `cache` / `cache_locks` tables

### CMS pages

Footer links resolve via `/p/{slug}` → `PageController` → looks up `mshop_cms.url = '/{slug}'`. To add a page: insert into `mshop_cms` with a leading-slash URL, then link text records via `mshop_cms_list`.
