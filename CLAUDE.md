# Laravel PHP + Vue 2 Development Guidelines

## Core Principles

- Use concise language
- No emojis
- Code should be self-documenting with minimal inline comments
- PHPDoc blocks required for classes and methods (IDE support)
- Strict typing throughout
- Test-driven development (flexible coverage goals)

## Development Flow

### 1. OpenSpec Workflow (Required for All Features)

Every feature/bug fix starts with OpenSpec:

```bash
# Start new feature
/openspec:proposal <feature-name>

# Review and refine specs
openspec show <feature-name>
openspec validate <feature-name>

# Implement
/openspec:apply <feature-name>

# Archive when complete
/openspec:archive <feature-name> --yes
```

### 2. Superpowers Integration

Superpowers skills activate automatically:

- **TDD**: test-driven-development skill for features
- **Debugging**: systematic-debugging for issues
- **Planning**: writing-plans and executing-plans for complex work
- **Verification**: verification-before-completion before claiming done

Manual activation:

```bash
/superpowers:brainstorm  # Design refinement
/superpowers:write-plan  # Implementation planning
/superpowers:execute-plan # Batch execution
```

## Required MCP Servers

### MySQL MCP

Use for all database inspection:

```json
{
  "name": "mysql",
  "repository": "hovecapital/read-only-local-mysql-mcp-server",
  "usage": "Query database schema, inspect data, analyze relationships"
}
```

### Context7 MCP

Use for library documentation:

```json
{
  "name": "context7",
  "usage": "Read up-to-date documentation for Laravel packages and dependencies"
}
```

## Environment Notes

- **Sail**: Use `./vendor/bin/sail` for all Laravel commands
- **NPM**: User runs npm commands - instruct and await
- **Dev servers**: Sail and `npm run dev` typically running
- **Environment**: Local development

## PHP/Laravel Backend

### Type Safety

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Models\User;
use App\DTOs\UserData;

/**
 * User management service
 */
final class UserService
{
    public function createUser(UserData $data): User
    {
        // Implementation
    }

    public function findById(int $id): ?User
    {
        // Implementation
    }
}
```

**Rules:**

- `declare(strict_types=1)` at top of all PHP files
- Parameter and return type hints on all methods
- Avoid `mixed` type unless legacy compatibility required
- Use `?Type` for nullable, handle null cases explicitly
- Readonly properties where applicable (PHP 8.1+)

### Modern PHP Features (8.3+)

```php
// Enums with backed values
enum UserRole: string
{
    case ADMIN = 'admin';
    case USER = 'user';
}

// Constructor property promotion
final class UserData
{
    public function __construct(
        public readonly string $email,
        public readonly string $name,
        public readonly UserRole $role,
    ) {}
}

// Named arguments
$user = new User(
    email: 'user@example.com',
    name: 'John Doe',
    role: UserRole::USER,
);

// Match expressions
$status = match ($user->role) {
    UserRole::ADMIN => 'Administrator',
    UserRole::USER => 'Regular User',
};
```

### Code Quality

- **No repetition**: Extract to services, traits, helpers
- **Single responsibility**: One clear purpose per class/method
- **Explicit returns**: Always return values, even void
- **Fail early**: Validate inputs at method start with clear exceptions

### Laravel Architecture

```
app/
├── Http/
│   ├── Controllers/        # Thin controllers, delegate to services
│   ├── Requests/          # Form request validation
│   └── Resources/         # API response formatting
├── Services/              # Business logic
├── Repositories/          # Complex database queries
├── DTOs/                  # Data Transfer Objects
├── Enums/                 # Constants and status values
└── Models/                # Eloquent models with typed relationships
```

### Controller Pattern

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers;

use App\Http\Requests\CreateUserRequest;
use App\Http\Resources\UserResource;
use App\Services\UserService;
use Illuminate\Http\JsonResponse;

/**
 * User management controller
 */
final class UserController extends Controller
{
    public function __construct(
        private readonly UserService $userService,
    ) {}

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->createUser(
            $request->validated()
        );

        return UserResource::make($user)
            ->response()
            ->setStatusCode(201);
    }
}
```

