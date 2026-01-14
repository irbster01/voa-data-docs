# How to Set Up Public Documentation Site

## Step 1: Create Public Repo on GitHub

1. Go to https://github.com/new
2. Repository name: `voa-data-docs`
3. Description: "Documentation for VOA data systems and reporting"
4. **Make it PUBLIC**
5. Don't initialize with README (we'll push our own files)
6. Click "Create repository"

## Step 2: Copy Your Docs to the New Repo

In your terminal (from `c:\dev\active\fab-cicd\docs`):

```powershell
# Create a new folder for the public repo
cd c:\dev\active
mkdir voa-data-docs
cd voa-data-docs

# Initialize git
git init
git branch -M main

# Copy your docs (do this manually or via PowerShell)
# Copy these files from c:\dev\active\fab-cicd\docs:
# - index.md (the home page)
# - CIS-Data-Reference.md
# - _config.yml
# - Any other docs you want public

# Add and commit
git add .
git commit -m "Initial documentation"

# Connect to GitHub (replace with your actual repo URL)
git remote add origin https://github.com/irbster01/voa-data-docs.git
git push -u origin main
```

## Step 3: Enable GitHub Pages

1. Go to your new repo on GitHub: https://github.com/irbster01/voa-data-docs
2. Click **Settings** (top right)
3. Click **Pages** (left sidebar)
4. Under "Build and deployment":
   - Source: **Deploy from a branch**
   - Branch: **main** 
   - Folder: **/ (root)**
5. Click **Save**

## Step 4: Wait 2-3 Minutes

GitHub will build your site. Refresh the Pages settings page to see the URL.

Your docs will be live at: **https://irbster01.github.io/voa-data-docs/**

## Step 5: Share the Link

Send that URL to your team. No login needed - they just click and read!

---

## How to Update Docs Later

When you update docs in your private repo:

1. Copy the updated markdown file to your public `voa-data-docs` folder
2. Commit and push:
   ```powershell
   cd c:\dev\active\voa-data-docs
   git add .
   git commit -m "Updated CIS documentation"
   git push
   ```
3. GitHub Pages automatically rebuilds (takes ~2 minutes)

---

## What Files to Copy

**Copy these from your private repo:**
- `docs/index.md` (the home page - GitHub Pages needs this filename)
- `docs/CIS-Data-Reference.md`
- `docs/Lighthouse-Attendance-Pipeline.md`
- `docs/_config.yml` (GitHub Pages theme config)
- Any future documentation you create

**DO NOT copy:**
- Code files (.py, .pq, .sql)
- Dataflow definitions
- Credentials or connection strings
- Any actual data files

---

## Need Help?

If something doesn't work:
1. Check that the repo is PUBLIC
2. Check that Pages is enabled in Settings → Pages
3. Check that files are in the root of the repo (not in a subfolder)
4. Wait a few minutes after pushing - GitHub needs time to build

---

*This file can be deleted from the public repo - it's just setup instructions*
