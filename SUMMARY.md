# Laravel Development Flow - Summary

## What We've Created

A comprehensive development environment for Laravel + Vue 2 projects using Claude Code with:

1. **Spec-driven development** (OpenSpec)
2. **Systematic workflows** (Superpowers)
3. **Database inspection** (MySQL MCP)
4. **Library documentation** (Context7 MCP)
5. **Strict PHP standards** (Type safety, PSR-12)

## The Files

### 1. CLAUDE.md

**Purpose:** Complete coding guidelines and standards

**Key sections:**

- Core principles (concise, strict typing, minimal comments, PHPDoc)
- OpenSpec workflow integration
- Superpowers skill activation
- MCP server usage
- PHP 8.3+ patterns (enums, readonly, match expressions)
- Laravel architecture (controllers, services, repositories, DTOs)
- Vue 2 component patterns
- Testing approach (meaningful over coverage)
- Error handling
- Database optimization
- Security practices

**Usage:** Reference for all development decisions

### 2. settings_local.json

**Purpose:** Claude Code permissions and configuration

**Key features:**

- Safe bash permissions (prevents destructive commands)
- All Sail/Artisan commands allowed
- Git operations enabled
- File operations scoped safely
- MCP server definitions
- Required plugins list
- Telemetry disabled

**Usage:** Copy to `~/.claude-code/settings_local.json`

### 3. SETUP.md

**Purpose:** Complete installation and configuration guide

**Covers:**

- OpenSpec installation and initialization
- Superpowers plugin setup
- MySQL MCP configuration (with Sail specifics)
- Context7 verification
- Project context population
- Testing all components
- Troubleshooting common issues

**Usage:** Follow once per project for initial setup

### 4. QUICKREF.md

**Purpose:** Daily development quick reference

**Contains:**

- Common commands (OpenSpec, Superpowers, Sail)
- Code pattern templates
- Testing examples
- Git workflow
- Pre-completion checklist
- File structure reference

**Usage:** Keep open during development

## How It All Works Together

### The Complete Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Developer: "Add user email verification feature"        │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Claude Code: /openspec:proposal add-email-verification  │
│    Creates: openspec/changes/add-email-verification/       │
│    - proposal.md (intent, approach)                         │
│    - tasks.md (implementation checklist)                    │
│    - specs/ (delta showing changes)                         │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Review & Refine                                          │
│    Developer: "Add acceptance criteria for email tokens"   │
│    Claude updates specs and tasks                          │
│    openspec validate add-email-verification                │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Implementation: /openspec:apply add-email-verification  │
│                                                              │
│    Superpowers TDD skill activates:                         │
│    - Write test first                                       │
│    - Implement feature                                      │
│    - Verify passing                                         │
│                                                              │
│    MySQL MCP for database:                                  │
│    Claude: "What's the users table schema?"                │
│                                                              │
│    Context7 for docs:                                       │
│    Claude: "Get Laravel notification examples"             │
│                                                              │
│    CLAUDE.md guides:                                        │
│    - Strict typing                                          │
│    - Service layer pattern                                  │
│    - Form request validation                                │
│    - Repository for queries                                 │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Verification                                             │
│    Superpowers verification-before-completion activates:    │
│    ✓ Tests passing                                          │
│    ✓ Types declared                                         │
│    ✓ PHPDoc present                                         │
│    ✓ No N+1 queries                                         │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Archive: /openspec:archive add-email-verification --yes │
│    Merges approved specs into openspec/specs/              │
│    Moves change to openspec/archive/                        │
│    Ready for next feature                                   │
└─────────────────────────────────────────────────────────────┘
```

## Key Benefits

### 1. Spec-Driven Development (OpenSpec)

- **Before:** Vague requirements in chat history
- **After:** Clear specs agreed before coding
- **Result:** Predictable, reviewable outputs

### 2. Systematic Workflows (Superpowers)

- **Before:** Ad-hoc debugging and implementation
- **After:** Proven TDD and debugging patterns
- **Result:** Consistent quality, fewer bugs

### 3. Database Inspection (MySQL MCP)

- **Before:** Manual queries, PHPMyAdmin, or guessing
- **After:** Natural language database questions
- **Result:** Faster schema understanding, better queries

### 4. Library Documentation (Context7)

- **Before:** Outdated Stack Overflow, guessing API
- **After:** Current, accurate package docs
- **Result:** Correct implementation first try

### 5. Strict PHP Standards (CLAUDE.md)

- **Before:** Inconsistent typing, mixed patterns
- **After:** PHP 8.3+ features, strict types everywhere
- **Result:** Fewer runtime errors, better IDE support

## Permission Strategy

### What's Allowed (Safe Operations)

- All Sail/Artisan commands
- Git operations (status, diff, log, add, commit)
- File viewing (cat, ls, grep, find)
- File creation/copying (controlled patterns)
- Package management (composer, npm)
- Database inspection via MCP (read-only)
- OpenSpec commands
- Code quality tools (phpstan, php-cs-fixer)

### What's Denied (Destructive Operations)

- System-wide deletions (rm -rf /)
- Disk operations (dd, mkfs)
- Piped script execution (curl | bash)
- Permission chaos (chmod 777 /)
- Process killing (killall, pkill -9)
- Environment file access (.env)

### Result

Claude can work efficiently without constant permission prompts while staying safe.

## Conflict Resolutions Applied

1. **Comments:** Minimal inline + PHPDoc for classes/methods
2. **Framework:** Laravel-only (no Symfony)
3. **Testing:** Flexible, meaningful tests (no coverage mandates)
4. **Style:** Concise language, no emojis
5. **Architecture:** Simple Laravel patterns (not DDD/Hexagonal)

## Migration Path

### From Current Setup

If you have an existing Laravel project:

1. Install OpenSpec globally: `npm install -g @fission-ai/openspec@latest`
2. Run `openspec init` in your project
3. Install Superpowers plugin in Claude Code
4. Configure MySQL MCP with your Sail credentials
5. Copy settings_local.json to `~/.claude-code/`
6. Copy CLAUDE.md to project root
7. Populate `openspec/project.md` with your project details

### For New Projects

1. Create Laravel project with Sail
2. Follow SETUP.md completely
3. Start first feature with `/openspec:proposal`

## Next Steps

1. **Read SETUP.md** - Complete installation
2. **Skim CLAUDE.md** - Understand standards
3. **Bookmark QUICKREF.md** - Daily reference
4. **Create first proposal** - Try the workflow
5. **Let Superpowers guide** - Trust the process

## TypeScript Version

Coming next! Will follow same pattern:

- OpenSpec for spec-driven development
- Superpowers for workflows
- Node.js-specific MCPs
- TypeScript strict mode guidelines
- Next.js/React patterns

## Support Resources

- **OpenSpec:** <https://github.com/fission-ai/openspec>
- **Superpowers:** <https://github.com/obra/superpowers>
- **MySQL MCP:** <https://github.com/hovecapital/read-only-local-mysql-mcp-server>
- **Laravel:** <https://laravel.com/docs>
- **Vue 2:** <https://v2.vuejs.org>

## Philosophy

> "Agree on what to build before writing code.
> Follow proven workflows automatically.
> Access knowledge when you need it.
> Enforce quality systematically."

This setup embodies:

- **Deterministic over unpredictable**
- **Systematic over ad-hoc**
- **Typed over dynamic**
- **Tested over hoped**
- **Documented over assumed**
