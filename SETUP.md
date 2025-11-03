# Laravel + Vue 2 Development Flow Setup

Complete setup guide for Claude Code with OpenSpec, Superpowers, and MCPs.

## Prerequisites

- Node.js >= 20.19.0
- PHP 8.3+
- Laravel Sail (Docker)
- Claude Code

## Installation Steps

### 1. Install OpenSpec

```bash
npm install -g @fission-ai/openspec@latest
openspec --version
```

### 2. Initialize OpenSpec in Your Project

```bash
cd your-laravel-project
openspec init
```

Select **Claude Code** when prompted for AI tools.

This creates:

```
openspec/
├── project.md          # Project context and conventions
├── specs/              # Current specifications (source of truth)
├── changes/            # Active feature proposals
└── archive/            # Completed changes
```

### 3. Install Superpowers Plugin

```bash
# In Claude Code
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace

# Verify installation
/help
```

You should see:

- `/superpowers:brainstorm`
- `/superpowers:write-plan`
- `/superpowers:execute-plan`

### 4. Configure MCP Servers

#### MySQL MCP (Read-Only)

1. Clone the repository:

```bash
git clone https://github.com/hovecapital/read-only-local-mysql-mcp-server
cd read-only-local-mysql-mcp-server
npm install
```

2. Configure in Claude Code settings (`~/.claude-code/config.json`):

```json
{
  "mcpServers": {
    "mysql": {
      "command": "node",
      "args": ["/path/to/read-only-local-mysql-mcp-server/index.js"],
      "env": {
        "MYSQL_HOST": "localhost",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "your_user",
        "MYSQL_PASSWORD": "your_password",
        "MYSQL_DATABASE": "your_database"
      }
    }
  }
}
```

**For Laravel Sail:**

```json
{
  "env": {
    "MYSQL_HOST": "127.0.0.1",
    "MYSQL_PORT": "3306",
    "MYSQL_USER": "sail",
    "MYSQL_PASSWORD": "password",
    "MYSQL_DATABASE": "your_db_name"
  }
}
```

#### Context7 MCP

Claude Code includes Context7 by default. Verify with:

```bash
# In Claude Code
"Get Laravel documentation for Eloquent relationships"
```

### 5. Copy Configuration Files

```bash
# Copy CLAUDE.md to your project root
cp CLAUDE.md /path/to/your-project/

# Copy settings_local.json to Claude Code
cp settings_local.json ~/.claude-code/settings_local.json
```

### 6. Populate Project Context

After setup, run this in Claude Code:

```
Please read openspec/project.md and help me fill it out with details about my project, tech stack, and conventions
```

Example project.md:

```markdown
# Project Context

## Overview
E-commerce platform built with Laravel 11 and Vue 2.

## Tech Stack
- PHP 8.3
- Laravel 11
- Vue 2.7
- MySQL 8.0
- Redis 7.0
- Laravel Sail

## Database
- Primary: MySQL via Sail
- Cache: Redis
- Queue: Redis

## Frontend
- Vue 2.7 with Vuex
- Tailwind CSS 3.x
- Axios for API calls

## Conventions
- API resources for JSON responses
- Form requests for validation
- Service layer for business logic
- Repository pattern for complex queries
- Enums for constants (PHP 8.1+)

## Authentication
- Laravel Sanctum
- SPA authentication

## Testing
- PHPUnit for backend
- Jest for frontend
- Feature tests for API endpoints
```

## Verification

### Test OpenSpec

```bash
# Create a test proposal
/openspec:proposal Add user email verification

# Verify it was created
openspec list

# View the proposal
openspec show add-user-email-verification
```

### Test Superpowers

```bash
# Test brainstorming
/superpowers:brainstorm

# Superpowers skills activate automatically when:
# - Implementing features (TDD skill)
# - Debugging issues (systematic-debugging)
# - Planning work (writing-plans)
# - Completing tasks (verification-before-completion)
```

