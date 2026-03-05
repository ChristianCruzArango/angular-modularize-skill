---
name: angular-modularize
description: >
  Modularizes existing Angular components and projects following official best
  practices, SOLID principles, and the Smart/Presentational pattern. Use when
  the user asks to "modularize", "refactor component", "split component",
  "extract component", "decompose component", "organize Angular project",
  "apply Angular best practices", "split into feature modules", "restructure
  project", or "improve project structure".
allowed-tools: Read, Grep, Glob, Edit, Write, Bash, Agent
---

# Angular Modularization — Universal Best Practices

You are an expert Angular architect. These guidelines are **universal** and apply
to any Angular project regardless of domain or size. Always adapt to the Angular
version detected in the project.

---

## Step 1: Analyze Before Changing

Before making ANY change:

1. **Read `package.json`** → identify `@angular/core` version
2. **Scan structure** → `Glob("**/*.component.ts")`, `Glob("**/*.module.ts")`, `Glob("**/*.routes.ts")`
3. **Detect standalone** → `Grep("standalone", glob="**/*.component.ts")`
4. **Identify large components** → read `.ts` and `.html` files, count lines, list responsibilities
5. **Map dependencies** → which components use which services, check imports
6. **Present findings** to the user and **ask confirmation** before proceeding

---

## Step 2: Identify Components That Need Modularization

### Single Responsibility Principle (SRP)

> "Each class should have only ONE reason to change." — SOLID

Before each code change, ask: **what responsibilities does this component have?**
If you can describe it with more than one "AND", it needs refactoring.

**Example violation:** "This component fetches users AND renders the list AND handles
the create form AND validates input AND manages pagination."

### Smell Indicators — When a Component MUST Be Split

| Indicator | Threshold | Action |
|-----------|-----------|--------|
| Template (HTML) lines | > 100 lines | Extract child components |
| Component class lines | > 200 lines | Extract logic to services |
| File total lines | > 400 lines | Split responsibilities (Angular style guide rule) |
| Constructor/inject dependencies | > 5 | Component has too many concerns |
| Distinct `@if` sections (or `*ngIf` in legacy code) | > 3 unrelated blocks | Each block → child component |
| Distinct `@for` loops (or `*ngFor` in legacy code) | > 2 different data sources | Each list → child component |
| Component fetches data AND renders it | Mixed smart/presentational | Split into container + presentational |
| Repeated HTML blocks | Same block appears 2+ times | Extract to shared component |
| Multiple form groups | > 2 in same component | Each form section → child component |
| Component name is generic | `MainComponent`, `PageComponent` | Rename to reflect single responsibility |

### When a Component Should NOT Be Split

| Criteria | Reason |
|----------|--------|
| Template < 50 lines with single responsibility | Already small enough |
| Component used only once with no children | No reuse benefit, splitting adds overhead |
| Simple wrapper (< 3 inputs, no logic) | Already granular |
| Splitting requires passing > 5 inputs down | Creates prop drilling (worse than monolith) |
| Component tightly coupled to parent lifecycle | Splitting breaks interaction model |
| Feature has < 3 files total | Module overhead exceeds benefit |
| App has < 10 components total | Over-engineering for small apps |

---

## Step 3: Decompose a Monolithic Component

This is the **core process** for modularizing an existing component.

### 3.1 List Every Responsibility

Read the component and enumerate what it does:

```
Analysis of a monolithic OrderPageComponent:
1. Fetches order data from API          → Extract to OrderService
2. Displays order header info           → Extract to OrderHeaderComponent (presentational)
3. Renders line items table             → Extract to OrderItemsTableComponent (presentational)
4. Handles payment form                 → Extract to PaymentFormComponent (presentational)
5. Shows order status timeline          → Extract to OrderTimelineComponent (presentational)
6. Manages form validation              → Extract to OrderFormService or validators file
7. Handles error/loading states         → Keep in smart component, pass state to children
```

