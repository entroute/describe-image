---
name: describe-image
description: Generate alt text or a natural-language description for an image via a verified x402 pay-per-request endpoint. Use when the user shares an image URL and wants it described, needs alt text for accessibility, wants to analyze/tag image contents, or needs to feed visual content to a text-only agent.
---

# Describing images via EntRoute

When the user needs an image described (alt text, caption, content tagging), use this skill to call a ranked, fulfillment-verified x402 endpoint.

## Steps

1. **Extract the image URL** from the user's request. Note the intended use (alt text = short, concise; caption = medium; detailed analysis = long).

2. **Discover the best endpoint.** Call EntRoute's `/discover`:

   ```bash
   curl -s -X POST https://api.entroute.com/discover \
     -H "Content-Type: application/json" \
     -d '{
       "capability_id": "image.describe",
       "constraints": {"network": "base", "verified_only": true},
       "preferences": {"ranking_preset": "balanced"}
     }'
   ```

   Take `ranked_endpoints[0]`. Check `sample_request` for exact param names. Common shapes: `{ image_url, detail }` or `{ image, mode }`. A few endpoints accept base64 `image_data` instead of a URL.

3. **Build the request matching `sample_request`.** If the user provided a URL, send `image_url`; if they uploaded an image or provided base64, use the corresponding param from the sample.

4. **Pay with x402.** With `@entroute/mcp-server` or `@entroute/sdk-agent-ts` installed, `call_paid_api` / `discoverAndCall` handles payment automatically.

5. **Parse and return.** Response typically contains a `description`, `caption`, or `text` field. Use `sample_response` to confirm the shape and extract the relevant field.

## Preferred approach

With Claude Code + `@entroute/mcp-server`, use the MCP tools `discover_paid_api` and `call_paid_api` directly — discovery + payment in one step.

## Tuning for use case

- **Alt text** (accessibility): request `detail: short` or `mode: alt_text`. Keep response under ~125 chars.
- **Caption / social post**: `detail: medium`.
- **Content moderation / tagging**: `detail: detailed` or request category tags.

## Notes

- Default network is Base; pricing is USDC. Image-description endpoints typically cost $0.003–$0.02 per call.
- Public image URLs work; some endpoints can't reach images behind auth.
- Full docs: https://entroute.com/docs/quickstart