### Test MySQL MCP

```bash
# In Claude Code
"Show me the schema for the users table"
"What are the last 10 users created?"
"Analyze the relationship between users and posts tables"
```

### Test Context7 MCP

```bash
# In Claude Code
"Get documentation for Laravel validation rules"
"Show me examples of Laravel resource controllers"
```

## Development Workflow

### Starting a New Feature

1. **Create OpenSpec proposal:**

```bash
/openspec:proposal Add two-factor authentication
```

2. **Review and refine:**

```bash
openspec show add-two-factor-authentication
```

3. **Make changes to specs/tasks as needed:**

```
"Can you add acceptance criteria for SMS verification?"
"Add a task for testing the OTP flow"
```

4. **Validate the proposal:**

```bash
openspec validate add-two-factor-authentication
```

5. **Implement:**

```bash
/openspec:apply add-two-factor-authentication
```

6. **Archive when complete:**

```bash
/openspec:archive add-two-factor-authentication --yes
```

### During Implementation

Superpowers skills activate automatically:

- **TDD**: Guides test-first development
- **Debugging**: Systematic 4-phase debugging process
- **Verification**: Ensures work is actually complete

Manual activation:

```bash
/superpowers:write-plan    # For complex implementations
/superpowers:execute-plan  # Batch execution with checkpoints
```

### Database Inspection

Use MySQL MCP instead of manual queries:

```
"What's the schema for the orders table?"
"Show me users created in the last week"
"Analyze the database relationships for the e-commerce domain"
```

### Using Libraries

Before using unfamiliar packages:

```
"Get Context7 documentation for Laravel Sanctum"
"Show me Laravel Horizon queue examples"
```

## Common Commands

### Sail

```bash
./vendor/bin/sail up -d
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan test
./vendor/bin/sail composer require package/name
```

### OpenSpec

```bash
openspec list                    # View active changes
openspec view                    # Interactive dashboard
openspec show <change>           # View change details
openspec validate <change>       # Check spec formatting
openspec archive <change> --yes  # Archive completed change
```

### Superpowers

```bash
/superpowers:brainstorm    # Design refinement
/superpowers:write-plan    # Implementation planning
/superpowers:execute-plan  # Batch execution
```

## Troubleshooting

### OpenSpec commands not found

```bash
# Restart Claude Code after installation
# Or manually reload:
/reload
```

### MySQL MCP connection issues

```bash
# Verify Sail MySQL is running
./vendor/bin/sail ps

# Check port forwarding
./vendor/bin/sail artisan tinker
# DB::connection()->getPdo();

# Update MCP config with correct Sail credentials
```

### Superpowers not activating

```bash
# Check installation
/plugin list

# Reinstall if needed
/plugin update superpowers
```

### Context7 not responding

```bash
# Context7 is built-in, ensure you're using correct syntax:
"Get documentation for Laravel [package-name]"

# Not: "context7 get docs"
```

## Best Practices

1. **Always start with OpenSpec** for features and bug fixes
2. **Let Superpowers guide** TDD and debugging automatically
3. **Use MySQL MCP** for database inspection instead of manual queries
4. **Use Context7** before implementing unfamiliar packages
5. **Follow CLAUDE.md** guidelines strictly
6. **Keep specs updated** by archiving completed changes
7. **Test meaningfully**, not for coverage numbers
8. **Commit regularly** with conventional commit messages

## Next Steps

1. Read through `CLAUDE.md` for complete coding guidelines
2. Create your first OpenSpec proposal
3. Let Superpowers guide your implementation
4. Use MCPs for database and documentation queries

## Support

- OpenSpec: <https://github.com/fission-ai/openspec>
- Superpowers: <https://github.com/obra/superpowers>
- MySQL MCP: <https://github.com/hovecapital/read-only-local-mysql-mcp-server>
- Context7: Built into Claude Code
