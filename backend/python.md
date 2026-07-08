# Backend Playbook: Python (FastAPI)

Use Python/FastAPI when: you want the Python ecosystem, or the API sits close to
ML/data work.

## Project layout

```txt
app/
  main.py
  deps.py         # auth dependency (JWKS)
  routers/        # health, me, webhooks
  db/
    models.py     # SQLAlchemy models
    session.py
  schemas.py      # Pydantic DTOs
alembic/          # migration env + versions
alembic.ini
Dockerfile
```

## Libraries

- HTTP framework: FastAPI.
- ORM: SQLAlchemy 2.x (or SQLModel) over Postgres.
- Migration tool: Alembic.
- Validation: Pydantic v2 (FastAPI's request/response models).
- Redis client: `redis` (redis-py, async).

## Auth verification

- Verify the token per `../auth.md`'s Auth Swap Contract via JWKS.
- Use **PyJWT** with `PyJWKClient`:
  `jwks_client = PyJWKClient(jwks_url)`, then
  `signing_key = jwks_client.get_signing_key_from_jwt(token)`, then
  `jwt.decode(token, signing_key.key, algorithms=[...], audience=..., issuer=...)`.
  Wire it as a FastAPI dependency that returns the verified user id. (PyJWT is the
  maintained choice; python-jose is effectively unmaintained.)

## Database & migrations

- SQLAlchemy models are the source of truth.
- Migrations via Alembic: `alembic revision --autogenerate` then
  `alembic upgrade head`.
- Run before production deploy. Use SQLAlchemy transactions for multi-table writes.

## Health, webhooks, security

- `GET /health` route with no auth.
- Webhook routers verify provider signatures against the raw body before mutating.
- FastAPI `CORSMiddleware` scoped to known clients; a rate-limit middleware
  (e.g. slowapi) before expensive endpoints.

## Coolify deploy

- Dockerfile running `uvicorn` (behind `gunicorn` workers for production).
  Expose `/health`.
- Run `alembic upgrade head` as a release step.

## Env vars

- Public (bundle-safe): none server-side.
- Secret (server only): `DATABASE_URL`, `REDIS_URL`, auth provider secrets, plus
  payment/AI keys in scope.

## Contract compliance

1. `GET /health` responds without auth. ✓
2. Protected routes verify the JWT via PyJWT + `PyJWKClient` dependency. ✓
3. No trusted values from client input; identity from the verified token. ✓
4. Pydantic v2 validates bodies/params before DB writes. ✓
5. Typed DTOs returned (Pydantic response models), not raw rows. ✓
6. Alembic migrations committed + run pre-deploy; transactions; indexes. ✓
7. Webhooks verify signatures against the raw body before mutating. ✓
8. Secrets stay in server env only. ✓
9. uvicorn/gunicorn Dockerfile + `/health` + env list + documented migrate
   command on Coolify. ✓
10. HTTPS, scoped CORS, rate limits, no secret logging. ✓

## Official References

- FastAPI: https://fastapi.tiangolo.com/
- SQLAlchemy: https://docs.sqlalchemy.org/
- Alembic: https://alembic.sqlalchemy.org/
- PyJWT: https://pyjwt.readthedocs.io/
