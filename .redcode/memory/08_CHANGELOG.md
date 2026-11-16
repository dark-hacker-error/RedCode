# 08 — Changelog

Har kaam ki entry yahan: date, kya hua, affected files.

## 2026-11-14 — Initial audit
- Poore repo ka read-only audit kiya (structure, tech stack, entry points, core modules,
  data flow, config/storage, branding, CI/CD, dependencies, risks).
- Naya folder banaya: `.redcode/memory/` (00 se 08 tak files).
- Root `README.md` ko RedCode branding ke saath rewrite kiya (purana content git/history
  me safe; source code change nahi).
- `REDCODE_AGENTS.md` banaya (existing `AGENTS.md` overwrite nahi kiya).
- Source code me **koi change nahi** kiya. License as-is.
- GitHub: private repo `Vempire-X/RedCode` create kiya.
  - Baseline source `main` par commit `f523a18` (original upstream README bhi baseline
    me preserved).
  - Docs commits `a45c2f6` + `4f6b903` branch `redcode/initial-audit` par.
  - PR #1: https://github.com/Vempire-X/RedCode/pull/1 (merge nahi kiya).
  - CI note: inherited `check-standards` workflow `dev` branch expect karta hai, isliye
    PR par fail hota hai. Source/workflow change nahi kiya.

### Audit facts captured
- Version: opencode 2.3.0 codebase; Bun 1.4.4; Effect 4.1.0; TypeScript 7.1 beta.
- Repo: 35+ packages, 1,721 test files, 60 e2e spec files.
- Audit time par folder git repository nahi tha.
- `node_modules` maujood nahi; install/build/test execute nahi hue.

## 2026-11-15 — Full rebrand + npm/installer release path
- Poora source-level rebrand `opencode` → **RedCode** (names/branding only; koi feature remove nahi, logic same).
- Product: `redcode` command (`redcode2` legacy alias), npm package **`redcode-ai`** + platform packages
  `redcode-ai-<os>-<arch>[-baseline][-musl]`, version **0.1.0**.
- Env prefix `OPENCODE_`→`REDCODE_`; config `~/.config/redcode/`, project `.redcode/`, `redcode.json(c)`,
  DB `redcode.db`, XDG data/cache/state `redcode`.
- Desktop: app id `ai.redcode.desktop[.dev/.beta]`, scheme `redcode://`, settings `redcode.settings`.
- Repo ka apna config dir `.opencode/` → `.redcode/` (agents/commands/skills/plugins/themes/tool + `redcode.jsonc`).
- Nix: `nix/opencode.nix` → `nix/redcode.nix`, flake attrs `redcode`/`redcode-desktop`.
- CI: blacksmith runner labels → GitHub-hosted (`ubuntu-24.04`, `macos-15`, `windows-2025`, `ubuntu-24.04-arm`).
- Naya `.github/workflows/redcode-release.yml`: `v*` tag par sab platforms build, npm publish (`NPM_TOKEN`),
  GitHub Release. Purana `release.yml` disable (upstream-only infra).
- `bun.lock` ke workspace package names update kiye; `@opencode/pty*` entries as-is.
- `LICENSE` aur upstream copyright as-is.

### Verification status
- Local `bun install`/build/test **abhi bhi possible nahi** (EACCES). Sab kuch **GitHub Actions par unverified**.
- `install` script ab `registry.npmjs.org/redcode-ai-*` se install karta hai aur `~/.redcode/bin` me daalta hai.
- CI branch refs `dev` -> `main` kiye (check*, triage-pr-standards, triage-compliance-close, bot-nix-hashes)
  taaki PR #2 par CI chale aur triage bots crash na karein. Deploy/nightly workflows jaan-boojh kar waise hi
  chhode (wo `dev`/`production` branch par hi trigger hote hain, is repo me wo branch nahi hai).
- PR #2 open/updated: https://github.com/Vempire-X/RedCode/pull/2 (`redcode/rebrand-and-installer` -> `main`).
  Merge **nahi** kiya (user approval baaki).
- PR #2 **merge** ho gaya (squash) -> `main` = `e224c13 chore: full rebrand to RedCode, npm package and installer`.
- PR #1 close kar diya (uske docs PR #2 ke through main me aa gaye).
- `main` par tag: abhi koi nahi.
- GitHub repo secret `NPM_TOKEN` set kiya (Actions secrets API + libsodium sealed box; value commit/disk par save nahi ki).
- Next: `v0.1.0` tag push -> `.github/workflows/redcode-release.yml` build + npm publish + GitHub Release.


### Fix rounds (2026-11-15/16, GitHub Actions par)
- `a5bf97c8` composite `setup-bun` (jo `--frozen-lockfile` hard-code karta tha) hataya; stale `bun.lock`
  workspace names (`redcode-aient` -> `@redcode/client`) theek kiye.
- `c4e464ac` release ko `plan` + 12-target `build` matrix + `pack` me toda (single job `ENOSPC` se fail hota tha).
- `470eb199` `.opencode` prefix replacement se toote identifiers wapas kiye: `sdk?.opencodeai`,
  `target.opencodePtyAsset`, `state?.opencodeChecks`, `opencodeLight`/`opencodeDark`, winget `anomaly.opencode_`.
