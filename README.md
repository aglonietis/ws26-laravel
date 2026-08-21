# Laravel 12.61.1 — WSC2026

A real **Laravel 12.61.1** application (WorldSkills 2026 Web Technologies, TP17) backed by a
self-contained **SQLite** database by default — no database server required. On start it
creates the SQLite file, runs migrations, then serves. A deployed instance can point at a
real database instead; see **Configuration** below.

## Run it

```bash
docker compose up --build
```

Then open **http://localhost**. By default there is no database service: the app uses a
SQLite file created inside the container at startup. Stop with `docker compose down`.

## Configuration

The entrypoint prefers **`.env.prod`** and copies it over `.env` on every start. That file is
the deployed configuration — written per competitor by Mission Control with their own
database, `APP_KEY` and hostname — and it is **gitignored**, because it carries real
credentials. Copy **`.env.prod.example`** to `.env.prod` and fill it in.

With no `.env.prod`, the container falls back to your `.env`, or to `.env.example` (SQLite)
if you have none. The SQLite file is only created when `DB_CONNECTION=sqlite`, so setting
another driver actually reaches that database.

## Develop

The simplest loop is Docker: edit the source (see below), then rebuild:

```bash
docker compose up --build
```

Edit **routes/web.php and resources/views/** to change routes, controllers and views.

To run it natively instead you need **PHP 8.3** (with `pdo_sqlite`) and **Composer 2.9.5**.
Then:

```bash
composer install
touch database/database.sqlite
php artisan migrate
php artisan serve
```

## Stack

- PHP 8.3 / Composer 2.9.5
- Laravel 12.61.1
- SQLite by default (bundled, no server); any Laravel driver via `.env.prod`
