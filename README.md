# e-vote test prototype

Responsive FastAPI and browser UI for learning and workflow checks. This is **not suitable for a real or binding election**. Face matching has no liveness/spoof protection; use synthetic test identities and consenting participants only.

## Features in this build

- New voter registrations are capped at 50 (`MAX_VOTERS`, default 50). Existing rows and duplicate candidate/voter entries are not silently removed or collapsed in lists.
- Admins can approve voters, create elections, assign approved voters per election, and add/remove candidates in Manage Parties. A candidate with a recorded vote cannot be deleted.
- Passwords use salted PBKDF2-HMAC-SHA256 hashes. Old plain-text password rows are rehashed after a successful login.
- Phone, email, document URL, and face templates are encrypted at the application layer with AES-256-GCM. Render Postgres also encrypts database storage at rest with AES-256.
- Face enrollment requires consent, uses a real `face_recognition` provider, and fails closed if the provider or encryption key is unavailable. Voters must verify before casting.
- The mobile sidebar has a dismissible backdrop and closes on navigation, backdrop tap, or Escape.

## Local setup

1. Use Python 3.11 and install `requirements.txt`.
2. Set `SESSION_SECRET` (32+ characters), `BOOTSTRAP_ADMIN_TOKEN` (32+ random characters), and `DATABASE_ENCRYPTION_KEY` (64 hex characters / 32 bytes).
3. Run `python seed.py` to create tables and the sample party/election (no accounts or login passwords are seeded).
4. Start with `uvicorn main:app --reload` and open `http://localhost:8000`.
5. Create the first administrator by POSTing to `/api/bootstrap-admin` with `token`, `username`, `phone`, and a 12+ character `password`. After success, remove `BOOTSTRAP_ADMIN_TOKEN`.

Generate local test secrets with `python -c "import secrets; print(secrets.token_urlsafe(48)); print(secrets.token_hex(32))"`. The first line is suitable for session/bootstrap secrets; use the hex line as `DATABASE_ENCRYPTION_KEY`. Never commit secret values. Back up the encryption key securely; without it, encrypted fields cannot be read.

## Render test deployment

The Blueprint needs `render.yaml` and `e-vote-github-upload.zip` at the GitHub repository root. Choose **New → Blueprint**, connect the repository, and set `BOOTSTRAP_ADMIN_TOKEN` and `DATABASE_ENCRYPTION_KEY` as secret environment values. The Blueprint uses free Render services and a free Postgres database, which expires after 30 days; treat all records as temporary. Do not use real voters or election data. The test service requires the Render Python build to successfully install `face-recognition`/dlib; if it cannot, face endpoints return an unavailable error rather than fake a match.

## Security limits

This prototype is not independently audited, does not use a formal secret-ballot protocol, does not provide face liveness detection, and must not be used for official elections. The demo page is a visual preview; backend operations require the FastAPI service.
