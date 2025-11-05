# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a DDEV addon that integrates the mittwald hosting platform with DDEV local development environments. The addon enables:
- Synchronizing DDEV configuration with mittwald cloud projects
- Pulling/pushing code and databases from/to mittwald cloud
- Running the mittwald CLI within DDEV web containers

## Technology Stack

- **Language**: Bash/Shell scripting
- **Testing**: Bats (Bash Automated Testing System)
- **CI/CD**: GitHub Actions
- **Dependencies**: DDEV >= v1.24.3, mittwald CLI (@mittwald/cli), jq

## Key Commands

### Development
```bash
# Install addon locally for testing
ddev add-on get /path/to/this/repo

# Install from registry
ddev add-on get mittwald/ddev-mittwald

# Pull from mittwald cloud
ddev pull mittwald

# Push to mittwald cloud (dangerous)
ddev push mittwald

# Run mittwald CLI in web container
ddev mw [command]
```

### Testing
```bash
# Run all tests
bats tests/test.bats

# Tests require these environment variables:
# - MITTWALD_API_TOKEN
# - MITTWALD_APP_INSTALLATION_ID
# - MITTWALD_SSH_USER
# - MITTWALD_SSH_PRIVATE_KEY
```

## Architecture

### DDEV Addon Structure
The addon follows DDEV's standard addon pattern with these key components:

1. **install.yaml**: Defines addon metadata, installation workflow, and interactive prompts for credentials
   - Pre-install: Prompts for MITTWALD_API_TOKEN (global) and MITTWALD_APP_INSTALLATION_ID (project)
   - Post-install: Renders mittwald-specific config using `mw ddev render-config`
   - Copies files to `.ddev/` directories

2. **providers/mittwald.yaml**: Implements DDEV's provider system for pull/push operations
   - `auth_command`: Establishes SSH connection and sets mittwald context
   - `db_pull_command`: Downloads database from mittwald cloud
   - `files_import_command`: Downloads files using `mw app download`
   - `db_push_command`: Uploads database (marked dangerous)
   - `files_push_command`: Uploads files using `mw app upload` (marked dangerous)

3. **commands/web/mw**: Custom DDEV command wrapper for mittwald CLI
   - Installed globally to allow `ddev mw [args]` from any project

4. **web-build/**: Docker integration
   - **Dockerfile.mittwald**: Extends DDEV web container, installs mittwald CLI via npm
   - **mw-util**: Utility script for writing rsync filter files (excludes wp-config.php, typo3temp, etc.)

### Configuration Management
- Sensitive credentials stored as DDEV environment variables
- MITTWALD_API_TOKEN: Global config (shared across projects)
- MITTWALD_APP_INSTALLATION_ID: Project config (per-project)
- Interactive prompts during installation if not set
- Supports DDEV_NONINTERACTIVE mode for CI environments

### File Organization
```
.ddev/
├── commands/web/mw           # mittwald CLI wrapper command
├── providers/mittwald.yaml   # Pull/push provider configuration
└── web-build/
    ├── Dockerfile.mittwald   # Docker extension for CLI
    └── mw-util               # rsync filter utility
```

## Testing Infrastructure

Tests use Bats framework and follow this pattern:
- Create temporary test project in ~/tmp/test-mittwald
- Configure DDEV with mittwald environment variables
- Test installation, pull operations, and CLI functionality
- Clean up after each test (delete project, remove directory)

Test tags:
- `@test "install from release"` has `# bats test_tags=release` and is skipped in PR pipelines

CI pipeline (`tests.yml`):
- Runs on PRs, pushes to main, daily schedule (8:25 AM UTC)
- Tests against DDEV stable and HEAD versions
- Uses `ddev/github-action-add-on-test@v1`
- Requires GitHub secrets for mittwald credentials

## Development Guidelines

### Scripting Conventions
- Use `/usr/bin/env bash` shebang for portability (not `/bin/bash`)
- All files should contain `#ddev-generated` comment for `ddev get` replacement tracking
- Use `#ddev-description:` comments for install action descriptions
- Use `#ddev-nodisplay` to suppress output in install actions

### Version Constraints
- Specify `ddev_version_constraint` in install.yaml for required DDEV features
- Current constraint: `>= v1.24.3`

### Dangerous Operations
- Mark destructive operations (push/upload) with `#ddev-dangerous` in provider config
- Include warnings in documentation and user prompts

### Testing Best Practices
- Always export `DDEV_NON_INTERACTIVE=true` in test setup
- Use `set -eu -o pipefail` for strict error handling
- Clean up test projects in teardown even on failure
- Skip release tests in PR pipelines to avoid version conflicts

## Common Tasks

### Adding New mittwald CLI Commands
Just use `ddev mw [command]` - no additional wrapper needed.

### Modifying Pull/Push Behavior
Edit `providers/mittwald.yaml` to change auth, pull, or push commands.

### Updating rsync Filters
Modify `web-build/mw-util` to add/remove excluded files/directories.

### Testing Changes Locally
1. Make changes to addon files
2. Install from local directory: `ddev add-on get /path/to/ddev-mittwald`
3. Restart DDEV: `ddev restart`
4. Test pull/push operations or run `ddev mw` commands

### Updating mittwald CLI Version
The CLI is installed from npm in `web-build/Dockerfile.mittwald`. To update:
1. Modify the Dockerfile to specify version: `npm install -g @mittwald/cli@x.y.z`
2. Rebuild web container: `ddev restart`
