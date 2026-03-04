---
title: "Zero Trust Beyond the Buzzword"
date: 2026-02-15
tags: ["zero-trust", "architecture", "enterprise"]
---

Every vendor sells "zero trust" now. Most of them are selling network segmentation with a new label. Here's what zero trust actually looks like when you implement it across an enterprise.

## Identity Is the New Perimeter

This isn't news, but it's still poorly executed. The number of organisations that have a "zero trust strategy" but still allow lateral movement with service accounts is staggering.

The real work is unglamorous:

- **Inventory every identity** (human and non-human)
- **Enforce least privilege** at the application layer, not just the network
- **Continuously verify**, don't just authenticate at the gate

## The Architecture That Actually Works

You need three layers working together:

| Layer | Purpose | Key Tech |
|-------|---------|----------|
| Identity | AuthN/AuthZ for every request | IdP, OIDC, mTLS |
| Policy | Centralised, auditable decisions | OPA, Cedar |
| Observability | Verify trust continuously | SIEM, UEBA |

None of this is revolutionary. The hard part is **integration** across legacy systems, cloud workloads, and third-party SaaS. That's where architecture matters more than any single product.

## Where Most Implementations Fail

They treat zero trust as a project with an end date. It's an operating model. You're never done.
