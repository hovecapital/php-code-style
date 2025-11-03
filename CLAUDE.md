# Laravel PHP + Vue 2 Development Guidelines v2

## Environment & Workflow

**IMPORTANT: Use Sail for all Laravel commands**

```bash
./vendor/bin/sail artisan migrate
./vendor/bin/sail composer require package/name
./vendor/bin/sail artisan test
```

**IMPORTANT: The user will run all npm commands - instruct and await**

**IMPORTANT: Sail and npm run dev will normally already be running**

**IMPORTANT: We are running on a local development environment**

**IMPORTANT: Use the MySQL MCP for inspecting the database**

**IMPORTANT: Comments should be used very rarely, only when absolutely necessary - code should be self-documenting**

**IMPORTANT: Use concise language**

**IMPORTANT: Don't use emojis**

## Architecture Overview

### This Project's ACTUAL Architecture

This codebase uses an **Actions + DTOs + ViewModels** pattern, NOT the traditional Service/Repository pattern.

```
Request → Form Request (validation)
       → Controller (thin, delegates to Actions)
       → DataTransferObject (structured data)
       → Action(s) (single-purpose business logic)
       → Events (side effects)
       → Resource (API response) / ViewModel (Blade views)
```

### Current Project Structure

```
app/
├── Actions/                    # PRIMARY business logic layer (45 files)
│   ├── Campaigns/
│   ├── Products/              # SaveProductAccessories, SaveProductCategories, etc.
│   ├── Users/
│   └── SaveChangelogAction.php
├── DataTransferObjects/        # NOT "DTOs/" - full name (20 files)
│   ├── Product/
│   ├── Campaigns/
│   └── ChangelogDataTransferObject.php
├── ViewModels/                 # Prepare data for Blade views (15 files)
│   └── ProductsViewModel.php
├── Http/
│   ├── Controllers/           # Thin controllers (98 files, some need refactoring)
│   ├── Requests/              # Form validation (29 files) ✅
│   └── Resources/             # API responses (53 files) ✅
├── Models/                     # Eloquent models (74 files)
├── Events/                     # Event-driven architecture ✅
├── Listeners/                  # Event handlers ✅
├── Jobs/                       # Queue jobs ✅
├── Mail/                       # Mailable classes ✅
├── Helpers/                    # Static utility methods (4 files)
├── Traits/                     # Shared behaviors (5 files)
├── Utilities/                  # Date, Revenue utilities (2 files)
└── Services/                   # Only 1 file (UserAuditService) - NOT primary pattern
```

**Note:** This project does NOT use:

- Repositories (no app/Repositories/)
- Services layer (only 1 service file exists)
- PHP Enums (uses model constants instead)

## PHP/Laravel Backend Standards

### Type Safety: Current vs Aspirational

**Current State (Reality Check):**

- ~5% of files use `declare(strict_types=1)`
- Type hints partially adopted
- Many legacy files without strict typing

**NEW CODE STANDARD (Required Going Forward):**

```php
<?php

declare(strict_types=1);

namespace App\Actions\Products;

use App\Models\Product;
use App\DataTransferObjects\Product\ProductDataTransferObject;

/**
 * Save product accessories
 */
final class SaveProductAccessories
{
    public function __invoke(Product $product, ProductDataTransferObject $data): void
    {
        // Implementation with full type hints
    }
}
```

**Type Safety Rules for New Code:**

- `declare(strict_types=1)` at top of ALL new PHP files
- Parameter and return type hints on ALL methods
- Avoid `mixed` type unless absolutely necessary
- Use `?Type` for nullable, handle null cases explicitly
- Use `readonly` properties where applicable (PHP 8.1+)

**Legacy Code:**

- Gradual improvement encouraged but not mandatory
- When modifying existing files, add type hints opportunistically
- No requirement to retrofit entire codebase

### Modern PHP Features (PHP 8.1+)

**Constructor Property Promotion (Currently used well in DTOs):**

```php
<?php

declare(strict_types=1);

namespace App\DataTransferObjects\Product;

final class ProductDataTransferObject
{
    public function __construct(
        public string $title,
        public string $slug,
        public ?string $notes,
        public int $categoryId,
        public readonly bool $isActive,
    ) {}
}
```

**Model Constants (Current approach):**

```php
// Current: Using class constants
class Product extends Model
{
    public const FELLOWSHIP = 5;
    public const THIRD_PARTY = 2;
}
```

