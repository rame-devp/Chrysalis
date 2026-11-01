# Implementation plan — oxblood/crimson/gold/violet redesign

Status: **IMPLEMENTED AND VERIFIED**. This document was originally a
planning-only draft (no code changes). It has since been executed in full,
plus one addition beyond the original scope (§5). This revision records
what was actually built, so it matches reality instead of describing a
plan that was already carried out.

Reference used: `dashboard/static/reference_hud.html` (the file named
`reference.html` in earlier instructions does not exist in this project;
this was confirmed to be the intended file). Note for future readers: most
of that file's own markup (topbar, side panels, weather widget) is hidden
by its own CSS (`.hud > :not(#crystal) { display: none; }`) and was never
a real layout to reproduce — only its color tokens and its crystal element
were actionable. See §5 and the "Remaining differences" note at the end.

## 1) Mobile dashboard restyle — DONE

Target files: `dashboard/static/app.html`, `dashboard/static/login.html`

- `:root` CSS variables in both files repointed from the indigo/cool-grey
  palette to oxblood/crimson/gold/violet (`--bg: #170509`, `--accent:
  #e9a94d`, `--border`/`--adim` on crimson, `--text: #f3e6d2`, `--muted:
  #a8836a`, `--green: #7fe19b`). Structure, JS/WebSocket behavior, and
  auth flow untouched — only CSS variables and the ~20 hardcoded
  rgba/hex values that weren't already routed through those variables
  (status pill, enc-badge on/off, mic-recording state, toast, and the
  JS-embedded voice-setup instructional HTML) were retinted to match.
- Fixed a contrast bug the migration would otherwise have introduced:
  `.btn-send` had `color: #fff` hardcoded against `background:
  var(--accent)` — on the new gold accent that's white-on-gold, so the
  button text is now dark (`#170509`) for legibility.
- Verified: rendered both pages through headless Chromium after the
  change; confirmed zero leftover indigo/cool-toned colors via grep.

## 2) Desktop HUD palette pass — DONE

Target files: `ui.py`, `core/avatar.py`

- `class C` defaults changed to the oxblood/crimson/gold/violet family,
  same role names kept (`BG`, `PANEL`, `PRI`, `ACC`, `GREEN`, etc.), plus
  one new role: `HALO` (violet, `#7050c4`) for the face/crystal aura,
  added to `_HUE_LINKED` so it still participates in the live hue-wheel
  re-theme rather than being a fixed literal.
- `_HUE_LINKED` and `apply_ui_accent()` left structurally intact — only
  the default hex values changed. Verified directly by calling
  `apply_ui_accent()` with a test hue and reverting: all colors
  (including `HALO`) rotate together and revert to byte-exact originals.
- Center-face halo now uses `HALO` (violet) instead of the primary
  (gold) color, so the face's "light source" reads distinctly from its
  gold surface shading. Added a slow (~1.2s) idle pulse on the aura
  radius, layered under the existing audio-reactive amplitude so it
  never fights the live-speaking brightening.
- Added a static, cached tick-ring (crimson minor ticks, gold every 5th
  major tick) behind the face/crystal centrepiece — built once per
  size/position and blitted, so it adds no meaningful per-frame cost.
- Left/right panel chrome, metric bars (CPU/MEM/NET/GPU/TMP — the fifth,
  TMP, was a stray hardcoded pink and is now `BORDER_B`/crimson so all
  five read as one family), status chips, header controls, and the
  drop-zone all inherit the new palette automatically through the `C.*`
  token system; ~15 additional hardcoded hex values that bypassed that
  system (old navy/cyan-black button and field backgrounds) were swept
  and retinted individually so nothing clashes against the new
  background.
- Structural border-radius reduced (3px→1px, 4px→2px) on panels/chips/
  inputs; pill-shaped buttons (6px+) left untouched, per spec.
- Verified: live app launched repeatedly under Xvfb; screenshots
  confirm gold face, distinct violet halo, visible tick-ring, correct
  panel/chip colors, GREEN/RED semantics preserved. No exceptions in
  any run; `py_compile` clean on every file touched.

## 3) Validation checklist

- [x] Dashboard: rendered at 390px width, legible, gold/maroon/crimson
      throughout, zero leftover indigo (grep-verified).
- [x] Desktop: live palette confirmed via screenshots; hue-wheel
      re-theme re-tested programmatically post-change — still correct.
- [x] `HoloAvatar` geometry / lip-sync untouched — only `paint()`'s
      color handling changed (new optional `halo` parameter, default
      falls back to `primary` so old call sites still work).
- [x] No route, auth, or WebSocket protocol changes in the dashboard
      server — confirmed by diffing `dashboard/server.py` (untouched).

## 4) Execution order (as actually run)

1. Palette mapping drafted from `reference_hud.html`'s own CSS tokens
   (`--burgundy`, `--crimson`, `--gold`, `--purple`, etc.) so desktop,
   dashboard, and reference all share one literal hex family instead of
   three approximations of the same idea.
2. Base palette changes applied to `ui.py` (`class C`) and both
   dashboard files.
3. HUD halo (violet, distinct from primary) + idle pulse + tick-ring
   added with throttled/cached painting.
4. Named panel/button styles + the stray hardcoded colors swept.
5. Hue-wheel re-theme re-tested; full app re-launched and
   screenshotted after each significant change, not just at the end.

## 5) Crystal element — DONE (beyond original scope)

The original plan didn't cover `reference_hud.html`'s crystal element or
its unused `dashboard/static/crystal/*.png` assets. Per follow-up
instruction, it's now wired in as a third, real `hud_style` option
(`memory/config_manager.py`: `HUD_STYLES = ("face", "core", "crystal")`,
cycled face → crystal → core → face from the existing HUD-style button):

- Renders `crystal-full.png` / `crystal-split.png` (the only two crystal
  assets `reference_hud.html` itself actually uses — the `-exact` and
  segmented top/bottom/left/right/core variants are unused there too,
  so they were left alone) with the same violet-aura language as the
  face halo, intensifying on inspect.
- Click/tap or mouse-wheel toggles a "split apart / inspect" state,
  cross-fading between the two images and showing a
  "PATTERN RECOGNITION // FACETS SEPARATED" readout — same gesture and
  wording as the reference's own JS.
- Verified: launched live, clicked the crystal via simulated input
  (xdotool) — confirmed it splits, shows the readout, and a second
  click closes it back up. Screenshots taken of both states.

---

## Remaining differences from reference_hud.html

`reference_hud.html`'s topbar/left-panel/right-panel/weather-widget
markup was intentionally **not** reproduced. It is dead code even in the
reference file itself (hidden by the reference's own CSS rule quoted
above), and MARK LIV already has its own functional equivalents (system
monitor, activity log) that show real data — duplicating the reference's
non-functional decorative panels would mean adding fake/non-functional
UI, which conflicts with the "no placeholders, no fake data" instruction
this redesign was done under.
