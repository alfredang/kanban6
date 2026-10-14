---
description: Security-scan the project, then push it to GitHub with a README, About section and GitHub Pages deploy
argument-hint: "[owner/repo or repo URL] [public|private] [branch]"
allowed-tools: Bash, PowerShell, Read, Edit, Write, Grep, Glob, mcp__playwright__browser_resize, mcp__playwright__browser_navigate, mcp__playwright__browser_take_screenshot, mcp__playwright__browser_close
---

# Publish this project to GitHub

Repo details from the user: `$ARGUMENTS`

Work through the steps below in order. **Step 1 is a gate: nothing leaves this machine until the scan passes or the user explicitly accepts each finding.** Follow every constraint in `CLAUDE.md` while editing files.

## 0. Resolve the target

1. Parse `$ARGUMENTS` for:
   - **Repo**: `owner/name`, `https://github.com/owner/name(.git)` or `git@github.com:owner/name.git`.
   - **Visibility**: `public` or `private`. Default is `public`, because GitHub Pages on a free plan needs a public repo.
   - **Branch**: default `main`.
2. If no repo was given, check `git remote get-url origin`. If there's no remote either, ask the user for the repo details and stop until they answer.
3. Check the tooling: `gh --version` and `gh auth status`. If `gh` is missing or not logged in, tell the user to run `gh auth login` and stop.
4. Check whether the repo exists: `gh repo view <owner/name> --json name,visibility,url,homepageUrl,description`. If it exists, say so. If it doesn't, you will create it in step 4.

## 1. Security scan (gate)

Scan **every file that would be pushed**: tracked files plus untracked files that aren't ignored (`git ls-files -co --exclude-standard`). Also scan **git history** (`git log -p --all`), because a secret deleted in a later commit is still published.

1. **Run a dedicated scanner if one is installed.** Use `gitleaks detect --source . --log-opts="--all" --redact -v` if available, or `trufflehog git file://. --no-update`. Don't install anything without asking.
2. **Always run a pattern scan as well** with Grep over the files to be pushed. Look for:
   - Private keys: `-----BEGIN (RSA |EC |OPENSSH |DSA |PGP )?PRIVATE KEY-----`
   - Cloud and SaaS tokens: `AKIA[0-9A-Z]{16}`, `ghp_[A-Za-z0-9]{36}`, `github_pat_[A-Za-z0-9_]{20,}`, `gh[ousr]_[A-Za-z0-9]{36}`, `xox[baprs]-[A-Za-z0-9-]+`, `sk-[A-Za-z0-9_-]{20,}`, `sk-ant-[A-Za-z0-9_-]{20,}`, `AIza[0-9A-Za-z_-]{35}`, `SG\.[A-Za-z0-9_-]{22}\.[A-Za-z0-9_-]{43}`, `-----BEGIN CERTIFICATE-----`
   - Generic assignments: `(?i)(api[_-]?key|secret|token|passw(or)?d|client[_-]?secret|auth)["'\s]*[:=]\s*["'][^"']{8,}["']`
   - Connection strings with credentials: `[a-z]+://[^/\s:@]+:[^/\s@]+@`
   - JWTs: `eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}`
3. **Check for sensitive files** by name: `.env*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `*.keystore`, `credentials*`, `secrets*`, `*.sqlite`, `*.db`, `.claude/settings.local.json`, and any large binaries. Each one must be removed or listed in `.gitignore`. Create or update `.gitignore` if needed, and include `.claude/settings.local.json`.
4. **Check for personal data and project rules:**
   - Email addresses, phone numbers and internal hostnames or IPs. The `FORMSUBMIT_ENDPOINT` address in `index.html` is intentional, but **it becomes public**. Show it to the user and confirm before pushing.
   - The constraint check from `CLAUDE.md`: `grep -nE 'localStorage|sessionStorage|indexedDB|document\.cookie|alert\(|confirm\(|!important|https?://' index.html`. Any match other than the `FORMSUBMIT_ENDPOINT` line is a finding.
   - Branding: no real bank logos, trademarks or imitation of real bank systems. Only the neutral "IT PMO" wordmark (see `CLAUDE.md`).
   - The JS syntax check from `CLAUDE.md`, if Node is available.
5. **Report the results** as a short table: file:line, rule, severity, and a redacted snippet. **Never print a full secret value.**
   - **Real secret found in the working tree:** stop. Tell the user to remove it **and rotate it**. Don't push.
   - **Real secret found only in history:** stop. Explain that the history must be rewritten (for example with `git filter-repo`) and the secret rotated. Don't rewrite history yourself without explicit approval.
   - **False positives or intentional public values:** list them and get the user's explicit OK before continuing.
   - **Clean:** say "Security scan passed" and continue.

## 2. Create or edit the README

Read `index.html`, `CLAUDE.md` and the existing `README.md`. Then update `README.md`, keeping what's still accurate and editing it rather than rewriting it from scratch. It should contain:

- The title and a one-line description, plus a **Live demo** link to `https://<owner>.github.io/<repo>/` (use the custom domain if one is set).
- Features (columns, drag and drop plus the keyboard "Move ▸" control, Add Task form with validation, filters, summary, overdue badges).
- How to run it: double-click `index.html`. No build and no server.
- A note that there's no persistence on purpose, so a refresh resets the board.
- FormSubmit email notifications: where to change the address and the one-time activation email.
- Tech: vanilla HTML/CSS/JS in a single file, with no dependencies.
- Deployment: GitHub Pages through `.github/workflows/pages.yml`.
- A short disclaimer that this is a fictitious demo and isn't affiliated with any real bank.

