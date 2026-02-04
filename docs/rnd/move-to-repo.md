# Moving vadocs to a Dedicated Repository

## Step 1: Create the new repository

```bash
mkdir ~/projects/vadocs
cd ~/projects/vadocs
git init
```

## Step 2: Copy package files

```bash
# Copy the package contents (not the parent directory)
cp -r /path/to/ai_engineering_book/packages/vadocs/* ~/projects/vadocs/

# Remove the local .venv (you'll create a new one)
rm -rf ~/projects/vadocs/.venv
```

## Step 3: Initialize the new repository

```bash
cd ~/projects/vadocs

# Create .gitignore
cat > .gitignore << 'EOF'
__pycache__/
*.py[cod]
.venv/
dist/
*.egg-info/
.pytest_cache/
.coverage
EOF

# Initialize uv and install dependencies
uv sync
uv pip install -e ".[dev]"

# Verify tests pass
uv run pytest tests/ -v
```

```bash
<<<<<<<< HEAD:docs/rnd/move-to-repo.md
uv run pytest tests/ --cov=. --cov-report=term-missing -q
========
env -u VIRTUAL_ENV uv run pytest tests/ --cov=. --cov-report=term-missing -q
>>>>>>>> dc28782 (feat: Complete proof of concept for vadocs v0.1.0):docs/move-to-repo.md
```

## Step 4: Create initial commit

```bash
git add .
git commit -m "feat: Initial vadocs v0.1.0 package

Documentation validation engine with YAML frontmatter sync.

Features:
- FrontmatterValidator for generic YAML validation
- AdrValidator for ADR-specific validation
- AdrTermValidator for MyST term references
- AdrFixer and SyncFixer for auto-fixes
- load_config() for YAML config loading

79 tests passing."
```

## Step 5: Push to GitHub

```bash
# Option A: Using GitHub CLI
gh repo create vadocs --public --source=. --push

# Option B: Manual
git remote add origin git@github.com:YOUR_USERNAME/vadocs.git
git push -u origin main
```

<<<<<<<< HEAD:docs/rnd/move-to-repo.md

```bash
$ sudo dnf install gh

gh auth login
? Where do you use GitHub? GitHub.com
? What is your preferred protocol for Git operations on this host? SSH
? Upload your SSH public key to your GitHub account? /home/username/.ssh/id_github.pub
? Title for your SSH key: gh_cli
? How would you like to authenticate GitHub CLI? Paste an authentication token
Tip: you can generate a Personal Access Token here https://github.com/settings/tokens
The minimum required scopes are 'repo', 'read:org', 'admin:public_key'.
? Paste your authentication token: ****************************************
- gh config set -h github.com git_protocol ssh
✓ Configured git protocol
✓ SSH key already existed on your GitHub account: /home/username/.ssh/id_github.pub
✓ Logged in as username
```
========
<!-- #region -->
```bash
$ sudo dnf install gh

gh auth login
? Where do you use GitHub? GitHub.com
? What is your preferred protocol for Git operations on this host? SSH
? Upload your SSH public key to your GitHub account? /home/username/.ssh/id_github.pub
? Title for your SSH key: gh_cli
? How would you like to authenticate GitHub CLI? Paste an authentication token
Tip: you can generate a Personal Access Token here https://github.com/settings/tokens
The minimum required scopes are 'repo', 'read:org', 'admin:public_key'.
? Paste your authentication token: ****************************************
- gh config set -h github.com git_protocol ssh
✓ Configured git protocol
✓ SSH key already existed on your GitHub account: /home/username/.ssh/id_github.pub
✓ Logged in as username
```
<!-- #endregion -->

## Step 6: Test installation in target project


Conduct [PoC validation](/docs/poc.ipynb).

Passed.

>>>>>>>> dc28782 (feat: Complete proof of concept for vadocs v0.1.0):docs/move-to-repo.md


## Step 6: Test installation in target project


Conduct [PoC validation](/docs/poc.ipynb).

Passed.


## Step 7: Clean up original location

```bash
rm -rf /path/to/ai_engineering_book/packages/vadocs
```

## Optional: Tag a release

```bash
cd ~/projects/vadocs
git tag v0.1.0
git push origin v0.1.0
```

Then install specific version:

```bash
uv add "vadocs @ git+https://github.com/YOUR_USERNAME/vadocs.git@v0.1.0"
```
