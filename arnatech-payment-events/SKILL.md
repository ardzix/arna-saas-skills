---
name: arnatech-payment-events
description: Implement or review Arnatech payment webhook, Commerce, and Pulsar workflows. Use for Xendit callbacks, payment events, invoice activation, entitlement propagation, or event migration work.
---

# Arnatech Payment Events

Read [the payment-event contract](references/event-contract.md) before changing Payment Router, Commerce consumers, or payment-facing UI.

Payment Router is the only external payment webhook ingress. Verify the provider callback before accepting it, persist the received event before asynchronous delivery, and return a provider-safe response. Commerce owns invoice, order, subscription, and entitlement transitions; consumer services must not infer payment state from a browser redirect.

Use idempotency keys derived from the provider event or stable external reference, make consumers safe for redelivery, and retain explicit retry/dead-letter observability. For a migration from the legacy `xendit-webhooks` topic, dual-publish and migrate named consumers before retiring the old topic. Do not break a live checkout path or publish provider secrets.
