# Ledger Client

Signs Governance Events with the configured post-quantum key and writes
them to the configured Immutable Ledger endpoint.

## Interface

Input:  Governance Event JSON (pre-signature)
Output: Signed Governance Event written to ledger; HTTP response status

## Signing

Uses the algorithm specified in govops.yml signature_algorithm field.
Default: dilithium3 (CRYSTALS-Dilithium, NIST PQC standard).
Per GOS §11.2.

## Write Endpoint Contract

POST  /v1/events
Body: Governance Event JSON
Auth: Bearer token (LEDGER_API_KEY)

On success: HTTP 201 Created + { "event_id": "<identity>", "chain_position": N }
On failure: HTTP 4xx/5xx + error body; bridge retries up to 3 times with
            exponential backoff before raising a governance alert.
