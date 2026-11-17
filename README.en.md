# Visagismo B · v1.1.0

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

Style consulting for barbershops, in Portuguese, with real facial analysis, professional review, and VPS deployment.

**[Public guide](https://inematds.github.io/visagismo-b/guia/en/)** · [VPS installation](VPS.md) · [Original plan](PLANO.md)

## What’s implemented

- FastAPI, responsive interface, public page, and per-barbershop dashboard.
- Login with scrypt-protected passwords, revocable sessions, and CSRF protection.
- Questionnaire, JPEG/PNG/WEBP/HEIC upload, and size/resolution validation.
- MediaPipe Tasks with 478 detected points; calculations and classification inherited from the kit.
- Recommendations in Portuguese, client preferences, and the barber’s final decision.
- Persistent SQLite queue and separate worker, with recovery of interrupted processing.
- Required review, professional notes, follow-up, and printable report/PDF through the browser.
- History isolated by barbershop, deletion, and automatic retention.
- Revocable 7-day links to reports **without photos**.
- Optional frontal simulation with flux2-klein via fal.ai or remote inemaimg; geometric filter and fallback without an image.
- In-store checkout and optional Mercado Pago Checkout Pro integration, with webhook signature verification, server-side payment lookup, and amount verification.
- Docker Compose, Caddy/HTTPS, persistent volume, backup, and test workflow.

The public journey is **barbershop page → payment → questionnaire/photo → analysis → professional review → report → follow-up**. The professional can also start the consultation from the dashboard. Share the page available in Settings. The client can follow the consultation through a private 7-day link; photos and drafts do not appear in the public report. The barber can authorize resubmission after a failure without charging again.

Automatic B2B subscriptions and appointment scheduling with reminders are not part of this version; the barbershop’s monthly fee is managed commercially outside the system.

## Installation

The target is a Linux VPS with Docker Compose, a domain, and ports 80/443 available. Starting with 2 vCPUs, 4 GB RAM, and 20 GB of disk is recommended; validate against actual load. A GPU isn’t needed when simulation uses fal.ai; without a provider, the consultation works without a generated image.

```bash
git clone https://github.com/inematds/visagismo-b.git
cd visagismo-b
cp .env.example .env
# Edit DOMAIN and PUBLIC_URL; configure optional integrations.
chmod 600 .env
docker compose up -d --build
docker compose exec app python -m app.manage create-user \
  --email voce@barbearia.com.br --barbearia 'Sua Barbearia' --profissional 'Seu nome'
```

The password is requested without being displayed in the terminal. There is no default login. Visit the configured domain and complete the barbershop details. Details on billing, backup, and updates are in [VPS.md](VPS.md).

## Verification

```bash
python3 -m venv .venv
.venv/bin/pip install --no-deps -r requirements.txt
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/python -m pytest -q tests
```

Dependencies are pinned; `--no-deps` avoids installing a second OpenCV package through MediaPipe. The test uses a public scikit-image test image, never client photos. Do not use the old `requisitos.txt` for the new application.

## Structure

- `app/`: current application, pt-BR interface, authentication, queue, payments, and administration.
- `03-motor/`: original facial engine reused; detector adjusted to reject more than one face.
- `tests/`: full flow, isolation, access, uploads, retention, signature verification, and reconciliation.
- `guia/` and `capa/`: public page and project cover.
- `06-app/`, `01-estrategia/`, `configura.py`, `marca.json`: reference for the legacy kit in Spanish, not used by the current Compose setup.
- `docs/legado/`: original instructions preserved for reference; they are not the current deployment path.

## Limitations and data handling

Proportions are 2D estimates influenced by the capture and do not validate beauty, personality, or health. Comparing proportions in the generated image **does not guarantee that identity is preserved**. Side-view simulation is disabled. The person reviews the result before delivery.

Photos are removed after 24 hours, reports after 90 days (configurable); the worker checks every minute. Backups need their own retention policy. The administrator must complete the data about the responsible party and the privacy policy before providing commercial service.

Mercado Pago uses the account configured during installation: there is no automatic split of proceeds among barbershops. For independent commercial operators, use one installation and set of credentials per company. Online payment requires credentials and sandbox validation in the destination account; automated tests use controlled responses.

## Origin

Adapted from the `visagismo-barberias.zip` file provided by the user. The kit does not include a license file; this adaptation does not declare a license for the original material. Check the source terms before redistributing or commercializing the kit.
