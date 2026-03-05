---
name: angular-modularize
description: >
  Modularizes and restructures Angular projects following official best practices.
  Use when the user asks to "modularize", "refactor module structure",
  "organize Angular project", "apply Angular best practices",
  "split into feature modules", "restructure project", "create feature module",
  "organize components", or "improve project structure".
allowed-tools: Read, Grep, Glob, Edit, Write, Bash, Agent
---

# Angular Modularization Best Practices

You are an expert Angular architect. When the user asks to modularize or restructure
an Angular project, follow these guidelines based on the official Angular style guide
and modern Angular v17+ patterns.

## Step 1: Analyze the Current Project

Before making changes, always:

1. **Scan the project structure** using Glob to understand the current layout
2. **Identify the Angular version** by reading `package.json`
3. **Check existing modules** with `Glob("**/*.module.ts")` and `Glob("**/*.routes.ts")`
4. **Detect standalone components** with `Grep("standalone", glob="**/*.component.ts")`
5. **Map dependencies** between components, services, and modules
6. **Report findings** to the user before proposing changes

## Step 2: Evaluate IF Modularization is Appropriate

**Not everything should be modularized.** Before restructuring, evaluate each component/module
against these criteria:

### When TO Modularize

| Criteria | Example |
|----------|---------|
| Feature has its own route | `/users`, `/dashboard`, `/settings` |
| 3+ related components work together | User list, user detail, user form |
| Feature is developed by a separate team | Team A owns orders, Team B owns inventory |
| Feature can benefit from lazy loading | Large feature with heavy dependencies |
| Components are reused across 2+ features | Shared table, shared modal, shared form controls |
| Feature has its own services and models | UserService, User model, user-specific guards |
| Business domain is clearly bounded | Authentication, billing, reporting |

### When NOT to Modularize

| Criteria | Example |
|----------|---------|
| Single component with no children | A simple static page or about page |
| Component is used only once in one place | A header-specific search bar |
| Feature has fewer than 2-3 files total | A single service + model |
| App is small (< 10 components total) | Simple CRUD app, prototype, POC |
| Over-modularization adds more complexity than it solves | Creating a module for a single pipe |
| Component is tightly coupled to its parent | A form step that only exists inside a wizard |
| No lazy loading benefit | Feature loaded at startup anyway |
| Splitting would create circular dependencies | Two features that share too much state |

### Decision Flowchart

Ask these questions in order:

1. **Does this feature have its own route?** → If yes, it's a candidate for a feature module/directory
2. **Are there 3+ related components?** → If yes, group them together
3. **Is this component reused elsewhere?** → If yes, move to `shared/`
4. **Is this a singleton service used app-wide?** → If yes, move to `core/`
5. **Would lazy loading improve performance?** → If yes, create a lazy-loaded feature
6. **Does splitting improve team autonomy?** → If yes, modularize
7. **If none of the above**, keep it where it is — don't modularize for the sake of it

### Hybrid Approach (Recommended for existing projects)

Not all parts of an application need the same level of modularization:

- **High-traffic features** → Full modularization with lazy loading
- **Simple CRUD screens** → Lightweight feature directory, no separate module needed
- **Utility components** → `shared/` directory without over-engineering
- **One-off pages** → Keep inline, don't create a feature module for a single component

## Step 3: Determine the Modularization Strategy

### For Angular v17+ (Standalone by default)

Prefer **standalone components** with route-based lazy loading:

```
src/app/
├── app.component.ts
├── app.config.ts
├── app.routes.ts
├── core/
│   ├── interceptors/
│   │   └── auth.interceptor.ts
│   ├── guards/
│   │   └── auth.guard.ts
│   └── services/
│       └── auth.service.ts
├── shared/
│   ├── components/
│   │   └── button/
│   │       ├── button.component.ts
│   │       ├── button.component.html
│   │       ├── button.component.css
│   │       └── button.component.spec.ts
│   ├── directives/
│   ├── pipes/
│   └── models/
└── features/
    ├── dashboard/
    │   ├── dashboard.component.ts
    │   ├── dashboard.component.html
    │   ├── dashboard.component.spec.ts
    │   ├── dashboard.routes.ts
    │   └── components/
    │       └── widget/
    │           ├── widget.component.ts
    │           └── widget.component.spec.ts
    └── users/
        ├── user-list/
        │   ├── user-list.component.ts
        │   ├── user-list.component.html
        │   └── user-list.component.spec.ts
        ├── user-detail/
        │   ├── user-detail.component.ts
        │   └── user-detail.component.spec.ts
        ├── services/
        │   └── user.service.ts
        ├── models/
        │   └── user.model.ts
        └── users.routes.ts
```

### For Angular v14-v16 (NgModules with standalone migration path)

Use **feature modules** with lazy loading:

```
src/app/
├── app.module.ts
├── app-routing.module.ts
├── core/
│   └── core.module.ts
├── shared/
│   └── shared.module.ts
└── features/
    └── users/
        ├── users.module.ts
        ├── users-routing.module.ts
        ├── components/
        ├── services/
        └── models/
```

### For Angular < v14 (NgModules)

Use the **classic module pattern**:

```
src/app/
├── app.module.ts
├── app-routing.module.ts
├── core/
│   └── core.module.ts
├── shared/
│   └── shared.module.ts
└── features/
    └── feature-name/
        ├── feature-name.module.ts
        ├── feature-name-routing.module.ts
        ├── components/
        ├── services/
        └── models/
```

## Step 4: Apply Modularization Rules

### File Naming Conventions

- Use **kebab-case** for all file names: `user-profile.component.ts`
- Follow the pattern: `<feature>.<type>.ts`
  - Components: `user-list.component.ts`
  - Services: `user.service.ts`
  - Guards: `auth.guard.ts`
  - Interceptors: `auth.interceptor.ts`
  - Pipes: `date-format.pipe.ts`
  - Directives: `highlight.directive.ts`
  - Models/Interfaces: `user.model.ts`
  - Specs: `user-list.component.spec.ts`
- **One concept per file**: never put multiple components or services in one file

### Core Module/Directory (`core/`)

Contains **singleton services** and **app-wide** functionality:

- Authentication services, guards, and interceptors
- HTTP interceptors (error handling, auth tokens)
- Application-wide services (logging, notification)
- Environment configuration services

Rules:
- Services must use `providedIn: 'root'` (Angular 6+)
- Never import Core into feature modules directly
- Core is imported **only once** in the root module/config

### Shared Module/Directory (`shared/`)

Contains **reusable** components, directives, and pipes:

- UI components (buttons, modals, cards, tables)
- Custom pipes (date formatting, text transforms)
- Custom directives (click-outside, lazy-load)
- Shared models and interfaces
- Shared validators

Rules:
- Everything in shared must be **stateless** and **presentation-only**
- No services with state in shared (use core for that)
- Export barrel files (`index.ts`) for clean imports
- Each shared component gets its own directory

### Feature Modules/Directories (`features/`)

Encapsulate **business domains**:

Rules:
- Each feature is a **self-contained unit** with its own routing
- Features are **lazy-loaded** for performance
- Features should not depend on other features directly
- Use services or a state management solution for cross-feature communication
- Smart (container) components vs Presentational (dumb) components pattern:
  - **Smart components**: handle data, call services, manage state
  - **Presentational components**: receive data via inputs, emit events via outputs

### Lazy Loading Configuration

#### Standalone (v17+):

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.routes')
      .then(m => m.USERS_ROUTES)
  },
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.component')
      .then(c => c.DashboardComponent)
  }
];
```

#### NgModules (v14-v16):

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.module')
      .then(m => m.UsersModule)
  }
];
```

### Barrel Exports (`index.ts`)

Create barrel exports **only for public APIs** of a feature or shared module:

