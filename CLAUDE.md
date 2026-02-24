# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mixpost is a self-hosted social media management software built as a Laravel package. It provides a comprehensive platform for scheduling and publishing content across multiple social media platforms, and managing analytics.

This is **Mixpost Lite** - the open-source version. There is also **Mixpost Pro** which contains additional features and is built on top of Lite.

## Porting Features from Pro to Lite

Features are typically developed first in Pro, then ported to Lite. When implementing a feature from Pro:

### Pro Repository Location
```
/Users/lao9s/web/packages/mixpost-pro-team
```

Pro has its own `CLAUDE.md` with architecture documentation.

### Key Differences Between Pro and Lite

1. **Workspace Isolation**: Pro has multi-tenancy through workspace scoping
   - Pro models use `OwnedByWorkspace` trait - remove this in Lite
   - Pro queue jobs implement `QueueWorkspaceAware` interface - remove in Lite
   - Pro commands support `--workspace=` option - remove in Lite

2. **Additional Pro Features** (not in Lite):
   - API endpoints (`src/Http/Api/`)
   - Webhook triggers
   - AI service integrations
   - URL shorteners
   - Team management

3. **Controller Structure**:
   - Pro: `src/Http/Base/` for web controllers, `src/Http/Api/` for API
   - Lite: `src/Http/Controllers/` (flat structure)

### Porting Checklist

When porting a feature from Pro to Lite:

1. **Copy relevant files** from Pro to Lite equivalent locations
2. **Remove workspace scoping**:
   - Remove `OwnedByWorkspace` trait from models
   - Remove `QueueWorkspaceAware` interface from jobs
   - Remove `--workspace` option from commands
   - Remove workspace-related middleware and checks
3. **Adjust controller paths**: Move from `Http/Base/` to `Http/Controllers/`
4. **Remove Pro-only dependencies**: AI providers, URL shorteners, webhooks, etc.
5. **Simplify routes**: Remove workspace prefixing if present
6. **Update imports/namespaces**: `Inovector\MixpostPro\` → `Inovector\Mixpost\`
7. **Run tests**: Ensure feature works without workspace context
8. **Run linters**: `composer format` and `npm run lint`

## Build & Development Commands

### Frontend Development
```bash
# Start development server with hot reload
npm run dev

# Build production assets
npm run build

# Run ESLint with auto-fix
npm run lint

# Check ESLint issues without fixing
npm run lint:check

# Format code with Prettier
npm run format

# Check formatting without changes
npm run format:check
```

### Backend Development
```bash
# Format PHP code with Laravel Pint
composer format

# Run static analysis with PHPStan (level 5)
composer analyse

# Run tests with Pest
composer test