- `7b5a8d21` `services/site` ka `window.opencode` console API wapas (site.tsx, console-stub.ts).
- `2f71b7bc` + `20fad234` 29 symlinks (jo Android checkout me regular files ban gaye the) git mode `120000` par wapas.
- `ba69cd05` round-2 identifier fixes: `...opencode` spread (voice.test), `?.opencode` (keymap test),
  `.opencode-pierre-file-tree` (e2e spec), `console\.opencode\.ai` (escaped dot ne protection bypass kiya).
- `b2e61df9` CLI packaging paths `redcode-ai-*` par: `remote-cli.ts`, `bootstrap.test.ts`, `wsl/local.ts`,
  `service-smoke.ts` (target + binary `redcode`).
- `6509fa8e` `patches/@silvia-odwyer%2Fphoton-node@0.3.4.patch` ab `__REDCODE_PHOTON_WASM_PATH` padhta hai
  (warna Node build `dist-node/photon_rs_bg.wasm` ENOENT); site ka `architectPatterns` id `redcode-grid`.
- Typecheck (`check.yml`) `ba69cd05` par **green**.

### Release blocker (root cause: billing, not code)
- Run `43684847437` (tag `v0.1.0`, sha `b2e61df9`): `plan` + **12/12 platform builds success**, lekin `pack` job
  start hi nahi hua — annotation: *"The job was not started because recent account payments have failed or your
  spending limit needs to be increased."* `Vempire-X` account par Actions billing fail hai.
- Isliye project `dark-hacker-error/RedCode` (private) par shift kiya (transfer API 422 de raha tha, isliye fresh
  repo + full push). Wahan `NPM_TOKEN` set kiya aur tag `v0.1.0` push kiya -> release dobara queue hua.
- `legal-search.json` fixture stale hai (rebrand ne text 1 char chhota kiya, truncation shift ho gayi):
  `services/site/test/legal-search.test.ts` fail. Regenerate ke liye temporary `regen-legal.yml` workflow.
- **npm par kuch bhi publish nahi hua** jab tak release run complete na ho.

### Release run 2 (account dark-hacker-error, 12/12 builds, pack fail)
- `pack` fail: `No binary packages found for redcode-ai`. Cause: `actions/upload-artifact` me
  `path: packages/cli/dist/redcode-ai-<target>` dene se directory ka naam drop ho jata hai (artifact root =
  directory ke contents), aur 12 artifacts merge hone par `redcode-ai-*` dirs banti hi nahi.
  Fix: `path: packages/cli/dist` (poora dist) -> `redcode-ai-<target>/...` structure bachta hai.
- `packages/script/test/highlights.test.ts` fail: test RedCode expect karta tha, `CHANGELOG.md` upstream
  "OpenCode" wording rakhta hai (rebrand ne CHANGELOG ko chhua nahi). Fix: expectations upstream wording par.
- Yaad rahe: repo **public** hai ab, isliye Actions minutes/concurrency free hain. Naye account par jobs
  pehle 1-2 concurrency par serialise ho rahe the; release ke waqt check/check-test runs cancel karna
  build ko tez karta hai.

### Release run 3 + 4 (publish par npm 429)
- Run 3 (`43810106883`): 12/12 builds + pack **success**, phir `Publish to npm` par
  `npm error code E429  429 Too Many Requests - PUT https://registry.npmjs.org/redcode-ai-windows-x64
  - Could not publish, as user undefined: rate limited exceeded`. 10 packages publish ho gaye, 11th par throttle.
- `redcode-ai-*` par `dist-tags.latest` check karo, `versions` me `0.0.0-stage` **npm ka staged-publish
  placeholder** hai (stub tarball); asli version `dist-tags.latest` = `0.1.0` hota hai. Panic mat karo.
- Run 4 (`43812494740`, retry-with-backoff ke saath): 12/12 builds success, publish ne ~21 min retry kiya
  (60/120/180/240/300/360s) lekin windows-x64 par 429 bana raha -> fail. Matlab block window >20 min.
- Structural fix (commit `1228dd3b`): `pack` job ab `npm-tarballs` + `archives` artifacts upload karta hai,
  aur publishing alag `publish` job me hai (`needs: [plan, pack]`, timeout 150 min) jo wahi tarballs
  download karke 7 attempts x 180s backoff ke saath publish karta hai, har success ke baad 20s gap.
  Fayda: publish fail hone par `gh run rerun --failed` sirf publish job rerun karega, 12 builds dobara nahi.
- GitHub Release ab apna alag `release` job hai (`needs: [plan, pack]`), publish par depend nahi karta —
  npm throttle hone par bhi archives/tarballs release me mil jate hain. `gh release create ... ||
  gh release upload --clobber`.
- Publish loop already-published versions ko `npm view <name>@<version>` se skip karta hai, isliye retry
  sirf bache hue packages publish karta hai (platform packages pehle, wrapper `redcode-ai` sabse aakhir me).
