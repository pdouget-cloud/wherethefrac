# Where the Frac? 🛢️

Real-time crowdsourced tracking of Permian Basin frac activity.
**Who is fracking right now — by operator and by pressure pumper.**

## Stack

- **Next.js 14** (App Router, TypeScript)
- **Tailwind CSS**
- **Supabase** (Postgres + Row Level Security)
- **Mapbox GL JS** (interactive map)
- **Resend** (transactional email for weekly digest)
- **Vercel** (hosting + cron for Monday digest)

---

## Local Setup (5 steps)

### 1. Clone & install

```bash
git clone https://github.com/YOUR_USERNAME/wherethefrac.git
cd wherethefrac
npm install
```

### 2. Set up Supabase

1. Go to [app.supabase.com](https://app.supabase.com) → New project
2. In the SQL editor, run the full contents of `supabase-schema.sql`
3. Copy your credentials from **Project Settings → API**:
   - `Project URL` → `NEXT_PUBLIC_SUPABASE_URL`
   - `anon public` key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `service_role` key → `SUPABASE_SERVICE_ROLE_KEY`

### 3. Set up Mapbox

1. Go to [account.mapbox.com](https://account.mapbox.com/access-tokens/)
2. Create a public token (default scopes are fine)
3. Copy it → `NEXT_PUBLIC_MAPBOX_TOKEN`

### 4. Set up Resend (email digest)

1. Go to [resend.com](https://resend.com) → create an account
2. Add & verify your sending domain
3. Create an API key → `RESEND_API_KEY`
4. Set `RESEND_FROM_EMAIL` to a verified address like `digest@yourdomain.com`

### 5. Configure env vars

```bash
cp .env.local.example .env.local
# Fill in all values in .env.local
```

**`.env.local` variables:**

| Variable | Where to get it |
|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase → Project Settings → API |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase → Project Settings → API |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase → Project Settings → API |
| `NEXT_PUBLIC_MAPBOX_TOKEN` | mapbox.com → Access Tokens |
| `RESEND_API_KEY` | resend.com → API Keys |
| `RESEND_FROM_EMAIL` | Your verified sending address |
| `CRON_SECRET` | Any random string, e.g. `openssl rand -hex 32` |

### 6. (Optional) Seed test data

```bash
npm run seed
```

This inserts ~27 realistic Permian Basin sightings spread across the last 24 hours.

### 7. Start dev server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Deploying to Vercel

### Option A — Vercel dashboard (recommended for first deploy)

1. Push this repo to GitHub
2. Go to [vercel.com/new](https://vercel.com/new) → Import your GitHub repo
3. Add all env vars from `.env.local` in Vercel → Settings → Environment Variables
4. Click **Deploy**

### Option B — GitHub Actions auto-deploy

1. Add `VERCEL_TOKEN` to GitHub → Settings → Secrets → Actions
   (get from vercel.com → Account → Tokens)
2. Push to `main` — the `deploy.yml` workflow handles the rest

### Vercel cron (weekly digest)

`vercel.json` is already configured to call `/api/weekly-report` every Monday at
14:00 UTC (8 am CT). Vercel sends the `Authorization: Bearer <CRON_SECRET>` header
automatically. Make sure `CRON_SECRET` matches between Vercel env vars and your local
`.env.local`.

---

## Project Structure

```
wherethefrac/
├── .github/
│   └── workflows/
│       ├── ci.yml                  # Build + lint check on PRs
│       ├── deploy.yml              # Auto-deploy main to Vercel
│       └── weekly-report-test.yml # Sunday health check ping
├── scripts/
│   └── seed.js                    # Local dev data seeder
├── src/
│   ├── app/
│   │   ├── globals.css
│   │   ├── layout.tsx              # Root layout with NavBar
│   │   ├── page.tsx                # Leaderboard home
│   │   ├── map/
│   │   │   └── page.tsx            # Map page
│   │   ├── submit/
│   │   │   └── page.tsx            # Submit form page
│   │   └── api/
│   │       ├── reports/route.ts    # GET + POST reports
│   │       ├── fleets/route.ts     # GET aggregated fleets
│   │       ├── subscribe/route.ts  # POST email subscription
│   │       └── weekly-report/route.ts  # Cron — sends digest
│   ├── components/
│   │   ├── NavBar.tsx
│   │   ├── LeaderboardTable.tsx    # Live-updating fleet table
│   │   ├── MapView.tsx             # Mapbox map with fleet pins
│   │   ├── SubmitForm.tsx          # GPS + URL pre-fill form
│   │   ├── SignalBadge.tsx         # Low/Medium/High badge
│   │   └── WeeklySignup.tsx        # Email digest sign-up
│   └── lib/
│       ├── types.ts                # Shared TypeScript types
│       ├── supabase.ts             # Browser Supabase client
│       ├── supabase-server.ts      # Server (service role) client
│       └── aggregate.ts            # Fleet clustering logic
├── supabase-schema.sql             # Run once in Supabase SQL editor
├── .env.local.example
├── vercel.json                     # Cron schedule
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## How Signal Points Work

| Source | Points |
|---|---|
| Anonymous sighting | 1 pt |
| Verified sighting (future) | 3 pts |

**Thresholds:**
- 🟢 Low — 1–2 pts
- 🟡 Medium — 3–5 pts
- 🔴 High — 6+ pts

Reports older than 48 hours are automatically excluded from the leaderboard and map.

---

## Fleet Clustering

Reports are grouped by **operator + pumper** (case-insensitive). Within each group,
individual sightings are clustered by:
1. Exact pad name match (case-insensitive), OR
2. GPS proximity within ~5 miles (0.072 degree threshold)

---

## URL Pre-fill (Share Links)

After submitting a sighting, a shareable URL is generated:
```
https://wherethefrac.com/submit?op=Diamondback+Energy&pump=Liberty+Oilfield+Services&pad=Rattler+18H
```

Anyone following this link gets the form pre-filled with the same operator/pumper/pad,
making it easy for field crews to confirm active sightings.

---

## Future Roadmap

- **RRC well lookup** — Autocomplete pad names from Texas Railroad Commission daily permit CSVs, auto-fill GPS from permit lat/lng
- **Email verification** — Double opt-in for 3-pt "verified" sightings
- **Push notifications** — Subscribe to alerts for specific operators or regions
- **Historical charts** — Fleet activity trends over 30/90 days

---

## License

MIT