**Future Consideration (PHP 8.1+ Enums):**

```php
// Could migrate to enums for better type safety
enum ProductType: int
{
    case FELLOWSHIP = 5;
    case THIRD_PARTY = 2;
}
```

### Actions Pattern (PRIMARY Business Logic Layer)

Actions are single-purpose, invokable classes. Use for discrete business operations.

**When to create an Action:**

- Saving related data (SaveProductAccessories)
- Complex business operations (ProcessOrderAction)
- Reusable operations called from multiple controllers
- Operations that trigger events

**Action Structure:**

```php
<?php

declare(strict_types=1);

namespace App\Actions\Products;

use App\Models\Product;
use App\DataTransferObjects\Product\ProductDataTransferObject;
use App\Events\ProductUpdated;

final class SaveProductAccessories
{
    public function __invoke(Product $product, ProductDataTransferObject $data): void
    {
        $product->accessories()->sync($data->accessoryIds);

        event(new ProductUpdated($product));
    }
}
```

**Examples from codebase:**

- `app/Actions/Products/SaveProductVariants.php`
- `app/Actions/Products/SaveProductBundles.php`
- `app/Actions/SaveChangelogAction.php`

### DataTransferObjects Pattern

**Directory:** `app/DataTransferObjects/` (NOT `app/DTOs/`)

Use DTOs to transform request data into structured, typed objects.

```php
<?php

declare(strict_types=1);

namespace App\DataTransferObjects\Product;

use App\Http\Requests\Admin\Product\SaveProduct;

final class ProductDataTransferObject
{
    public function __construct(
        public string $title,
        public string $slug,
        public ?string $description,
        public array $categoryIds,
        public array $accessoryIds,
    ) {}

    public static function fromSaveProductRequest(SaveProduct $request): self
    {
        return new self(
            title: $request->input('title'),
            slug: $request->input('slug'),
            description: $request->input('description'),
            categoryIds: $request->input('categories', []),
            accessoryIds: $request->input('accessories', []),
        );
    }
}
```

### ViewModels Pattern

**Directory:** `app/ViewModels/`

Use ViewModels to prepare data for Blade views (not API responses).

```php
<?php

declare(strict_types=1);

namespace App\ViewModels;

use App\Models\Product;
use App\Models\Category;

final class ProductsViewModel
{
    public readonly Collection $products;
    public readonly Collection $categories;

    public function __construct(?int $categoryId = null)
    {
        $this->products = Product::with('categories', 'points')
            ->when($categoryId, fn($q) => $q->whereHas('categories', fn($q) => $q->where('id', $categoryId)))
            ->get();

        $this->categories = Category::all();
    }
}
```

**Usage in controller:**

```php
public function index(?int $categoryId = null)
{
    $viewModel = new ProductsViewModel($categoryId);
    return view('products.index', ['viewModel' => $viewModel]);
}
```

### Controllers: Thin by Design

**Standard (Good):**

```php
<?php

declare(strict_types=1);

namespace App\Http\Controllers\Admin;

use App\Http\Requests\Admin\Product\SaveProduct;
use App\Http\Resources\Admin\ProductResource;
use App\DataTransferObjects\Product\ProductDataTransferObject;
use App\Actions\Products\SaveProductAction;

final class ProductsController extends Controller
{
    public function __construct(
        private readonly SaveProductAction $saveProduct,
    ) {}

    public function store(SaveProduct $request): JsonResponse
    {
        $productData = ProductDataTransferObject::fromSaveProductRequest($request);

        $product = ($this->saveProduct)($productData);

        return ProductResource::make($product)
            ->response()
            ->setStatusCode(201);
    }
}
```

**WARNING: Some controllers are too large**

- `app/Http/Controllers/Admin/ManualOrderController.php` - 1,776 lines (needs refactoring)
- `app/Http/Controllers/Retail/WishlistController.php` - 415 lines
- `app/Http/Controllers/Admin/SearchController.php` - 349 lines

**Rule:** Controllers >200 lines should be refactored into multiple Actions.

### Form Requests (Well Implemented ✅)

**Directory:** `app/Http/Requests/` (29 files)

Always use Form Requests for validation, not inline controller validation.

```php
<?php

declare(strict_types=1);

namespace App\Http\Requests\Admin\Product;

use Illuminate\Foundation\Http\FormRequest;

final class SaveProduct extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => ['required', 'string', 'max:255'],
            'slug' => ['required', 'string', 'unique:products,slug,' . $this->route('id')],
            'categories' => ['array'],
            'categories.*' => ['exists:categories,id'],
        ];
    }
}
```

