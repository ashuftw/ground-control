# Ground Control

A lightweight, private productivity dashboard for managing university tasks, deadlines, and exams — with passwordless sign-in and cloud sync across devices.

**Live:** https://gc.ashuftw.com

It's a single self-contained `index.html` file backed by [Supabase](https://supabase.com) (free tier). Each user signs in with an email **magic link** and gets their own private data, isolated by row-level security. No passwords, no per-user setup — open the page, enter your email, click the link, done.

---

## Features

- **📝 Task boards** — manage tasks across labs/areas in four horizons: *Today*, *Next up*, *Pending*, *Backlog*. Add, check off, **edit** (✎ or double-click the text), and delete.
- **📅 Deadlines & exams** — track due dates with urgency colouring and a countdown bar. Add, **edit** (✎), and delete.
- **📸 Snapshots** — save a named restore point of your entire state and roll back to it later. Restoring auto-saves a backup first, so it's always undoable. Capped at 30 per user.
- **🔄 Cloud sync** — everything syncs to your account; use it on any device.
- **📴 Offline / local mode** — works without signing in; data is kept in the browser's local storage.

---

## Architecture

```
        ┌──────────────────┐
        │   Your Browser   │
        │  (index.html UI) │
        └────────┬─────────┘
                 │  Supabase JS SDK
                 ▼
        ┌──────────────────────────────┐
        │          Supabase            │
        │  Auth: email magic link      │
        │  Postgres + Row Level Security│
        │  Tables: tasks, deadlines,    │
        │          snapshots            │
        └──────────────────────────────┘
```

- **Frontend**: pure HTML/CSS/JS in one file. The Supabase JS SDK loads from a CDN.
- **Auth**: Supabase email OTP / magic link (`signInWithOtp`).
- **Data isolation**: every table has an RLS policy `auth.uid() = user_id`, so users only ever see their own rows. The anon key embedded in the page is public by design — RLS, not the key, is what protects data.

---

## Run it locally

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

For magic-link login to work locally, add `http://localhost:8000/**` to the Supabase **Redirect URLs** (see below). Without sign-in, the dashboard still runs in offline/local mode.

---

## Backend setup (one-time, by the owner)

The app points at a shared Supabase project. To stand up your own:

1. **Create a project** at [supabase.com](https://supabase.com) (free, no card).

2. **Create the tables + security policies** — run this in the SQL Editor:

   ```sql
   create table public.tasks (
     id uuid primary key default gen_random_uuid(),
     user_id uuid not null references auth.users(id) on delete cascade default auth.uid(),
     hiwi_id text not null,
     horizon_id text not null,
     text text not null,
     done boolean not null default false,
     created_at timestamptz not null default now()
   );

   create table public.deadlines (
     id uuid primary key default gen_random_uuid(),
     user_id uuid not null references auth.users(id) on delete cascade default auth.uid(),
     title text not null,
     date date not null,
     type text not null,
     created_at timestamptz not null default now()
   );

   create table public.snapshots (
     id uuid primary key default gen_random_uuid(),
     user_id uuid not null references auth.users(id) on delete cascade default auth.uid(),
     label text not null,
     data jsonb not null,
     created_at timestamptz not null default now()
   );

   alter table public.tasks enable row level security;
   alter table public.deadlines enable row level security;
   alter table public.snapshots enable row level security;

   create policy "own tasks"     on public.tasks     for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
   create policy "own deadlines" on public.deadlines for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
   create policy "own snapshots" on public.snapshots for all using (auth.uid() = user_id) with check (auth.uid() = user_id);
   ```

3. **Set redirect URLs** under **Authentication → URL Configuration**:
   - **Site URL**: your hosting URL (e.g. `https://gc.ashuftw.com`)
   - **Redirect URLs**: add `https://gc.ashuftw.com/**` and `http://localhost:8000/**`

4. **Wire up the keys** — from **Project Settings → API**, copy the **Project URL** and the **anon / public** key into the constants near the top of `index.html`'s script:

   ```javascript
   const SUPABASE_URL = 'https://<project-ref>.supabase.co';
   const SUPABASE_ANON_KEY = 'eyJhbGci...';
   ```

> **Email limits:** Supabase's built-in email sender is rate-limited (a few magic links per hour) and intended for testing. For more volume, configure a custom SMTP provider (e.g. Resend) under **Authentication → Emails**.

---

## Deploy

The app is a static file, so any static host works. This repo deploys via **GitHub Pages** (Settings → Pages → Deploy from branch → `main` / root), auto-publishing on every push to `main`.

It's served at the custom domain **gc.ashuftw.com** via the `CNAME` file in the repo root. To wire up the domain:

1. In your DNS provider, add a `CNAME` record: `gc` → `ashuftw.github.io`.
2. In **Settings → Pages → Custom domain**, confirm `gc.ashuftw.com` and enable **Enforce HTTPS** once the certificate is issued.
3. Add `https://gc.ashuftw.com/**` to the Supabase Redirect URLs (and set it as the Site URL).
