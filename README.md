# Job Watch sign-up page

A small Streamlit app. A friend enters the invite code, fills in a form, uploads a resume, and the app saves their
profile to the **private** GitHub repo `jobwatch-profiles`. The Job Watch engine on apex-vm pulls that repo at the
start of every run, so they appear automatically (7 AM and 2 PM Pacific, weekdays).

## What a person sees
1. **Sign up** — invite code → form → "Your ID" and a one-time "edit code".
2. **Edit my filters** — ID + edit code → the same form, pre-filled. Saved changes apply on the next run.
3. **Download my CSV** — ID + edit code → their jobs as a CSV (refreshed weekly by the engine).
After the engine's next run the page shows their private job-list link.

## Secrets (Streamlit "Secrets" box — never in code, never committed)
```
INVITE_CODE = "pick-a-phrase-to-share-with-friends"
GITHUB_TOKEN = "github_pat_or_ghp_...   (the token with 'repo' scope)"
```
Optional: `PROFILES_REPO = "yourname/jobwatch-profiles"` (otherwise it is `<token owner>/jobwatch-profiles`).

## Deploy (the owner's three clicks)
1. Go to https://share.streamlit.io → **Create app** → *Deploy a public app from GitHub* is fine; pick the **private** repo
   `jobwatch-signup`, branch `main`, main file `app.py`. (If the private repo isn't listed, click "Connect GitHub" and allow access to it.)
2. Before deploying, open **Advanced settings → Secrets** and paste the two lines above with your real values.
3. Click **Deploy**. Share the app URL plus the invite code with the person you want to add.

## Notes
- Files written to the repo per person: `profiles/<id>/profile.json`, `resume.txt`, `auth.json` (salted hash of the edit code),
  `contact.json` (email, for the owner's record). The engine reads only `profile.json` and `resume.txt`, and re-sanitizes `profile.json`.
- The engine writes back `public.json` (their page link) and `export.csv` (weekly). The app only reads those.
- To remove someone: delete their folder from `jobwatch-profiles` **and** run `run_jobwatch.bat --remove-profile <id>` on apex-vm.
- Limits: 25 job titles, 5 cities, resume ≤ 5 MB.
- Local test: `py -m pip install -r requirements.txt` then `py -m streamlit run app.py` (needs `.streamlit/secrets.toml`, which is git-ignored).
