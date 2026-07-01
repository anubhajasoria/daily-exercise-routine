# Roz Ka Hisaab

A single-page daily workout &amp; habit tracker, built around a specific weekly strength
plan (lower body, walk, upper body, rest/stretch, full body, weekend activity), plus
two daily diet habits and an optional 7,000-step walk on non-walk days.

Everything lives in one file: `index.html`. No build step, no framework, no bundler.

## Features

- Monthly calendar with a fill-ring per day showing how much of that day's checklist
  is done
- Day panel with the weekday's specific exercises + diet habits, each toggleable
- "Watch real demo videos" link per exercise (opens a YouTube search for that
  movement's form, rather than a drawn animation)
- Weekly / monthly summary view with completion stats and a day-by-day breakdown
- Streak counter
- Signed-in sync across devices via Firebase Auth (email/password) + Firestore, so
  the same account shows the same data on your phone and laptop

## Local development

No build step — just open `index.html` in a browser, or serve the folder with any
static file server, e.g.:

```bash
python3 -m http.server 8000
```

Then visit `http://localhost:8000`.

## Firebase setup (required before first use)

The app is wired to Firebase Auth + Firestore for storage, but ships with placeholder
config values. To connect it to your own free Firebase project:

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add
   project** (the free Spark plan is enough).
2. In **Project settings**, click the `</>` (web app) icon to register a web app.
   Skip Firebase Hosting if asked. Copy the `firebaseConfig` object it gives you.
3. In `index.html`, find the `firebaseConfig` object near the top of the
   `<script type="module">` block and replace the placeholder values with your own.
4. **Authentication** → **Sign-in method** → enable **Email/Password**.
5. **Firestore Database** → **Create database** → start in production mode.
6. In Firestore's **Rules** tab, paste the rules below and click **Publish**. This
   restricts each account to only its own data:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /entries/{uid} {
         allow read, write: if request.auth != null && request.auth.uid == uid;
       }
     }
   }
   ```

## Deploying

Any static host works since there's no server-side code. Two free options:

- **Netlify Drop** — drag `index.html` onto [app.netlify.com/drop](https://app.netlify.com/drop)
  for an instant URL. Click "Claim this site" and create a free account to keep it
  permanently (otherwise it expires after 24 hours).
- **GitHub Pages** — push this repo to GitHub, then in the repo's
  **Settings → Pages**, set the source to your default branch. Your site will be live
  at `https://<username>.github.io/<repo>`.

## Data model

Each signed-in user's data lives in a single Firestore document at
`entries/{uid}`, with one field `data` holding a JSON blob keyed by date
(`YYYY-MM-DD`) → which checklist items are checked. Keeping it as one document keeps
reads/writes simple and cheap on Firestore's free tier.

## Customizing the plan

The weekly exercise plan, diet habits, and exercise-demo search queries are all
defined near the top of the `<script type="module">` block in `index.html`:

- `WORKOUTS` — one entry per weekday (0 = Sunday … 6 = Saturday) with a label and
  list of exercises
- `DIET_ITEMS` — the two daily habits (protein, portion awareness)
- `OPTIONAL_WALK_ITEM` — the bonus 7,000-step walk shown on days without a walk
- `EXERCISE_DEMOS` — the YouTube search query and form cues shown per exercise
