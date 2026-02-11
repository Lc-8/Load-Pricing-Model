# GitHub Bootstrap — Fix “repository is empty” / create main branch

If Codex (or Replit Git) says:
> “This repository is empty. Create a default branch (e.g. main) by pushing an initial commit, then retry.”

It means GitHub does not have a branch yet. You must push **one commit**.

## Path A (easiest): create repo with README on GitHub
1. GitHub → **New repository**
2. Check **“Add a README”**
3. Create repo  
✅ GitHub creates `main` automatically

Then connect Replit to that repo.

## Path B: create repo empty, push from Replit
1. GitHub → **New repository** (leave “Add README” unchecked)
2. Copy the repo HTTPS URL
3. In Replit:
   - Open **Version Control** (Git icon)
   - Click **Initialize repository** (if needed)
   - Ensure default branch name is `main`:
     - Replit shell:
       ```bash
       git branch -M main
       ```
   - Add files, commit:
     ```bash
     git add .
     git commit -m "Initial commit"
     ```
   - Add remote + push:
     ```bash
     git remote add origin <YOUR_GITHUB_REPO_URL>
     git push -u origin main
     ```
✅ Repo now has a default branch

## Common gotchas
- If you get auth errors: use GitHub token (not password).
- If remote already exists: `git remote -v` then `git remote set-url origin <url>`
- If branch mismatch: `git branch` and `git status` to confirm you’re on `main`.
