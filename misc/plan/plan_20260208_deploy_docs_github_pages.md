# Plan: Deploy vadocs Documentation to GitHub Pages

## Context

The vadocs project has complete documentation (Developer Guide, API Reference, ADRs) built with MyST Markdown, but no CI/CD or deployment pipeline exists. ADR-26022 mandates GitHub Pages via `actions/deploy-pages` for all public repos. Both `vadocs` and `ai_engineering_book` repos will be transferred to the `soviar-systems` GitHub organization before deploying.

## Step 1: Transfer Repos to `soviar-systems` (Manual — User)

These are GitHub UI steps the user performs:
1. Go to each repo's **Settings > General > Danger Zone > Transfer repository**
2. Transfer `lefthand67/vadocs` → `soviar-systems/vadocs`
3. Transfer `lefthand67/ai_engineering_book` → `soviar-systems/ai_engineering_book`

GitHub automatically creates redirects from old URLs.

## Step 2: Update References in vadocs

Three files reference `lefthand67` and need updating to `soviar-systems`:

| File | Change |
|------|--------|
| `myst.yml:9` | `github: https://github.com/lefthand67/vadocs` → `github: https://github.com/soviar-systems/vadocs` |
| `CLAUDE.md:7` | `github.com/lefthand67/ai_engineering_book` → `github.com/soviar-systems/ai_engineering_book` |
| `README.md:94` | `github.com/lefthand67/ai_engineering_handbook` → `github.com/soviar-systems/ai_engineering_book` (also fixes repo name typo) |

## Step 3: Create GitHub Actions Workflow

**New file:** `.github/workflows/deploy.yml`

### Workflow Design

**Triggers:**
- `push` to `main` — full validate + build + deploy
- `pull_request` to `main` — validate + build only (catches broken docs pre-merge)
- `workflow_dispatch` — manual re-deploy

**Job 1: `validate`** (all triggers)
1. Checkout code
2. Install `uv` (`astral-sh/setup-uv@v3`, cache enabled)
3. `uv sync --frozen`
4. `uv run pytest`

**Job 2: `build-deploy`** (depends on `validate`)
1. Checkout code
2. `actions/configure-pages@v3`
3. Setup Node.js 20
4. `npm install -g mystmd`
5. `myst build --html` → output to `_build/html/`
6. `actions/upload-pages-artifact@v3` — upload `_build/html/`
7. `actions/deploy-pages@v4` — deploy (**only on `main`**, guarded by `if:`)

**Key configuration:**
- `BASE_URL: /${{ github.event.repository.name }}` — correct asset paths for `soviar-systems.github.io/vadocs`
- Permissions: `contents: read`, `pages: write`, `id-token: write`
- Concurrency group: `'pages'`, `cancel-in-progress: false`
- Environment: `github-pages`

## Step 4: Configure GitHub Pages (Manual — User)

After the workflow merges to `main`:
1. Go to `soviar-systems/vadocs` **Settings > Pages**
2. Set **Source** to **GitHub Actions**

## Step 5: Write GitHub Pages Setup Instruction

Save a reusable instruction to `ai_engineering_book/misc/plan/plan_20260208_github_pages_setup_instruction.md` covering:
- How to configure GitHub Pages with GitHub Actions as the source
- The repo Settings > Pages UI steps
- Required workflow permissions (`pages: write`, `id-token: write`)
- The `actions/deploy-pages@v4` pattern
- How to verify deployment

## Files Summary

| File | Action |
|------|--------|
| `.github/workflows/deploy.yml` | **Create** |
| `myst.yml` | **Edit** — update github URL |
| `CLAUDE.md` | **Edit** — update parent project link |
| `README.md` | **Edit** — update parent project link |
| `../ai_engineering_book/misc/plan/plan_20260208_github_pages_setup_instruction.md` | **Create** — reusable GH Pages setup guide |

## Verification

1. User transfers repos to `soviar-systems`
2. Update references and create workflow, commit
3. Push PR to `main` — verify `validate` + `build-deploy` jobs pass (deploy skipped)
4. Merge to `main` — verify full pipeline including deploy
5. Configure Pages source in repo settings
6. Verify site at `https://soviar-systems.github.io/vadocs/`
