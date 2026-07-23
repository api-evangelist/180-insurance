---
name: Quote to policy (cotação → proposta → venda)
description: Run the 180 Seguros embedded-insurance happy path — list products, quote a risk, issue a proposal, and confirm the resulting sale/policy.
api: https://docs.180s.com.br/reference/introducao
operations:
  - GET /sagas/v1/combos
  - POST /sagas/v1/cotacoes
  - POST /sagas/v1/propostas
  - GET /sagas/v1/propostas/{id-proposta}
  - GET /sagas/v1/vendas/{id-venda}
---

# Quote to policy

Sell an embedded insurance policy through the 180 Seguros Sagas API.

## Auth
Get an OAuth2 client-credentials token from Auth0 and reuse it (valid ~12h):

```
POST https://180s-staging.us.auth0.com/oauth/token
{ "grant_type": "client_credentials", "client_id": "...", "client_secret": "..." }
```

Send `Authorization: Bearer {access_token}` on every call to `https://sagas.staging.180s.com.br`.

## Steps
1. **List products** — `GET /sagas/v1/combos` to get the `id-combo` and `id-produto` available to your channel.
2. **Quote** — `POST /sagas/v1/cotacoes` with the combo/product ids, the risk, and chosen coverages. Attach any partner context via the `metadados` key/value map. Keep the returned `id-cotacao`.
3. **Issue a proposal** — `POST /sagas/v1/propostas` referencing the `id-cotacao`. The proposal returns status `pendente` (outstanding requirements, e.g. payment) or `completa`.
4. **Poll / confirm** — `GET /sagas/v1/propostas/{id-proposta}`. When it reaches `completa`, a **venda** (sale/policy) is created automatically and the policy is issued.
5. **Confirm the sale** — `GET /sagas/v1/vendas/{id-venda}` for the issued policy and related entities.

## Conventions
- Prefer webhooks over polling: subscribe to `proposta-criada`, `venda-criada`, `venda-completa` (see the webhook catalog) and verify the `i80-signature` HMAC-SHA256 header.
- Errors are RFC 9457 `application/problem+json`; branch on the `problem-id` field. Business-rule failures are HTTP 422.
- List endpoints are cursor-paginated (`pagina-seguinte` / `pagina-anterior` / `limite`).