### Service Layer

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\DTOs\UserData;
use App\Models\User;
use App\Repositories\UserRepository;
use Illuminate\Support\Facades\Hash;

/**
 * User management service
 */
final class UserService
{
    public function __construct(
        private readonly UserRepository $userRepository,
    ) {}

    public function createUser(array $data): User
    {
        $userData = new UserData(
            email: $data['email'],
            name: $data['name'],
            role: UserRole::from($data['role']),
        );

        return $this->userRepository->create([
            'email' => $userData->email,
            'name' => $userData->name,
            'role' => $userData->role->value,
            'password' => Hash::make($data['password']),
        ]);
    }
}
```

### Form Request Validation

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Enum;
use App\Enums\UserRole;

/**
 * Create user request validation
 */
final class CreateUserRequest extends FormRequest
{
    /**
     * @return array<string, mixed>
     */
    public function rules(): array
    {
        return [
            'email' => ['required', 'email', 'unique:users'],
            'name' => ['required', 'string', 'max:255'],
            'password' => ['required', 'string', 'min:8'],
            'role' => ['required', new Enum(UserRole::class)],
        ];
    }
}
```

### Eloquent Models

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use App\Enums\UserRole;

/**
 * User model
 * 
 * @property int $id
 * @property string $email
 * @property string $name
 * @property UserRole $role
 */
final class User extends Model
{
    protected $fillable = [
        'email',
        'name',
        'role',
    ];

    /**
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'role' => UserRole::class,
            'email_verified_at' => 'datetime',
        ];
    }

    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

### Testing Approach

Focus on meaningful tests over coverage metrics:

```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use App\Enums\UserRole;

final class UserManagementTest extends TestCase
{
    public function test_creates_user_with_valid_data(): void
    {
        $response = $this->postJson('/api/users', [
            'email' => 'test@example.com',
            'name' => 'Test User',
            'password' => 'password123',
            'role' => 'user',
        ]);

        $response->assertCreated();
        $this->assertDatabaseHas('users', [
            'email' => 'test@example.com',
        ]);
    }
}
```

**Testing priorities:**

- Feature tests for API endpoints
- Unit tests for complex business logic
- Integration tests for external services
- Test what matters, not lines of code

## Vue 2 Frontend

### Component Design

```vue
<template>
  <div class="user-profile">
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
    <user-role-badge :role="user.role" />
  </div>
</template>

<script>
import UserRoleBadge from './UserRoleBadge.vue';

export default {
  name: 'UserProfile',
  
  components: {
    UserRoleBadge,
  },
  
  props: {
    user: {
      type: Object,
      required: true,
      validator: (user) => {
        return user.id && user.name && user.email;
      },
    },
  },
  
  computed: {
    isAdmin() {
      return this.user.role === 'admin';
    },
  },
};
</script>
```

**Rules:**

- Single File Components (.vue)
- PascalCase component names, kebab-case in templates
- Always validate props with types and required flags
- Single responsibility per component
- Computed properties for derived state

### Project Structure (Vue)

```
resources/js/
├── components/           # By feature/domain
│   ├── users/
│   ├── posts/
│   └── shared/
├── services/            # API calls
├── store/               # Vuex modules
├── utils/               # Shared utilities
└── constants/           # App constants
```

### Vuex Pattern

```javascript
// store/modules/users.js
export default {
  namespaced: true,
  
  state: {
    users: [],
    loading: false,
    error: null,
  },
  
  mutations: {
    SET_USERS(state, users) {
      state.users = users;
    },
    
    SET_LOADING(state, loading) {
      state.loading = loading;
    },
  },
  
  actions: {
    async fetchUsers({ commit }) {
      commit('SET_LOADING', true);
      
      try {
        const users = await userService.getAll();
        commit('SET_USERS', users);
      } finally {
        commit('SET_LOADING', false);
      }
    },
  },
  
  getters: {
    adminUsers: (state) => {
      return state.users.filter(u => u.role === 'admin');
    },
  },
};
```

### API Service Layer

```javascript
// services/userService.js
import axios from 'axios';

class UserService {
  async getAll() {
    const response = await axios.get('/api/users');
    return response.data.data;
  }
  
  async create(userData) {
    const response = await axios.post('/api/users', userData);
    return response.data.data;
  }
  
  async update(id, userData) {
    const response = await axios.patch(`/api/users/${id}`, userData);
    return response.data.data;
  }
}

export default new UserService();
```

## Error Handling

### Backend (Laravel)

```php
<?php

declare(strict_types=1);

namespace App\Exceptions;

use Exception;
use Illuminate\Http\JsonResponse;

final class UserNotFoundException extends Exception
{
    public function render(): JsonResponse
    {
        return response()->json([
            'error' => 'User not found',
            'message' => 'The requested user does not exist',
        ], 404);
    }
}
```

### Frontend (Vue)

```javascript
async createUser(userData) {
  try {
    const user = await userService.create(userData);
    this.$toast.success('User created successfully');
    return user;
  } catch (error) {
    if (error.response?.status === 422) {
      this.$toast.error('Validation failed: ' + error.response.data.message);
    } else {
      this.$toast.error('Failed to create user');
    }
    throw error;
  }
}
```

**Error handling rules:**

- Validate inputs early
- Structured error response formats
- Clear, actionable error messages
- Proper HTTP status codes

## Database Patterns

### Query Optimization

```php
// Bad: N+1 queries
$users = User::all();
foreach ($users as $user) {
    echo $user->posts->count();
}

// Good: Eager loading
$users = User::withCount('posts')->get();
foreach ($users as $user) {
    echo $user->posts_count;
}
```

### Repository Pattern

```php
<?php

declare(strict_types=1);

namespace App\Repositories;

use App\Models\User;
use Illuminate\Database\Eloquent\Collection;

/**
 * User repository for complex queries
 */
final class UserRepository
{
    public function findActiveAdmins(): Collection
    {
        return User::query()
            ->where('role', 'admin')
            ->where('active', true)
            ->with('posts')
            ->get();
    }

