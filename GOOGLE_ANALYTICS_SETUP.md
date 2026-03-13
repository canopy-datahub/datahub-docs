# Google Analytics Setup

The UI uses **Google Analytics 4 (GA4)** via Next.js's official `@next/third-parties/google` library. It automatically tracks pageviews on every route change, plus custom events for key user interactions.

---

## Prerequisites

- A Google Analytics 4 account and property ([create one here](https://analytics.google.com))
- Your **GA4 Measurement ID** — format: `G-XXXXXXXXXX` (found in GA4 → Admin → Data Streams → your web stream)

---

## Step 1: Set the Environment Variable

The Measurement ID is injected via a single environment variable:

```
NEXT_PUBLIC_GTAG=G-XXXXXXXXXX
```

### Local development

Add it to `.env.local` in the `datahub-ui-main` root:

```bash
# datahub-ui-main/.env.local
NEXT_PUBLIC_GTAG=G-XXXXXXXXXX
```

### Docker / ECS deployment

Set `NEXT_PUBLIC_GTAG` in `datahub-ui-main/.env.local` — `deploy.py` reads this file automatically and passes the value as a Docker build arg when deploying the UI:

```bash
# datahub-ui-main/.env.local
NEXT_PUBLIC_GTAG=G-XXXXXXXXXX
```

Then deploy as normal:

```bash
cd ~/dataHub/datahub-deployment-scripts
python deploy.py ${PROJECT_NAME} ui ${ENV}
```

For a local Docker build/test before deploying:

```bash
cd ~/dataHub/datahub-ui-main
docker-compose build   # reads .env.local automatically
docker-compose up      # runs on http://localhost:3000
```

> ⚠️ **`NEXT_PUBLIC_GTAG` is baked into the JS bundle at build time.** Setting it in the ECS task definition environment or AWS Secrets Manager has **no effect** — Next.js inlines `NEXT_PUBLIC_*` values during the Docker build, not at container startup. To change the value, update `.env.local` and redeploy the UI.
---

## Step 2: Verify It's Working

1. Run the app locally (`npm run dev`) or deploy it
2. Open your site in a browser and open DevTools → Network tab
3. Filter by `google-analytics` or `gtag` — you should see requests to `https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX`
4. In GA4 → Reports → Realtime, confirm your session appears within ~30 seconds

---

## Where Analytics Are Collected

### Automatic Page View Tracking

`<GoogleAnalytics>` is mounted at the app root in `pages/_app.js` and fires a `page_view` event automatically on every Next.js client-side route change. No code is needed on individual pages. Every page in the app is covered:

| Page | Route |
|---|---|
| Homepage | `/` |
| Study Explorer — Studies tab | `/studyExplorer/studies` |
| Study Explorer — Variables tab | `/studyExplorer/variables` |
| Study Detail | `/studyExplorer/studies/[id]` |
| Resource Center | `/resourceCenter` |
| About | `/about` |
| News | `/news` |
| Events | `/events` |
| FAQ | `/faq` |
| Funding Opportunities | `/fundingOpportunities` |
| Glossary | `/glossary` |
| Admin / Portal pages | `/portal/*` |

This is the primary source of **user activity metrics** — you can see total visits, unique users, bounce rate, and session duration per page in GA4 → Reports → Engagement → Pages and screens.

### Custom Event Tracking

In addition to automatic pageviews, the following user interactions fire custom events:

| Event Name | Where in the UI | Trigger | Parameters |
|---|---|---|---|
| `homePage` | Homepage search bar | User submits a search | `value: 'Home Page Search Made'`, `query: <search filters as JSON>` |
| `studyExplorer` | Study Explorer search bar | User submits a search | `value: 'Study Explorer Search Made'`, `query: <active filters as JSON>` |
| `studyExplorer` | Study Explorer results toolbar | User clicks **Download Results** (studies CSV) | `value: 'Download Results'` |
| `studyExplorer` | Study Explorer results toolbar | User clicks **Download Variables Results** (variables CSV) | `value: 'Download Variables Results'` |
| `advancedSearch` | Query Builder (advanced search panel) | User submits an advanced query | `value: 'Advanced Search Made'`, `query: <query builder rules as JSON>` |
| `toggleResults` | Results view toggle (Studies / Variables) | User switches to **List View** | `value: 'List View'` |
| `toggleResults` | Results view toggle | User switches to **Table View** | `value: 'Table View'` |
| `close_button` | Side widget (right-hand panel) | User closes the side panel | _(no params)_ |

### What You Can Measure in GA4

| Metric | Where to find it in GA4 |
|---|---|
| Total visits and unique users per page | Reports → Engagement → Pages and screens |
| Most-searched queries on Homepage | Reports → Engagement → Events → `homePage` → `query` parameter |
| Most-used Study Explorer filters | Reports → Engagement → Events → `studyExplorer` → `query` parameter |
| How often users download data | Reports → Engagement → Events → `studyExplorer`, filter `value = 'Download Results'` |
| Advanced search usage rate | Reports → Engagement → Events → `advancedSearch` |
| Preferred results view (list vs table) | Reports → Engagement → Events → `toggleResults` |
| Real-time active users | Reports → Realtime |

---

## Disabling Analytics

To disable GA entirely for an environment, simply omit `NEXT_PUBLIC_GTAG` from the environment variables (or set it to an empty string). No code changes required.

---

## Files Reference

| File | Role |
|---|---|
| `pages/_app.js` | Mounts `<GoogleAnalytics>` at app root — handles init and automatic `page_view` on every route change |
| `views/Homepage/Homepage.jsx` | `homePage` search event |
| `views/StudyExplorer/StudyExplorer.jsx` | `studyExplorer` search event |
| `views/StudyExplorer/Components/Results/ResultsActions.jsx` | `studyExplorer` download events (studies CSV and variables CSV) |
| `components/QueryBuilder/QueryBuilder.jsx` | `advancedSearch` event |
| `components/Toggle/.../SearchResultViewToggle.jsx` | `toggleResults` events (list / table view switch) |
| `components/SideWidget/SideWidget.jsx` | `close_button` event |
| `.env.local` | Measurement ID for local dev and ECS deployments |
| `Dockerfile` | Declares `NEXT_PUBLIC_GTAG` as a build `ARG` (value supplied at build time) |
| `docker-compose.yml` | Reads `.env.local` and forwards `NEXT_PUBLIC_GTAG` as a build arg for local Docker builds |
| `datahub-deployment-scripts/deploy.py` | Reads `.env.local` and passes `NEXT_PUBLIC_GTAG` as `--build-arg` when deploying to ECS |
