---
title: "Cloudflare Pages Custom Domain - API Token Limitation"
type: idea
tags: [cloudflare, pages, dns, custom-domain, deployment]
created: 2026-05-25
updated: 2026-05-25
quality: 5
source_session: 20260525-020000_daemon-deploy-personalization
related:
  - slug: daemon-mcp-server
    type: caused-by
---

# Cloudflare Pages Custom Domain Limitation

## Thesis
Adding a custom domain to a Cloudflare Pages project via API requires a separate DNS CNAME record in the zone. A wrangler API token with `Cloudflare Pages:Edit` permission can add the domain to the project, but it does NOT automatically create the DNS record — that requires separate `Zone:DNS:Edit` permission, which a typical deploy-only token doesn't have.

## Evidence
- Added `daemon.cloudenz.org` to Pages project `daemon-cloudenz` → `status: "pending"`, `"CNAME record not set"`
- Tried creating CNAME via API with the same token → `"Authentication error"`
- Zone is on Cloudflare (`zone_tag: 1c5c89f35cf18dfaf3f7454edcdb1256`) but token scopes don't include DNS write
- PATCH to domain endpoint still returns `"pending"` — doesn't trigger auto-provisioning

## Resolution
DNS record must be created manually:
- Type: CNAME
- Name: daemon
- Content: daemon-cloudenz.pages.dev
- Proxy: proxied
- Do this at dash.cloudflare.com → zone → DNS → Add record

## Implications
- For fully automated deploy pipelines, the deploy token needs **both** `Cloudflare Pages:Edit` and `Zone:DNS:Edit` permissions
- For manual setup (SSH-based deploys), just note the DNS record requirement in deploy docs
- If the zone is already on Cloudflare, Pages auto-provisions the cert once the CNAME is detected — no manual SSL setup needed
