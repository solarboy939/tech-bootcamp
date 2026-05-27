# TECH Bootcamp — From Problem to Partnership

Interactive workshop deck for **Handelsblatt TECH · Heilbronn · 01.06.2026 · 16:15–17:05**.

Speakers: Patrick Burkert (Campus Founders), Henrike Luszick (Bridgemaker), Florian Peter (Bridgemaker).

## What's in here

- `index.html` — the full 11-slide deck. Self-contained; can be opened directly with a browser as well.
- `assets/` — logos and speaker photos.
- `vercel.json` — rewrite so the root URL serves the deck.

## How the multiplayer slides work

- **Slide 9 — Problem Canvas.** Each participant types their problem on their own phone or laptop. New problems appear live on the wall for everyone in the room.
- **Slide 10 — Decision Matrix.** Drag problems into the 2×2 (Generic ↔ Proprietary × Quick Win ↔ Strategic Bet). Placements sync live for everyone.

State is stored in **Supabase** in a table called `tech_bootcamp_problems`, with Realtime push so updates appear instantly (no polling lag). The Supabase project URL and anon public key are embedded in the page — this is fine and intended: the anon key is designed to be public, and RLS policies on the table govern what it can actually do.

When the page is opened via `file://` (no network), the deck falls back to per-browser `localStorage` so it still works offline for a solo rehearsal.

A small badge at the top of the screen tells you which mode is active:
- Green **Live · synced across devices** = Supabase reachable.
- Orange **Local mode · this browser only** = fallback active.

## Supabase schema (already deployed)

```sql
create table if not exists public.tech_bootcamp_problems (
  id          text primary key,
  who         text not null default 'Anonymous',
  title       text not null,
  affected    text,
  tried       text,
  placement   text not null default 'pool'
              check (placement in ('pool','q1','q2','q3','q4')),
  ts          bigint not null default (extract(epoch from now()) * 1000)::bigint,
  created_at  timestamptz not null default now()
);

alter table public.tech_bootcamp_problems enable row level security;
create policy "anon read"   on public.tech_bootcamp_problems for select using (true);
create policy "anon insert" on public.tech_bootcamp_problems for insert with check (true);
create policy "anon update" on public.tech_bootcamp_problems for update using (true);
create policy "anon delete" on public.tech_bootcamp_problems for delete using (true);

alter publication supabase_realtime add table public.tech_bootcamp_problems;
```

## Deploy

The repo is already connected to a Vercel project (`tech-bootcamp`). Push to `main` and Vercel will redeploy automatically. No build step, no env vars needed (all Supabase config is in the HTML).

```bash
git add .
git commit -m "your message"
git push
```

## Keyboard shortcuts (when presenting)

- `←` / `→` — navigate slides
- `O` — slide overview
- `F` — fullscreen
- `Cmd/Ctrl + Enter` (in the canvas form) — submit problem

## Resetting the wall before / during a workshop

In the Supabase dashboard SQL Editor:

```sql
truncate public.tech_bootcamp_problems;
```

Or directly from the running page (host laptop): on Slide 9 click **Clear wall**.
