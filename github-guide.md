# CGLab GitHub Organization Guide

**Comparative Genomics Lab · IMBB-FORTH**
**Organization:** [github.com/cgenomicslab](https://github.com/cgenomicslab)

This guide covers how we organize code in the lab, how to bring existing projects into the organization, and the daily git workflow everyone should follow.

---

## The core principle

**All lab code lives in the organization.** When you create a repo for a project, paper, pipeline, or analysis — create it under `cgenomicslab`, not your personal account. This ensures code stays in the lab when people move on, while your contributions remain permanently visible on your personal GitHub profile.

Your personal GitHub account is for personal projects, learning experiments, or tools so uniquely tied to you that they'll follow you across institutions. Everything else belongs in the org.

---

## Organization structure

### Repository types and naming

We use consistent naming to keep things findable:

| Type | Naming pattern | Examples |
|------|---------------|----------|
| Published tools | `toolname` | `orthoscape`, `trna-evolution` |
| Paper repos | `YYYY-lastname-shorttopic` | `2025-papadak-olfactory-receptors` |
| Shared pipelines | `pipeline-description` | `pipeline-rnaseq`, `pipeline-structure-prediction` |
| Lab infrastructure | `descriptive-name` | `cgenomicslab.github.io`, `lab-docs`, `uniprot-refproteomes` |
| Course/teaching | `course-name-year` | `bioinfo-intro-2025` |

### Teams and permissions

We use GitHub Teams to manage access. The org base permission is **Read** (everyone can see everything). Specific teams grant write access:

| Team | Access | Members |
|------|--------|---------|
| `owners` | Admin (all repos) | PI + one senior member |
| `core-dev` | Write (all active repos) | Senior postdocs, lab manager |
| `members` | Write (assigned repos) | Current students and postdocs |
| `alumni` | Read (all repos) | Former lab members |

---

## Setting up your environment

### 1. Link your email to GitHub

This is **critical** — your commits only appear on your GitHub profile if the email in your git config matches one registered on your GitHub account.

```bash
# Check your current git email
git config --global user.email

# Set it to the email linked to your GitHub account
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

Go to [github.com/settings/emails](https://github.com/settings/emails) and confirm the same email is listed there (you can add multiple).

### 2. Set up SSH authentication (recommended)

```bash
# Generate an SSH key (if you don't have one)
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start the SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy the public key
cat ~/.ssh/id_ed25519.pub
```

Go to [github.com/settings/keys](https://github.com/settings/keys), click "New SSH key", and paste it.

Test the connection:
```bash
ssh -T git@github.com
# Should say: Hi username! You've successfully authenticated...
```

### 3. Enable two-factor authentication

This is strongly recommended for all org members. Go to [github.com/settings/security](https://github.com/settings/security) and enable 2FA using an authenticator app.

---

## Daily workflow: the branch-and-PR model

We use a simple branching workflow. Nobody pushes directly to `main`. Instead, work happens on feature branches that get merged via pull requests.

### Starting new work on an existing repo

```bash
# Clone the repo (first time only)
git clone git@github.com:cgenomicslab/repo-name.git
cd repo-name

# Make sure you're up to date
git checkout main
git pull origin main

# Create a feature branch
git checkout -b your-initials/short-description
# Examples:
#   git checkout -b ap/add-uapa-analysis
#   git checkout -b mk/fix-alignment-bug
#   git checkout -b nk/update-documentation
```

### Making commits

```bash
# Stage your changes
git add script.py data/results.csv

# Or stage everything (be careful — check what's changed first)
git status          # Always check first!
git add .

# Commit with a clear message
git commit -m "Add UapA binding site prediction using Boltz-2"

# Push your branch to GitHub
git push origin your-initials/short-description
```

**Commit message conventions:**
- Start with a verb: Add, Fix, Update, Remove, Refactor
- Be specific: `Fix off-by-one error in exon boundary detection` not `fix bug`
- Keep the first line under 72 characters

### Opening a pull request

After pushing your branch, go to the repo on GitHub. You'll see a prompt to open a pull request. Click it and:

1. Write a brief description of what changed and why
2. Assign a reviewer (anyone from the team)
3. Link any related issues

Or from the command line using GitHub CLI:
```bash
# Install gh if needed: https://cli.github.com/
gh pr create --title "Add UapA binding site analysis" --body "Implements Boltz-2 predictions for UapA transporter"
```

### Reviewing and merging

Once approved, the author merges their own PR (using "Squash and merge" for clean history or "Create a merge commit" for detailed history). Delete the branch after merging.

### Keeping your branch up to date

If `main` has moved ahead while you were working:

```bash
git checkout main
git pull origin main
git checkout your-initials/short-description
git merge main
# Resolve any conflicts, then continue working
```

---

## Bringing existing code into the organization

This is the most common scenario: a lab member has been developing code on their personal GitHub account and it needs to move to the org.

### Option A: Transfer the repository (preferred)

This preserves everything — stars, issues, commit history, and creates an automatic URL redirect from the old location.

**The member does this themselves:**

1. Go to `github.com/your-username/repo-name`
2. Click **Settings** → scroll to **Danger Zone** → **Transfer repository**
3. Type `cgenomicslab` as the new owner
4. Confirm the transfer

After transfer:
- The repo lives at `github.com/cgenomicslab/repo-name`
- The old URL (`github.com/your-username/repo-name`) automatically redirects
- All commit history, issues, and PRs are preserved
- The member retains their contribution credit

**Important:** Only repo admins (typically the creator) can initiate a transfer. An org owner must accept it.

Update your local clone to point to the new location:
```bash
cd repo-name
git remote set-url origin git@github.com:cgenomicslab/repo-name.git

# Verify
git remote -v
```

### Option B: Mirror the repository

Use this when the member wants to keep their original repo as well (e.g., a personal tool that they want backed up in the org).

**Step 1: Create an empty repo in the org**

Go to `github.com/organizations/cgenomicslab/repositories/new`, create a new repo with the desired name. Do NOT initialize with README/license/gitignore.

**Step 2: Push a mirror**

```bash
# Clone the personal repo as a bare mirror
git clone --bare git@github.com:personal-username/repo-name.git
cd repo-name.git

# Push to the org
git push --mirror git@github.com:cgenomicslab/repo-name.git

# Clean up
cd ..
rm -rf repo-name.git
```

**Step 3: Clone the org version for future work**

```bash
git clone git@github.com:cgenomicslab/repo-name.git
```

All future development happens in the org repo. The personal repo becomes a read-only archive or fork.

### Option C: Start fresh in the org, import history

If the code has been in a local directory (not on GitHub):

```bash
# Navigate to your project directory
cd my-project

# Initialize git (if not already)
git init
git add .
git commit -m "Initial commit: import existing analysis code"

# Create the repo on GitHub (using gh CLI)
gh repo create cgenomicslab/repo-name --private --source=. --remote=origin

# Or manually: create the repo on github.com/organizations/cgenomicslab/repositories/new
# Then:
git remote add origin git@github.com:cgenomicslab/repo-name.git
git branch -M main
git push -u origin main
```

---

## Creating a new repo from scratch

### From the command line

```bash
mkdir my-new-project
cd my-new-project
git init

# Create essential files
touch README.md LICENSE CITATION.cff .gitignore

# Create the repo on GitHub under the org
gh repo create cgenomicslab/my-new-project --private --source=. --remote=origin

# Make your first commit
git add .
git commit -m "Initial project setup"
git push -u origin main
```

### From the GitHub web interface

1. Go to [github.com/organizations/cgenomicslab/repositories/new](https://github.com/organizations/cgenomicslab/repositories/new)
2. Name your repo following our conventions
3. Set visibility (private for work-in-progress, public when ready)
4. Add a README, .gitignore (Python or R), and LICENSE (MIT)
5. Clone it locally:

```bash
git clone git@github.com:cgenomicslab/my-new-project.git
```

---

## Essential files every repo should have

### README.md (minimum template)

```markdown
# Project Name

Brief one-sentence description of what this does.

## Installation

How to install dependencies and set up.

## Usage

Minimal example of how to run it.

## Citation

If you use this software, please cite:
> Author et al. (Year). Title. Journal. DOI

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributors

- Name (role/affiliation) — what they contributed
```

### .gitignore

For a bioinformatics project, start with:

```
# Python
__pycache__/
*.py[cod]
*.egg-info/
.eggs/
dist/
build/
*.egg
.venv/
env/

# R
.Rhistory
.RData
.Rproj.user/

# Jupyter
.ipynb_checkpoints/

# Data (don't commit large files)
*.bam
*.bai
*.sam
*.fastq
*.fastq.gz
*.fasta.gz
*.vcf
*.vcf.gz
*.bed.gz
*.h5
*.hdf5

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Results (optional — sometimes you want to track these)
# results/
# figures/
```

### LICENSE

We use the **MIT License** by default. When creating a repo on GitHub, select "MIT License" from the dropdown. This maximizes reuse and is standard in bioinformatics.

```
MIT License

Copyright (c) 2024 Comparative Genomics Lab, IMBB-FORTH

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction...
```

### CITATION.cff

For repos associated with a publication:

```yaml
cff-version: 1.2.0
message: "If you use this software, please cite the paper below."
title: "Your Tool Name"
version: 1.0.0
date-released: 2025-06-01
url: "https://github.com/cgenomicslab/your-tool"
authors:
  - family-names: Lastname
    given-names: Firstname
    orcid: "https://orcid.org/0000-0000-0000-0000"
    affiliation: "IMBB-FORTH"
preferred-citation:
  type: article
  title: "Your Paper Title"
  journal: "Journal Name"
  doi: "10.xxxx/xxxxx"
  year: 2025
  authors:
    - family-names: Lastname
      given-names: Firstname
```

GitHub will display a "Cite this repository" button in the sidebar automatically.

---

## Working with large files

Genomic data files (FASTA, BAM, VCF, etc.) should **never** be committed to git. Instead:

- Store raw data on the lab server (`/data/` or shared storage)
- Use relative paths in scripts and document the expected data location in the README
- For small reference files needed for reproducibility, consider [Git LFS](https://git-lfs.github.com/)
- For archival/publication, deposit data in repositories like Zenodo, Figshare, or domain-specific databases (SRA, ENA)

```bash
# If you accidentally committed a large file
git rm --cached large_file.bam
echo "large_file.bam" >> .gitignore
git add .gitignore
git commit -m "Remove large file, add to gitignore"
```

---

## Public vs. private repos

**Start private, go public when ready.** A repo should become public when:

- The associated paper is submitted or published
- The tool is ready for external users
- You want to share it with collaborators outside the org

Benefits of public repos on the free plan:
- Branch protection rules (not available for private repos on free plan)
- GitHub Pages (our website is public for this reason)
- Visibility for your career — public contributions appear on your profile

To change visibility: repo **Settings** → **Danger Zone** → **Change visibility**.

---

## When you leave the lab

When a member departs (graduation, new position, etc.), we follow this offboarding checklist:

1. **Merge or close** all your open pull requests
2. **Verify** your commit email is linked to your GitHub account (so contributions stay on your profile)
3. **Transfer** any personal repos that contain lab work to the org
4. **Update** CONTRIBUTORS.md in your repos with your affiliation dates
5. **Fork** any org repos you want a personal copy of (public repos can be forked anytime)

Your GitHub membership will be moved from `members` to `alumni` (read access). All your commits, authorship, and contributions remain permanently visible.

---

## Quick reference: common git commands

```bash
# Daily workflow
git status                          # What's changed?
git diff                            # See exact changes
git add file.py                     # Stage a specific file
git add .                           # Stage everything
git commit -m "message"             # Commit staged changes
git push                            # Push to GitHub
git pull                            # Get latest from GitHub

# Branches
git checkout -b name                # Create + switch to new branch
git checkout main                   # Switch to main
git branch                          # List branches
git branch -d name                  # Delete branch (after merge)

# Undo mistakes
git checkout -- file.py             # Discard unstaged changes to a file
git reset HEAD file.py              # Unstage a file
git stash                           # Temporarily shelve changes
git stash pop                       # Restore shelved changes

# History
git log --oneline -10               # Last 10 commits, compact
git log --all --graph --oneline     # Visual branch history
git blame file.py                   # Who changed each line

# Remote
git remote -v                       # Show remote URLs
git remote set-url origin URL       # Change remote URL
git fetch                           # Download remote changes (don't merge)
```

---

## Useful tools

- **[GitHub CLI (gh)](https://cli.github.com/):** Create repos, PRs, issues from the terminal
- **[Zenodo](https://zenodo.org/):** Mint DOIs for releases (integrates directly with GitHub)
- **[cffinit](https://citation-file-format.github.io/cff-initializer-javascript/):** Generate CITATION.cff files
- **[gitignore.io](https://www.toptal.com/developers/gitignore):** Generate .gitignore for any stack
- **[pre-commit](https://pre-commit.com/):** Automated checks before each commit (linting, formatting)

---

## Getting help

If you're stuck or unsure:

1. Check `git status` — it usually tells you what to do next
2. Ask in the lab Slack/chat — someone has hit the same issue
3. Read the error message carefully — git errors are verbose but informative
4. The [Pro Git book](https://git-scm.com/book/en/v2) (free) is an excellent reference

**Golden rule:** Never use `git push --force` on `main`. If you think you need to force-push, ask first.