### 3.2 Classify: Smart vs Presentational

| Type | Characteristics | Rules |
|------|----------------|-------|
| **Smart (Container)** | Injects services, fetches data, manages state, coordinates children | One per route/feature. Knows WHERE data comes from. |
| **Presentational (Dumb)** | Receives data via `input()`, emits events via `output()`, renders UI only | No service injection. No business logic. Reusable across contexts. Does NOT know where data comes from. |

**Key test:** If the component wouldn't work in a completely different application
without modification → it's smart. If it would → it's presentational.

### 3.3 Classify Each Responsibility

| Responsibility Type | Destination | Rule |
|---------------------|------------|------|
| API calls, data fetching | Service in `services/` | Never in presentational components |
| Business logic, calculations | Service or utility file | Components should not contain business rules |
| Reusable UI block | `shared/components/` | If used in 2+ features |
| Feature-specific UI block | Child component in same feature | If used only here |
| Form validation rules | Validators file or service | Keep testable and reusable |
| State management | Service with signals or BehaviorSubject | Components are NOT the source of truth |

### 3.4 Extract — Before and After

**BEFORE — Monolithic (violates SRP, uses legacy patterns to show what needs refactoring):**

```typescript
// BEFORE: legacy patterns — *ngFor, *ngIf, constructor injection, no OnPush, no signals
@Component({
  selector: 'app-order-page',
  template: `
    <!-- Responsibility 1: Order header -->
    <div class="header">
      <h1>{{ order?.title }}</h1>
      <span>{{ order?.status }}</span>
      <p>{{ order?.date | date:'longDate' }}</p>
    </div>

    <!-- Responsibility 2: Items table -->
    <table>
      <tr *ngFor="let item of order?.items">
        <td>{{ item.name }}</td>
        <td>{{ item.price | currency }}</td>
        <td>{{ item.quantity }}</td>
      </tr>
      <tr>
        <td colspan="2">Total</td>
        <td>{{ calculateTotal() | currency }}</td>
      </tr>
    </table>

    <!-- Responsibility 3: Payment form -->
    <form [formGroup]="paymentForm" (ngSubmit)="submitPayment()">
      <input formControlName="cardNumber" placeholder="Card number">
      <input formControlName="expDate" placeholder="MM/YY">
      <input formControlName="cvv" placeholder="CVV">
      <div *ngIf="paymentForm.get('cardNumber')?.errors?.['required']">
        Card number is required
      </div>
      <button type="submit" [disabled]="paymentForm.invalid">Pay</button>
    </form>
  `
})
export class OrderPageComponent implements OnInit {
  order: Order | null = null;
  paymentForm: FormGroup;

  constructor(
    private http: HttpClient,
    private route: ActivatedRoute,
    private fb: FormBuilder,
    private router: Router,
    private notificationService: NotificationService
  ) {
    this.paymentForm = this.fb.group({
      cardNumber: ['', [Validators.required, Validators.minLength(16)]],
      expDate: ['', Validators.required],
      cvv: ['', [Validators.required, Validators.minLength(3)]]
    });
  }

  ngOnInit(): void {
    const id = this.route.snapshot.params['id'];
    this.http.get<Order>(`/api/orders/${id}`).subscribe(order => {
      this.order = order;
    });
  }

  calculateTotal(): number {
    return this.order?.items.reduce((sum, item) => sum + item.price * item.quantity, 0) ?? 0;
  }

  submitPayment(): void {
    if (this.paymentForm.valid) {
      this.http.post('/api/payments', {
        orderId: this.order?.id,
        ...this.paymentForm.value
      }).subscribe({
        next: () => {
          this.notificationService.success('Payment completed');
          this.router.navigate(['/orders']);
        },
        error: () => this.notificationService.error('Payment failed')
      });
    }
  }
}
```

**AFTER — Modularized (each piece has ONE responsibility):**

