---
description: Publish the project to GitHub, configure GitHub Pages via GitHub Actions, update the README and repo metadata, and scan for secrets before pushing.
---

# Publish project to GitHub

Use this command when the user wants to upload the current project to GitHub, enable GitHub Pages, keep the README in sync, update the repo About metadata, and check for sensitive data before publication.

## Inputs

Ask the user for one of these:
- a GitHub repo URL to push to, or
- a GitHub repo name and owner to create, or
- a GitHub CLI auth status confirmation if the repo already exists.

Do not assume a repo exists. If the user provides a repo URL or repository name, use it.

## Workflow

1. Confirm the repo is a Git working tree.
   - If not a git repo: initialize it using `git init` and create an initial commit if needed.

2. Confirm GitHub authentication.
   - Run `gh auth status`.
   - If gh is not installed or not authenticated, tell the user to install GitHub CLI and run `gh auth login` or use an SSH key for GitHub access.
   - Do not proceed with a push until GitHub auth is confirmed.

3. Resolve the target repository.
   - If the user supplied a repo URL, set it as the remote:
     - `git remote set-url origin <repo-url>`
   - If the repo does not exist yet and the user wants a new repo, create it via:
     - `gh repo create <owner>/<repo> --public --source=. --remote=origin --push`
   - If the user supplied only a repo name and owner, use `gh repo create`.

4. Upload the code to GitHub.
   - Ensure the current branch is `main`:
     - `git branch -M main`
   - Push the branch:
     - `git push -u origin main`

5. Create or update the GitHub Pages workflow.
   - Ensure the repo has a `.github/workflows` directory.
   - Create or update `.github/workflows/pages.yml` with a GitHub Actions workflow that deploys the static site to GitHub Pages:

```yaml
name: Deploy static site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Pages
        uses: actions/configure-pages@v5

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

   - Commit and push the workflow:
     - `git add .github/workflows/pages.yml`
     - `git commit -m "Add GitHub Pages workflow"`
     - `git push origin main`

6. Enable GitHub Pages in the repository settings.
   - Open the repo settings page in the browser if the user is signed in to GitHub.
   - Navigate to Settings → Pages.
   - Set the source to `GitHub Actions`.
   - Save the setting.
   - If GitHub Pages is not available or the save button is disabled, explain the repo permission issue and ask the user to confirm they are the repo owner or admin.

7. Capture a current browser screenshot with the Playwright MCP tool.
  - Open `index.html` directly in a browser page.
  - Set the viewport to a representative desktop size such as 1440 × 1100.
  - Use Playwright to capture the full page to `screenshots/board.png`.
  - Create the `screenshots` directory first if it does not exist.
  - Reference the image from `README.md` using a relative Markdown image link.

8. Create or edit the repository README.
   - Create `README.md` if it does not exist.
   - Include:
     - project name
     - short description
     - key features
     - setup instructions
     - local run instructions
     - GitHub Pages URL if available
     - screenshots or usage notes if relevant
   - Use clear markdown headings and keep it concise.
   - Commit and push:
     - `git add README.md`
     - `git commit -m "Update README"`
     - `git push origin main`

9. Update the GitHub repository About field and homepage link.
   - Use the repo description and homepage field in GitHub:
     - `gh repo edit <owner>/<repo> --description "<short repo description>" --homepage "https://<user>.github.io/<repo>/"`
   - If the repo is not public or the gh repo edit command is not available, instruct the user to edit the About section in the GitHub web UI manually.
   - Add the GitHub Pages URL into the About/homepage field.

10. Scan the code for sensitive data before finalizing the upload.
   - Prefer a real secret scanner when installed:
     - `gitleaks detect --source . --no-banner --redact`
     - or `trufflehog filesystem .`
   - If no scanner is installed, run a defensive pattern scan with `git grep` and common secret patterns:
     - API keys, tokens, secrets, passwords, private keys, cloud credentials, and connection strings.
   - Search for patterns like:
     - `AKIA[0-9A-Z]{16}`
     - `ghp_[A-Za-z0-9]{36}`
     - `sk_live_[A-Za-z0-9]+`
     - `BEGIN [A-Z ]*PRIVATE KEY`
     - `aws_access_key_id|aws_secret_access_key`
     - `password\s*[:=]`
     - `token\s*[:=]`
     - `api[_-]?key\s*[:=]`
   - If sensitive values are found, remove them from the working tree before final push and tell the user exactly what was removed.

11. Final verification.
   - Confirm the repo is on GitHub.
   - Confirm the workflow is active in the Actions tab.
   - Confirm the Pages URL is live.
   - Check the README and repository About metadata are present.
   - Provide the user with the final public URL.

## Safe behavior

- Never upload or expose secrets, credentials, tokens, or private keys.
- Never commit `.env`, secret configuration, credentials, logs, or local-only deployment data.
- Stop the process immediately if a secret is found and ask the user to confirm whether to scrub it or keep it local.

## Final response to the user

After the repo is published and verified, provide a short summary with:
- repo URL
- GitHub Pages URL
- README status
- About/homepage status
- secret scan result
- any follow-up actions required
