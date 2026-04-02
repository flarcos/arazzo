# Open Payments Arazzo Workflows

Machine-readable workflow specifications for the [Open Payments API](https://openpayments.dev), written in [Arazzo 1.0.1](https://spec.openapis.org/arazzo/v1.0.1).

These documents describe the exact API call sequences needed to execute common Open Payments operations — including inputs, request bodies, success criteria, data flow between steps, and outputs.

## Workflows

### Payments

| File | Workflow | Steps | Description |
|------|----------|-------|-------------|
| [one-time-payment-fixed-receive.arazzo.yaml](one-time-payment-fixed-receive.arazzo.yaml) | `oneTimePaymentFixedReceive` | 9 | Send a payment where the **recipient** specifies the amount to receive (e.g., e-commerce checkout) |
| [one-time-payment-fixed-send.arazzo.yaml](one-time-payment-fixed-send.arazzo.yaml) | `oneTimePaymentFixedSend` | 9 | Send a payment where the **sender** specifies the amount to send (e.g., peer-to-peer transfer) |
| [recurring-payment.arazzo.yaml](recurring-payment.arazzo.yaml) | `setupRecurringPayment` | 9 | Set up a recurring payment with interval-based spend limits (e.g., subscription billing) |

### Transaction History

| File | Workflow | Steps | Description |
|------|----------|-------|-------------|
| [transaction-history.arazzo.yaml](transaction-history.arazzo.yaml) | `listIncomingPayments` | 3 | List incoming payments with pagination |
| | `listOutgoingPayments` | 3 | List outgoing payments with pagination |
| | `getPaymentDetails` | 4 | Get details for a specific payment (incoming or outgoing) |

### Token & Grant Management

| File | Workflow | Steps | Description |
|------|----------|-------|-------------|
| [token-management.arazzo.yaml](token-management.arazzo.yaml) | `rotateAccessToken` | 1 | Rotate a GNAP access token |
| | `revokeAccessToken` | 1 | Revoke a GNAP access token |
| | `cancelGrant` | 1 | Cancel an active GNAP grant |

## Payment Flow Overview

A typical one-time payment involves 9 steps across 3 servers:

```
┌──────────┐     ┌──────────────┐     ┌─────────────────┐
│  Client  │     │  Auth Server │     │ Resource Server  │
└────┬─────┘     └──────┬───────┘     └────────┬────────┘
     │                  │                      │
     │ 1. GET wallet address ─────────────────►│ Discover servers
     │                  │                      │
     │ 2. POST grant ──►│                      │ Get incoming-payment token
     │                  │                      │
     │ 3. POST ────────────────────────────────►│ Create incoming payment
     │                  │                      │
     │ 4. GET wallet address ─────────────────►│ Discover sender's servers
     │                  │                      │
     │ 5. POST grant ──►│                      │ Get quote token
     │                  │                      │
     │ 6. POST ────────────────────────────────►│ Create quote
     │                  │                      │
     │ 7. POST grant ──►│ ◄── interactive ──── │ Get outgoing-payment token
     │                  │     (user consent)   │ (requires redirect)
     │                  │                      │
     │ 8. POST continue►│                      │ Finalize grant after consent
     │                  │                      │
     │ 9. POST ────────────────────────────────►│ Create outgoing payment
     │                  │                      │
```

## Arazzo Expression Examples

These workflows use [Arazzo runtime expressions](https://spec.openapis.org/arazzo/v1.0.1#runtime-expressions) for data flow between steps:

```yaml
# Reference an input value
$inputs.recipientWalletAddressUrl

# Reference a previous step's output
$steps.createIncomingPayment.outputs.incomingPaymentUrl

# Reference the current response
$response.body.access_token.value

# Reference the HTTP status code
$statusCode
```

## Source Descriptions

Each workflow references three OpenAPI source descriptions:

| Source | Description |
|--------|-------------|
| `walletAddressServer` | Wallet address resolution (discover auth + resource server URLs) |
| `authServer` | GNAP authorization server (grant requests, token management) |
| `resourceServer` | Open Payments resource server (payments, quotes) |

## Usage

### With the Arazzo SDK

These files are the input for [@flarcos/arazzo-sdk](https://github.com/flarcos/arazzo-sdk), which can:

```bash
# Validate the workflow specs
arazzo-sdk validate -i "*.arazzo.yaml"

# Generate a typed TypeScript SDK
arazzo-sdk generate -i "*.arazzo.yaml" -o src/generated/

# Inspect workflow structure
arazzo-sdk inspect -i one-time-payment-fixed-receive.arazzo.yaml
```

### As Documentation

Each YAML file is self-documenting. Open any file to see:
- **Step-by-step descriptions** of what each API call does
- **Request bodies** with all required fields
- **Success criteria** for each step
- **Output mappings** showing how data flows between steps

## Related Projects

- [@flarcos/arazzo-sdk](https://github.com/flarcos/arazzo-sdk) — Parse these specs and generate typed TypeScript SDKs
- [@flarcos/kiota-authentication-gnap](https://github.com/flarcos/kiota-authentication-gnap) — GNAP authentication for Open Payments
- [Open Payments](https://openpayments.dev) — The API standard these workflows document
- [Arazzo Specification](https://spec.openapis.org/arazzo/v1.0.1) — The workflow description format

## License

Apache-2.0
