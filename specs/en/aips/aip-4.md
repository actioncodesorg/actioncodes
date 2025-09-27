# AIP-4: Prefix System

- **Author**: Action Codes Core Team  
- **Status**: Dropped  
- **Created**: 2025-06-29
- **Updated**: 2025-09-27

## Overview

**!! This AIP has been dropped and is no longer in use. !!**

Prefix system was a trust layer enabling branded, verifiable, and auditable prefixes while keeping user UX simple (prefixless codes) but was dropped in favor of elasticity and decentralization.

**Key goals:**
- Ensure prefix authenticity and authority verification
- Support manual verification with future DAO/admin automation  
- Enable dynamic fees and governance
- Fully cross-chain ready and relayer-agnostic  

## Motivation

Prefixes provide namespaces, trust branding, and metadata routing in the Action Codes Protocol:

- Allow apps, wallets, and relayers to verify code authenticity and trust
- Enable branding and metadata routing for organizations
- UX is simple: users enter prefixless codes; prefixes exist only for trust verification
- Fully audit-ready and ready for automation or DAO governance

## Specification

### Prefix Structure

- **Prefix**: 3–12 uppercase alphanumeric characters
- **PrefixAccount**: On-chain representation of a verified prefix
- **Purpose**: Provide canonical trust for relayers/wallets and link prefix to owner, metadata, and fee reference

#### PrefixAccount Fields

| Field | Type | Description |
|-------|------|-------------|
| `prefix` | string | Uppercase alphanumeric prefix (3–12 chars) |
| `owner_pubkey` | pubkey | Main wallet controlling prefix |
| `authority_keys` | pubkey[] | Keys authorized to generate / sign codes for this prefix |
| `metadata_uri` | string | URI to JSON metadata following Prefix Metadata Schema v1 ([Default is v1](../../schemas/meta_v1.json)) |
| `ref` | hash | Hash of approval record (verifier signatures + timestamp) |
| `status` | enum | pending, approved, rejected, active, inactive |
| `fee_paid` | bool | True if registration fee paid |
| `created_at` | u64 | Submission / approval timestamp |
| `updated_at` | u64 | Last update timestamp |
| `bump` | u8 | PDA bump for on-chain account |

**Notes:**
- `authority_keys` allow multiple backend servers, relayer programs, or delegated signing keys
- Protocol verifies code signatures against any key in `authority_keys`

### Prefix Registration Flow

#### 1. Submission (Off-Chain / Platform)

- Restricted to registered verifiers (see Verifiers list)
- Submission includes:
  - Prefix name
  - Owner pubkey
  - Metadata URI (JSON following Prefix Metadata Schema v1 (Default is v1))
  - Optional KYC info or supporting evidence (token ownership, domain proof)
- Platform stores request as pending

#### 2. Manual Verification

- Only registered verifiers can approve / reject:
  - Check KYC / identity
  - Validate metadata against Prefix Metadata Schema v1 (Default is v1)
  - Validate token / domain ownership if required
- Status updated:
  - `approved` → moves to pending fee payment
  - `rejected` → reason stored

#### 3. Fee Payment (Pending → Active)

- Fee is dynamic, stored in FeeRegistry
- User pays fee → `fee_paid = true` → status becomes `active`
- On-chain PrefixAccount created at this point

#### 4. On-Chain PrefixAccount

- Stores verified prefix + ref to approval record
- Authority keys included for code signature verification
- Prefix used only in metadata (`pre=PREFIX`), not shown to end users

### FeeRegistry / FeeSystem

| Field | Type | Description |
|-------|------|-------------|
| `current_fee` | u64 | Fee in lamports (SOL smallest unit) |
| `updated_at` | u64 | Timestamp of last update |
| `admin_pubkey` | pubkey | Admin / DAO wallet pubkey |

- Fees can be edited by admin/DAO
- Collected after approval to ensure UX simplicity
- Prevents spam / squatting

### Verifiers List

