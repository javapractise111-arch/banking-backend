# Threat Model

## Assets

- Customer identity and authentication data
- Account balances and transaction histories
- Ledger integrity
- Customer beneficiary data and scheduled payments
- Audit logs and operational telemetry

## Threats considered

- Credential stuffing and brute force attempts
- IDOR / authorization bypass against account data
- Duplicate financial commands and replay attacks
- Concurrent withdrawals and transfers
- Ledger imbalance and tampering
- Sensitive data leakage through logs or exceptions
- External service failures and outbox gaps

## Controls in design

- JWT-based auth with RBAC and least privilege
- MFA-ready design and secure password storage
- Role and account ownership validation on every financial operation
- Idempotency keys for commands and reconciliation support
- Optimistic locking and transaction boundaries around updates
- BigDecimal-based money calculations to prevent float precision issues
- Redacted logging and secure secret handling patterns
- Transactional outbox for reliable downstream notifications/events

## Residual risk

Production deployment requires independent security review, secrets management, penetration testing, and formal operational controls before claiming readiness.
