# Laravel + Vue 2 Development Flow for Claude Code

Spec-driven development workflow combining OpenSpec, Superpowers, and MCPs for predictable, high-quality Laravel applications.

## Quick Start

```bash
# Install OpenSpec
npm install -g @fission-ai/openspec@latest

# Initialize in your project
cd your-laravel-project
openspec init

# Install Superpowers in Claude Code
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# Copy configuration
cp settings_local.json ~/.claude-code/settings_local.json
cp CLAUDE.md /path/to/your-project/

# Start building
/openspec:proposal <your-feature-name>
```

## What You Get

✅ **Spec-driven workflow** - Agree on requirements before coding  
✅ **Automatic TDD** - Test-first development built-in  
✅ **Systematic debugging** - 4-phase root cause process  
✅ **Database inspection** - Natural language MySQL queries  
✅ **Current documentation** - Up-to-date library docs  
✅ **Strict PHP 8.3+** - Type safety, enums, readonly properties  
✅ **Safe permissions** - Work efficiently without constant prompts  

## The Files

| File | Purpose | When to Use |
|------|---------|-------------|
| **SETUP.md** | Installation guide | Once per project setup |
| **CLAUDE.md** | Coding guidelines | Reference during development |
| **QUICKREF.md** | Daily commands | Keep open while coding |
| **SUMMARY.md** | How it all works | Understanding the flow |
| **settings_local.json** | Claude Code config | Copy to `~/.claude-code/` |

## The Workflow

### 1. Propose

```bash
/openspec:proposal Add user authentication
```

Creates structured proposal with specs and tasks.

### 2. Refine

```bash
openspec show add-user-authentication
```

Review and iterate until specs are clear.

### 3. Implement

```bash
/openspec:apply add-user-authentication
```

Superpowers guides TDD automatically. MySQL MCP inspects database. Context7 provides docs.

### 4. Archive

```bash
/openspec:archive add-user-authentication --yes
```

Merges approved specs, archives change.

## Core Technologies

### OpenSpec

Spec-driven development - agree on requirements before code.

- Proposals capture intent
- Tasks break down work
- Spec deltas show changes
- Archive maintains history

### Superpowers

Proven workflows activate automatically:

- **TDD** - Red-green-refactor cycle
- **Debugging** - Systematic root cause analysis
- **Verification** - Ensure work is complete
- **Planning** - Break down complex features

### MySQL MCP (Read-Only)

Natural language database inspection:

```
"What's the users table schema?"
"Show recent orders with totals"
"Analyze the post-comment relationship"
```

### Context7 MCP

Up-to-date library documentation:

```
"Get Laravel Sanctum docs"
"Show validation rule examples"
```

## Development Standards

### PHP 8.3+

```php
<?php

declare(strict_types=1);

// Enums
enum Status: string {
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

// Readonly properties
final class UserData
{
    public function __construct(
        public readonly string $email,
        public readonly Status $status,
    ) {}
}

// Match expressions
$message = match($status) {
    Status::ACTIVE => 'User is active',
    Status::INACTIVE => 'User is inactive',
};
```

### Laravel Architecture

```
Controllers → Services → Repositories → Models
     ↓            ↓
Form Requests  DTOs/Enums
```

- **Controllers:** Thin, delegate to services
- **Services:** Business logic
- **Repositories:** Complex queries
- **Form Requests:** Validation
- **DTOs:** Structured data
- **Enums:** Constants

### Testing

```php
// Feature test - test what matters
public function test_user_can_register(): void
{
    $response = $this->postJson('/api/register', [
        'email' => 'user@test.com',
        'password' => 'password123',
    ]);

    $response->assertCreated();
    $this->assertDatabaseHas('users', [
        'email' => 'user@test.com',
    ]);
}
```

Focus on meaningful tests, not coverage percentages.

## Example Session

