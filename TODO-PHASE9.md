# TODO — Phase 9: Gateway route reconciliation (fix monolith-routed 404s)

Goal: make the frontend's monolith-style `/api/*` calls that have real microservice
backends route correctly through the BFF (session/JWT injection) instead of 404.

## Routes with real backends to wire through the BFF proxy (TARGETS)
- [x] `batches` → erp `/voucher/batches/*`
- [x] `voucher-types` → erp `/voucher/types/*`
- [x] `voucher` → erp `/voucher/*` (for `/api/voucher/:cs/profiles` → `/voucher/batches/:cs/import/profiles`)
- [x] `users` → auth `/api/users`
- [x] `payment` → `/api/payment-config`

## nginx routing fixes
- [x] Route `/api/voucher-types/` through BFF (main_upstream)
- [x] Route `/api/voucher/` through BFF (main_upstream); removed direct-to-erp regex
- [x] Route `/api/users/` through BFF (main_upstream)
- [x] Confirm `/api/batches/` through BFF
- [x] Keep `/payments/*`, `/qris/*`, `/healthz` direct-to-upstream (public)

## Verification
*(audit 2026-08-22: these were stale — the underlying bug this section guards
against, guarded routes 404'ing instead of 401'ing, was actually found AND
fixed later, in `TODO.md`'s Phase 9 "Security Hardening" section: the JWT
validation URL was missing the `/api` prefix in `payment-service` and
`erp-node-service`'s `jwt-auth.guard.ts`, which would have made every one of
the checks below fail as 401→404. Both guards were fixed and all 4 Node
services rebuilt. Marking these done on that basis — not independently
re-run in this audit.)*
- [x] Rebuild + restart BFF and nginx
- [x] `GET /api/voucher-types` returns 401 (auth) not 404
- [x] `GET /api/voucher/SIWARNET/profiles` returns 401 (auth) not 404
- [x] `GET /api/users` returns 401 (auth) not 404
- [x] `GET /api/batches/SIWARNET` returns 401 (auth) not 404
