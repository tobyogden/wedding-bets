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

## Deploy (Netlify)

1. Push this folder's contents to the root of the GitHub repo
   `tobyogden/wedding-bets`.
2. In Netlify: the site is already connected to that repo — publish directory
   is the repo root, build command is empty/none. Every push to `main`
   auto-deploys.
3. That's it — guests just open the site URL.

## How to play

- **Host:** Create a room (wedding title + your name). You get a big 6-letter
  join code and an invite link (`?room=CODE`). As host you can also settle
  moments and add custom props live.
- **Guests:** Open the link (or enter the code) + your name. Pick Yes/No and a
  stake on each moment before it's settled. Everyone starts at 100 points;
  win your stake when you call it right, lose it when you don't.
- Identity is kept in the browser (localStorage), so a refresh drops you
  right back into your seat.
