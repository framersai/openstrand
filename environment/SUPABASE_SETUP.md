# Supabase Setup Guide

This project uses Supabase for authentication (email/password, OAuth) and eventually per-user quotas. Follow the steps below to configure a project for local development.

## 1. Create a Supabase project
1. Visit [https://supabase.com](https://supabase.com) and sign in.
2. Create a new project (free tier works fine for development).
3. Note the **Project URL** and **Anon/Public Key** under _Project Settings -> API_.

## 2. Configure authentication providers
1. Go to _Authentication -> Providers_.
2. Enable **Email** (optional), **GitHub**, and **Google**.  
   - For GitHub: supply your GitHub OAuth Client ID/Secret (create via GitHub Developer settings).
   - For Google: supply the OAuth Client ID/Secret from Google Cloud Console.
3. Under _Authentication -> URL Configuration_, set:
   - **Site URL**: `http://localhost:3000` (update with production URL later).
   - **Redirect URLs**: add `http://localhost:3000/auth`.

## 3. Add environment variables
Update `.env` (or `.env.local`) with:
```env
NEXT_PUBLIC_SUPABASE_URL=https://YOUR-PROJECT.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=YOUR-ANON-KEY
NEXT_PUBLIC_SITE_URL=http://localhost:3000

SUPABASE_URL=https://YOUR-PROJECT.supabase.co
SUPABASE_ANON_KEY=YOUR-ANON-KEY
SUPABASE_SERVICE_ROLE_KEY=YOUR-SERVICE-ROLE-KEY # optional for server tasks
SUPABASE_JWT_ISSUER=https://YOUR-PROJECT.supabase.co/auth/v1
SUPABASE_JWT_AUDIENCE=authenticated
SUPABASE_ENFORCE_AUTH=false
```
> Set `SUPABASE_ENFORCE_AUTH=true` once you want the backend to reject unauthenticated requests.

## 4. (Optional) Service role key
If you plan to manage quotas or sync profiles server-side, grab the **service role key** under _Project Settings -> API_ and store it as `SUPABASE_SERVICE_ROLE_KEY`. **Never expose** this key to the browser or commit it to Git.

## 5. Test locally
1. Install workspace dependencies (`npm install` from the repo root).
2. Start both backend and frontend:
   ```bash
   ./start-local.sh
   ```
   or, if you prefer manual control:
   ```bash
   npm run dev --workspace @framers/openstrand-teams-backend
   npm run dev --workspace openstrand-app
   ```
3. Visit `http://localhost:3000/auth` and sign in via GitHub/Google. Successful login should redirect back to `/` and show your email in the header.

## 6. Production readiness checklist
- Update `NEXT_PUBLIC_SITE_URL` to your hosted domain (e.g., `https://openstrand.ai`).
- Add production redirect URLs in Supabase (e.g., `https://openstrand.ai/auth`).
- Consider enabling _Row Level Security_ and storing user profiles/usage data.
- Turn on `SUPABASE_ENFORCE_AUTH=true` so API routes require Bearer tokens.

---

Once Supabase is configured, proceed to the LemonSqueezy plan (`docs/LEMON_SQUEEZY_PLAN.md`) to wire up billing and subscriptions.


