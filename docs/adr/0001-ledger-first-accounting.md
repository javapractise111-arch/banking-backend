# Architecture Decision Record 0001

Date: 2026-10-12

Status: Accepted

## Context

A banking backend must preserve integrity, auditability, and defensible traceability for all money movements. Conventional balance-only updates are insufficient because they obscure the source, destination, and business intent of each movement and are prone to race conditions and duplicate processing.

## Decision

The system uses a ledger-first model where every monetary movement is represented as balanced journal entries. All transfers and adjustments must net to zero across debit and credit accounts. The ledger is the authoritative record; account balances are derived and reconciled from it rather than treated as the single source of truth.

## Consequences

- Financial integrity is stronger and easier to audit.
- Transfers cannot create or destroy money by accident.
- Duplicate or concurrent commands must be handled with idempotency and locking.
- More database writes are required, but correctness and auditability improve materially.
- The architecture supports independent reconciliation and statement generation.
