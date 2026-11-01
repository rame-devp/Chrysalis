Project: Mark-LIV-main — a PyQt6 desktop assistant. Target file: ui.py (plus
core/avatar.py, which paints the holographic face inside the HUD). This is a
native, hand-painted UI (QPainter/QWidget/stylesheets) — there is no HTML/CSS/
React here. Everything below must be expressed as Qt code: QPainter drawing,
QSS stylesheets, QFont, QColor.

Read ui.py fully before planning, specifically: the class C colour palette and
apply_ui_accent() (lines ~73–145), HudCanvas (the painted centrepiece,
~line 382), MetricBar (~920), LogWidget (~981), FileDropZone (~1111), and
MainWindow._build_header / _build_left_panel / _build_right_panel /
_build_footer (~3631–4520). Read core/avatar.py to see how HoloAvatar draws the
face inside HudCanvas — do not change its geometry or lip-sync logic, only how
it is lit/coloured if the new palette requires it.

Stay in PLANNING and write implementation_plan.md before touching code.

======================================================================
WHAT THIS ALREADY IS, AND WHAT'S CHANGING
======================================================================
The app already has a real cockpit HUD: fixed-width left rail (system
monitor bars, uptime/OS info, status chips), a centre column dominated by
HudCanvas (the painted face/core with a halo and audio-reactive scale), a
fixed-width right rail (activity log, file drop zone, command input,
interrupt/mute buttons), a 54px header (title, clock, settings gear), and a
22px footer. This structure is not changing. What changes is the palette,
the chrome style, and the shape of a few painted elements, to match a
different reference aesthetic described below. Nothing that reads real data
(CPU/MEM/GPU/NET/TMP bars, the log, the clock, the file drop zone, the mute/
interrupt buttons) gets removed or replaced with decoration — it gets restyled.

