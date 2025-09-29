# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Nx workspace demo repository specifically designed to demonstrate the `gridatek/nx-migrate-action` GitHub Action. The project uses Nx 21.5.3 with TypeScript and focuses on automated Nx migrations via GitHub Actions.

## Key Commands

### Development Commands
```bash
# Build all projects
npx nx run-many -t build

# Type check all projects
npx nx run-many -t typecheck

# Lint all projects
npx nx run-many -t lint

# Test all projects
npx nx run-many -t test

# Run all CI tasks (lint, test, build, typecheck)
npx nx run-many -t lint test build typecheck

# Format code with Prettier
npx prettier . --write

# Sync TypeScript project references
npx nx sync

# Check if TypeScript project references are in sync
npx nx sync:check
```

### Nx Project Management
```bash
# Generate a new library
npx nx g @nx/js:lib packages/pkg1 --publishable --importPath=@my-org/pkg1

# Build specific project
npx nx build <project-name>

# Run any target on a project
npx nx <target> <project-name>

# View project graph
npx nx graph

# List available plugins
npx nx list

# Show all projects
npx nx show projects

# Release packages
npx nx release

# Release packages (dry run)
npx nx release --dry-run
```

## Architecture

### Workspace Structure
- **Monorepo Setup**: Uses Nx workspaces with package-based structure
- **TypeScript Configuration**: Strict TypeScript setup with project references
- **Package Management**: NPM workspaces in `packages/*` directory (currently empty)
- **Build System**: Uses SWC for compilation via `@nx/js/typescript` plugin

### Key Configuration Files
- `nx.json`: Nx workspace configuration with TypeScript plugin setup
- `tsconfig.base.json`: Base TypeScript configuration with strict settings
- `package.json`: Root package with workspaces configuration for `packages/*`
- `.prettierrc`: Code formatting with single quotes

### GitHub Actions Integration
- **CI Pipeline** (`.github/workflows/ci.yml`): Runs on push/PR with lint, test, build, typecheck using Node.js 20
- **Nx Migration** (`.github/workflows/nx-migrate.yml`): Daily automated Nx migrations using `gridatek/nx-migrate-action@v0` with Node.js 22
- **Auto-merge** (`.github/workflows/auto-merge-dependency-prs.yml`): Unified workflow for auto-merging both Dependabot and Nx migration PRs
- **Dependabot** (`.github/dependabot.yml`): Weekly dependency updates excluding Nx packages (handled by nx-migrate-action)

### Branch Protection Rules (Rulesets)
Here's how to protect your main branch using GitHub's modern Rulesets approach:

**Navigate to Repository Rules**
1. Go to your repository on GitHub
2. Click Settings > Rules > Rulesets

**Create New Branch Ruleset**
1. Click "New ruleset" > "New branch ruleset"
2. Enter a Ruleset name (e.g., "Main Branch Protection")
3. Set Enforcement status to "Active"

**Configure Branch Targeting**
1. Click "Add a target"
2. Select "Include default branch" or specify `main` as the branch pattern

**Key Protection Rules**
- **Require pull requests before merging**
  - Enable "Require a pull request before merging"
  - Set required number of reviewers (recommended: at least 1)
  - Enable "Dismiss stale reviews when new commits are pushed"

- **Require status checks to pass**
  - Enable "Require status checks to pass"
  - Enable "Require branches to be up to date before merging"
  - Add required status checks: `ci` (job name from CI workflow)

- **Additional protections**
  - Enable "Require conversation resolution before merging"
  - Enable "Block force pushes"
  - Configure "Restrict pushes that create files" if needed

**Rulesets Benefits**: Modern approach (2024+) with better flexibility, multiple rules can layer together, and improved organization-level management.

### Development Environment
- **VSCode Extensions**: Nx Console and Prettier recommended
- **Node.js**: Version 20 for CI, Version 22 for Nx migrations
- **Package Manager**: NPM with legacy peer deps flag
- **Code Style**: Prettier with single quotes, consistent formatting

### Automated Dependency Management
- **Nx Updates**: Handled by `gridatek/nx-migrate-action` with proper migrations
- **Other Dependencies**: Managed by Dependabot (patch/minor auto-merge, major requires review)
- **Conflict Prevention**: Dependabot ignores `@nx/*`, `nx`, and `@nrwl/*` packages
- **Auto-merge Strategy**: Separate jobs for different bot types with CI validation

## Important Notes

- This is a demo repository with minimal actual code - the main purpose is demonstrating the Nx migration action
- The `packages/` directory is currently empty but ready for library generation
- TypeScript project references are automatically maintained by Nx
- CI runs `npx nx fix-ci` to provide recommendations for failures
- The workspace is connected to Nx Cloud (ID: 68d7b76076755e5707bbb011)
- Auto-merge workflows respect branch protection rules and require CI validation