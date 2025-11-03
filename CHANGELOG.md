# Changelog

## [1.0.0] - Setup

Initial release with release-please automation configured.

### Release-Please Configuration

This repository now uses [release-please](https://github.com/googleapis/release-please) to automate releases and changelog generation.

**How it works:**

1. Use conventional commit messages:
   - `feat:` - New features (minor version bump: 1.0.0 → 1.1.0)
   - `fix:` - Bug fixes (patch version bump: 1.0.0 → 1.0.1)
   - `feat!:` or `fix!:` - Breaking changes (major version bump: 1.0.0 → 2.0.0)
   - `docs:`, `chore:`, `style:`, `refactor:`, `test:`, `build:`, `ci:` - Included in changelog

2. Release-please will automatically:
   - Create/update a Release PR with version bump and changelog
   - Update this CHANGELOG.md file
   - Create GitHub releases when the Release PR is merged

**Examples:**
```bash
git commit -m "feat: add new coding guidelines section"
git commit -m "fix: correct typo in PHP standards"
git commit -m "docs: update Vue 2 component examples"
git commit -m "feat!: restructure project architecture guidelines

BREAKING-CHANGE: Complete rewrite of architecture section"
```

Future releases will be automatically documented below this section.
