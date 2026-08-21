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

`.env.prod` is written per competitor by Mission Control and is **gitignored** — it carries
real credentials. See `.env.prod.example` for the keys it should define.

The SQLite file (`database/database.sqlite`) is created only when `DB_CONNECTION=sqlite`, so
pointing the app at MySQL actually reaches MySQL. Migrations never fail the boot: an
unreachable database leaves the app serving its error page instead of crash-looping the pod.
