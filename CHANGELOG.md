# Changelog

## [1.0.2](https://github.com/hovecapital/php-code-style/compare/v1.0.1...v1.0.2) (2025-11-04)


### Bug Fixes

* formatting for permissions ([b380015](https://github.com/hovecapital/php-code-style/commit/b380015b5fe6eea71b81225f18c30d478549c726))

## [1.0.1](https://github.com/hovecapital/php-code-style/compare/v1.0.0...v1.0.1) (2025-11-03)


### Bug Fixes

* claude permissions local ([a585f06](https://github.com/hovecapital/php-code-style/commit/a585f06945a9d2f574099f0f0cc8d9ab3b5e886c))

## 1.0.0 (2025-11-03)


### Bug Fixes

* setup release please, update claude md ([17b2543](https://github.com/hovecapital/php-code-style/commit/17b2543e936c652d89dae004a9220ba25941b208))

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
