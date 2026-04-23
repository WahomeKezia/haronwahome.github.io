# Haron Wahome Portfolio — Deployment Guide
## Supabase + GitHub Pages · 100% Free · ~30 minutes setup

---

## What you'll have when done

- A live website at **haronwahome.github.io** (or a custom domain)
- Permanent photo uploads stored in the cloud
- Blog posts and journal entries saved to a database
- Owner-only login — only Haron can post; visitors only read
- Zero monthly cost

---

## STEP 1 — Set up Supabase (the database)

1. Go to **https://supabase.com** and click **Start your project**
2. Sign up with GitHub or email (free)
3. Click **New Project**, give it a name like `haron-portfolio`, choose a region close to Kenya (e.g. AWS EU West)
4. Set a strong database password — save it somewhere safe
5. Wait ~2 minutes for the project to start

### Create the database tables

Once your project is ready, click **SQL Editor** in the left sidebar, then click **New query**. Paste this SQL and click **Run**:

```sql
-- Blog posts table
create table blog_posts (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  title text not null,
  category text,
  body text not null,
  user_id uuid references auth.users(id)
);

-- Journal entries table
create table journal_entries (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  title text not null,
  body text not null,
  user_id uuid references auth.users(id)
);

-- Photos metadata table
create table photos (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  url text not null,
  caption text,
  tag text default 'general',
  storage_path text not null,
  user_id uuid references auth.users(id)
);
```

### Set Row Level Security (who can read/write)

Still in the SQL Editor, run this second query:

```sql
-- Anyone can READ blog posts, journal entries, and photos
alter table blog_posts enable row level security;
alter table journal_entries enable row level security;
alter table photos enable row level security;

create policy "Public read blog" on blog_posts for select using (true);
create policy "Owner write blog" on blog_posts for insert with check (auth.uid() = user_id);
create policy "Owner delete blog" on blog_posts for delete using (auth.uid() = user_id);

create policy "Public read journal" on journal_entries for select using (true);
create policy "Owner write journal" on journal_entries for insert with check (auth.uid() = user_id);
create policy "Owner delete journal" on journal_entries for delete using (auth.uid() = user_id);

create policy "Public read photos" on photos for select using (true);
create policy "Owner write photos" on photos for insert with check (auth.uid() = user_id);
create policy "Owner delete photos" on photos for delete using (auth.uid() = user_id);
```

### Create the photo storage bucket

1. In the left sidebar click **Storage**
2. Click **New bucket**, name it exactly: `photos`
3. Toggle **Public bucket** ON (so visitors can see images)
4. Click **Save**

Then in Storage, click **Policies** → **New policy** for the `photos` bucket:
- For SELECT: allow for everyone (public)
- For INSERT: allow for authenticated users only

---

## STEP 2 — Create Haron's login account

1. In Supabase left sidebar, click **Authentication** → **Users**
2. Click **Add user** → **Create new user**
3. Enter Haron's email and a strong password
4. Click **Create user**

This is the only account that can log in and post to the site.

---

## STEP 3 — Get your Supabase keys

1. In the left sidebar click **Project Settings** → **API**
2. Copy these two values:
   - **Project URL** (looks like `https://abcdefgh.supabase.co`)
   - **anon / public key** (a long string starting with `eyJ…`)

   NEXT_PUBLIC_SUPABASE_URL=https://pltmyhgzjbxpmyawijvb.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_4S46OFvSzB4TGxSYgSpnRA_4UWNqK9h

---

## STEP 4 — Update the portfolio file

Open the file **index.html** in any text editor (Notepad, VS Code, etc.)

Find these two lines near the top of the `<script>` section:

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

Replace them with your actual values:

```javascript
const SUPABASE_URL = 'https://abcdefgh.supabase.co';
const SUPABASE_ANON_KEY = 'eyJhbGci...your-full-key-here';
```

Save the file.

---

## STEP 5 — Deploy to GitHub Pages

1. Go to **https://github.com** and log in to your account
2. Click the **+** icon → **New repository**
3. Name it exactly: `haronwahome.github.io`
   _(Replace `haronwahome` with whatever GitHub username Haron uses)_
4. Set it to **Public**
5. Click **Create repository**

### Upload the file

6. On the new repository page, click **Add file** → **Upload files**
7. Drag your `index.html` file into the upload area
8. At the bottom, click **Commit changes**

### Enable GitHub Pages

9. Click **Settings** (top of the repository)
10. In the left sidebar click **Pages**
11. Under **Source**, select **Deploy from a branch**
12. Choose branch: **main**, folder: **/ (root)**
13. Click **Save**

After ~2 minutes, the site will be live at:
**https://haronwahome.github.io** (or whatever the repo name is)

---

## STEP 6 — Test everything

1. Visit the live URL
2. Click **Owner Login** in the nav
3. Log in with the email and password you created in Step 2
4. You should see the amber admin banner at the top
5. Try uploading a photo, writing a blog post, and adding a journal entry
6. Log out, refresh — all content should still be there
7. Confirm there is no login button visible (it's there but only "Owner Login" — visitors can see it but can't post anything)

---

## Updating the site in the future

Whenever Haron wants to change the static content (about text, research info, etc.), simply:
1. Edit `index.html`
2. Go to the GitHub repository
3. Click the file → click the pencil icon to edit → commit changes

The site updates within seconds.

---

## Optional: Custom domain (e.g. haronwahome.com)

If Haron wants a proper domain:
1. Buy a domain from Namecheap (~$12/year) or Google Domains
2. In GitHub Pages settings, enter the custom domain
3. Follow the DNS setup instructions GitHub provides

---

## Free tier limits (you will almost certainly never hit these)

| Service | Free limit |
|---|---|
| GitHub Pages | 1 GB storage, 100 GB bandwidth/month |
| Supabase Storage | 500 MB |
| Supabase Database | 500 MB, 50,000 rows |
| Supabase Bandwidth | 2 GB/month |

A personal portfolio with photos and blog posts will use a tiny fraction of these limits.

---

## Troubleshooting

**Photos not showing up** → Check that the `photos` bucket is set to Public in Supabase Storage

**Login not working** → Double-check the email/password in Supabase Authentication → Users

**"Error saving post"** → Make sure the SQL policies were run correctly in Step 1

**Site not loading** → Wait 5 minutes after enabling GitHub Pages and hard-refresh (Ctrl+Shift+R)
