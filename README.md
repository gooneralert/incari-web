# Incari web build (split-hosted)

Large asset (`incari.wasm`, ~72 MB) lives on GitHub raw so Cloudflare Pages never has to serve a
single file over its 25 MB limit. The small loader assets deploy to Cloudflare Pages.

## Layout

- `incari.wasm` — the TeaVM WebAssembly GC module, fetched at runtime from GitHub raw:
  `https://raw.githubusercontent.com/gooneralert/incari-web/main/incari.wasm`
- `boot.ts` — the page loader (`app-web/src/main/ts/boot.ts`). When the build has no `build.json`
  (standalone page uses `DEFAULT_MANIFEST`), `startBuild` loads the module from the raw URL above
  instead of a page-relative path.
- Deployed small assets (this folder minus `incari.wasm`) go to Cloudflare Pages via
  `wrangler pages deploy`. Live at `https://incari-web.pages.dev`.

## How it works

`boot.ts` `startBuild` resolves the module URL:

```ts
const moduleUrl =
  manifest.id === "development"
    ? "https://raw.githubusercontent.com/gooneralert/incari-web/main/incari.wasm"
    : versioned(manifest, target.module);
```

The raw wasm is served with `Access-Control-Allow-Origin: *`, matching the horizon-web split-deploy
method. Cloudflare Pages only serves the ~2 MB of loader assets (`index.html`, `boot.js`,
`incari.wasm-runtime.js`, …).

## Deploy

```sh
# Build: ./gradlew :app-web:teavm   (writes app-web/build/teavm/)
# Strip TS: pageScripts transform -> app-web/build/page/scripts/
# Deploy folder = build/teavm/ minus incari.wasm, plus build/page/scripts/*
npx wrangler pages deploy <deploy-folder> --project-name incari-web
```