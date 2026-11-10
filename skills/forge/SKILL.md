---
name: forge
description: Generate a textured 3D GLB from a text prompt, image URLs or a sketch, and auto-rig GLBs into animation-ready humanoids, using @three-ws/forge. Use when code or the user needs a generated 3D model, a rigged character, or the live Forge tier and cost catalog.
---

# Forge SDK

`@three-ws/forge` (Node 18+, zero dependencies, ESM) is a thin client over the public three.ws `/api/forge` endpoint. It submits a generation, polls the job, and resolves with a GLB URL.

Install: `npm install @three-ws/forge`

## Calls

```js
import { forge, rig, catalog, getJob, createForge } from '@three-ws/forge';
```

- `forge(promptOrInput, opts)` where input is a string prompt or `{ prompt, images: [urls], aspectRatio }`. Needs a prompt or at least one image. Options: `path` (`image|geometry|sketch`; sketch needs a prompt naming the drawing), `tier` (`draft|standard|high`), `backend`, `payWith` (`credits|x402`), `providerKey` (BYOK, sent as `x-forge-provider-key`), `onProgress(job)`, `pollIntervalMs` (2500), `timeoutMs` (180000), `signal`.
- `rig(glbUrl, opts)` auto-rigs an existing GLB into a humanoid.
- `catalog()` returns the live tier, backend and cost matrix. Call it before choosing a paid tier.
- `getJob(jobId)` reads one job once without waiting.
- `createForge({ baseUrl, fetch, apiKey, headers, providerKey })` reuses configuration. `THREE_WS_BASE_URL` overrides the default origin `https://three.ws`.

Result: `{ jobId, creationId, status, glbUrl, viewerUrl, path, tier, backend, etaSeconds, estimatedCredits, durable, raw }`. Errors are `ThreeWsError` with `.code` and `.status`, and `PaymentRequiredError` (HTTP 402) with `.accepts`, the x402 challenge.

## Lanes

- Default credits lane: includes the free TRELLIS lane, plus account credits for paid tiers. Pass `apiKey` to `createForge` for account use.
- `payWith: 'x402'` buys one generation with one USDC payment, no account. It cannot be combined with `backend`, `path` or `providerKey`. Without a payment-aware `fetch` the first call throws `PaymentRequiredError` carrying the price.

## Worked example

```js
import { forge, rig } from '@three-ws/forge';

const model = await forge('a small ceramic frog figurine', {
  tier: 'draft',
  onProgress: (j) => console.log(j.status, j.etaSeconds),
});
console.log(model.glbUrl, model.viewerUrl);

const character = await forge('a friendly cartoon astronaut', { tier: 'draft' });
const rigged = await rig(character.glbUrl);
console.log(rigged.glbUrl);
```

## Guidance

- Describe one subject per prompt. For a character you intend to animate, generate then `rig()`.
- Payment is an on-chain spend. If a call throws `PaymentRequiredError`, show the owner `err.accepts` (amount, asset, network, recipient) and wait for an explicit yes before configuring a paying `fetch`. Never pay automatically.
- Never log or commit an API or provider key.