```typescript
// order.service.ts — Data fetching responsibility
@Injectable({ providedIn: 'root' })
export class OrderService {
  private readonly http = inject(HttpClient);

  loadOrder(id: string): Observable<Order> {
    return this.http.get<Order>(`/api/orders/${id}`);
  }

  submitPayment(orderId: string, payment: PaymentData): Observable<PaymentResult> {
    return this.http.post<PaymentResult>('/api/payments', { orderId, ...payment });
  }
}
```

```typescript
// order-header.component.ts — Presentational (display only)
@Component({
  selector: 'app-order-header',
  template: `
    <div class="header">
      <h1>{{ order().title }}</h1>
      <span class="status">{{ order().status }}</span>
      <p>{{ order().date | date:'longDate' }}</p>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [DatePipe]
})
export class OrderHeaderComponent {
  readonly order = input.required<Order>();
}
```

```typescript
// order-items-table.component.ts — Presentational
@Component({
  selector: 'app-order-items-table',
  template: `
    <table>
      @for (item of items(); track item.id) {
        <tr>
          <td>{{ item.name }}</td>
          <td>{{ item.price | currency }}</td>
          <td>{{ item.quantity }}</td>
        </tr>
      }
      <tr>
        <td colspan="2">Total</td>
        <td>{{ total() | currency }}</td>
      </tr>
    </table>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [CurrencyPipe]
})
export class OrderItemsTableComponent {
  readonly items = input.required<OrderItem[]>();
  protected readonly total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
}
```

```typescript
// payment-form.component.ts — Presentational (form UI + validation display)
@Component({
  selector: 'app-payment-form',
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="cardNumber" placeholder="Card number">
      <input formControlName="expDate" placeholder="MM/YY">
      <input formControlName="cvv" placeholder="CVV">
      @if (form.get('cardNumber')?.errors?.['required']) {
        <div class="error">Card number is required</div>
      }
      <button type="submit" [disabled]="form.invalid || submitting()">Pay</button>
    </form>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [ReactiveFormsModule]
})
export class PaymentFormComponent {
  private readonly fb = inject(FormBuilder);

  readonly submitting = input<boolean>(false);
  readonly paymentSubmitted = output<PaymentData>();

  protected readonly form = this.fb.group({
    cardNumber: ['', [Validators.required, Validators.minLength(16)]],
    expDate: ['', Validators.required],
    cvv: ['', [Validators.required, Validators.minLength(3)]]
  });

  protected onSubmit(): void {
    if (this.form.valid) {
      this.paymentSubmitted.emit(this.form.value as PaymentData);
    }
  }
}
```

```typescript
// order-page.component.ts — Smart/Container (orchestrates everything)
@Component({
  selector: 'app-order-page',
  template: `
    @if (order(); as order) {
      <app-order-header [order]="order" />
      <app-order-items-table [items]="order.items" />
      <app-payment-form
        [submitting]="submitting()"
        (paymentSubmitted)="onPaymentSubmit($event)" />
    } @else {
      <p>Loading order...</p>
    }
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [OrderHeaderComponent, OrderItemsTableComponent, PaymentFormComponent]
})
export class OrderPageComponent {
  private readonly orderService = inject(OrderService);
  private readonly route = inject(ActivatedRoute);
  private readonly router = inject(Router);
  private readonly notificationService = inject(NotificationService);

  protected readonly order = signal<Order | null>(null);
  protected readonly submitting = signal(false);

  constructor() {
    const id = this.route.snapshot.params['id'];
    this.orderService.loadOrder(id).subscribe(order => this.order.set(order));
  }

