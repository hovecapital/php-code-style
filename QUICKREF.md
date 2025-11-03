# Laravel Development Quick Reference

## Daily Workflow

### 1. Start Feature

```bash
/openspec:proposal <feature-name>
```

### 2. Review & Refine

```bash
openspec show <feature-name>
openspec validate <feature-name>
```

### 3. Implement

```bash
/openspec:apply <feature-name>
```

### 4. Complete

```bash
/openspec:archive <feature-name> --yes
```

## Superpowers Commands

```bash
/superpowers:brainstorm     # Interactive design
/superpowers:write-plan     # Create plan
/superpowers:execute-plan   # Execute in batches
```

**Auto-activated skills:**

- TDD when implementing
- Debugging when fixing issues
- Verification before completion

## Database Queries (MySQL MCP)

Instead of manual queries, ask Claude:

```
"Show me the users table schema"
"What are the recent orders?"
"Analyze relationships between posts and comments"
```

## Documentation (Context7)

```
"Get Laravel Sanctum documentation"
"Show me Laravel validation examples"
"Context7 docs for Inertia.js"
```

## Laravel Commands

```bash
# Sail prefix
./vendor/bin/sail up -d
./vendor/bin/sail down

# Artisan
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan migrate:fresh --seed
./vendor/bin/sail artisan make:model Post -mfsc
./vendor/bin/sail artisan make:controller PostController --resource
./vendor/bin/sail artisan make:request StorePostRequest
./vendor/bin/sail artisan test
./vendor/bin/sail artisan test --filter=PostTest

# Composer
./vendor/bin/sail composer require package/name
./vendor/bin/sail composer update

# Tinker
./vendor/bin/sail artisan tinker
```

## Code Patterns

### Model with Types

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

/**
 * @property int $id
 * @property string $title
 * @property int $user_id
 */
final class Post extends Model
{
    protected $fillable = ['title', 'content', 'user_id'];

    protected function casts(): array
    {
        return [
            'published_at' => 'datetime',
        ];
    }

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

### Controller (Thin)

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Http\Requests\StorePostRequest;
use App\Services\PostService;

final class PostController extends Controller
{
    public function __construct(
        private readonly PostService $postService,
    ) {}

    public function store(StorePostRequest $request)
    {
        $post = $this->postService->create(
            $request->validated()
        );

        return response()->json($post, 201);
    }
}
```

### Service Layer

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\Post;
use App\Repositories\PostRepository;

final class PostService
{
    public function __construct(
        private readonly PostRepository $repository,
    ) {}

    public function create(array $data): Post
    {
        return $this->repository->create([
            'title' => $data['title'],
            'content' => $data['content'],
            'user_id' => auth()->id(),
        ]);
    }
}
```

### Form Request

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

final class StorePostRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'content' => ['required', 'string'],
        ];
    }
}
```

### Enum

```php
<?php

declare(strict_types=1);

namespace App\Enums;

enum PostStatus: string
{
    case DRAFT = 'draft';
    case PUBLISHED = 'published';
    case ARCHIVED = 'archived';
}
```

### Vue Component

```vue
<template>
  <div class="post-card">
    <h2>{{ post.title }}</h2>
    <p>{{ post.content }}</p>
    <span v-if="isPublished">Published</span>
  </div>
</template>

<script>
export default {
  name: 'PostCard',
  
  props: {
    post: {
      type: Object,
      required: true,
    },
  },
  
  computed: {
    isPublished() {
      return this.post.status === 'published';
    },
  },
};
</script>
```

## Testing

### Feature Test

```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;

final class PostTest extends TestCase
{
    public function test_user_can_create_post(): void
    {
        $user = User::factory()->create();
        
        $response = $this->actingAs($user)
            ->postJson('/api/posts', [
                'title' => 'Test Post',
                'content' => 'Test content',
            ]);

        $response->assertCreated();
        $this->assertDatabaseHas('posts', [
            'title' => 'Test Post',
        ]);
    }
}
```

## Git Workflow

```bash
git status
git add .
git commit -m "feat: add user authentication"
git push origin feature-branch
```

### Commit Types

- `feat:` New feature
- `fix:` Bug fix
- `refactor:` Code refactoring
- `test:` Adding tests
- `docs:` Documentation
- `chore:` Maintenance

## Checklist Before Completion

- [ ] `declare(strict_types=1)` in all PHP files
- [ ] PHPDoc on classes and public methods
- [ ] Type hints on parameters and returns
- [ ] Tests for business logic
- [ ] No N+1 queries
- [ ] Form requests for validation
- [ ] OpenSpec change archived
- [ ] Superpowers verification passed

## Common Issues

### Sail not starting

```bash
./vendor/bin/sail down
docker system prune -a
./vendor/bin/sail up -d
```

### Database connection

```bash
# Check Sail services
./vendor/bin/sail ps

# Reset database
./vendor/bin/sail artisan migrate:fresh --seed
```

### Node modules

```bash
npm install
npm run dev
```

## File Structure

```
app/
├── Http/
│   ├── Controllers/
│   ├── Requests/
│   └── Resources/
├── Services/
├── Repositories/
├── DTOs/
├── Enums/
└── Models/

resources/
└── js/
    ├── components/
    ├── services/
    ├── store/
    ├── utils/
    └── constants/

tests/
├── Feature/
└── Unit/

openspec/
├── project.md
├── specs/
├── changes/
└── archive/
```

## Resources

- CLAUDE.md - Full coding guidelines
- SETUP.md - Complete setup instructions
- Laravel Docs - <https://laravel.com/docs>
- Vue 2 Docs - <https://v2.vuejs.org>
