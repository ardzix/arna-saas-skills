# Arnatech payment-event contract

## Current and target flow

Current flow is `Xendit -> Payment Router -> Pulsar -> Arna Commerce`. Payment Router persists a `WebhookLog` then emits a legacy wrapper to `persistent://public/default/xendit-webhooks`:

```json
{
  "webhook_data": {},
  "metadata": {
    "webhook_log_id": 1,
    "xendit_event": "invoice.paid"
  }
}
```

Commerce consumes this wrapper, finds the invoice by `external_id`, and idempotently activates the invoice, order, subscription, entitlements, and any provisioning job.

The target normalized topic is `persistent://public/default/payment.xendit.webhook.received.v1`. Its envelope is:

```json
{
  "event_id": "uuid",
  "event_type": "payment.xendit.webhook.received.v1",
  "occurred_at": "2026-09-18T00:00:00Z",
  "producer": "payment-router",
  "correlation_id": "provider-event-or-webhook-log-id",
  "organization_id": null,
  "tenant_id": null,
  "data": {
    "provider": "xendit",
    "provider_event": "invoice.paid",
    "provider_event_id": "stable-id-if-present",
    "external_id": "commerce-invoice-number",
    "payload": {}
  }
}
```

`organization_id` and `tenant_id` can be absent at ingress because a provider event may not contain them. Commerce resolves them through the invoice and must not trust provider-supplied tenant context.

## Migration

1. Add normalized-event publication with a stable idempotency key while retaining legacy publication.
2. Add a Commerce consumer/subscription for the v1 topic and test equivalent idempotent state transitions.
3. Migrate each named consumer, observe lag, failures, duplicates, and invoice reconciliation.
4. Stop legacy publishing only after all consumers have moved and the retention window has passed.

Do not use an event as a command. A payment event states a provider fact; Commerce decides the resulting subscription and entitlement transition.

## Webhook and consumer requirements

- Require and constant-time verify the configured Xendit callback token in production. Keep the endpoint unauthenticated only for the provider, not unverified.
- Persist receipt before publishing; retain only the minimal headers/body necessary for reconciliation and protect/redact sensitive values.
- Producer retries must be bounded. Consumers must acknowledge only after a durable, idempotent effect; negative-ack/retry and dead-letter handling must be observable.
- Idempotency cannot depend only on a transient delivery attempt. Prefer provider event ID; otherwise use a stable external payment/invoice reference plus event type.
- Browser payment success is advisory UX. The authoritative state is Commerce after its event consumer commits.