    public function create(array $data): User
    {
        return User::create($data);
    }
}
```

## Performance

- **OpCache**: Enable in production
- **Query optimization**: Use eager loading, avoid N+1
- **Caching**: Redis for sessions, query caching
- **Route caching**: `sail artisan route:cache`
- **Config caching**: `sail artisan config:cache`
- **View caching**: Automatic in production

## Security

- Input validation via Form Requests
- SQL injection prevention via Eloquent/Query Builder
- XSS protection via Blade escaping
- CSRF token handling built-in
- Password hashing via `Hash::make()`
- Rate limiting on routes
- Sanctum for API authentication

## PSR Standards

Follow PSR-12 coding standard:

- 4 spaces for indentation
- Opening braces on same line for classes/methods
- One blank line after namespace declaration
- Declare visibility for all properties/methods

## Tools Integration

### Use MCP for Database

```bash
# Instead of manual queries, use MySQL MCP
"Check the users table schema and recent records"
```

### Use Context7 for Documentation

```bash
# Before using unfamiliar packages
"Get documentation for Laravel Sanctum authentication"
```

### Sail Commands

```bash
./vendor/bin/sail artisan migrate
./vendor/bin/sail artisan make:model User
./vendor/bin/sail artisan test
./vendor/bin/sail composer require package/name
```

## Development Checklist

Before marking work complete:

- [ ] Strict types declared in all PHP files
- [ ] PHPDoc blocks on classes and public methods
- [ ] Type hints on all parameters and returns
- [ ] Form requests for validation
- [ ] Tests written for business logic
- [ ] No N+1 queries
- [ ] OpenSpec change archived
- [ ] Superpowers verification passed
