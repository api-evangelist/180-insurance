---
name: Subscribe to and verify 180 Seguros webhooks
description: Register a webhook destination for the insurance lifecycle, verify the HMAC-SHA256 signature on every delivery, and rotate the signing key safely.
api: https://docs.180s.com.br/reference/webhooks-eventos
operations:
  - POST /sagas/v1/webhooks
  - GET /sagas/v1/webhooks
  - PATCH /sagas/v1/webhooks/{id-webhook}
  - DELETE /sagas/v1/webhooks/{id-webhook}
  - POST /sagas/v1/webhooks/{id-webhook}/rotacionar-chave-hmac
---

# Webhook subscription & verification

## Auth
Use the same OAuth2 client-credentials bearer token as the rest of the Sagas API.

## Steps
1. **Register** — `POST /sagas/v1/webhooks` with your destination URL. It must answer `POST` with a 2xx (e.g. 204). Optionally set `segredo-compartilhado` to also receive an `Authorization: Bearer {secret}` header. Store the returned HMAC signing key.
2. **List / confirm** — `GET /sagas/v1/webhooks` returns active subscriptions and their URLs.
3. **Receive events** — every delivery carries the envelope `{ id-evento, evento, criado-em, versao, detalhes }`. React to `evento` (e.g. `venda-completa`, `renovacao-disponivel`, `sinistro-pagamento-concluido`). Reply 2xx quickly.
4. **Verify the signature** — read the `i80-signature` header (`t={timestamp},{version}={signature}`). Compute `HMAC-SHA256(key, "{timestamp}.{raw_body}")` in hex and compare to the header signature. Reject stale timestamps. Verify on the **raw** body before parsing.
5. **Rotate the key** — `POST /sagas/v1/webhooks/{id-webhook}/rotacionar-chave-hmac`. For 7 days both keys sign each delivery (two signatures); accept either, then cut over to the new key.
6. **Update / remove** — `PATCH` to toggle `ativo` or change the URL; `DELETE` to permanently remove (cannot be re-activated).

## Notes
- Always start by handling the `ping` verification event.
- Idempotency is not provided by the API, so de-duplicate on `id-evento` on your side.
