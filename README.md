# Nx Migrate Action Demo

This repository demonstrates how to set up automated Nx migrations using the [`gridatek/nx-migrate-action`](https://github.com/gridatek/nx-migrate-action) GitHub Action with auto-merge capabilities.

## 🚀 Quick Setup Guide

Follow these steps to create your own Nx workspace with automated migrations:

### Step 1: Create a New Nx Workspace

```bash
# Create a new Nx workspace
npx create-nx-workspace@latest my-nx-workspace --preset=ts --packageManager=npm

# Navigate to your workspace
cd my-nx-workspace

# Initialize git repository
git init
git add .
git commit -m "Initial commit"
```

### Step 2: Set Up GitHub Repository

1. Create a new repository on GitHub
2. Push your local workspace to GitHub:

```bash
git remote add origin https://github.com/your-username/my-nx-workspace.git
git branch -M main
git push -u origin main
```

### Step 3: Configure Branch Protection (Recommended)

Here's how to protect your main branch using GitHub's modern **Rulesets** approach:

#### Navigate to Repository Rules
1. Go to your repository on GitHub
2. Click **Settings** → **Rules** → **Rulesets**

#### Create New Branch Ruleset
1. Click **"New ruleset"** → **"New branch ruleset"**
2. Enter a **Ruleset name** (e.g., "Main Branch Protection")
3. Set **Enforcement status** to **"Active"**

#### Configure Branch Targeting
1. Click **"Add a target"**
2. Select **"Include default branch"** (this will show as `~DEFAULT_BRANCH` in the configuration)

#### Key Protection Rules

**Require pull requests**
- ✅ Enable **"Require a pull request before merging"**
- Set **Required number of reviewers** to **0** (for solo projects) or at least 1 for team projects

**Require status checks**
- ✅ Enable **"Require status checks to pass"**
- ✅ Enable **"Require branches to be up to date before merging"**
- In the **"Additional settings"** section:
  - Type `ci` in the status check name field
  - Click the **➕ plus icon** to add it
  - This is the job name from your CI workflow in `.github/workflows/ci.yml`

**Additional protections**
- ✅ Enable **"Require conversation resolution before merging"**
- ✅ Enable **"Block force pushes"** (prevents `non_fast_forward` pushes)
- ✅ Enable **"Restrict deletions"** (prevents branch deletion)

**Merge method restrictions (within pull request rule)**
- In the pull request settings, you can specify **"allowed_merge_methods"**
- Set to **["squash"]** to match your auto-merge workflow and enforce squash-only merging

#### Finalize Ruleset
1. Click **"Create"** to activate the ruleset

#### Configure Repository Merge Methods (Optional)
After creating the ruleset, you may also want to configure allowed merge methods at the repository level:

1. Go to **Settings** → **General** → **Pull Requests**
2. Under **"Merge button"**, configure:
   - ✅ **"Allow merge commits"** (optional)
   - ✅ **"Allow squash merging"** (recommended, matches auto-merge workflow)
   - ✅ **"Allow rebase merging"** (optional)

> **Note**: At least one merge method must be enabled. If you want to enforce only squash merging, enable only "Allow squash merging" here and use the "Require a merge type" rule in your ruleset.

> **Important**: Rulesets are GitHub's modern approach to branch protection (2024+). They offer better flexibility and can layer multiple rules together. Legacy branch protection rules are still supported but rulesets are recommended for new repositories.

### Step 4: Configure Dependabot (Optional)

Dependabot automatically creates pull requests for dependency updates. While the Nx Migration Action handles Nx-specific updates, Dependabot can manage other dependencies.

Create `.github/dependabot.yml`:

```yaml
version: 2
updates:
  # Enable version updates for npm
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
    open-pull-requests-limit: 10
    reviewers:
      - "your-username"  # Replace with your GitHub username
    assignees:
      - "your-username"  # Replace with your GitHub username
    commit-message:
      prefix: "deps"
      include: "scope"
    # Ignore Nx dependencies - let nx-migrate-action handle these
    ignore:
      - dependency-name: "@nx/*"
      - dependency-name: "nx"
      - dependency-name: "@nrwl/*"
    # Group dependency updates to reduce PR noise
    groups:
      dev-dependencies:
        patterns:
          - "@types/*"
          - "*eslint*"
          - "*prettier*"
          - "*jest*"
          - "*typescript*"

  # Enable version updates for GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
```

> **Note**: This configuration completely ignores all Nx packages since the `nx-migrate-action` handles all Nx updates with proper migrations. Dependabot will only manage non-Nx dependencies.

### Step 5: Add Nx Migration Workflow

Create `.github/workflows/nx-migrate.yml`:

```yaml
name: Nx Migrate
on:
  workflow_dispatch:
  schedule:
    - cron: '0 1 * * *' # Daily at 1 AM

jobs:
  nx-migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: 'npm' # or 'yarn', 'pnpm'

      - uses: gridatek/nx-migrate-action@v0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Step 6: Add CI Workflow

Create `.github/workflows/ci.yml` (or use the one generated by Nx when creating your workspace):

```yaml
name: CI

on:
  push:
    branches:
      - main
  pull_request:

permissions:
  actions: read
  contents: read

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          filter: tree:0
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - run: npm ci --legacy-peer-deps
      - run: npx nx run-many -t lint test build typecheck
```

> **Note**: If you created your Nx workspace with `create-nx-workspace`, it may have already generated a CI workflow for you. You can use that instead of creating a new one.

### Step 7: Add Auto-merge Workflow (Optional)

Create `.github/workflows/auto-merge-dependency-prs.yml`:

```yaml
name: Auto-merge Dependency PRs

on:
  pull_request:
    types: [opened, synchronize, reopened]
  check_suite:
    types: [completed]
  status: {}

permissions:
  contents: write
  pull-requests: write
  checks: read

jobs:
  dependabot:
    name: Auto-merge Dependabot PR
    runs-on: ubuntu-latest
    if: github.event.pull_request.user.login == 'dependabot[bot]'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v2
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"

      - name: Approve patch and minor updates
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Enable auto-merge for patch and minor updates
        if: |
          steps.metadata.outputs.update-type == 'version-update:semver-patch' ||
          steps.metadata.outputs.update-type == 'version-update:semver-minor'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  nx-migrate:
    name: Auto-merge Nx Migration PR
    runs-on: ubuntu-latest
    if: |
      github.event.pull_request.user.login == 'github-actions[bot]' &&
      contains(github.event.pull_request.labels.*.name, 'nx-migrate-action')
    steps:
      - name: Auto-merge PR
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This unified workflow handles both:
- **Dependabot PRs**: Only auto-merges patch and minor updates, requires manual review for major versions
- **Nx Migration PRs**: Auto-merges after CI validation passes

> **Security Notes**:
> - Dependabot major version updates require manual approval
> - Both job types respect branch protection rules and require CI validation
> - Uses official GitHub recommended approaches for each bot type


## 🎯 How It Works

1. **Daily Check**: The migration workflow runs daily at 1 AM
2. **Version Detection**: Checks for new Nx versions
3. **Migration**: Automatically runs `nx migrate` if updates are available
4. **PR Creation**: Creates a pull request with migration changes
5. **CI Validation**: Your existing CI workflow (generated by Nx or custom) validates the changes
6. **Auto-merge**: If CI passes, the PR is automatically merged (if enabled)

## 📚 Learn More About This Setup

### Advanced Configuration

The `nx-migrate-action` supports additional configuration options:

```yaml
- uses: gridatek/nx-migrate-action@v0
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    package-manager: npm  # or yarn, pnpm
    pr-labels: nx-migrate-action,dependencies  # Custom labels
    target-branch: main  # Target branch for PRs
    dev-mode: false  # Enable for testing
```

### Migration Schedules

Choose the schedule that fits your team:

- **Daily** (`'0 1 * * *'`): Aggressive, get updates immediately
- **Weekly** (`'0 1 * * 1'`): Recommended for most teams
- **Monthly** (`'0 1 1 * *'`): Conservative approach


## 🎉 Benefits of Automated Nx Migrations

- **Stay Current**: Always running the latest Nx version with security patches
- **Reduce Manual Work**: No more remembering to check for updates
- **Team Consistency**: Everyone works with the same Nx version
- **CI Integration**: Changes are validated before merging
- **Safe Migrations**: Review process through pull requests

## 🔧 Troubleshooting

### Common Issues

1. **Migration fails**: Check the PR for migration errors and fix manually
2. **Auto-merge not working**: Ensure branch protection rules allow the bot to merge
3. **CI fails**: Migration might introduce breaking changes requiring manual fixes
4. **No CI workflow**: If Nx didn't generate a CI workflow, create one following Step 4 above

### Manual Override

You can always run migrations manually:

```bash
npx nx migrate latest
npx nx migrate --run-migrations
```

---

## 🚀 About nx-migrate-action

This demo uses the [`gridatek/nx-migrate-action`](https://github.com/gridatek/nx-migrate-action) GitHub Action.

- **GitHub Repository**: [gridatek/nx-migrate-action](https://github.com/gridatek/nx-migrate-action)
- **GitHub Marketplace**: [Nx Migration Action](https://github.com/marketplace/actions/nx-migration)
- **Issues & Support**: [Report issues](https://github.com/gridatek/nx-migrate-action/issues)

### Contributing

Found a bug or have a feature request? Please [open an issue](https://github.com/gridatek/nx-migrate-action/issues) or submit a pull request!