  protected onPaymentSubmit(payment: PaymentData): void {
    this.submitting.set(true);
    this.orderService.submitPayment(this.order()!.id, payment).subscribe({
      next: () => {
        this.notificationService.success('Payment completed');
        this.router.navigate(['/orders']);
      },
      error: () => {
        this.notificationService.error('Payment failed');
        this.submitting.set(false);
      }
    });
  }
}
```

### 3.5 Handle Shared State Between Split Components

| Pattern | When to Use |
|---------|------------|
| Parent passes data via `input()` | Simple data flow, < 3 levels deep |
| Shared service with signals | Multiple siblings need same reactive state |
| `output()` events up + `input()` down | Parent orchestrates child interactions |
| NgRx / signal store | Complex state with many consumers across features |

**Rule:** Never create a service just to avoid passing one input. Use services when
3+ components need the same state or when prop drilling exceeds 3 levels.

---

## Step 4: Project-Level Modularization

### Directory Structure by Angular Version

#### Angular v17+ (Standalone)

```
src/app/
├── app.component.ts
├── app.config.ts
├── app.routes.ts
├── core/                          # Singletons — imported ONCE at root
│   ├── interceptors/
│   ├── guards/
│   └── services/
├── shared/                        # Reusable, STATELESS components
│   ├── components/
│   ├── directives/
│   ├── pipes/
│   └── models/
└── features/                      # Lazy-loaded business domains
    └── users/
        ├── user-list/
        │   ├── user-list.component.ts
        │   ├── user-list.component.html
        │   ├── user-list.component.spec.ts
        │   └── components/        # Child presentational components
        │       └── user-card/
        ├── user-detail/
        ├── services/
        ├── models/
        └── users.routes.ts
```

#### Angular v14-v16 (NgModules)

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

### Module Boundary Rules

| Module | Contains | Rules |
|--------|----------|-------|
| **Core** | Singletons: auth service, interceptors, guards, app-wide services | `providedIn: 'root'`. Imported ONCE at root. Never in features. |
| **Shared** | Reusable UI: buttons, tables, pipes, directives, models | **Stateless only**. No services with state. Barrel exports for public API. |
| **Feature** | Business domain: components, routes, feature services, models | Self-contained. Lazy-loaded. No cross-feature imports. Smart + presentational pattern. |

### When TO Create a Feature Module/Directory

- Feature has its own route (`/users`, `/orders`)
- 3+ related components work together
- Feature benefits from lazy loading
- Different team owns the feature
- Business domain is clearly bounded

### When NOT to Create a Feature Module/Directory

- App has < 10 components total
- Single component with no children or subroutes
- Splitting would create circular dependencies
- Feature always loads at startup (no lazy loading benefit)

---

## Step 5: Lazy Loading

### Standalone (v17+)

```typescript
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

### NgModules (v14-v16)

```typescript
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.module')
      .then(m => m.UsersModule)
  }
];
```

### Deferred Loading with `@defer` (v17+)

For heavy components within a page:

```html
@defer (on viewport) {
  <app-heavy-chart [data]="chartData()" />
} @placeholder {
  <div class="skeleton">Loading chart...</div>
}
```

---

## Step 6: Component Best Practices

### Modern Pattern (v17+ Standalone)

```typescript
@Component({
  selector: 'app-user-list',
  templateUrl: './user-list.component.html',
  styleUrl: './user-list.component.css',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [DatePipe, RouterLink]
})
export class UserListComponent {
  // Inputs/Outputs first
  readonly users = input.required<User[]>();
  readonly userSelected = output<User>();

  // Computed state
  protected readonly activeUsers = computed(() =>
    this.users().filter(u => u.active)
  );
}
```

### Key Rules

- **`ChangeDetectionStrategy.OnPush`** on ALL components — avoids unnecessary re-renders
- **`inject()` function** instead of constructor injection — better type inference
- **Signals** (`signal`, `computed`, `effect`) for reactive state
- **`input()` / `output()`** signal-based APIs for component communication (v17+)
- **Never mutate inputs** — create new references, Angular uses `===` equality check
- **Template control flow** — `@if`, `@for`, `@switch` instead of `*ngIf`, `*ngFor` (v17+)
- **Co-locate files** — `.ts`, `.html`, `.css`, `.spec.ts` in the same directory

