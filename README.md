# Applicant Status Tracker

A public page where job applicants can enter a tracking code and see a safe, non-sensitive status — built with security as the main focus.

**Live demo:** _add your Netlify URL here_

## What it does
- Applicant enters a tracking code → sees status (e.g. "Under review"), a 4-step progress bar, and a short note.
- Invalid and malformed codes return the exact same generic error, so no one can tell a code is "almost right."
- Repeated wrong guesses from the same IP get rate-limited (5 tries / 10 min → 15 min block).
- Only non-sensitive fields are ever returned — no names, emails, or IDs.

## How it works
```
Browser (index.html)
   → Supabase Edge Function (check-status)
       → Postgres RPC (check_status)
           → applications table (RLS on, hashed codes)
```

- **Frontend:** plain HTML/CSS/JavaScript, no framework.
- **Edge Function:** TypeScript (Deno), adds a constant response delay and maps results to HTTP status codes (200 / 404 / 429).
- **Database:** PostgreSQL on Supabase. Row Level Security is on with no public policies, so the table can't be read directly. The RPC is `SECURITY DEFINER` and only callable by the service role. Codes and IPs are stored as salted SHA-256 hashes, never plain text.

## Project structure
```
index.html                              frontend
supabase/schema.sql                     tables, RLS, RPC, sample data
supabase/functions/check-status/index.ts  Edge Function
docs/architecture.svg                   architecture diagram
SELF_REVIEW.md                          what's tested, what's not, known limits
```

## Setup
1. Create a free project at supabase.com and run `supabase/schema.sql` in the SQL Editor.
2. Deploy `supabase/functions/check-status` as an Edge Function (JWT verification off — it's a public endpoint).
3. In `index.html`, set `SUPABASE_URL` and the publishable `SUPABASE_ANON_KEY` from Project Settings → API.
4. Host `index.html` anywhere static (Netlify, Vercel, GitHub Pages).

## Sample codes
| Code | Status |
|---|---|
| `APP-7K3M-9QXH` | Under review |
| `APP-2WZN-4RTB` | Interview stage |
| `APP-8HDC-5VYA` | Received |

## Security measures
| Risk | Mitigation |
|---|---|
| Data leakage | RLS + no table access from the browser; RPC returns only safe fields |
| Enumeration | Identical error for invalid/malformed codes |
| Brute force | Per-IP rate limit (5 fails / 10 min → 15 min block) |
| Timing attacks | Minimum 600 ms response time |
| DB leak | Codes/IPs stored as salted hashes, not plain text |

See `SELF_REVIEW.md` for what's fully tested and what's left to verify.
