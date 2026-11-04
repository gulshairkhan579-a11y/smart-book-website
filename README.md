# SmartBook

SmartBook is a public-facing Flask website and a private SQLite workspace for freelancers and independent creative businesses. Visitors can explore the public tool catalog, read feature pages, and use local JSON and URL helpers without an account. Sign-in is only required for the account-scoped client hub, appointments, invoices, and studio workspace. SmartBook does not invent search-volume, CPC, ranking, marketplace, or store-performance data.

## Architecture

```text
app.py                 Development-server entry point
smartbook/__init__.py  Flask app factory, security middleware, blueprint registration
smartbook/auth.py      Registration, login, logout, session status
smartbook/db.py        SQLite schema, connection lifecycle and initialization
smartbook/public.py    Public homepage, tool pages, auth pages, sitemap and robots
smartbook/utils.py     Shared validation and authentication helpers
smartbook/workspace.py Client, appointment, invoice and dashboard routes
smartbook/studios.py   SEO audit/keyword organization and content outline routes
templates/public.html  Public SmartBook homepage and searchable tool catalog
templates/tool.html    Crawlable public tool detail pages and local tool helpers
templates/auth.html    Dedicated login and registration pages
templates/index.html   Authenticated, noindex workspace shell
static/public.css      Responsive public website and tool detail design
static/public.js       Public navigation, category filters and live search
static/public-tools.js Browser-local public JSON and URL slug helpers
static/auth.js         CSRF-protected login and registration flow
static/style.css       Light/dark design system and responsive layouts
static/calendar.css    Responsive appointment calendar styles
static/design.css      Responsive digital design library/editor styles
static/studio.css      Workspace palette and keyword comparison styles
static/app.js          Navigation, forms, rendering and same-origin API client
tests/test_app.py      Flask integration tests
instance/              Local SQLite database (created automatically; ignored by Git)
```

### Application modules

| Module | Current implementation | Planned extension |
|---|---|---|
| Freelancer Hub | Tenant-scoped clients/leads, appointment/project scheduling, invoice totals and currencies | Proposals, contracts, payments, reminders, and project task workflows |
| SEO & keywords | Supplied-copy audit, heuristic keyword ideas, local CSV export, manual phrase-list competitor gap comparison | Search provider integration for volume/CPC/competition, technical crawl, and live competitor data |
| Content Studio | Deterministic article-outline templates | Optional generation provider, editorial workflow, and publishing integrations |
| Developer tools | Browser-local JSON validation/formatting and URL slug helper | Sandboxed API testing and safe, consent-based performance checks |
| Digital design | Thirty original, editable starter canvases; search, category/price filters, product details, local text/color editing, image placement, PNG export | 650+ product marketplace, catalog persistence, mockups, collaboration, and product-selling workflow |
| Commerce / marketing | No provider-backed commerce, sales, or campaign integrations | Separate modules with selected providers and explicit credentials |

### Data model

- `users`: unique case-insensitive email and Werkzeug password hash.
- `clients`: account-owned lead/client details with `Lead`, `Active`, or `Past` status.
- `invoices`: account-owned, client-linked documents with integer minor-unit amounts, tax, currency, due date, and status.
- `appointments`: account-owned schedule records linked to clients and stored with service, date/time, amount, and status.
- Every customer-owned query scopes by `user_id`; SQLite foreign keys enforce client relationships. Monetary calculations use `Decimal` and are stored as integer cents/paisa.

## Run locally

1. Open this folder in VS Code.
2. In the integrated PowerShell terminal, create and activate a virtual environment:

   ```powershell
   py -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

3. Install dependencies:

   ```powershell
   python -m pip install -r requirements.txt
   ```

4. Set a stable local session secret and start Flask:

   ```powershell
   $env:SECRET_KEY = (py -c "import secrets; print(secrets.token_hex(32))")
   python app.py
   ```

5. Open <http://127.0.0.1:5000>, create an account (12+ character password), and add your own records.

The database is created automatically at `instance/smartbook.sqlite3` (the established database filename is retained). To use another location, set `SMARTBOOK_DATABASE`. For deployment, use a persistent database volume, a strong secret stored outside source control, HTTPS, `COOKIE_SECURE=true`, and `FLASK_DEBUG=false`. The built-in Flask development server is not a production server.

## Security and integration notes

- Authentication uses Werkzeug password hashing, signed `HttpOnly`/`SameSite=Lax` sessions, CSRF tokens for API mutations, input validation, and user-scoped database operations.
- Production API keys are not required by the current features. Real keyword volume, CPC, organic difficulty, paid competition, live competitor analysis, AI generation, payment processing, email delivery, and marketplace/store analytics require provider accounts and credentials configured as environment variables. Provider integrations must identify source/freshness and must not silently substitute estimated values.
- SEO audit requests analyze text the user provides; the server does not fetch arbitrary URLs. This avoids implying live rankings and prevents an open URL-fetch/SSRF feature.
- Design uploads are limited to PNG, JPEG, and WebP files up to 5 MB; they are processed with browser object URLs and are not sent to or stored by the server. PNG export uses the browser canvas.
- The public homepage and individual `/tools/<slug>` pages have canonical URLs and indexable metadata and are listed in `/sitemap.xml`. `/robots.txt` excludes API routes; dashboard and authentication pages remain crawlable so search engines can see their `noindex, nofollow` directives. The dashboard is also sent with `Cache-Control: private, no-store` and is never listed in the sitemap.
- Account recovery, verification, rate limiting, payment operations, and production deployment hardening remain roadmap work before offering a public paid service.

## Development roadmap

1. **Foundation (implemented):** Flask app, account isolation, SQLite schema, CSRF-protected session auth, responsive workspace shell, public marketing site, tool discovery and SEO pages.
2. **Freelancer operations (implemented first slice):** client CRM, appointment scheduling, and invoice creation with USD/PKR/EUR/GBP totals.
3. **Local creative tools (implemented first slice):** 30 editable design starters with browser-local image handling and PNG export, heuristic SEO organization and CSV export, manual competitor phrase comparison, supplied-copy audit, and outline templates.
4. **Quality and readiness:** schema migrations, password recovery/email verification, rate limits, audit logging, backups, and deployment behind a production WSGI server.
5. **Provider-backed studios:** integrate a selected SEO data vendor and optional content model behind server-side adapters; disclose source, coverage, cost, and freshness.
6. **Commerce and SaaS operations:** build a real catalog, order sync, payment integrations, subscription billing, quotas, and team workspaces as separately scoped modules.
