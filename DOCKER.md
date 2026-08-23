# Laravel 12.61.1 — WSC2026 app

```bash
docker compose up --build
```

Open **http://localhost**. On start the entrypoint picks the configuration, ensures an app
key, runs migrations, then serves. Pinned: PHP 8.3 / Composer 2.9.5, laravel/framework 12.61.1.

## Which configuration is used

The entrypoint prefers **`.env.prod`** and copies it over `.env` on every start:

| Situation | What runs |
|---|---|
| `.env.prod` present | Deployed config — this competitor's own database, `APP_KEY` and hostname |
| No `.env.prod`, no `.env` | `.env.example` is copied — SQLite, no database server |
| No `.env.prod`, `.env` exists | Your local `.env` is left alone |

`.env.prod` is written per competitor by Mission Control. It ships here as a template with
the credentials blank — fill those in on the deployment, not in the repository.

The SQLite file (`database/database.sqlite`) is created only when `DB_CONNECTION=sqlite`, so
pointing the app at MySQL actually reaches MySQL: the image builds `pdo_mysql` alongside
`pdo_sqlite`. Migrations never fail the boot: an unreachable database leaves the app serving
its error page instead of crash-looping the pod.

The image is built in two stages. A Node stage runs `npm run build`, so `public/build` — the
Vite manifest and hashed assets, all gitignored — exists in the image and `@vite(...)`
resolves at runtime. `bootstrap/app.php` trusts the ingress's `X-Forwarded-*` headers, so the
app knows it is served over https and generates `https://` URLs.
