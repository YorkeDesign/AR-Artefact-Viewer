---
harvestry_item: 9fc5e4fd-eeb8-4590-9850-792c170e6294
captured: 2026-06-03
proposed: 2026-09-14
status: awaiting-approval   # awaiting-approval | approved | rejected | implemented
approved_by:
approved_on:
implemented_commit:
primary_source: https://x.com/boona11/status/2062053207770124676 (capture only, tool not independently verified)
authority: WEAK
informs_decision: none
---

# Document a GLB-decimation fallback step for oversized exports

**What it is.** A capture pointing at "GLB Shrink," a free drop-in-browser
tool that shrinks oversized `.glb` files (the Operator's tweet cites a
58MB to 869KB test). The tweet is the only source; there is no independent
writeup, so treat the tool's own claims as unverified marketing until tried.

**What it would change here.** `README.md`'s "Recommended File Size Targets"
section (around line 445) already states the target (2-6MB GLB, 2-4MB USDZ)
but the 9-step scan-to-GLB conversion workflow above it (around lines 430-443)
has no step for what to do when Blender's export lands above that target - it
jumps straight from "export as GLB" to "upload." This proposal adds one
fallback line naming a decimation option for that gap. Nothing else in the
repo touches file size (there's no build step or asset pipeline to hook into -
this is a static HTML + CDN project per `CLAUDE.md`).

**Why now.** "Hosting and CDN delivery of 3D assets" is a high-weight signal
interest in `PROJECT.yml`, and file size is the direct lever on jsDelivr load
time for AR hand-off on mobile. `open_decisions` in `PROJECT.yml` is empty, so
this doesn't answer a live decision - it closes a documentation gap the
project already flagged (a size target with no stated way to hit it when
Blender's default export misses).

## Implementation plan

1. In `README.md`, after the existing "Recommended File Size Targets" bullets
   (~line 448), add one sentence: if a converted GLB exceeds the 2-6MB target,
   try a browser-based decimator (name GLB Shrink as one option, unverified)
   or Blender's own Decimate modifier before re-exporting, then re-check with
   the existing gltf-viewer.donmccurdy.com step.
2. No code, no new dependency, no CDN change - documentation only.
3. Verification: read the updated section back and confirm it still matches
   the 9-step workflow's step numbering; no build or test exists for this
   static-HTML repo, so a re-read is the check.

**Effort:** 0.25 hours. **Risk:** none - additive doc sentence, easily
reverted; the tool itself is not installed or depended on, only named as an
option. **Cost or requirements:** none; GLB Shrink is free and web-based, and
this proposal does not ask Simon to sign up for or install anything.

## Decision

Simon sets `status:` above. `approved` unlocks implementation by a session
following the repo's normal verification and commit autonomy; `rejected`
needs a one-line reason so the vault records why. Until then nothing is
built.
