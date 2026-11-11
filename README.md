# Wedding Bets 💍🎉

A live, multiplayer wedding-day betting game. A host creates a room for a
wedding, guests join with a 6-letter code, everyone stakes points (5 / 10 / 25)
on whether classic wedding moments will happen, and the host settles each
moment as the day unfolds. Points update on every phone in near-real-time.

## What's in here

- `index.html` — the entire app (markup, styles, and logic in one file).
  It uses the Supabase JS client from CDN; there is no build step.

## Backend

Supabase project **"wedding bets"** (ref `etojeszmiwfgncokrsay`):

- URL: `https://etojeszmiwfgncokrsay.supabase.co`
- The client config (project URL + anon public key) is embedded at the top of
  the script in `index.html`. The anon key is a public client key — access is
  governed by the database's Row Level Security policies (open anon policies,
  room-code-gated at the app level).
- Tables: `rooms`, `players`, `props`, `picks` (public schema), all published
  to Supabase Realtime so picks, settlements, and standings sync live.
- `props.locks_at` (nullable timestamptz) carries a per-prop betting deadline.
  The `picks` table's INSERT/UPDATE policies reject writes for a prop whose
  `locks_at` is in the past — that's the real enforcement; the app mirrors the
  lock state in its UI and handles a rejection gracefully.
- `rooms.ref_code` (nullable text) is the room's 8-character referee code, and
  `players.is_referee` flags players who joined with it. Both were added after
  the original schema; the app degrades gracefully if they're absent (room
  creation falls back to no ref code, and the host panel offers a "Generate"
  button for rooms that don't have one yet).
- `rooms.finished` (boolean, default false) is the end-game flag. The host
  sets it when the game ends; the room view and the big-screen display both
  read it live over Realtime. Until the column exists the app simply treats
  every room as in progress.

## Deploy (Netlify)

1. Push this folder's contents to the root of the GitHub repo
   `tobyogden/wedding-bets`.
2. In Netlify: the site is already connected to that repo — publish directory
   is the repo root, build command is empty/none. Every push to `main`
   auto-deploys.
3. That's it — guests just open the site URL.

## How to play

- **Host:** Create a room (wedding title + your name). You get a big 6-letter
  join code, an invite link (`?room=CODE`), and a QR code of that link right in
  the room header. As host you can also settle moments and add custom props live.
- **QR joining:** Everyone in the room (not just the host) sees the QR card —
  anyone can recruit more players. Tap the QR to open a fullscreen scan view
  (giant QR, wedding title, and the room code in big letters) that the host can
  hold up across the room for guests to scan. Scanning opens the invite link,
  which pre-fills the room code on the join form, so a guest only types their
  name. If the QR library (loaded from CDN) ever fails, the code and the
  copy-invite-link button still work on their own.
- **Guests:** Open the link (or enter the code) + your name. Pick Yes/No and a
  stake on each moment before it's settled. Everyone starts at 100 points;
  win your stake when you call it right, lose it when you don't.
- **Betting deadlines (host):** The "⏰ Betting deadlines" panel sets a lock
  time per category (Getting Ready, First Look, Ceremony, Portraits,
  Reception, plus any custom categories). Setting one writes `locks_at` to
  every prop in that category. Props show "🔒 Locks 3:30 PM" while betting is
  open; once the time passes the card flips to "🔒 Betting closed" (existing
  picks stay visible, controls disappear) — the app re-checks every 30
  seconds, so locks engage on their own. Hosts can still settle locked
  moments, and can change or clear a deadline anytime. A custom prop added to
  a category inherits that category's deadline.
- **Game log & stats (host):** A collapsible "📊 Game log & stats" section
  shows the room totals (picks placed, points wagered, moments settled,
  showdowns), a per-player table (picks, W–L, win %, net points, current
  points), and a full bet log — every pick, newest first, with side, stake,
  time placed, and its result once settled.
- **Referee access (host):** Every room gets a second secret besides the
  guest code: an 8-character **referee code**, shown only in the host-only
  "🏅 Referee access" panel (with a copyable **referee link**,
  `?room=CODE&ref=REFCODE`). Hand it to a designated game-runner — best man,
  maid of honor, planner — and they join with full host powers (settle
  moments, set deadlines, see the log & stats, regenerate the code) without
  needing the room creator's phone. Referees can also join manually: the join
  form has a "Joining as the referee?" expander with its own ref-code field.
  A referee link flips the join form into a visible "🏅 Joining as referee"
  mode. Validation is exact-match against the room's stored `ref_code` — a
  wrong code gets a clear error and no join. Referee players carry
  `is_host=true` and `is_referee=true` and show a 🏅 REFEREE badge in the
  standings and bet log (the creator shows 👑 HOST). **Regenerate code**
  (behind an in-app confirm) writes a fresh code that invalidates the old one
  for new joins immediately; referees already in the room keep their seats.
  Security note: the UI never renders the ref code for non-hosts, but with
  the anon key and open RLS every client fetches the full room row, so a
  determined guest could read it from the raw API response. That's accepted
  by design — this is an honor-system party game, and regenerating is the
  remedy if a code leaks.
- **Ending the game (host):** The "🏁 The finish line" card ends the game
  (behind an in-app confirm): it sets `rooms.finished`, which crowns the
  winner in a banner at the top of the room, renames the standings to "Final
  standings," and freezes everything — pick controls disappear for everyone
  behind a "🏁 Game over" badge, and settle/unsettle buttons stay disabled
  until the host taps **↩ Reopen game** in the banner. Ending is fully
  reversible.
- **Big-screen / DJ mode:** Open `?screen=CODE` (e.g.
  `https://weddingbets.netlify.app/?screen=37PF2N`) on a TV, projector, or
  the DJ's laptop. It's a pure read-only display — no login, no session, no
  controls — that stays live over Realtime: wedding title, top-8 standings
  with the leader marked "IN THE LEAD 👑", a "X / Y moments called"
  progress line, the latest settled moments with their biggest winners, and
  the join QR in the corner so late guests can scan in from the screen. The
  host gets a **📺 Copy big-screen link** button in the room header. When
  the host ends the game, the screen flips to a winner spotlight: the
  champion's name in huge glowing type, their final points, and the final
  top 5. A bad code gets a friendly "Room not found" screen.
- Identity is kept in the browser (localStorage), so a refresh drops you
  right back into your seat. **Leave room** fully clears that saved seat (via
  an in-app confirm — native `confirm()` dialogs can be auto-dismissed or
  suppressed in some browsers/webviews, which used to silently swallow the
  leave action). And if an invite link (`?room=CODE`) names a different room
  than the saved seat, the invite wins: you land on the join form with the new
  code prefilled instead of being auto-rejoined to the old room.