| Field | Type | Description |
|-------|------|-------------|
| `verifier_pubkey` | pubkey | Wallet authorized to approve / submit prefix |
| `status` | enum | active, inactive |
| `added_at` | u64 | Timestamp verifier added |
| `added_by` | pubkey | Who added this verifier |
| `removed_at` | u64 | Optional removal timestamp |

- Multiple verifiers possible → consensus-based approval

### Prefix Metadata Schema v1

All prefix metadata must conform to the Prefix Metadata Schema v1, which includes:

**Required fields:**
- `title`: Source of the action request (brand, app, or person) - max 50 chars
- `icon`: HTTP/HTTPS URL to icon image (SVG, PNG, or WebP)
- `description`: Information about the action - max 200 chars
- `label`: Text for user action button (verb + max 5 words) - max 30 chars
- `intentCategories`: Array of action categories (payment, stake, vote, etc.)
- `contact`: Flexible contact object with key-value pairs (default required is email)

**Optional fields:**
- `website`: Official website for branding/verification
- `schemaVersion`: Must be "1"

**Contact object example:**
```json
{
  "contact": {
    "email": "support@ota.codes",
    "telegram": "@otacodes", 
    "discord": "otacodes#1234"
  }
}
```

**Schema validation:**
- Verifiers must validate metadata against the canonical schema
- Protocol provides methods to validate metadata against the schema by version (Default is v1)
- Invalid metadata results in rejection with specific error details
- Metadata URI must be accessible and return valid JSON

### Code Usage & Verification

#### User Side
- Users enter prefixless codes (e.g., 12345678)
- Prefix is metadata only for trust verification

#### Relayer / Protocol Verification
1. Fetch PrefixAccount for `pre=PREFIX`
2. Verify:
   - `status = active`
   - Code signature matches any key in `authority_keys`
3. May fetch and validate metadata from `metadata_uri`:
   - Must conform to Prefix Metadata Schema v1 (Default is v1)
   - Verify `icon` URL is accessible and valid format
   - May check provided`intentCategories` match expected action type
4. Optional: relayer-specific checks (domain, registration, SAS)
5. If all checks pass → attach TX / execute action

**Cross-chain ready:** PrefixAccount + authority_keys can validate codes from any blockchain relayer

### Security & Anti-Spam

- Fee prevents spam
- Prefixes reserved for branded tokens or businesses require verification
- Only registered verifiers can approve → prevents abuse
- `ref` ensures full auditability
- Supports revocation / deactivation by owner or DAO

### Manual Flow Diagram

```mermaid
flowchart TD
    A[User submits prefix request + metadata URI + KYC] --> B[Platform stores request pending]
    B --> C[Registered verifier checks submission]
    C --> D[Validate metadata against Schema v1 Default is v1]
    D --> E{Approve or Reject?}
    E -->|Reject| F[Submission rejected, reason stored]
    E -->|Approve| G[Status = approved, fee pending]
    G --> H[User completes fee payment]
    H --> I[Platform creates PrefixAccount on-chain with ref + authority_keys]
    I --> J[Relayers/wallets fetch PrefixAccount to verify trust & signature]
    J --> K[User enters code; relayer verifies code signature using authority_keys]
    K --> L[Relayer validates metadata from URI]
    L --> M[Attach TX / execute action if valid]
```

## Key Principles

1. **Users never see prefixes**: UX = one-time code entry
2. **Prefix = trust anchor only**: verified by authority_keys
3. **Manual approval now, scalable to multiple verifiers / attestation services**
4. **Dynamic fees**: anti-spam, flexible governance
5. **Auditability**: ref stores hash of approval record
6. **Cross-chain ready**: metadata / authority verification works across blockchains
7. **Delegated / multiple authority keys**: future-proof for backend servers and relayer programs

## Notes

- Fully manual for now: future automation is possible with multiple verifiers or attestation services
- Protocol remains chain-agnostic; PrefixAccount + ref are canonical trust anchors
- UX is simple: users type code, see trust badge, and confirm transaction
- Supports dynamic fee, revocation, metadata update, and auditability
- Compatible with AIP-1…AIP-5 canonical code formats, delegated authenticators