---

## Step 7: Naming Conventions

- **kebab-case** for file names: `user-profile.component.ts`
- **Pattern**: `<name>.<type>.ts`
- **Suffixes**: `Component`, `Service`, `Directive`, `Pipe`, `Guard`, `Interceptor`
- **One concept per file** — never multiple components in one file
- **Avoid generic names** — `utils.ts`, `helpers.ts`, `common.ts` → be specific
- **Barrel exports** (`index.ts`) only for public APIs, never for internals

---

## Step 8: NgModule → Standalone Migration

When project uses NgModules and Angular v14+:

1. **Leaf components first** (no children) → add `standalone: true` + `imports` array
2. **Shared components** next → convert and update all consumers
3. **Feature modules** one at a time → replace `.module.ts` with `.routes.ts`
4. **Update routing** → `loadChildren` with module → `loadComponent`/`loadChildren` with routes
5. **Remove empty modules** → delete `.module.ts` when all declarations are standalone
6. **Hybrid is OK** → not everything must be migrated at once

### Do NOT Migrate

- Third-party modules without standalone support
- Modules with complex providers → migrate providers to `app.config.ts` first
- Library entry point modules

---

## Step 9: Anti-Patterns

| Anti-Pattern | Why It's Bad | Fix |
|-------------|-------------|-----|
| God component (> 200 lines class) | Violates SRP, hard to test | Split into smart + presentational |
| God module (50+ declarations) | Hard to navigate, slow compilation | Split into feature modules |
| Prop drilling (> 3 levels) | Fragile, hard to refactor | Use shared service with signals |
| Cross-feature imports | Creates coupling, breaks lazy loading | Communicate via shared services |
| Services in `shared/` | Shared should be stateless | Move stateful services to `core/` |
| Business logic in templates | Untestable, hard to read | Move to computed signals or methods |
| Fat templates (> 100 lines) | Multiple responsibilities in one view | Extract child components |
| Generic file names | Hard to find and understand | Name by specific responsibility |
| Circular dependencies | Build failures, tight coupling | Restructure boundaries |
| Mixed concerns in component | Fetching + rendering + validation | Separate into service + smart + presentational |

---

## Step 10: Refactoring Checklist

Execute in this order:

1. **Identify large components** — read all `.html` and `.ts` files, list candidates
2. **Split monolithic components** — apply Step 3 for each candidate
3. **Create directory structure** — `core/`, `shared/`, `features/`
4. **Extract core services** — auth, interceptors, guards → `core/`
5. **Extract shared components** — reusable UI → `shared/`
6. **Group into features** — related components → `features/<name>/`
7. **Set up lazy loading** — configure routes for each feature
8. **Update all imports** — fix paths across the application
9. **Add barrel exports** — only for public APIs
10. **Build the project** — `npx ng build` to detect circular dependencies and errors
11. **Run ALL tests** — `npx ng test --watch=false`
12. **Serve and verify** — `npx ng serve`, test navigation and lazy loading

---

## Step 11: Unit Testing

Every new or moved file must have a corresponding `.spec.ts`:

- **Presentational components**: test rendering with different inputs, output emissions, user interactions
- **Smart components**: test service calls, state management, child component coordination
- **Services**: test public methods, HTTP calls with `HttpTestingController`
- **Guards**: test access conditions, redirects
- **Pipes**: test transformations with edge cases
- After restructuring, run ALL existing tests to verify nothing broke

---

## Important Reminders

- **Always present before/after** structure to the user
- **Ask confirmation** before moving files or splitting components
- **Preserve git history** — use `git mv` for file moves
- **Update `tsconfig.json`** path aliases after restructuring
- **Verify `angular.json`** references after file moves
- **Adapt to Angular version** — these practices apply universally but syntax differs per version
- **Do not over-modularize** — if splitting adds more complexity than it solves, don't split
