# OpenJEV Support

This fork adds **optional** support for [OpenJEV](https://openjev.sh) — a free community gateway to the same Jev model built by [TypeSafe](https://typesafe.ai) — alongside the existing TypeSafe integration. **TypeSafe remains the default; nothing changes for anyone with a `TYPESAFE_API_KEY`.**

## What was added

| File | Change |
|---|---|
| `src/typesafe-client.ts` | Added `OPENJEV_BASE_URL`, `OPENJEV_MODEL`, `JevProvider` type, `resolveOpenJevApiKey()`, `resolveProvider()`; constructor now resolves the provider; error message updated to mention both keys. |
| `src/types.ts` | Added `provider?: 'typesafe' \| 'openjev'` to `TypeSafeClientConfig`. |
| `lib/typesafe-client.js` | Compiled JS kept in sync with the source changes above. |
| `lib/typesafe-client.d.ts` | Type declarations kept in sync (new exports + `provider` field). |
| `lib/types.d.ts` | Added `provider` to the compiled `TypeSafeClientConfig`. |
| `README.md` | Short note after the project intro + OpenJEV key example in the Usage section. |
| `docs/configuration.md` | Documented the `provider` config field and OpenJEV endpoint/model/key. |

No TypeSafe code was removed, renamed, or re-defaulted. The `Authorization: Bearer` header and request/response contract are identical for both providers.

## Provider selection rule

1. **Explicit choice wins** — `config.provider` or the `JEV_PROVIDER` environment variable (`typesafe` / `openjev`).
2. **Otherwise, if a TypeSafe key is available** (`config.apiKey` or `TYPESAFE_API_KEY` env / `~/.dsh/.env`) → **TypeSafe** (unchanged default).
3. **Otherwise, if only an OpenJEV key is available** (`OPENJEV_API_KEY` env / `~/.dsh/.env`) → **OpenJEV**.

Explicit `config.baseUrl` / `config.model` overrides still take precedence over the provider defaults.

## How to configure

```bash
# ~/.dsh/.env  (or export in your shell)

# TypeSafe (default — unchanged):
TYPESAFE_API_KEY=your_typesafe_api_key_here

# OpenJEV (optional — used when no TypeSafe key is set, or forced):
OPENJEV_API_KEY=your_openjev_api_key_here

# Force OpenJEV even when a TypeSafe key is also present:
JEV_PROVIDER=openjev
```

Or programmatically:

```ts
ctx.plugin(TypeSafeClient, { provider: 'openjev' })
```

| Provider | Endpoint | Model | Key env var |
|---|---|---|---|
| TypeSafe (default) | `https://api.typesafe.ai/v1/systemone` | `jev-latest` | `TYPESAFE_API_KEY` |
| OpenJEV | `https://api.openjev.sh/v1/systemone` | `openjev` | `OPENJEV_API_KEY` |

Both providers share the same request/response contract. Retryable HTTP statuses (429, 5xx including 503 and 529) are already handled by the existing `status >= 500` transient-error check in `safety-guard.ts`.

## How it was verified

A live `POST https://api.openjev.sh/v1/systemone` request with model `openjev`, state `ping`, and one `noul` question returned HTTP 200 with a valid answer. No repository code was executed during verification.

## Upstream

Original project: https://github.com/zhangxaochen/dsh-jev by [@zhangxaochen](https://github.com/zhangxaochen) (MIT license).
