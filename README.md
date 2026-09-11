# com-etzhayyim-app-ki

ki (木) — vascular synthesis layer. Absorbs raw signals via xylem, synthesizes
structured knowledge artifacts via LLM, publishes via phloem bloom, and
records growth ring checkpoints (per `wrangler.jsonc`'s `APP_DESCRIPTION`).

## Frontend migrated to ClojureScript (2026-09-07)

The `svelte/` directory (SvelteKit) is gone. The frontend is now
ClojureScript — reagent + re-frame + `jp-go-dds` (デジタル庁デザインシステム) —
at [`cljs/`](cljs). This was a **frontend-only** migration; the backend
Worker/XRPC logic was moved, not rewritten:

| Then | Now |
|---|---|
| `svelte/src/routes/+page.svelte` (the status page) | [`cljs/src/ki/app.cljk`](cljs/src/ki/app.cljk) — same facts + own path, faithfully ported (route/var counts corrected against `wrangler.jsonc`, see that namespace's docstring) |
| `svelte/src/routes/xrpc/[...path]/+server.ts` (the deployed XRPC handler, per `wrangler.jsonc`'s old `main`) | [`src/xrpc-dispatcher.ts`](src/xrpc-dispatcher.ts) — moved byte-for-byte, only a provenance header comment added |
| `wrangler.jsonc` `main: svelte/.svelte-kit/cloudflare/_worker.js` | `main` dropped entirely (a Cloudflare Worker can be assets-only) |
| `wrangler.jsonc` `assets.directory: ./svelte/.svelte-kit/cloudflare/client` | `assets.directory: ./cljs/public` |
| `wrangler.jsonc` `vars.APP_FRAMEWORK: sveltekit-edge-bff` | `vars.APP_FRAMEWORK: cljs-reagent-re-frame-jp-go-dds` |

`main` is **not** repointed at `src/xrpc-dispatcher.ts`: that handler does not
call `env.ASSETS.fetch`, so putting it in front of the static assets would
mean nothing serves the frontend. It remains preserved, unwired source —
whether/how to revive it as a standalone Worker fetch handler is a product
decision outside the scope of this migration.

This change is **UNVERIFIED**: no `wrangler deploy` or `wrangler dev` was run
against it.

`lg-clj/`, `kotoba/`, and `xrpc-adapter/` were not touched by this migration.

### Build / test

```bash
cd cljs
npm install
npm run build   # amu compile --target wasm32-browser app -> public/js/app.js
npm test        # amu compile --target wasm32-browser test && node out/tests.js
```
