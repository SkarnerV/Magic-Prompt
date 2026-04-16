# Pitfall: Release Pipeline — "No artifacts were found"

**Source:** `token-tracker/docs/release-pipeline-debugging.md`, `openman/docs/release-process.md`

## Symptom

GitHub Actions release workflow completes the build but fails with `No artifacts were found.` during artifact upload.

## Root Cause

In Tauri v2, the bundler is **opt-in** via `--bundles` CLI arguments. Tauri v1 ran the bundler automatically when `bundle.active: true` was set in config. In v2, that field was removed and `tauri build` only produces the binary executable.

The CI logs show:
```
Finished `release` profile [optimized] target(s) in 2m 10s
Built application at: .../release/token-tracker
Looking for artifacts in: .../bundle/dmg/...
[error] No artifacts were found.
```
Notice: **No "Bundling" output between build and artifact lookup** — the bundler never ran.

### Why It Was Hard to Diagnose

1. **Misleading similarity to v0.1.0** — The working v0.1.0 used Tauri v1, while current code was Tauri v2. Diffing against v0.1.0 config didn't reveal the behavioral change.
2. **Ambiguous error message** — "No artifacts were found" suggests many causes but doesn't indicate bundling was never attempted.
3. **CI environment obscured behavior** — Without local testing, the missing "Bundling" phase output wasn't noticed.

## Fix

Add explicit `--bundles` arguments per platform:

```yaml
matrix:
  include:
    - os: ubuntu-22.04
      target: x86_64-unknown-linux-gnu
      bundles: --bundles deb --bundles appimage
    - os: macos-latest
      target: x86_64-apple-darwin
      bundles: --bundles dmg --bundles app
    - os: macos-latest
      target: aarch64-apple-darwin
      bundles: --bundles dmg --bundles app
    - os: windows-latest
      target: x86_64-pc-windows-msvc
      bundles: --bundles msi --bundles nsis
args: --target ${{ matrix.target }} ${{ matrix.bundles }}
```

Windows also needs NSIS/Wix installed:
```yaml
- name: Install Windows dependencies
  if: matrix.os == 'windows-latest'
  run: |
    choco install nsis
    choco install wixtoolset
```

## How to Avoid

- Run `pnpm tauri build --help` — `--bundles` being optional is the tell
- Test locally before pushing to CI
- Check for framework behavioral changes when diffing against a working reference (check versions first)
- Look for **missing output**, not just errors — the absence of "Bundling" phase between "Built" and "Looking for artifacts" was the key evidence