- A screenshot directly under the Live demo link: `![IT PMO Kanban board showing the Backlog, In Progress, Blocked and Done columns](docs/screenshot.png)`. Step 7 captures or refreshes the image.

Don't add badges, other images or claims you can't verify. Don't put any email address in the README.

## 3. Create or update the GitHub Pages workflow

Make sure `.github/workflows/pages.yml` exists and does the following:
- Triggers on `push` to the target branch and on `workflow_dispatch`.
- Sets `permissions: contents: read, pages: write, id-token: write` and `concurrency: group: pages`.
- Uses `actions/checkout@v4`, `actions/configure-pages@v5`, `actions/upload-pages-artifact@v3` and `actions/deploy-pages@v4`.
- Copies **only the site files** (`index.html`, plus any other assets the app needs) into `_site`, so that `CLAUDE.md`, `.claude/` and other repo files aren't published to the site.

If the file already matches, leave it alone. If the branch differs from the one in the file, update the trigger.

## 4. Commit and push

1. If the repo doesn't exist, create it: `gh repo create <owner/name> --<visibility> --source . --remote origin`. Don't push yet.
   If `origin` points somewhere other than the target, ask before running `git remote set-url origin https://github.com/<owner/name>.git`.
2. Run `git status` and show the user what will be committed. Stage files by name rather than with `git add -A`, and never stage anything that step 1 flagged.
3. Commit with a clear message such as `Update README and Pages deploy`. End the message with the attribution line from the current system instructions, if there is one.
4. If you're on a branch other than the target, switch or ask first. Push with `git push -u origin <branch>`. **Never force-push.** If the push is rejected, explain why and ask what to do.

## 5. Set the repo About section

1. Write a description of 350 characters or less from the README, for example: "Single-file IT PMO Kanban board for demos and training: vanilla HTML/CSS/JS, drag and drop, filters, no build step."
2. Run `gh repo edit <owner/name> --description "<desc>" --homepage "https://<owner>.github.io/<repo>/"`.
3. Add topics such as `kanban`, `project-management`, `vanilla-javascript`, `html`, `css`, `github-pages` and `demo` with `--add-topic` (one flag per topic). Keep any existing topics.

## 6. Enable Pages and verify the deploy

1. Enable Pages with GitHub Actions as the source:
   `gh api -X POST repos/<owner/name>/pages -f build_type=workflow`.
   If it already exists (409 or 422), run `gh api -X PUT repos/<owner/name>/pages -f build_type=workflow` instead.
2. Find the run: `gh run list --workflow pages.yml --limit 1`. Then wait for it with `gh run watch <id> --exit-status`.
   If the run started before Pages was enabled and failed, re-run it with `gh workflow run pages.yml --ref <branch>` and watch again.
3. Confirm the site URL with `gh api repos/<owner/name>/pages --jq .html_url`.
4. If the run fails, show the failing step with `gh run view <id> --log-failed`, fix the problem if it's in the workflow, and ask before pushing again.

## 7. Capture the README screenshot

Run this once the deploy in step 6 has succeeded, so the screenshot shows the live site.

1. With the Playwright MCP tools, resize the browser to 1440×900 (`browser_resize`), open the live Pages URL (`browser_navigate`), and save a viewport screenshot to `docs/screenshot.png` (`browser_take_screenshot` with `scale: "css"`). Then close the browser (`browser_close`).
   If the Playwright MCP tools aren't available, tell the user and skip this step.
2. Look at the image. It should show the header, summary strip, filter bar and all four columns with seed cards. Don't open the Add Task modal or change the board first. A 404 for `favicon.ico` in the console is harmless.
3. Make sure the README has the screenshot line from step 2, and that `.playwright-mcp/` is in `.gitignore` (Playwright writes logs and snapshots there). Delete the `.playwright-mcp/` folder.
4. Stage `docs/screenshot.png` (plus `README.md` and `.gitignore` if they changed) by name, commit with a message such as `Update README screenshot`, and push as in step 4. **Never force-push.** If the image is byte-identical to the committed one, skip the commit.

The screenshot isn't copied into `_site`, so it lives only in the repo for the README.

## 8. Report

Finish with a short summary:
- The security scan result, including any findings the user accepted.
- The repo URL, its visibility, and the commit pushed.
- What changed in the README and the workflow (or "unchanged").
- The About description and topics.
- The live Pages URL and whether the deploy succeeded.
- Whether the screenshot was captured and pushed (or why it was skipped).
- Any manual follow-ups, such as clicking the FormSubmit activation email or rotating a key.
