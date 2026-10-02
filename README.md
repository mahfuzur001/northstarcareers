# Northstar Careers

A server-rendered Django job portal for curated public job listings. Administrators create and publish jobs in Django Admin; visitors can search and browse published listings, then apply through the employer's official application page.

## Features

- Responsive Django Templates UI with Bootstrap 5 and vanilla JavaScript
- Slug-based jobs, category, company, and location pages
- Search and shareable filters for keywords, location, category, type, workplace, experience, salary text, and posting date
- Draft/published workflow, featured jobs, expiry-aware public querysets, and admin bulk actions
- JobPosting/Organization structured data, canonical metadata, sitemap and robots.txt
- Reusable ad positions configured through environment variables
- SQLite for local use; PostgreSQL through `DATABASE_URL` for production
- Contact form, legal information pages, accessible not-found and server-error pages

## Requirements

- Python 3.11 or newer
- pip
- PostgreSQL for production deployments

Install dependencies from `requirements.txt`. SQLite ships with Python and requires no separate service.

## Local installation

PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
Copy-Item .env.example .env
```

Edit `.env` and replace `SECRET_KEY` with a random local key. One can be generated with:

```powershell
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

SQLite is used when `DATABASE_URL` is blank. Initialize the database and administrator:

```powershell
python manage.py makemigrations jobs
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Open `http://127.0.0.1:8000/` for the site and `http://127.0.0.1:8000/admin/` to manage it. Add a category, company, location, and job in that order. Save a job as a draft, review it, and publish it using the `is_published` field or the bulk admin action.

## Static and media files

CSS and JavaScript live in `static/`. In production, collect static assets with:

```powershell
python manage.py collectstatic --noinput
```

WhiteNoise serves versioned static assets. `MEDIA_ROOT` and `MEDIA_URL` are configured for local uploaded media; local media is served by Django only with `DEBUG=True`. Company logos currently use validated public HTTPS/HTTP URLs, so no uploaded media storage is required. For uploaded images, configure an object-storage backend before deployment rather than serving uploads through Django.

## PostgreSQL and production

Set deployment environment variables (do not commit `.env`):

```text
DEBUG=False
SECRET_KEY=<unique-long-random-value>
ALLOWED_HOSTS=jobs.example.com,www.jobs.example.com
SITE_URL=https://jobs.example.com
DATABASE_URL=postgresql://USER:PASSWORD@HOST:5432/DATABASE
SECURE_SSL_REDIRECT=True
SESSION_COOKIE_SECURE=True
CSRF_COOKIE_SECURE=True
```

`dj-database-url` parses `DATABASE_URL`; production connections require SSL. Configure DNS, TLS termination, allowed hosts, backups, monitoring, and your platform's PostgreSQL network access before launch. Run:

```powershell
python manage.py check --deploy
python manage.py migrate
python manage.py collectstatic --noinput
```

Run the WSGI application at `config.wsgi:application` behind a production WSGI server/reverse proxy. Set a durable PostgreSQL database and keep secrets in the platform's secret manager. The project intentionally refuses to start in production with the development secret.

## Email

The contact form uses Django's console email backend locally. In production set `CONTACT_EMAIL`, `DEFAULT_FROM_EMAIL`, `EMAIL_HOST`, `EMAIL_PORT`, `EMAIL_HOST_USER`, `EMAIL_HOST_PASSWORD`, and `EMAIL_USE_TLS` to the values from your email provider. Keep SMTP credentials outside source control.

## Adsterra

The templates use reusable `components/ad_slot.html` placements: header, inline/home and job detail/listings, sidebar, and footer. Set `ADSTERRA_HEADER`, `ADSTERRA_INLINE`, `ADSTERRA_SIDEBAR`, and/or `ADSTERRA_FOOTER` in the deployment environment to trusted embed snippets supplied by Adsterra. Review network/privacy obligations and test placements on mobile; no advertising scripts are included by default. Treat these settings as trusted HTML and never populate them from visitor input.

## SEO

Set `SITE_URL` to the final HTTPS origin. `/sitemap.xml` includes public jobs and active category, company, location, and static pages. `/robots.txt` references that sitemap and disallows the admin. Job pages emit Schema.org `JobPosting` JSON-LD; company pages emit `Organization` JSON-LD. Confirm the live sitemap and structured data with search-engine webmaster tools after deployment.

## Testing

```powershell
python manage.py test jobs
python manage.py check
```

Normal visitors have no job submission route; creating and publishing listings is limited to authorized Django staff.