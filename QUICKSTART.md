# 5-Minute Quick Start

Get up and running with the Laravel development flow immediately.

## Prerequisites Check

```bash
# Verify you have these installed
node --version    # Need >= 20.19.0
php --version     # Need >= 8.3
docker --version  # For Laravel Sail
```

## 1. Install OpenSpec (2 minutes)

```bash
npm install -g @fission-ai/openspec@latest
cd /path/to/your-laravel-project
openspec init
```

When prompted:

- Select **Claude Code**
- Answer project context questions

## 2. Install Superpowers (1 minute)

Open Claude Code:

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

## 3. Copy Configuration (30 seconds)

```bash
# From downloaded files to your project
cp CLAUDE.md /path/to/your-project/

# From downloaded files to Claude Code config
cp settings_local.json ~/.claude-code/settings_local.json
```

## 4. Configure MySQL MCP (1 minute)

Edit `~/.claude-code/config.json`:

```json
{
  "mcpServers": {
    "mysql": {
      "command": "npx",
      "args": ["-y", "@hovecapital/read-only-local-mysql-mcp"],
      "env": {
        "MYSQL_HOST": "127.0.0.1",
        "MYSQL_PORT": "3306",
        "MYSQL_USER": "sail",
        "MYSQL_PASSWORD": "password",
        "MYSQL_DATABASE": "your_database_name"
      }
    }
  }
}
```

Replace `your_database_name` with your actual database name.

## 5. Restart Claude Code (30 seconds)

Restart Claude Code to load new configuration.

## Verify It's Working

### Test OpenSpec

```bash
/openspec:proposal test-feature
openspec list
```

You should see `test-feature` in the list.

### Test Superpowers

```bash
/help
```

You should see:

- `/superpowers:brainstorm`
- `/superpowers:write-plan`
- `/superpowers:execute-plan`

### Test MySQL MCP

In Claude Code:

```
"What tables exist in the database?"
```

Should return your database schema.

### Test Context7

In Claude Code:

```
"Get Laravel validation documentation"
```

Should return current Laravel validation docs.

## Your First Feature (5 minutes)

Now build a real feature:

```bash
# 1. Create proposal
/openspec:proposal Add user profile editing

# 2. Review (Claude creates specs and tasks)
openspec show add-user-profile-editing

# 3. Refine if needed
"Can you add a task for avatar upload?"

# 4. Implement (Superpowers guides TDD automatically)
/openspec:apply add-user-profile-editing

# 5. Archive when done
/openspec:archive add-user-profile-editing --yes
```

## Common Issues

### "openspec: command not found"

```bash
npm install -g @fission-ai/openspec@latest
# Make sure npm global bin is in PATH
```

### MySQL MCP not connecting

```bash
# Check Sail is running
./vendor/bin/sail ps

# Verify database name
./vendor/bin/sail artisan tinker
# DB::connection()->getDatabaseName();

# Update config.json with correct name
```

### Superpowers commands not showing

```bash
# Restart Claude Code completely
# Commands load at startup

# Or try reinstall
/plugin update superpowers
```

### Permission denied errors

```bash
# Verify settings_local.json is in right place
ls -la ~/.claude-code/settings_local.json

# Restart Claude Code after copying
```

## What You Just Got

✅ **Spec-driven workflow** - Clear requirements before code  
✅ **Automatic TDD** - Tests written first  
✅ **Database inspection** - Natural language queries  
✅ **Current docs** - Up-to-date library info  
✅ **Strict typing** - PHP 8.3+ standards  
✅ **Smart permissions** - Work without spam  

## Daily Workflow

Every feature follows this pattern:

```bash
# Start
/openspec:proposal <feature-name>

# Review & refine
openspec show <feature-name>

# Implement (Superpowers guides you)
/openspec:apply <feature-name>

# Complete
/openspec:archive <feature-name> --yes
```

## Example Commands

### Database Questions

```
"Show me the users table schema"
"What are the most recent orders?"
"Explain the relationship between posts and comments"
```

### Documentation Queries

```
"Get Laravel Sanctum authentication docs"
"Show me validation rule examples"
"How do I use Laravel queues?"
```

### Code Patterns

```
"Create a service for managing user subscriptions"
"Add a form request for updating profiles"
"Create a repository for complex product queries"
```

## Cheat Sheet

Keep these commands handy:

```bash
# OpenSpec
/openspec:proposal <name>      # Start feature
openspec list                  # View all changes
openspec show <name>           # Review details
/openspec:apply <name>         # Implement
/openspec:archive <name> --yes # Complete

# Superpowers
/superpowers:brainstorm        # Design help
/superpowers:write-plan        # Plan complex work
/superpowers:execute-plan      # Execute in batches

# Sail
./vendor/bin/sail up -d        # Start
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan test
./vendor/bin/sail down         # Stop
```

## Learn More

- **Full setup:** [SETUP.md](SETUP.md)
- **Coding standards:** [CLAUDE.md](CLAUDE.md)
- **Daily reference:** [QUICKREF.md](QUICKREF.md)
- **How it works:** [SUMMARY.md](SUMMARY.md)
- **What changed:** [CHANGES.md](CHANGES.md)

## Support

If something isn't working:

1. Check the issue in this guide first
2. Read [SETUP.md](SETUP.md) troubleshooting section
3. Verify all prerequisites are installed
4. Restart Claude Code completely
5. Check Claude Code logs for errors

## Next Steps

1. ✅ Setup complete? Create your first real feature
2. Read [CLAUDE.md](CLAUDE.md) for coding standards
3. Bookmark [QUICKREF.md](QUICKREF.md) for daily use
4. Explore Superpowers skills as they activate
5. Ask Claude questions via MySQL and Context7 MCPs

---

**You're ready!** Start with `/openspec:proposal your-feature-name` and let the workflow guide you.
