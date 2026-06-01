# Cross-Device Sync — Setup (≈10 minutes, free)

Your dashboard is ready to go live. To make your data follow you across every device, do these steps once. Until you finish them, the site still works perfectly — it just stores data locally on each device (the chip in the top-right will read **SYNC OFF · ADD KEYS**).

---

## 1. Create a free Supabase project

1. Go to **https://supabase.com** → **Start your project** → sign in with GitHub or email.
2. Click **New project**. Give it any name (e.g. `identity-pro`), set a database password (save it somewhere), pick the closest region.
3. Wait ~2 minutes for it to finish provisioning.

## 2. Create the data table

1. In your project, open **SQL Editor** (left sidebar) → **New query**.
2. Paste this in and click **Run**:

```sql
create table if not exists public.app_state (
  user_id    uuid primary key references auth.users(id) on delete cascade,
  data       jsonb not null default '{}'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.app_state enable row level security;

create policy "Users manage their own row"
  on public.app_state for all
  to authenticated
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

alter publication supabase_realtime add table public.app_state;
```

This creates one private row per user, locks it so only you can read/write your own data, and turns on live updates so changes push to your other devices instantly.

## 3. Allow your website to sign you in

1. Left sidebar → **Authentication** → **URL Configuration**.
2. Set **Site URL** to your live address (e.g. `https://your-site.netlify.app`).
3. Under **Redirect URLs**, add the same address. Save.

> Email sign-in is on by default — nothing else to enable. (While testing locally you can also add `http://localhost` here.)

## 4. Copy your two keys

1. Left sidebar → **Project Settings** → **API**.
2. Copy the **Project URL** and the **anon / public** key (the long one labeled `anon`).

## 5. Paste them into the dashboard

Open `index.html`, find this block near the very bottom, and replace the two placeholders:

```js
const SUPABASE_URL      = 'PASTE_YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'PASTE_YOUR_SUPABASE_ANON_KEY';
```

Save the file.

## 6. Deploy & test

1. Put the folder live (e.g. drag it onto **app.netlify.com/drop**).
2. Open the site. The top-right chip now reads **◌ SYNC · SIGN IN** — click it, enter your email, and click the link Supabase emails you.
3. After sign-in the chip shows **✓ SYNCED · your@email**. Add a note or tick a protocol, then open the site on your phone, sign in with the same email — your data is there.

---

## Good to know

- **One account, every device.** Sign in with the same email anywhere and your data loads. Changes save automatically (the chip flashes *SAVING…* then *SYNCED*) and update your other open devices live.
- **What syncs:** your operating-protocol state (focus wheel, vortex, identity journal, habits), the protocol-metric tracker, your coherence notes, and all the folder pages (trade journal, pre-market, beliefs, phase notes, metrics).
- **Conflict rule:** if you edit the *same* data on two devices at once, the most recent save wins for the whole bundle. For normal one-place-at-a-time use this never comes up.
- **Privacy:** row-level security means only your logged-in account can ever read your row.
- **Free tier** is ample for personal use. No credit card required.
- The **Coherence Notes → folder** feature (writing .md files to a disk folder) is separate from cloud sync and still works per-device in Chrome/Edge.
