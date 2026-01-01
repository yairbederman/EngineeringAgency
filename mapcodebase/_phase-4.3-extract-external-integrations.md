---
description: Phase 4.3 - Extract external third-party integration contracts
---

# Phase 4.3: Extract External Integrations

## Goal
Extract contracts for third-party service integrations (Stripe, SendGrid, Twilio, AWS, etc.)—enabling Tech Specs to use existing wrappers correctly.

## Trigger Condition
**Execute this phase IF** any third-party SDK imports detected:
- Payment: `stripe`, `@stripe/stripe-js`, `braintree`, `paypal`
- Email: `@sendgrid/mail`, `nodemailer`, `mailgun`
- SMS: `twilio`, `nexmo`, `vonage`
- Cloud: `aws-sdk`, `@aws-sdk/*`, `@google-cloud/*`, `@azure/*`
- Auth: `auth0`, `@auth0/auth0-spa-js`, `firebase-admin`
- Analytics: `segment`, `mixpanel`, `amplitude`
- Other: Any package with clear external service purpose

**Skip IF**: No third-party service SDKs detected.

## Input
- `package.json` / `pom.xml` / `requirements.txt` for dependencies
- `source-structure.json.discoveredLocations.services` for wrapper implementations

## Steps

### 1. Detect Third-Party Dependencies
Scan dependency files for known integrations:

| Category | Package Patterns |
|----------|-----------------|
| Payment | `stripe`, `braintree`, `paypal-*`, `square` |
| Email | `@sendgrid/*`, `nodemailer`, `mailgun.js`, `postmark` |
| SMS | `twilio`, `@vonage/*`, `plivo` |
| AWS | `aws-sdk`, `@aws-sdk/*` |
| GCP | `@google-cloud/*` |
| Azure | `@azure/*` |
| Auth | `auth0`, `firebase-admin`, `passport-*` |
| Storage | `cloudinary`, `@supabase/*`, `firebase/storage` |

### 2. Find Wrapper Implementations
Search for files that import and wrap the SDK:

```javascript
// Pattern: import SDK + export wrapper function/class
import Stripe from 'stripe';
export const stripeClient = new Stripe(process.env.STRIPE_KEY);
export async function createPaymentIntent(amount: number, currency: string) { ... }
```

### 3. Extract Wrapper Methods
For EACH wrapper function/method:

| Field | Source |
|-------|--------|
| `methodName` | Exported function/method name |
| `sdkMethod` | Which SDK method it calls internally |
| `parameters` | Input parameters with types |
| `returns` | Return type |
| `errorHandling` | How errors are caught/transformed |

### 4. Document Configuration Requirements
For EACH integration:

| Field | Source |
|-------|--------|
| `envVars` | Required environment variables |
| `initFile` | Where client is initialized |
| `singleton` | Is client a singleton instance? |

### 5. Map Error Handling Patterns
Document how the project handles SDK errors:

| Pattern | Detection |
|---------|-----------|
| Re-throw | Error caught and re-thrown as-is |
| Transform | Error transformed to custom error type |
| Fallback | Error caught with fallback behavior |
| Log-only | Error logged but swallowed |

## Output

### `analysis/external-integrations.json`
```json
{
  "integrations": {
    "[IntegrationName]": {
      "category": "payment | email | sms | storage | auth | analytics | cloud",
      "sdk": {
        "package": "stripe",
        "version": "^14.0.0",
        "importedFrom": "[file:line]"
      },
      "client": {
        "initFile": "[path]",
        "singleton": true,
        "instanceName": "stripeClient"
      },
      "wrapperMethods": {
        "[methodName]": {
          "file": "[path]",
          "sdkMethod": "stripe.paymentIntents.create",
          "parameters": {
            "[param]": {
              "type": "[type]",
              "required": true
            }
          },
          "returns": {
            "type": "[type]",
            "fields": {
              "[field]": "[type]"
            }
          },
          "errorHandling": {
            "pattern": "transform",
            "customErrorType": "[ErrorClassName]",
            "caughtErrors": ["StripeError", "StripeCardError"]
          }
        }
      },
      "configKeys": [
        {
          "name": "STRIPE_SECRET_KEY",
          "required": true,
          "usedIn": "[file:line]"
        },
        {
          "name": "STRIPE_WEBHOOK_SECRET",
          "required": false,
          "usedIn": "[file:line]"
        }
      ],
      "webhooks": {
        "handlerFile": "[path to webhook handler]",
        "events": ["payment_intent.succeeded", "invoice.paid"]
      }
    }
  },
  "_unusedIntegrations": [
    {
      "package": "[package name]",
      "installedButNotUsed": true,
      "reason": "No import found in source files"
    }
  ],
  "_coverage": {
    "integrationsDetected": 5,
    "wrapperMethodsExtracted": 23,
    "configKeysDocumented": 12
  }
}
```

## Webhook Handler Detection

For integrations that use webhooks (Stripe, SendGrid, etc.):

1. Search for webhook endpoint handlers (`/webhook`, `/api/webhook/*`)
2. Document which events are handled
3. Map event → handler function

```json
"webhooks": {
  "handlerFile": "src/api/webhooks/stripe.ts",
  "endpoint": "/api/webhooks/stripe",
  "events": {
    "payment_intent.succeeded": "handlePaymentSuccess",
    "invoice.paid": "handleInvoicePaid"
  }
}
```

## Critical Rules

1. **Full Wrapper Extraction**: Document ALL wrapper methods, not just common ones
2. **SDK Method Mapping**: Map each wrapper to exact SDK method called
3. **Error Patterns**: Document error handling approach for reliability
4. **Config Completeness**: List ALL required environment variables
5. **Webhook Coverage**: Document webhook handlers when present
6. **Unused Detection**: Flag installed but unused SDKs