### API Resources (Well Implemented ✅)

**Directory:** `app/Http/Resources/` (53 files)

Use API Resources for consistent JSON response formatting.

```php
<?php

declare(strict_types=1);

namespace App\Http\Resources\Admin;

use Illuminate\Http\Resources\Json\JsonResource;

final class ProductResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'categories' => CategoryResource::collection($this->whenLoaded('categories')),
            'created_at' => $this->created_at?->toISOString(),
        ];
    }
}
```

### Eloquent Models

**Directory:** `app/Models/` (74 files)

**Current state:** Most models lack strict types, but relationships have return types.

**New model standard:**

```php
<?php

declare(strict_types=1);

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\HasMany;

/**
 * Product model
 *
 * @property int $id
 * @property string $title
 * @property string $slug
 */
final class Product extends Model
{
    public const FELLOWSHIP = 5;
    public const THIRD_PARTY = 2;

    protected $fillable = [
        'title',
        'slug',
        'description',
    ];

    protected function casts(): array
    {
        return [
            'is_active' => 'boolean',
            'published_at' => 'datetime',
        ];
    }

    public function categories(): BelongsToMany
    {
        return $this->belongsToMany(Category::class);
    }

    public function variants(): HasMany
    {
        return $this->hasMany(ProductVariant::class);
    }
}
```

### Event-Driven Architecture (Well Implemented ✅)

**Directories:** `app/Events/`, `app/Listeners/`, `app/Jobs/`

Use events for side effects and decoupling.

```php
// In Action
use App\Events\ProductCreated;

public function __invoke(ProductDataTransferObject $data): Product
{
    $product = Product::create([...]);

    event(new ProductCreated($product, auth()->user()->name));

    return $product;
}
```

### Code Quality Standards

**For All Code:**

- **No repetition**: Extract to Actions, Traits, Helpers
- **Single responsibility**: One clear purpose per class/method
- **Explicit returns**: Always return values, even void
- **Fail early**: Validate inputs at method start with clear exceptions
- **PHPDoc blocks**: Required for classes and public methods (IDE support)

**Query Optimization:**

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

## Vue 2 Frontend Standards

### Current Reality vs New Standards

**CURRENT STATE (Existing Code):**

- Component-local state (no Vuex store)
- Inline axios calls in components
- Minimal/missing prop validation
- Components in `resources/js/admin/` and `resources/js/retail/`

**NEW STANDARDS (For New Features):**

- Create API service layer in `resources/js/services/`
- Implement Vuex for shared state management
- Require prop validation with types
- Create `resources/js/constants/` for app constants

**Migration Approach:**

- New features MUST follow new standards
- Refactor existing code opportunistically, not mandatorily

### Component Design

**Current (Acceptable for Existing Code):**

```vue
<template>
  <div class="gallery">
    <img :src="src" :alt="alt">
  </div>
</template>

<script>
export default {
  name: 'Gallery',
  props: ['src', 'alt', 'single'],  // No validation
}
</script>
```

**NEW STANDARD (Required for New Components):**

```vue
<template>
  <div class="product-card">
    <h3>{{ product.title }}</h3>
    <p>{{ product.description }}</p>
  </div>
</template>

<script>
export default {
  name: 'ProductCard',

  props: {
    product: {
      type: Object,
      required: true,
      validator: (product) => {
        return product.id && product.title;
      },
    },
  },

  computed: {
    displayPrice() {
      return this.formatMoney(this.product.price);
    },
  },
}
</script>
```

**Component Rules:**

- Single File Components (.vue)
- PascalCase component names, kebab-case in templates
- **New components:** MUST validate props with types and required flags
- Single responsibility per component
- Computed properties for derived state, not methods

### Project Structure (Vue)

**Current Structure:**

```
resources/js/
├── admin/                      # Admin interface (98 components)
│   ├── components/
│   ├── pages/
│   ├── mixins/                # Well-implemented (7 files) ✅
│   │   ├── money.js
│   │   ├── Format.js
│   │   └── ...
│   └── util/
├── retail/                     # Customer-facing (100 components)
│   ├── account/
│   ├── cart/
│   ├── product/
│   └── wishlist/
└── tracking/                   # Tracking functionality
```

**NEW STRUCTURE (Add These):**