# Run tests with coverage
composer test-coverage
```

### Git Hooks
The project uses Husky for pre-commit hooks that automatically run:
- ESLint and Prettier on staged JS/Vue/CSS files
- Laravel Pint on staged PHP files

## Architecture Overview

### Package Structure
This is a Laravel package (not a standalone application) that integrates into existing Laravel applications. The service provider (`MixpostServiceProvider`) registers all package components.

### Core Directories

- **src/**: Main package source code
    - **Http/**: Controllers, middleware, requests, and resources
    - **Models/**: Eloquent models (workspace-scoped via traits)
    - **SocialProviders/**: Platform-specific integrations (Facebook, Twitter, etc.)
    - **Jobs/**: Queue jobs for async operations
    - **Commands/**: Artisan commands (workspace and system level)
    - **Services/**: Business logic and external service integrations
    - **Actions/**: Single-responsibility action classes

- **resources/**: Frontend assets
    - **js/**: Vue 3 components with Composition API
    - **css/**: Tailwind CSS v4 styles
    - **views/**: Blade templates (minimal, mainly for app shell)

- **database/**: Migrations and factories
    - All tables use `mixpost_` prefix
    - Workspace-scoped data isolation

### Key Architectural Patterns

1. **Service Provider Pattern**: External services (Unsplash, social platforms) use provider abstraction
    - Abstract classes define contracts
    - Concrete implementations in provider classes
    - Manager classes handle provider registration

2. **Action Classes**: (src/Actions/)
    - Single responsibility principle
    - Reusable across controllers and jobs

3. **Frontend Architecture**: Vue 3 + Inertia.js
    - Pages in `resources/js/Pages/`
    - Shared components in `resources/js/Components/`
    - Composables for reusable logic

4. **Event-Driven**: Extensive use of Laravel events
    - Post lifecycle events (created, published, failed)
    - Account events (added, unauthorized)

## Testing Approach

- **Test Framework**: Pest PHP
- **Test Location**: `tests/` directory
- **Run Single Test**: `vendor/bin/pest tests/path/to/Test.php`
- **Run Specific Test Method**: `vendor/bin/pest tests/path/to/Test.php --filter="test method name"`

## Important Development Notes

- Always check for workspace context when working with data operations
- Social provider integrations require OAuth setup and token management
- Media uploads support images and videos with automatic conversions
- Queue workers must be workspace-aware for proper data isolation
- Frontend uses Ziggy for route generation - use `route()` helper
- The project uses Tailwind CSS v4 with custom configuration
- When working with Tailwind CSS v4, use the custom spacing values defined in `resources/css/app.css` @theme section:
    - `--spacing-xs: 0.5rem`
    - `--spacing-sm: 0.75rem`
    - `--spacing-md: 1rem`
    - `--spacing-lg: 1.5rem`
    - `--spacing-xl: 2rem`
    - `--spacing-2xl: 2.5rem`
- When setting max-width in Tailwind CSS, NEVER use standard Tailwind classes like `max-w-2xl`, `max-w-md`, `max-w-lg`, etc. Instead, ALWAYS use the custom container size variables:
    - Use `max-w-(--container-xs)` instead of `max-w-xs`
    - Use `max-w-(--container-sm)` instead of `max-w-sm`
    - Use `max-w-(--container-md)` instead of `max-w-md`
    - Use `max-w-(--container-lg)` instead of `max-w-lg`
    - Use `max-w-(--container-xl)` instead of `max-w-xl`
    - Use `max-w-(--container-2xl)` instead of `max-w-2xl`
    - This applies to ALL container sizes - always use the custom CSS variable format
- When creating buttons in UI components, always reuse existing button components from `resources/js/Components/Button/`:
    - `PrimaryButton.vue` - Primary actions and CTAs
    - `SecondaryButton.vue` - Secondary actions
    - `DangerButton.vue` - Destructive actions
    - `SuccessButton.vue` - Success/confirmation actions
    - `WarningButton.vue` - Warning actions
    - `PureButton.vue` - Minimal styling buttons
    - `DarkButtonLink.vue` - Dark themed button links
    - Never create custom button elements - use these standardized components for consistency

## Coding Standards

- **UI/UX Design**: Follow the guidelines in `.guidelines/ui-guideline.pdf`
    - Maintain consistency with existing UI patterns and components
    - Follow the design system and component hierarchy

- **PHP/Laravel Code**: Follow the guidelines in `.guidelines/laravel-php-guideline.md`
    - PSR-1, PSR-2, and PSR-12 compliance
    - CamelCase for variables, functions, and methods
    - Snake_case for object keys (except Inertia data which uses camelCase)
    - Typed properties and return types preferred
    - Early returns and avoid else statements

- **JavaScript/Vue Code**: Follow the guidelines in `.guidelines/javascript-guideline.md`
    - CamelCase for variables, functions, and methods
    - Snake_case for object keys
    - Prefer const over let, never use var
    - Arrow functions preferred over function keyword
    - Use destructuring for objects and arrays
    - Early returns and avoid else statements

## Database Considerations

- All tables prefixed with `mixpost_`
- UUID fields for public identifiers
- Soft deletes on posts and critical data
- JSON columns for flexible configuration storage

## Security Considerations

- OAuth tokens stored encrypted
- API authentication via personal access tokens
- File uploads validated for type and size
- XSS protection via Vue's automatic escaping
