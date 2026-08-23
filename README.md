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
database, `APP_KEY` and hostname. It ships here as a template with the credentials left
blank; fill those in on the deployment, not in the repository.

With no `.env.prod`, the container falls back to your `.env`, or to `.env.example` (SQLite)
if you have none. The SQLite file is only created when `DB_CONNECTION=sqlite`, so setting
another driver actually reaches that database — the image carries both `pdo_sqlite` and
`pdo_mysql`, so the same build serves either.

Behind the ingress the app is reached over https while the container itself is spoken to
over plain http. `bootstrap/app.php` trusts the forwarded headers, so `route()` and `url()`
emit `https://` links and form posts are not blocked as mixed content.

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

## Front-end assets

CSS and JS go through **Vite**. `@vite([...])` resolves against `public/build/manifest.json`,
which is generated — never committed — so the Docker image builds it in a Node stage before
the app image is assembled. `docker compose up --build` therefore needs no npm on your
machine, and `@vite(...)` works in the container exactly as it does locally.

For a hot-reloading front-end loop natively you need **Node 24.1.0** and **npm 11.5.0**:

```bash
npm install
npm run dev     # or: npm run build
```

## Stack

- PHP 8.3 / Composer 2.9.5
- Laravel 12.61.1
- SQLite by default (bundled, no server); MySQL via `.env.prod` (`pdo_mysql` is built in)
- Node 24.1.0 / npm 11.5.0, Vite 7 + Tailwind 4 (compiled during the image build)