```
resources/js/
├── services/                   # NEW: API layer
│   ├── productService.js
│   ├── cartService.js
│   └── axios.js
├── store/                      # NEW: Vuex modules
│   ├── modules/
│   │   ├── products.js
│   │   └── cart.js
│   └── index.js
├── constants/                  # NEW: App constants
│   └── productTypes.js
└── (existing directories)
```

### API Service Layer (NEW STANDARD)

**Create:** `resources/js/services/productService.js`

```javascript
import axios from 'axios';

class ProductService {
  async getAll(categoryId = null) {
    const params = categoryId ? { category: categoryId } : {};
    const response = await axios.get('/api/products', { params });
    return response.data.data;
  }

  async getById(id) {
    const response = await axios.get(`/api/products/${id}`);
    return response.data.data;
  }

  async create(productData) {
    const response = await axios.post('/api/products', productData);
    return response.data.data;
  }

  async update(id, productData) {
    const response = await axios.patch(`/api/products/${id}`, productData);
    return response.data.data;
  }
}

export default new ProductService();
```

**Usage in component:**

```javascript
import productService from '@/services/productService';

export default {
  async mounted() {
    try {
      this.products = await productService.getAll(this.categoryId);
    } catch (error) {
      this.$toast.error('Failed to load products');
    }
  },
}
```

### Vuex Store (NEW STANDARD for Shared State)

**When to use Vuex:**

- Shared state across multiple components
- Complex state management needs
- User authentication state
- Shopping cart state

**Create:** `resources/js/store/modules/products.js`

```javascript
import productService from '@/services/productService';

export default {
  namespaced: true,

  state: {
    products: [],
    loading: false,
    error: null,
  },

  mutations: {
    SET_PRODUCTS(state, products) {
      state.products = products;
    },

    SET_LOADING(state, loading) {
      state.loading = loading;
    },

    SET_ERROR(state, error) {
      state.error = error;
    },
  },

  actions: {
    async fetchProducts({ commit }, categoryId = null) {
      commit('SET_LOADING', true);
      commit('SET_ERROR', null);

      try {
        const products = await productService.getAll(categoryId);
        commit('SET_PRODUCTS', products);
      } catch (error) {
        commit('SET_ERROR', error.message);
      } finally {
        commit('SET_LOADING', false);
      }
    },
  },

  getters: {
    activeProducts: (state) => {
      return state.products.filter(p => p.is_active);
    },
  },
};
```

### Mixins (Well Implemented ✅)

**Directory:** `resources/js/admin/mixins/` (7 files)

Mixins are currently used well for shared functionality.

```javascript
// resources/js/admin/mixins/money.js
export default {
    methods: {
        money(value) {
            var formatter = new Intl.NumberFormat('en-GB', {
                style: 'currency',
                currency: 'GBP',
            });
            return formatter.format(value);
        }
    }
}
```

**Continue using mixins for:**

- Formatting utilities
- Shared methods
- Common computed properties

## Testing Standards

### Current State

**Feature Tests:** 50 files (well-covered ✅)
**Unit Tests:** 2 files (minimal coverage)

```
tests/
├── Feature/
│   ├── Admin/
│   ├── WishlistTest.php
│   ├── FavouritesTest.php
│   ├── ThirdPartyApprovalTest.php
│   └── ... (47 more)
└── Unit/
    ├── ExampleTest.php
    └── OuBusinessSettingTest.php
```

### Testing Standards

**Priorities:**

1. Feature tests for API endpoints (continue current approach)
2. Unit tests for complex Actions and business logic (increase coverage)
3. Integration tests for external services
4. **Test what matters, not lines of code**

**Feature Test Example:**

```php
<?php

declare(strict_types=1);

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use App\Models\Product;
use Illuminate\Foundation\Testing\RefreshDatabase;

final class ProductManagementTest extends TestCase
{
    use RefreshDatabase;

    public function test_admin_can_create_product(): void
    {
        $admin = User::factory()->admin()->create();

        $response = $this->actingAs($admin)
            ->postJson('/api/admin/products', [
                'title' => 'Test Product',
                'slug' => 'test-product',
                'description' => 'Test description',
            ]);

        $response->assertCreated();
        $this->assertDatabaseHas('products', [
            'title' => 'Test Product',
        ]);
    }
}
```

**Unit Test for Actions (NEW STANDARD):**

