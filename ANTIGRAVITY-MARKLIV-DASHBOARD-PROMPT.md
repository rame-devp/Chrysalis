Project: Mark-LIV-main — target file: dashboard/static/app.html (a single
self-contained HTML/CSS/JS file served by dashboard/server.py over a paired
WebSocket + REST connection). Also check dashboard/static/login.html, which
should get the same palette treatment for visual consistency, even though its
layout doesn't need to change. Do not touch dashboard/server.py's routes,
auth, or WebSocket protocol — this is a visual-only pass over static assets.

Read app.html fully before planning: the <style> block (~lines 1–135) defines
every token this page uses; the body is just `<header>` / `<main id="feed">` /
`<footer>`, driven entirely by the inline `<script>` at the bottom (WebSocket
message handling, AES encryption via crypto-js, the message feed, the
composer). This is a phone-first single page — assume a narrow viewport
(360–430px wide) as the primary target, not desktop.

Stay in PLANNING and write implementation_plan.md before touching code.

======================================================================
WHAT THIS ALREADY IS, AND WHAT'S CHANGING
======================================================================
This is a minimal chat client: a header with a logo, a live/sleeping status
pill, an encryption badge, and the paired IP:port; a scrolling message feed
with two message styles (JARVIS vs system); and a footer composer with a text
input, a file-attach button, a mic button, a WAKE button, and SEND. All of
that stays — this is a restyle, not a rebuild. The current look is a generic
dark indigo chat app (`--bg: #07090f`, `--accent: #6366f1`). Replace the
token set and the chrome so it reads as the same instrument-panel family as
the rest of the project, scaled down to fit a phone screen without turning
into a cluttered miniature of a desktop HUD.

======================================================================
TARGET AESTHETIC (no image attached — build from this description)
======================================================================
Same family as the rest of this project's redesign: deep oxblood/near-black
background with a violet undertone, crimson hairlines for structure, gold for
active/readable values, violet reserved for the one "alive" indicator on the
page. Monospace for anything numeric or system-status (the IP:port badge,
timestamps if you add any); the existing sans-serif stays for message body
text, since that's prose someone is reading, not instrument text — don't make
chat bubbles monospace.

Token replacements in :root (keep the same variable names so nothing else in
the file needs touching — the file already references `var(--bg)`,
`var(--accent)`, etc. throughout):
  --bg:      near-black with maroon cast (#0e0409 → #170509 range)
  --surface: very low-opacity warm white, unchanged role, slightly warmer tint
  --border:  very low-opacity gold/crimson rather than plain white
  --accent:  gold (#e9a94d-ish) replacing the current indigo #6366f1 —
             this drives the logo's "J" in JARVIS, the active-pill state,
             and the send button
  --adim:    the accent at low opacity, same role as today (JARVIS message
             bubble background)
  --text:    warm off-white/ink, not the current cool blue-grey
  --muted:   muted rose/ash, not the current cool grey
  --green / --gdim: keep as-is — these already correctly mean "connected/
             active" and that semantic doesn't change

Specific elements:
- `.logo em` (the accent-coloured "J"): gold, with a very subtle text-glow
  rather than flat colour — this is the one place on the page allowed a soft
  glow, echoing the crystal/halo motif from the rest of the project.
- `.pill.on` (connected/active status): keep green, but change its border
  and background tint to sit correctly against the new maroon background
  rather than the current cool-toned green tint.
- `.enc-badge`: keep its green/red on/off semantics, retint to the new
  palette's green/crimson.
- `.url` badge (IP:port): switch to the gold/crimson bordered pill style used
  elsewhere in this project's HUD language — thin border, monospace, small
  caps letter-spacing on any label text around it if you add one.
- Message bubbles (`.msg-j` for JARVIS, and whatever class the system/user
  style uses): JARVIS bubbles pick up a subtle gold-on-oxblood tint instead
  of the current indigo tint; keep the existing rounded-corner amount and
  the `pop` entry animation as-is — no new bubble animation needed.
- Footer composer: input field border/background move to the new palette;
  the SEND button becomes a solid gold pill (this is the one clearly
  "primary action" button on the page and should read as such); WAKE and the
  mic/attach icon buttons stay as secondary/ghost style in the muted/crimson
  tones, not competing with SEND for attention.
- `@keyframes blink` (the connection dot) and `@keyframes pop` (message
  entry) keep their current timing and mechanics — only colours change.

Do not introduce anything decorative that doesn't already exist on this page
functionally — no fake weather, no extra instrument rings, no additional
panels. The phone dashboard's job is to be a fast, legible chat client; the
"HUD" feeling should come entirely from the colour system, the monospace
status chrome, and the logo glow, not from added visual complexity that would
crowd a 360px-wide screen.

======================================================================
WHAT MUST NOT CHANGE
======================================================================
- The WebSocket message handling, the AES-256 encryption flow (crypto-js
  usage, `_encrypt`/`_initCrypto`), the auth token flow (`sessionStorage`,
  redirect to `/login` when missing): none of this is touched. This is a
  `<style>`-block and class-attribute pass only.
- `main.js`-equivalent logic embedded in the `<script>` tag: functions like
  `setStatus`, `append`, `sys`, `toast`, file handling — behaviour unchanged,
  only any inline style strings they set (if any) get retinted to match.
- Layout structure (`header` / `main#feed` / `footer`, fixed positioning,
  flex behaviour): unchanged. This is not gaining new regions.
- Must still render correctly from 360px up through desktop widths — this
  page is reached from a phone via QR-code pairing as the primary path, so
  the phone breakpoint is the one that must not regress.
- `login.html` gets the same token/colour treatment for consistency but its
  form layout and auth logic are untouched.

======================================================================
PROCESS
======================================================================
1. PLANNING: list the exact token value changes (old hex → new hex) for
   every `:root` variable in app.html, and confirm login.html's structure
   needs no layout change, only token reuse. Get my approval before writing
   code.
2. EXECUTION: change the `:root` block first and reload — since every rule
   in the file already references the variables, confirm most of the page
   re-colours from that one edit before touching any individual selector by
   hand. Then do the few per-element touch-ups listed above (logo glow, pill
   borders, bubble tint, SEND button).
3. VERIFICATION: run the dashboard server, open app.html in a browser sized
   to 390×844 (a standard phone viewport) and again at a normal desktop
   width, and in walkthrough.md capture:
   - The page at rest with the status pill in both "Connecting" and
     "Active"/on states (trigger via the dev tools or a real paired session
     if available).
   - A JARVIS message and a system message both visible in the feed, bubble
     tint and legibility confirmed against the new background.
   - The encryption badge in both on/off states.
   - The composer with something typed, SEND clearly the most prominent
     control, WAKE and mic/attach clearly secondary.
   - login.html alongside app.html, confirming they read as the same visual
     family.
   - The 360px-wide view specifically, confirming nothing clips or wraps
     badly.

Do not commit. Flag anything in this prompt that conflicts with what you find
in app.html, login.html, or server.py before you start building.
