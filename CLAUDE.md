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
- **CI Pipeline** (`.github/workflows/ci.yml`): Runs on push/PR with lint, test, build, typecheck
- **Nx Migration** (`.github/workflows/nx-migrate.yml`): Daily automated Nx migrations using `gridatek/nx-migrate-action@v0`
- **Auto-merge**: Configured for automatic PR merging of Nx migration PRs

### Branch Protection Rules
Here's how to protect your main branch using GitHub Actions:

**Navigate to Settings**
1. Go to your repository on GitHub
2. Click Settings > Branches

**Add Branch Protection Rule**
1. Click "Add branch protection rule"
2. Enter `main` (or your default branch name) in the branch name pattern

**Key Protection Settings**
- **Require pull requests before merging**
  - Check "Require a pull request before merging"
  - Set required number of reviewers (recommended: at least 1)
  - Enable "Dismiss stale reviews when new commits are pushed"

- **Require status checks to pass**
  - Check "Require status checks to pass before merging"
  - Check "Require branches to be up to date before merging"
  - Add your GitHub Actions workflow jobs as required status checks

- **Additional protections**
  - Check "Require conversation resolution before merging"
  - Check "Restrict pushes that create files"
  - Check "Do not allow bypassing the above settings" (removes admin override)

### Development Environment
- **VSCode Extensions**: Nx Console and Prettier recommended
- **Node.js**: Version 20 specified in CI
- **Package Manager**: NPM with legacy peer deps flag
- **Code Style**: Prettier with single quotes, consistent formatting

## Important Notes

- This is a demo repository with minimal actual code - the main purpose is demonstrating the Nx migration action
- The `packages/` directory is currently empty but ready for library generation
- TypeScript project references are automatically maintained by Nx
- CI runs `npx nx fix-ci` to provide recommendations for failures
- The workspace is connected to Nx Cloud (ID: 68d7b76076755e5707bbb011)