```
Developer: Create a feature for user profile editing

Claude: I'll create an OpenSpec proposal for user profile editing.
        /openspec:proposal add-profile-editing
        
        ✓ Created openspec/changes/add-profile-editing/
        ✓ proposal.md - Intent and approach
        ✓ tasks.md - Implementation checklist  
        ✓ specs/users/spec.md - Profile spec delta
        
        Ready to review. Use: openspec show add-profile-editing

Developer: Looks good, let's implement

Claude: Starting implementation for add-profile-editing.
        Superpowers TDD skill activated.
        
        [Checks database schema via MySQL MCP]
        "What's the users table schema?"
        
        [Gets documentation via Context7]
        "Get Laravel validation rules for file uploads"
        
        [Implements following CLAUDE.md standards]
        ✓ Task 1.1: Create UpdateProfileRequest
        ✓ Task 1.2: Add ProfileService
        ✓ Task 1.3: Create ProfileController
        ✓ Task 2.1: Write feature tests
        ✓ Task 2.2: Create Vue profile component
        
        All tasks complete. Tests passing.

Developer: Archive the change

Claude: /openspec:archive add-profile-editing --yes
        
        ✓ Specs merged to openspec/specs/users/
        ✓ Change archived
        ✓ Ready for next feature
```

## Safety Features

### Allowed Operations

✅ Sail commands (`./vendor/bin/sail artisan`, `sail composer`)  
✅ Git operations (status, diff, commit)  
✅ File viewing (cat, ls, grep)  
✅ Package management (composer, npm)  
✅ Database inspection (via MCP, read-only)  
✅ Code quality tools (phpstan, php-cs-fixer)  

### Blocked Operations

❌ System-wide deletions (`rm -rf /`)  
❌ Destructive disk ops (`dd`, `mkfs`)  
❌ Piped execution (`curl | bash`)  
❌ Environment access (`.env` files)  
❌ Process killing (`killall`)  

## Pre-Completion Checklist

Before marking work done:

- [ ] `declare(strict_types=1)` in all PHP files
- [ ] PHPDoc blocks on classes and public methods
- [ ] Type hints on all parameters and returns
- [ ] Form requests for validation
- [ ] Tests written for business logic
- [ ] No N+1 queries
- [ ] OpenSpec change archived
- [ ] Superpowers verification passed

## Common Commands

### Daily Workflow

```bash
/openspec:proposal <feature>      # Start feature
openspec show <feature>            # Review specs
/openspec:apply <feature>          # Implement
/openspec:archive <feature> --yes  # Complete
```

### Superpowers

```bash
/superpowers:brainstorm    # Design refinement
/superpowers:write-plan    # Create plan
/superpowers:execute-plan  # Batch execution
```

### Laravel/Sail

```bash
./vendor/bin/sail up -d
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan test
./vendor/bin/sail composer require package/name
```

## Documentation

- **[SETUP.md](SETUP.md)** - Complete installation and configuration
- **[CLAUDE.md](CLAUDE.md)** - Full coding guidelines and patterns
- **[QUICKREF.md](QUICKREF.md)** - Daily command reference
- **[SUMMARY.md](SUMMARY.md)** - How everything works together

## Coming Soon

**TypeScript Version** - Same workflow for Node.js/TypeScript projects:

- Next.js/React patterns
- TypeScript strict mode
- Node.js MCPs
- Prisma/Drizzle patterns

## Philosophy

> Write specs before code.  
> Follow proven workflows.  
> Enforce quality systematically.

This setup embodies:

- **Deterministic** over unpredictable
- **Systematic** over ad-hoc  
- **Typed** over dynamic
- **Tested** over hoped
- **Documented** over assumed

## Resources

- [OpenSpec](https://github.com/fission-ai/openspec)
- [Superpowers](https://github.com/obra/superpowers)
- [MySQL MCP](https://github.com/hovecapital/read-only-local-mysql-mcp-server)
- [Laravel Docs](https://laravel.com/docs)
- [Vue 2 Docs](https://v2.vuejs.org)

## License

MIT

---

**Ready to build?** Start with [SETUP.md](SETUP.md) for installation, then create your first feature with `/openspec:proposal`.
