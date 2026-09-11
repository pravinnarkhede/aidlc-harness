# Repo Catalog — PLACEHOLDER, not yet filled in

This is a template, not real data. `auxiliary-project-onboarding` treats
this file as "not yet configured" until you either fill it in yourself
before running your first ticket, or answer its questions interactively (in
which case it overwrites this placeholder with your real answers).

## How to fill this in yourself (optional — skip if you'd rather answer
## onboarding's questions interactively instead)

Replace the JSON block below with your project's real repos. Shape:
`{org, repos: [{name, url, branch, tags, role}]}`.

- `name` — the repo's folder name once cloned (must match its actual repo name)
- `url` — the clone URL (SSH or HTTPS)
- `branch` — the branch to clone/track (its release or default branch)
- `tags` — short free-text labels used to match a ticket's labels/components
  against this repo (e.g. `["backend", "api"]`)
- `role` — **in your own words**: what this repo is and what it owns vs.
  consumes from another repo in the set. This is the part onboarding
  combines with what reverse-engineering actually finds in the code — not
  guessed from the repo name alone.

```json
{
  "org": "your-git-org",
  "repos": [
    {
      "name": "backend-api",
      "url": "git@github.com:your-org/backend-api.git",
      "branch": "main",
      "tags": ["backend", "api"],
      "role": "Backend API and database — owns the contract every other repo consumes"
    },
    {
      "name": "web-app",
      "url": "git@github.com:your-org/web-app.git",
      "branch": "main",
      "tags": ["frontend", "web"],
      "role": "Customer-facing web app — consumes backend-api"
    }
  ]
}
```

Add one entry per repo in your project — there's no fixed count, list
however many repos actually make up your project.