```php
<?php

declare(strict_types=1);

namespace Tests\Unit\Actions;

use Tests\TestCase;
use App\Actions\Products\SaveProductAccessories;
use App\Models\Product;
use App\Models\Accessory;

final class SaveProductAccessoriesTest extends TestCase
{
    public function test_saves_product_accessories(): void
    {
        $product = Product::factory()->create();
        $accessories = Accessory::factory()->count(3)->create();

        $dto = new ProductDataTransferObject(
            accessoryIds: $accessories->pluck('id')->toArray(),
            // ... other required fields
        );

        $action = new SaveProductAccessories();
        $action($product, $dto);

        $this->assertCount(3, $product->fresh()->accessories);
    }
}
```

## Error Handling

### Backend (Laravel)

```php
<?php

declare(strict_types=1);

namespace App\Exceptions;

use Exception;
use Illuminate\Http\JsonResponse;

final class ProductNotFoundException extends Exception
{
    public function render(): JsonResponse
    {
        return response()->json([
            'error' => 'Product not found',
            'message' => 'The requested product does not exist',
        ], 404);
    }
}
```

### Frontend (Vue)

```javascript
async createProduct(productData) {
  try {
    const product = await productService.create(productData);
    this.$toast.success('Product created successfully');
    return product;
  } catch (error) {
    if (error.response?.status === 422) {
      // Validation errors
      this.$toast.error('Validation failed: ' + error.response.data.message);
    } else {
      this.$toast.error('Failed to create product');
    }
    throw error;
  }
}
```

**Error handling rules:**

- Validate inputs early (Form Requests)
- Structured error response formats
- Clear, actionable error messages
- Proper HTTP status codes

## Database & Performance

### Query Optimization

Always use eager loading to prevent N+1 queries:

```php
// Bad: N+1 queries
$products = Product::all();
foreach ($products as $product) {
    echo $product->categories->count();
}

// Good: Eager loading
$products = Product::withCount('categories')->get();
foreach ($products as $product) {
    echo $product->categories_count;
}

// Good: With relationships
$products = Product::with('categories', 'variants')->get();
```

### Performance Best Practices

- **OpCache:** Enable in production
- **Query optimization:** Use eager loading, avoid N+1
- **Caching:** Redis for sessions, query caching
- **Route caching:** `sail artisan route:cache`
- **Config caching:** `sail artisan config:cache`
- **View caching:** Automatic in production

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

- 4 spaces for indentation (not tabs)
- Opening braces on same line for methods
- One blank line after namespace declaration
- Declare visibility for all properties/methods

## Development Checklist

Before marking work complete:

**Backend:**

- [ ] `declare(strict_types=1)` in all NEW PHP files
- [ ] PHPDoc blocks on classes and public methods
- [ ] Type hints on all parameters and returns (new code)
- [ ] Form Requests for validation
- [ ] Actions for business logic (not bloated controllers)
- [ ] No N+1 queries (use eager loading)
- [ ] Events for side effects
- [ ] Tests for business logic

**Frontend (New Features):**

- [ ] Props validated with types and required flags
- [ ] API calls in service layer (not inline)
- [ ] Shared state in Vuex (if applicable)
- [ ] No direct axios calls in components
- [ ] Mixins for shared logic

**Both:**

- [ ] No code repetition
- [ ] Clear error messages
- [ ] Tests passing (`sail artisan test`)

## Summary: What Makes This Project Unique

This Laravel + Vue 2 project differs from typical Laravel apps:

**What's Different:**

- Uses **Actions pattern** instead of Services/Repositories
- Uses **ViewModels** for Blade view data preparation
- Directory is `DataTransferObjects/` not `DTOs/`
- No Vuex store (component-local state)
- Inline axios calls (no service layer yet)
- Event-driven architecture emphasized
- ~5% strict typing adoption (aspirational goal for new code)

**What Works Well:**

- Form Requests (29 files)
- API Resources (53 files)
- DTOs with PHP 8.1 constructor property promotion
- Feature test coverage (50 files)
- Event-driven architecture
- Vue mixins (7 files)

**What Needs Improvement:**

- Large controllers (ManualOrderController = 1,776 lines)
- Inconsistent type safety in legacy code
- Missing frontend service layer
- Minimal unit test coverage
- Missing prop validation in Vue components

**Path Forward:**

- NEW code follows strict standards
- LEGACY code improved opportunistically
- GRADUALLY introduce frontend best practices (services, Vuex)
- REFACTOR large controllers into Actions
