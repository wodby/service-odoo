# Odoo on Wodby

What Wodby sets up for Odoo Community Edition on this service.

## Image

The service runs the prebuilt `wodby/odoo` image, with nothing built for it: Odoo's official image (Ubuntu, Odoo's
Debian package, the patched wkhtmltopdf for PDF reports) running as the `wodby` user (uid 1000). Every Community app
ships in the image and is installed from Odoo's Apps menu.

Custom or community modules need an image of their own built `FROM wodby/odoo:<version>` with the modules copied to
`/usr/src/app`: every directory up to three levels deep there that contains modules is added to the addons path. Use
it from a custom service based on this one.

## Configuration

`/etc/odoo/odoo.conf` is rendered from environment variables on every start; never edit it in a running container.

- **Database.** One database, from the `db` link: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`. Odoo serves
  only that database (`db_name` and `dbfilter`), and the database manager is off (`list_db = False`), so databases are
  never created, copied or dropped from the web. When the linked server has a CA certificate (`DB_SSL_CA`), Odoo
  connects with `verify-full` TLS. The database must be PostgreSQL.
- **Data.** `data_dir` is `/mnt/files`, the `filestore` volume shared by every replica through the linked storage
  service. Uploaded files (the filestore) and sessions are kept there; the database holds everything else.
- **Processes.** Odoo runs with workers (`workers` setting, default 2) and `max-cron-threads` (default 1) processes for
  scheduled actions in each replica. Live updates (`/websocket`: chat, notifications) are served by a separate gevent
  process on port 8072, which the Odoo Nginx service routes to. Each worker can use up to 2.5 GB
  (`ODOO_LIMIT_MEMORY_HARD`) and a request up to 120 s (`ODOO_LIMIT_TIME_REAL`).
- **Proxy.** `proxy_mode` is on: Odoo takes the host and scheme from the Nginx service's forwarded headers.
- **Email.** With a mail server linked as `sendmail`, `SMTP_HOST` and `SMTP_PORT` become Odoo's default outgoing server.
  Without one, set up an outgoing mail server in Odoo's settings.
- Other options: the variables listed in the [wodby/odoo](https://github.com/wodby/odoo) README.

## Installation and updates

After every deployment the **Install or update Odoo** action (`odoo-update`) runs:

- **First deployment:** installs `base` and the `init-modules` setting's apps into the empty database, without demo
  data. The administrator gets the `admin-login` (default `admin`), `admin-password` (required) and `admin-email`
  settings; the `language` setting is loaded and given to the administrator. The company is named after the app.
  `web.base.url` is set to the app's main address (`WODBY_PRIMARY_URL`, passed with `routeMetadata`).
- Without an admin password the action fails and nothing is installed.
- **Later deployments:** updates installed modules whose manifest `version` went up, for example after a new image, and
  installed modules under `/usr/src/app` whose files changed (except `static/` and `tests/`). Checksums are kept in the
  database (`wodby.module_checksums`). New modules are added to the Apps list but not installed.
- A database where an earlier installation stopped halfway is never installed over: the action fails.
- Changing the admin settings after the first deployment does not change the account; change it in Odoo.

The **Update all modules** button runs `odoo-update all` (`odoo -u all`), for example after an Odoo image update that
changed core modules without raising their versions.

## Limits

- Community Edition only. Custom and Enterprise modules need an image built on `wodby/odoo`.
- Odoo 18 to 20; one database per service.
- No development workspace.

## Check the result

- `curl -s localhost:8069/web/health` in the container returns `{"status": "pass"}`.
- The deployment's post-deploy log shows the install or update output.
- `odoo shell -d "$DB_NAME"` opens a Python shell in the container.
