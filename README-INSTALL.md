# Lae Secondary School CMS — Backend Integration Package

This package connects the existing principal-only Supabase login to secure draft storage and direct publishing to GitHub repository `elwi123-cell/Secondary-School-Web-Template-1` on branch `main`.

## Important repository check

The GitHub repository's visible root listing and the supplied updated ZIP are not identical. Before replacing files, compare the files in this package with the current `main` branch and merge changes carefully. The public repository listing currently shows 18 root-level public HTML pages (listed in the function allowlist). The supplied ZIP contains many more HTML files than the current GitHub repository listing. The publisher is deliberately restricted to the 18 pages verified in the repository listing; other dropdown entries will not publish until those pages are actually added to `main` and the allowlist is reviewed and updated.

## Files

- `page-editor.html` — editor updated with Save Draft and Publish to Live Website.
- `supabase/migrations/202610270001_cms_publishing.sql` — draft and audit-log tables with principal-only RLS.
- `supabase/functions/cms-github-publisher/index.ts` — Edge Function; validates JWT and principal role, restricts filenames, checks GitHub SHA to avoid overwriting newer edits, and commits the HTML to `main`.
- `.github/workflows/deploy-pages.yml` — optional Pages deployment workflow. Use it only if Settings → Pages → Build and deployment → Source is set to **GitHub Actions**.
- `supabase-config.js` — the existing public browser configuration; contains no service-role or GitHub secret.

## Step-by-step installation

### 1. Back up the live site

Create a GitHub branch or download a ZIP of the current `main` branch. Do not replace the live repository wholesale with the supplied ZIP because the two versions may differ.

### 2. Confirm GitHub Pages source

Open repository Settings → Pages. If it says **Deploy from a branch**, confirm branch `main` and folder `/(root)`. In this mode, a successful commit to `main` triggers the Pages deployment automatically. If it says **GitHub Actions**, add the optional workflow in this package and keep the Pages source set to GitHub Actions. Do not switch a working setup without checking it first.

Official docs: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site

### 3. Apply the SQL migration

In Supabase → SQL Editor, open and run `supabase/migrations/202610270001_cms_publishing.sql`. This expects your existing `public.cms_admins(user_id, role)` table and the existing principal row. If the real table schema differs, adjust the migration to match before running it.

### 4. Create a GitHub token for the server only

Create a fine-grained personal access token restricted to this repository with **Contents: Read and write** and Metadata read access. Give it an expiry and rotate it periodically. Never put it in `supabase-config.js`, HTML, browser JavaScript, GitHub Pages files, or this chat.

### 5. Set Supabase Edge Function secrets

From the Supabase project directory, use Supabase CLI (or Dashboard → Edge Functions → Secrets) to set:

- `GITHUB_TOKEN` = fine-grained token
- `GITHUB_OWNER` = `elwi123-cell`
- `GITHUB_REPO` = `Secondary-School-Web-Template-1`
- `GITHUB_BRANCH` = `main`
- `CMS_ALLOWED_ORIGIN` = `https://elwi123-cell.github.io`

Supabase provides `SUPABASE_URL`, `SUPABASE_ANON_KEY` and `SUPABASE_SERVICE_ROLE_KEY` in the Edge Function environment in normal hosted deployments. If your environment does not inject them, set them as function secrets using values from Supabase Project Settings → API. Never expose the service-role key to the browser.

### 6. Deploy the Edge Function

Install/login to Supabase CLI and link the existing project (`dhslaxdmsmbcmgkzyzfl`), then deploy:

```bash
supabase login
supabase link --project-ref dhslaxdmsmbcmgkzyzfl
supabase functions deploy cms-github-publisher
```

For local testing, Supabase CLI docs: https://supabase.com/docs/guides/functions

### 7. Commit the updated editor

Copy the updated `page-editor.html` to the repository root. Keep `supabase-config.js` beside it. Make sure `dashboard.html`, `login.html`, and `page-editor.html` remain excluded from the publisher's allowlist, so the principal cannot use the public page editor to rewrite the CMS itself. The dashboard selector currently lists pages from the ZIP; remove or disable entries that do not exist on GitHub `main` until the repository is reconciled.

### 8. Test safely

1. Sign in through the existing principal login.
2. Open a normal public page such as `about.html` from the dashboard.
3. Edit a harmless word and click **Save Draft to Supabase**. Verify a row appears in `cms_page_drafts`.
4. Click **Preview & Publish**, inspect the whole page, then click **Publish to Live Website**.
5. Check the returned commit link, GitHub repository commit history, and GitHub Pages deployment status.
6. Verify the live page on a separate browser/private tab.
7. Test that a non-principal user gets HTTP 403 and that a stale editor session gets HTTP 409 instead of overwriting a newer commit.

## Security and limitations

- The GitHub token is only used in the Supabase Edge Function.
- The function verifies the signed-in user and checks the `principal` role server-side; hiding buttons in HTML is not a security control.
- Filename allowlisting and GitHub SHA conflict checks prevent path traversal and accidental overwrite of newer page versions.
- Direct publishing commits to `main`, so each approved publish is a production change. For stronger governance, use a pull-request approval workflow instead of direct commits.
- The current Retain Professionalism action remains browser-based layout checks and simple text cleanup. It is **not** genuine AI proofreading; an AI provider would need a separate server-side integration and secret.
- GitHub Pages deployment is asynchronous. A successful commit does not mean the public page has finished deploying.
- Static HTML pages are public. Do not put private student records, credentials, or confidential information into page content.
- Keep the allowed page list synchronized with real root-level HTML files on `main`; the function intentionally refuses files outside the allowlist.
