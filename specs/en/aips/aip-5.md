# AIP-5: Delegated Authenticators (Mode A)

- **Author**: Action Codes Core Team
- **Status**: Draft
- **Created**: 2025-09-08

## Overview

This AIP introduces _Delegated Authenticators_ _Mode A_, allowing users to generate action codes without requiring a wallet signature per code. Delegated authenticators, or passkeys, enable devices or external authenticators to sign codes on behalf of a wallet, improving usability and flexibility.

## Motivation

Delegated authenticators provide a better user experience by enabling device-based delegation of signing authority. They function similarly to session keys, avoiding constant wallet popups and enabling smoother interactions. This approach supports modern authentication methods like passkeys and WebAuthn while maintaining security.

## Specification

### Delegation Certificate Structure

A delegation certificate is issued by a wallet and includes the following fields:

- `wallet_pubkey`: The public key of the wallet delegating authority.
- `auth_pubkey`: The public key of the delegated authenticator.
- `scopes`: Optional permitted actions or code types the authenticator can sign.
- `origin`: Optional origin or domain restriction for the authenticator.
- `not_before`: Timestamp indicating when the certificate becomes valid.
- `not_after`: Timestamp indicating when the certificate expires.
- `signature`: Signature by the wallet over the certificate fields.

### Signing Flow

The delegated authenticator signs the canonical message representing the code using its `auth_pubkey`. The resulting signature is attached along with the delegation certificate issued by the wallet. This allows the authenticator to generate valid signatures on behalf of the wallet without requiring the wallet to sign each code individually.

### Verification Rules

Protocol-level verification requires that the delegation certificate is validly signed by the wallet's `wallet_pubkey`. The certificate must be within its validity period, as defined by `not_before` and `not_after`. If present, the `scopes` field restricts the actions or code types the authenticator is permitted to sign, and the `origin` field binds the certificate to a specific domain or origin. The signature over the canonical message must be verified using the `auth_pubkey` contained in the delegation certificate.

### Resolution Flow

Resolution uses the code and its canonical metadata to verify authenticity. Verification of Mode A signatures follows the rules described above, ensuring the delegation certificate is valid and the delegated authenticator's signature on the canonical message is correct.

## Security Considerations

- Revocation is managed by issuing new certificates or setting expiry timestamps.
- Certificates are bound to origins to prevent misuse across domains.
- Replay attacks are mitigated by including timestamps and scopes.
- The wallet remains the root of trust, controlling delegation and revocation.

## Notes

- This mode is backward-compatible with Mode W (wallet signatures).
- Delegated authenticators are optional for integrators.
- Enables integration with passkeys, WebAuthn, and other modern authentication workflows.

---