```typescript
// shared/components/index.ts
export { ButtonComponent } from './button/button.component';
export { CardComponent } from './card/card.component';
export { ModalComponent } from './modal/modal.component';
```

Rules:
- Only export what other modules need to consume
- Do not create barrels for internal/private components
- Keep barrels flat (avoid re-exporting from nested barrels)

## Step 5: Component Best Practices

### Modern Component Pattern (v17+)

```typescript
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  styleUrl: './user-list.component.css',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CommonModule, RouterLink]
})
export class UserListComponent {
  private readonly userService = inject(UserService);

  readonly users = input.required<User[]>();
  readonly selectedUser = output<User>();

  protected readonly filteredUsers = computed(() =>
    this.users().filter(u => u.active)
  );
}
```

### Dependency Injection

- Use `inject()` function instead of constructor injection
- Use `providedIn: 'root'` for singleton services
- Use component-level providers for scoped instances
- Create `InjectionToken` for non-class dependencies

### Change Detection

- Always use `ChangeDetectionStrategy.OnPush` for better performance
- Use signals (`signal`, `computed`, `effect`) for reactive state
- Avoid mutating objects directly; create new references

## Step 6: Refactoring Checklist

When modularizing an existing project, follow this order:

1. **Create the directory structure** (`core/`, `shared/`, `features/`)
2. **Extract core services** (auth, http interceptors, guards) to `core/`
3. **Extract shared components** (reusable UI pieces) to `shared/`
4. **Group related components** into feature directories
5. **Set up lazy loading** for each feature
6. **Update imports** across the application
7. **Add barrel exports** where appropriate
8. **Verify circular dependencies** with `Grep("circular", glob="**/*")`
9. **Run tests** to ensure nothing broke: `npx ng test --watch=false`
10. **Run the application** to verify lazy loading works

## Step 7: Anti-Patterns to Avoid

- **God modules**: a single module with 50+ declarations
- **Cross-feature imports**: features importing directly from other features
- **Services in shared**: stateful services belong in `core/`, not `shared/`
- **Deep nesting**: more than 3-4 levels of nested directories
- **Generic filenames**: `utils.ts`, `helpers.ts`, `common.ts` — be specific
- **Circular dependencies**: feature A imports from feature B and vice versa
- **Barrel file bloat**: re-exporting everything including internals
- **Mixed concerns**: business logic inside components instead of services
- **Importing entire libraries**: use tree-shakeable imports

## Step 8: NgModule to Standalone Migration

When the project uses NgModules but Angular v14+, offer migration guidance:

### Migration Steps

1. **Start with leaf components** (no child components) — convert to standalone first
2. **Convert shared components** next — add `imports` array directly to `@Component`
3. **Convert feature modules** one at a time — replace module with routes file
4. **Update routing** — switch from `loadChildren` (module) to `loadComponent`/`loadChildren` (routes)
5. **Remove empty modules** — once all declarations are standalone, delete the `.module.ts`
6. **Keep hybrid approach** if full migration is too risky

### What Changes in Each Component

```typescript
// BEFORE (NgModule-based)
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html'
})
export class UserListComponent { }

// AFTER (Standalone)
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  imports: [CommonModule, RouterLink, UserCardComponent]
})
export class UserListComponent { }
```

### What NOT to Migrate

- Third-party modules that don't support standalone yet
- Complex modules with many providers — migrate providers to `app.config.ts` first
- Modules used as library entry points

## Step 9: Unit Testing

Every moved or created file must include a corresponding `.spec.ts` file:

- Components: test rendering, inputs, outputs, and user interactions
- Services: test public methods, HTTP calls with `HttpTestingController`
- Guards: test route access conditions
- Pipes: test transformation logic
- After restructuring, run all tests to verify nothing broke

## Additional Guidelines

- Always present the **before and after** structure to the user
- Ask for confirmation before moving files
- Preserve git history when possible (use `git mv` for moves)
- Update all import paths after restructuring
- Check `tsconfig.json` for path aliases and update them
- Verify `angular.json` references are still correct after moves
