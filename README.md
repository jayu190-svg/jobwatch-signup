# Job Watch sign-up page

A small Streamlit app. A friend enters the invite code, uploads a resume (PDF or DOCX) and answers three questions
(where, what kind of role next, optional Pushover key). Claude builds their profile from the resume, they confirm a plain-English
summary (and can correct it in a sentence), and the app saves it to the **private** GitHub repo `jobwatch-profiles`. The Job Watch engine on apex-vm pulls that repo at the
start of every run, so they appear automatically (7 AM and 2 PM Pacific, weekdays).

## What a person sees
1. **Sign up** — invite code → resume + 3 questions → "Here's what I'll search for" (correct it or confirm) → "Your ID" and a one-time "edit code". The old long form is under **Advanced options** and works without the Anthropic key.
2. **Edit my filters** — ID + edit code → the same plain-English summary and correction box (Advanced options has the long form). Changes apply on the next run, and only after the person confirms them.
3. **Download my CSV** — ID + edit code → their jobs as a CSV (refreshed weekly by the engine).
After the engine's next run the page shows their private job-list link.

## Secrets (Streamlit "Secrets" box — never in code, never committed)
```
INVITE_CODE = "pick-a-phrase-to-share-with-friends"
GITHUB_TOKEN = "github_pat_or_ghp_...   (the token with 'repo' scope)"
ANTHROPIC_API_KEY = "sk-ant-...   (same key the engine uses; powers the resume-first sign-up)"
```
Optional: `PROFILES_REPO = "yourname/jobwatch-profiles"` (otherwise it is `<token owner>/jobwatch-profiles`).

## Deploy (the owner's three clicks)
1. Go to https://share.streamlit.io → **Create app** → *Deploy a public app from GitHub* is fine; pick the **private** repo
   `jobwatch-signup`, branch `main`, main file `app.py`. (If the private repo isn't listed, click "Connect GitHub" and allow access to it.)
2. Before deploying, open **Advanced settings → Secrets** and paste the three lines above with your real values.
3. Click **Deploy**. Share the app URL plus the invite code with the person you want to add.

## Notes
- Files written to the repo per person: `profiles/<id>/profile.json`, `resume.txt`, `auth.json` (salted hash of the edit code),
  `contact.json` (email, for the owner's record). The engine reads only `profile.json` and `resume.txt`, and re-sanitizes `profile.json`.
- The engine writes back `public.json` (their page link) and `export.csv` (weekly). The app only reads those.
- To remove someone: delete their folder from `jobwatch-profiles` **and** run `run_jobwatch.bat --remove-profile <id>` on apex-vm.
- Claude (`claude-sonnet-4-6`, same as the engine) reads the resume as untrusted data; the person's answers override it on place and level, and everything goes through the engine's sanitizer again. Each sign-up or correction is one API call; a browser session is limited to 3 sign-ups and 6 corrections.
- Limits: 25 job titles, 5 cities, resume ≤ 5 MB.
- Local test: `py -m pip install -r requirements.txt` then `py -m streamlit run app.py` (needs `.streamlit/secrets.toml`, which is git-ignored).
