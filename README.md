# guivo-files

Static asset mirror for [Guivo](https://guivo.io), served over GitHub Pages as one hop in the game's
CDN fallback chain.

The client walks `Cloudflare Pages -> Cloudflare R2 -> GitHub Pages -> guivo.io`, measuring every hop
in parallel at boot and downloading from the fastest one that answers. This hop exists because the
other two are both behind Cloudflare, so a bad client-to-Cloudflare route takes them out together.
GitHub Pages is served by Fastly, which gives the chain a genuinely independent third option.

## Why these files

Only the types the game routes through a CDN: `ogg`, `mp3`, `m4a`, `png`, `svg`. Filenames are
content-hashed by Vite, so a name always refers to the same bytes and the files are safe to cache
forever.

Files live at the repo root, so the base url is `https://ivorcosta.github.io/guivo-files/` and each
asset is that plus its filename.

## Updating before a game deploy

In the guivo repo, `npm run build` (in `src/main/resources/static/main`) ends by listing any `dist/assets`
file of those five types that is missing here. Run `npm run cdn:sync` there to push them. It then waits
until all three hops serve them. Hashed names mean new builds add files rather than replace them, so
prune names no longer referenced by a live build occasionally to keep the repo small.

A push to `main` also publishes the same files to Cloudflare Pages (project `guivo`) and the R2 bucket
`guivo` via `.github/workflows/publish.yml`. Files deleted here are deleted there too. Run the workflow
manually to re-upload everything. It needs the repo secrets `CLOUDFLARE_API_TOKEN` (Pages Edit and R2
Edit) and `CLOUDFLARE_ACCOUNT_ID`.

## Requirements this hop has to keep meeting

- `Access-Control-Allow-Origin` on every response, or the browser blocks the load (Phaser fetches via
  XHR). GitHub Pages sends `*` by default.
- Stable urls that survive a re-upload.

Check with:

```bash
curl -s -D - -o /dev/null -H "Origin: https://guivo.io" \
  "https://ivorcosta.github.io/guivo-files/pieces_1-CyoN6RbB.png"
```

`.nojekyll` is present so Pages publishes the files as-is instead of running them through Jekyll.
