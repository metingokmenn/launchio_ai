# LaunchIO — website

Marketing site and legal pages for the LaunchIO app, hosted on **Firebase Hosting**.

```
public/            # everything that gets deployed
  index.html       # landing page
  privacy.html     # served at /privacy
  terms.html       # served at /terms
  404.html
  robots.txt, sitemap.xml
  assets/          # site.css, site.js, logos, favicons, og-image
firebase.json      # hosting config (clean URLs, headers, caching)
.firebaserc        # Firebase project ID
index.html, privacy.html, terms.html   # GitHub Pages redirects (see below)
```

The site is plain HTML, CSS and a few lines of JavaScript. There is no build step.

## Local preview

```bash
npm i -g firebase-tools
firebase emulators:start --only hosting --project demo-launchio
# → http://127.0.0.1:5000
```

## Deploy

### Manual

```bash
firebase login
firebase deploy --only hosting
```

The project comes from `.firebaserc`. To deploy to a different project, add `--project <project-id>`.

### Automatic (GitHub Actions)

- `.github/workflows/firebase-hosting-merge.yml` deploys to the **live** channel on every push to `main`.
  It can also be started by hand from the Actions tab ("Run workflow").
- `.github/workflows/firebase-hosting-pull-request.yml` deploys a 7-day **preview** channel for each pull request
  and comments the preview URL on it.

Both need a repository secret named `FIREBASE_SERVICE_ACCOUNT` that holds a service-account JSON key.
Until the secret exists, both workflows fail.

### Service account setup (one time)

The Firebase project is `launchio-420f7` (see `.firebaserc`); the site is served at `https://launchio-420f7.web.app`.

```bash
PROJECT_ID=launchio-420f7
SA=github-action-hosting@$PROJECT_ID.iam.gserviceaccount.com

gcloud iam service-accounts create github-action-hosting \
  --project "$PROJECT_ID" --display-name "GitHub Actions (Firebase Hosting)"

for ROLE in \
  roles/firebasehosting.admin \
  roles/firebaseauth.admin \
  roles/run.viewer \
  roles/serviceusage.apiKeysViewer \
  roles/serviceusage.serviceUsageConsumer
do
  gcloud projects add-iam-policy-binding "$PROJECT_ID" \
    --member "serviceAccount:$SA" --role "$ROLE" --condition None
done

gcloud iam service-accounts keys create key.json --iam-account "$SA"
gh secret set FIREBASE_SERVICE_ACCOUNT --repo metingokmenn/launchio_ai < key.json
rm key.json
```

The five roles, as they are named in the Google Cloud console (IAM & Admin → IAM):

| Role | Why it is needed |
| --- | --- |
| Firebase Hosting Admin | Upload and release the site |
| Firebase Authentication Admin | Register preview-channel domains |
| Cloud Run Viewer | Required by the deploy action |
| API Keys Viewer | Required by the deploy action |
| Service Usage Consumer | Without it the deploy fails with a 403 on `serviceusage.services.get` |

Without `gcloud` and `gh`: create the service account and a JSON key in the Google Cloud console
(IAM & Admin → Service Accounts), grant the five roles under IAM, then paste the whole JSON file into
GitHub → Settings → Secrets and variables → Actions → New repository secret, named `FIREBASE_SERVICE_ACCOUNT`.

`firebase init hosting:github` can also create the service account and the secret. It names the secret
differently and does not grant Service Usage Consumer, so rename the secret and add that role afterwards.

The key gives access to the app's Firebase project. Keep it only in the GitHub secret and delete local copies.

## Old GitHub Pages addresses

The site used to be served by GitHub Pages at `https://metingokmenn.github.io/launchio_ai/`.
The App Store listing and the released app link to those addresses, so they must keep working:

- The root-level `index.html`, `privacy.html` and `terms.html` redirect to the Firebase site.
- Keep GitHub Pages enabled, keep this repository public and do not delete the root files.

After a deploy, check that these three addresses land on the new site:

- `https://metingokmenn.github.io/launchio_ai/`
- `https://metingokmenn.github.io/launchio_ai/privacy.html`
- `https://metingokmenn.github.io/launchio_ai/terms.html`
