# Remove `.env` From Git History and Add to `.gitignore`

Your project is already in production and `.env` got committed. Here’s how to fix it properly.

---

## 1) Stop tracking `.env` going forward (quick + safe)

> This prevents future commits of `.env`, but the old versions still exist in history.

### Update `.gitignore`
```gitignore
# never commit env files
.env
.env.*
!.env.example
```

### Stop tracking & commit
```bash
# from repo root
git rm --cached .env .env.*  # remove from index only (keeps local file)
git add .gitignore
git commit -m "Stop tracking env files and ignore them"
git push
```

### (Optional) Create a template env
```bash
cp .env .env.example
# open .env.example and replace real secrets with placeholders (e.g., YOUR_DB_PASSWORD)
git add .env.example
git commit -m "Add .env.example template"
git push
```

---

## 2) Remove leaked `.env` from *history* (recommended if secrets were pushed)

> This rewrites history to purge the files everywhere. After doing this, you **must rotate all exposed secrets**.

### Option A — `git filter-repo` (recommended)
```bash
# Install once (requires Python):
# macOS (Homebrew):
brew install git-filter-repo
# or pipx:
pipx install git-filter-repo  # (or: pip install git-filter-repo)

# From repo root (non-bare clone):
git filter-repo --path-glob ".env*" --invert-paths

# Force-push rewritten history:
git push --force --all
git push --force --tags
```

After this, **everyone must re-clone** (or reset to the new history):
```bash
# Tell collaborators:
git fetch --all
git reset --hard origin/<default-branch>
# or simply re-clone the repo
```

### Option B — BFG Repo-Cleaner (good alternative)
```bash
# Download BFG jar: https://rtyley.github.io/bfg-repo-cleaner/

# Use on a bare clone:
git clone --mirror <REMOTE_URL> repo.git
java -jar bfg.jar --delete-files ".env" --delete-files ".env.*" repo.git

cd repo.git
git push --force
```

---

## 3) Immediately rotate secrets

Because `.env` was public in history, assume it’s compromised:

- Regenerate DB passwords, API keys, OAuth secrets, JWT keys, etc.
- Update the new values on your server and in your **local** `.env` (not committed).
- Restart your services so the new env takes effect.

---

## 4) (Nice-to-have) Prevent future accidents

- Add a pre-commit hook to block `.env`:
  ```bash
  # .git/hooks/pre-commit (make executable: chmod +x .git/hooks/pre-commit)
  if git diff --cached --name-only | grep -E '^\.env(\.|$)'; then
    echo "Error: .env files are ignored and must not be committed."
    exit 1
  fi
  ```
- Use a secrets scanner in CI (GitHub secret scanning, TruffleHog, Gitleaks).
- Keep a sanitized `.env.example` up to date.

---

## TL;DR

1. Add `.env` to `.gitignore`, then:
   ```bash
   git rm --cached .env .env.*
   git commit -m "Stop tracking env files and ignore them"
   git push
   ```
2. If secrets were exposed, **rewrite history** with `git filter-repo` (or BFG), **force-push**, have collaborators re-clone, and **rotate all secrets**.