======================================================================
TARGET AESTHETIC (no image attached — build from this description)
======================================================================
The current theme is a cyan/orange "Iron Man reactor" look (class C: BG
#00060a, PRI #00d4ff cyan, ACC #ff6b00 orange, GREEN #00ff88). Replace it with
a deep oxblood/crimson-and-gold instrument panel with one violet light source
at the centre — the same family as a jet cockpit or reactor containment
display, not a flat sci-fi blue:

- Background: near-black with a maroon/wine cast, not pure black (#0e0409 →
  #260a14 range), rather than the current cold #00060a.
- Structural lines, borders, panel hairlines: thin crimson/red
  (#d8324f-ish), replacing the current cyan borders.
- Primary readable text and active values: warm gold/amber (#e9a94d-ish),
  replacing cyan as the "PRI" role. Keep a dimmer gold for secondary labels
  (mirrors the existing PRI_DIM role).
- The one saturated glow in the whole interface is violet, on the HudCanvas
  centrepiece only (the halo/glow currently drawn around the face) — not on
  panel chrome elsewhere. This mirrors "one light source, everything else is
  quiet instrument grey."
- Keep GREEN and RED as status colours (connected/active vs error/muted) —
  those semantics don't change, only their exact hex can shift slightly to
  sit well against the new background.
- Typography stays monospace throughout (the app already uses "Courier New"
  everywhere) — do not introduce a second typeface. Keep the existing small
  caps-style, letter-spaced short labels (e.g. "◈ SYS MONITOR") as they are;
  this pattern already matches the target aesthetic.
- Panel chrome: the current flat-fill rounded rectangles (border-radius 3–4px)
  should move toward sharper, more instrument-panel corners — reduce
  border-radius on structural panels (left rail, right rail, info boxes)
  toward 1–2px, keep it slightly higher only on pill-shaped buttons.

Restyle these specific painted/drawn elements in HudCanvas:
- The grid-dot background layer (_grid_cache): recolour from cyan-on-black to
  a very dim crimson-on-oxblood, low opacity, unchanged density/pattern.
  (Locate the drawing code near `_grid_cache`/`_grid_key`.)
- The halo/glow around the face (`_halo`, `_tgt_halo`, `_base_halo`): recolour
  to violet, and add a slow independent pulse to its radius (roughly 1 cycle
  per 1.1–1.3 seconds, a short-then-soft double pulse if practical in
  QPainter — mirrors a heartbeat rhythm) layered on top of the existing
  audio-reactive scale (`_scale`/`_tgt_scale`/`_live_amp`), not replacing it.
  Audio reactivity remains the dominant driver while JARVIS is speaking; the
  idle pulse is only visible at rest.
- Any ring/tick-mark chrome already drawn around the face (if present) should
  pick up the crimson/gold ring language instead of cyan; if no ring
  currently exists around HudCanvas's centrepiece, add one: a thin outer
  circle of fine tick marks (crimson, a longer gold tick every 5th–10th) as a
  static overlay behind the face, non-interactive, respecting the existing
  paint-throttle pattern (`_paint_tick`) rather than adding new per-frame cost.

Left rail ("SYS MONITOR"):
- MetricBar fills: keep per-metric colour coding (CPU/MEM/NET/GPU/TMP each
  already have distinct colours via class C) but pull all five from the new
  gold/crimson/violet/green family rather than the current cyan/orange mix,
  so they read as one instrument set.
- The three status chips at the bottom ("AI CORE ACTIVE", "SEC CLEARED",
  "PROTOCOL …") keep their pill shape but restyle border/text colour to gold
  on dark, matching the header's badge style.

Right rail ("ACTIVITY LOG" / "FILE UPLOAD" / "COMMAND INPUT"):
- LogWidget text colour and its border move to the new palette; keep its
  existing autoscroll and append behaviour untouched.
- FileDropZone's border/icon colours restyle to gold-on-crimson-dash rather
  than cyan-on-dash; its drag-over highlight state keeps using GREEN.
- INTERRUPT button keeps its red/alert styling as-is (MUTED_C role) — that is
  already correct for the new palette and should not be touched.
- MUTE button restyles its active/inactive states to the new palette,
  preserving the existing text ("MICROPHONE ACTIVE" / muted equivalent).

Header: clock and title move from cyan to gold; the settings-gear button's
hover/checked states move from cyan to gold, keeping its existing checkable
behaviour untouched.

======================================================================
HOW TO DO THIS WITHOUT BREAKING LIVE THEMING
======================================================================
This app already has a live re-theme system: class C holds the palette, the
_HUE_LINKED tuple marks which keys shift together when the user picks a
custom accent colour from the hue wheel, and apply_ui_accent() /
retheme_all_widgets() apply changes live with no restart. Do this:

1. Change the *default* values in class C to the new oxblood/crimson/gold/
   violet palette. Do not remove or rename any key other keeping the same
   role names (BG, PANEL, PRI, ACC, etc.) so nothing elsewhere in the 5000+
   line file that references `C.PRI` etc. breaks.
2. Leave `_HUE_LINKED` and `apply_ui_accent()` structurally alone — the user's
   existing "recolour from a hue wheel" feature must keep working against
   the new defaults exactly as it does today against the old ones. Do not
   hardcode the new hex values anywhere outside class C; every colour
   reference in the file should still go through `C.<NAME>` or `qcol()` so
   the live re-theme continues to reach every widget.
3. Where you add anything genuinely new (the tick-ring around the face, if
   it doesn't already exist), add its colour as a new attribute on class C so
   it also participates in `_HUE_LINKED` and re-themes live, rather than a
   literal hex baked into the paint call.

======================================================================
WHAT MUST NOT CHANGE
======================================================================
- HoloAvatar's facial geometry, lip-sync/viseme logic, blink/gaze/glance
  behaviour: untouched. Only its lighting/tint, if any, may shift to sit
  correctly against the new background — check core/avatar.py for whether it
  already reads from class C or has its own hardcoded colours before
  deciding whether it needs a change at all.
- The panel layout (left rail / centre / right rail / header / footer), the
  QSplitter behaviour, the quick drawer, all overlays (SetupOverlay,
  CustomizeOverlay, PluginManagerOverlay, MemoryOverlay, ConfirmBanner,
  AudioDeviceOverlay, ClipboardPanel, RemoteKeyOverlay): none of these move,
  resize, or change behaviour. Only class C's palette and the specific
  painted elements named above change appearance.
- Every widget that displays live data (CPU/MEM/GPU/NET/TMP, uptime, process
  count, OS name, the clock, the activity log, file info) keeps showing real
  values from the same source it does today — no placeholder text, no
  decorative-only readouts introduced.
- The existing paint-throttle / repaint-rate pattern in HudCanvas
  (`_paint_tick`, idle frames dropping to ~20Hz) must not regress — new
  drawing (the tick-ring, the idle pulse) has to respect it, not add an
  unthrottled per-frame cost.
- CustomizeOverlay's hue wheel and live re-theme must still work after the
  change — verify this explicitly (see below).

======================================================================
PROCESS
======================================================================
1. PLANNING: list every class-C key you're changing (old hex → new hex),
   confirm HoloAvatar needs no geometry change, and confirm the plan for the
   tick-ring and idle pulse respects the existing paint-throttle pattern.
   Get my approval before writing code.
2. EXECUTION: change class C first, run the app, and visually confirm the
   whole UI re-themes from that one edit before touching HudCanvas's custom
   paint code. Then add the halo recolour + idle pulse + tick-ring. Then the
   per-panel QSS touch-ups (left rail chips, drop zone, buttons).
3. VERIFICATION: launch the app (`python main.py` or the project's documented
   entry point) and, in walkthrough.md, describe/record:
   - The full window at idle: gold/crimson/violet palette applied everywhere,
     face visible with its new halo colour, idle pulse visible on the halo.
   - JARVIS speaking (or a simulated speaking state if a live session isn't
     available): confirm audio-reactive scale still dominates over the idle
     pulse, not fighting it.
   - Open the hue wheel in CustomizeOverlay, pick a different accent colour,
     confirm the whole UI re-themes live and reverts cleanly back to the new
     default on reset — this is the regression test that matters most here.
   - Left rail: all five MetricBars showing live, moving values in the new
     palette.
   - Drop a file onto FileDropZone, confirm its states (idle/drag-over/
     loaded) all read correctly in the new colours.
   - Resize the window down to the app's documented minimum size
     (`_MIN_W`/`_MIN_H`) and confirm nothing clips or goes unreadable.

Do not commit. Flag anything in this prompt that conflicts with what you find
in ui.py or core/avatar.py before you start building.
