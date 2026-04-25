---
name: primeng
description: >
  PrimeNG +19 patterns for Angular +19 enterprise applications with standalone bootstrapping,
  Signals state management, reactive forms, data tables, drawers, and service-layer
  conventions.
  Trigger: When building Angular apps with PrimeNG, configuring PrimeNG providers and
  themes, or generating data-heavy admin, MES, and ERP interfaces.
license: Apache-2.0
metadata:
  author: seikaikyo
  contributor: rordenerena
  version: "1.0"
---

## When to Use

Load this skill when:
- Building Angular +19 applications with PrimeNG +19
- Configuring `providePrimeNG`, themes, ripple, or CSS layer ordering
- Implementing admin, MES, ERP, or other data-heavy business screens
- Generating PrimeNG forms, tables, drawers, tags, toasts, and loading states

## Critical Patterns

### Pattern 1: Project Structure

```text
src/app/
├── core/                    # Core modules (singleton services)
│   ├── auth/               # Authentication-related
│   ├── guards/             # Route guards
│   ├── interceptors/       # HTTP interceptors
│   ├── models/             # Data models
│   └── services/           # Shared services
├── shared/                  # Shared modules
│   ├── components/         # Reusable components
│   ├── directives/         # Custom directives
│   ├── pipes/              # Pipes
│   └── utils/              # Utility functions
├── features/               # Feature modules
│   ├── dashboard/
│   ├── production/
│   └── settings/
└── layout/                 # Layouts
```

### Pattern 2: PrimeNG Configuration (`app.config.ts`)

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';
import { providePrimeNG } from 'primeng/config';
import Aura from '@primeng/themes/aura';

export const appConfig: ApplicationConfig = {
  providers: [
    provideAnimationsAsync(),
    providePrimeNG({
      theme: {
        preset: Aura,
        options: {
          darkModeSelector: '.dark-mode',
          cssLayer: {
            name: 'primeng',
            order: 'tailwind-base, primeng, tailwind-utilities'
          }
        }
      },
      ripple: true
    })
  ]
};
```

### Pattern 3: PrimeNG Component Reference

| Purpose | Component | Example |
|------|------|------|
| Form/details sidebar | `<p-drawer>` | Edit work orders, create customers |
| Confirmation dialog | `<p-dialog>` | Delete confirmation, warning messages |
| Data table | `<p-table>` | Work order lists, reports |
| Button | `<p-button>` | All button actions |
| Text input | `<input pInputText>` | Used with `ngModel` |
| Dropdown | `<p-select>` | Status selection, categories |
| Date picker | `<p-datepicker>` | Date range filters |
| Tag/badge | `<p-tag>` | Status indicators |
| Toast notifications | `<p-toast>` | Action feedback |
| Loading overlay | `<p-blockui>` | Asynchronous operations |

### Pattern 4: Signals State Management (Angular 19)

```typescript
// Recommended: use Signals
import { signal, computed, effect } from '@angular/core';

@Component({...})
export class WorkOrderListComponent {
  // State
  workOrders = signal<WorkOrder[]>([]);
  selectedId = signal<string | null>(null);
  isLoading = signal(false);

  // Computed properties
  selectedOrder = computed(() =>
    this.workOrders().find(o => o.id === this.selectedId())
  );

  pendingCount = computed(() =>
    this.workOrders().filter(o => o.status === 'pending').length
  );

  // Side effects
  constructor() {
    effect(() => {
      console.log('Selected work order:', this.selectedOrder());
    });
  }
}
```

### Pattern 5: Service Layer Design

```typescript
@Injectable({ providedIn: 'root' })
export class WorkOrderService {
  private apiUrl = environment.apiUrl + '/work-orders';

  constructor(private http: HttpClient) {}

  // List query (supports filtering)
  getList(params?: WorkOrderQueryParams): Observable<ApiResponse<WorkOrder[]>> {
    return this.http.get<ApiResponse<WorkOrder[]>>(this.apiUrl, { params });
  }

  // Single item query
  getById(id: string): Observable<ApiResponse<WorkOrder>> {
    return this.http.get<ApiResponse<WorkOrder>>(`${this.apiUrl}/${id}`);
  }

  // Create
  create(data: CreateWorkOrderDto): Observable<ApiResponse<WorkOrder>> {
    return this.http.post<ApiResponse<WorkOrder>>(this.apiUrl, data);
  }

  // Update
  update(id: string, data: UpdateWorkOrderDto): Observable<ApiResponse<WorkOrder>> {
    return this.http.put<ApiResponse<WorkOrder>>(`${this.apiUrl}/${id}`, data);
  }

  // Delete
  delete(id: string): Observable<ApiResponse<void>> {
    return this.http.delete<ApiResponse<void>>(`${this.apiUrl}/${id}`);
  }
}
```

### Pattern 6: API Response Format

```typescript
// Unified response format
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
}

// Paginated response
interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: {
    total: number;
    page: number;
    limit: number;
  };
}
```

### Pattern 7: Form Handling

```typescript
// Recommended: Reactive Forms + PrimeNG
@Component({
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div class="field">
        <label for="orderNumber">Work Order Number</label>
        <input pInputText id="orderNumber" formControlName="orderNumber" />
        <small *ngIf="form.get('orderNumber')?.errors?.['required']" class="p-error">
          Required field
        </small>
      </div>

      <div class="field">
        <label for="customer">Customer</label>
        <p-select
          id="customer"
          formControlName="customerId"
          [options]="customers()"
          optionLabel="name"
          optionValue="id"
          placeholder="Select customer"
        />
      </div>

      <div class="field">
        <label for="dueDate">Due Date</label>
        <p-datepicker
          id="dueDate"
          formControlName="dueDate"
          dateFormat="yy-mm-dd"
        />
      </div>

      <p-button type="submit" label="Save" [loading]="isSubmitting()" />
    </form>
  `
})
export class WorkOrderFormComponent {
  form = new FormGroup({
    orderNumber: new FormControl('', Validators.required),
    customerId: new FormControl('', Validators.required),
    dueDate: new FormControl<Date | null>(null)
  });

  customers = signal<Customer[]>([]);
  isSubmitting = signal(false);
}
```

### Pattern 8: Table Best Practices

```typescript
@Component({
  template: `
    <p-table
      [value]="workOrders()"
      [paginator]="true"
      [rows]="20"
      [rowsPerPageOptions]="[10, 20, 50]"
      [loading]="isLoading()"
      [globalFilterFields]="['orderNumber', 'customerName']"
      styleClass="p-datatable-sm"
    >
      <ng-template pTemplate="header">
        <tr>
          <th pSortableColumn="orderNumber">
            Work Order Number <p-sortIcon field="orderNumber" />
          </th>
          <th pSortableColumn="status">Status</th>
          <th>Actions</th>
        </tr>
      </ng-template>

      <ng-template pTemplate="body" let-order>
        <tr>
          <td>{{ order.orderNumber }}</td>
          <td>
            <p-tag [value]="order.status" [severity]="getStatusSeverity(order.status)" />
          </td>
          <td>
            <p-button icon="pi pi-pencil" [text]="true" (click)="edit(order)" />
            <p-button icon="pi pi-trash" [text]="true" severity="danger" (click)="confirmDelete(order)" />
          </td>
        </tr>
      </ng-template>

      <ng-template pTemplate="emptymessage">
        <tr>
          <td colspan="3" class="text-center">No data</td>
        </tr>
      </ng-template>
    </p-table>
  `
})
export class WorkOrderTableComponent {
  workOrders = signal<WorkOrder[]>([]);
  isLoading = signal(false);

  getStatusSeverity(status: string): 'success' | 'info' | 'warn' | 'danger' {
    const map: Record<string, 'success' | 'info' | 'warn' | 'danger'> = {
      completed: 'success',
      in_progress: 'info',
      pending: 'warn',
      cancelled: 'danger'
    };
    return map[status] || 'info';
  }
}
```

### Pattern 9: Drawer Sidebar Pattern

```typescript
@Component({
  template: `
    <p-drawer
      [(visible)]="drawerVisible"
      [header]="isEditMode() ? 'Edit Work Order' : 'New Work Order'"
      position="right"
      [style]="{ width: '500px' }"
      (onHide)="onClose()"
    >
      <app-work-order-form
        [workOrder]="selectedOrder()"
        (save)="onSave($event)"
        (cancel)="drawerVisible = false"
      />
    </p-drawer>
  `
})
export class WorkOrderListComponent {
  drawerVisible = false;
  selectedOrder = signal<WorkOrder | null>(null);

  isEditMode = computed(() => this.selectedOrder() !== null);

  openCreate() {
    this.selectedOrder.set(null);
    this.drawerVisible = true;
  }

  openEdit(order: WorkOrder) {
    this.selectedOrder.set(order);
    this.drawerVisible = true;
  }
}
```

### Pattern 10: Error Handling

```typescript
// HTTP interceptor
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  constructor(private messageService: MessageService) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        let message = 'An error occurred. Please try again later';

        if (error.status === 401) {
          message = 'Please sign in again';
        } else if (error.status === 403) {
          message = 'Insufficient permissions';
        } else if (error.status === 404) {
          message = 'Data not found';
        } else if (error.status === 503) {
          message = 'Service is temporarily unavailable';
        } else if (error.error?.error?.message) {
          message = error.error.error.message;
        }

        this.messageService.add({
          severity: 'error',
          summary: 'Error',
          detail: message
        });

        return throwError(() => error);
      })
    );
  }
}
```

## Code Examples

### Example 1: Standalone list filter with Signals

```typescript
import { Component, computed, signal } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { InputTextModule } from 'primeng/inputtext';
import { TableModule } from 'primeng/table';

interface WorkOrder {
  id: string;
  orderNumber: string;
  customerName: string;
}

@Component({
  selector: 'app-work-order-search',
  standalone: true,
  imports: [FormsModule, InputTextModule, TableModule],
  template: `
    <input
      pInputText
      [ngModel]="query()"
      (ngModelChange)="query.set($event)"
      placeholder="Search work orders"
    />

    <p-table [value]="filteredOrders()" [paginator]="true" [rows]="10">
      <ng-template pTemplate="header">
        <tr>
          <th>Order</th>
          <th>Customer</th>
        </tr>
      </ng-template>
      <ng-template pTemplate="body" let-order>
        <tr>
          <td>{{ order.orderNumber }}</td>
          <td>{{ order.customerName }}</td>
        </tr>
      </ng-template>
    </p-table>
  `
})
export class WorkOrderSearchComponent {
  query = signal('');
  orders = signal<WorkOrder[]>([]);

  filteredOrders = computed(() => {
    const term = this.query().trim().toLowerCase();

    if (!term) {
      return this.orders();
    }

    return this.orders().filter(order =>
      order.orderNumber.toLowerCase().includes(term) ||
      order.customerName.toLowerCase().includes(term)
    );
  });
}
```

### Example 2: Reactive form dialog action

```typescript
import { Component, inject, signal } from '@angular/core';
import { FormControl, FormGroup, ReactiveFormsModule, Validators } from '@angular/forms';
import { finalize } from 'rxjs';
import { ButtonModule } from 'primeng/button';
import { DialogModule } from 'primeng/dialog';
import { InputTextModule } from 'primeng/inputtext';

@Component({
  selector: 'app-customer-dialog',
  standalone: true,
  imports: [ReactiveFormsModule, ButtonModule, DialogModule, InputTextModule],
  template: `
    <p-dialog [(visible)]="visible" header="New customer" [modal]="true">
      <form [formGroup]="form" class="flex flex-col gap-3" (ngSubmit)="save()">
        <input pInputText formControlName="name" placeholder="Customer name" />
        <input pInputText formControlName="email" placeholder="Email" />
        <p-button type="submit" label="Save" [loading]="saving()" [disabled]="form.invalid" />
      </form>
    </p-dialog>
  `
})
export class CustomerDialogComponent {
  private readonly customerService = inject(CustomerService);

  visible = false;
  saving = signal(false);

  form = new FormGroup({
    name: new FormControl('', { nonNullable: true, validators: [Validators.required] }),
    email: new FormControl('', { nonNullable: true, validators: [Validators.email] })
  });

  save() {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.saving.set(true);

    this.customerService.create(this.form.getRawValue())
      .pipe(finalize(() => this.saving.set(false)))
      .subscribe(() => {
        this.visible = false;
        this.form.reset({ name: '', email: '' });
      });
  }
}
```

### Example 3: Toast-backed delete confirmation flow

```typescript
import { Component, inject } from '@angular/core';
import { ButtonModule } from 'primeng/button';
import { ConfirmDialogModule } from 'primeng/confirmdialog';
import { ConfirmationService, MessageService } from 'primeng/api';

@Component({
  selector: 'app-delete-action',
  standalone: true,
  imports: [ButtonModule, ConfirmDialogModule],
  providers: [ConfirmationService],
  template: `
    <p-confirmdialog />
    <p-button label="Delete" severity="danger" (click)="confirmDelete('WO-1001')" />
  `
})
export class DeleteActionComponent {
  private readonly confirmationService = inject(ConfirmationService);
  private readonly messageService = inject(MessageService);
  private readonly workOrderService = inject(WorkOrderService);

  confirmDelete(id: string) {
    this.confirmationService.confirm({
      header: 'Delete work order',
      message: 'This action cannot be undone.',
      accept: () => {
        this.workOrderService.delete(id).subscribe(() => {
          this.messageService.add({
            severity: 'success',
            summary: 'Deleted',
            detail: `Work order ${id} was removed`
          });
        });
      }
    });
  }
}
```

## Anti-Patterns

### Don't: Mix template-driven and reactive forms on the same flow

This makes validation and state transitions harder to reason about.

```typescript
// Bad example
form = new FormGroup({
  status: new FormControl('pending')
});

template = `
  <input pInputText [(ngModel)]="query" formControlName="status" />
`;
```

### Don't: Hide production API failures with fallback mock data

Business screens should surface the failure and preserve traceability.

```typescript
// Bad example
this.workOrderService.getList().subscribe({
  next: response => this.workOrders.set(response.data ?? []),
  error: () => this.workOrders.set([{ id: 'demo', orderNumber: 'MOCK-001' } as WorkOrder])
});
```

### Don't: Render large PrimeNG tables without explicit loading and scaling strategy

Large datasets need pagination, lazy loading, or virtual scrolling.

```html
<!-- Bad example -->
<p-table [value]="workOrders()">
  <ng-template pTemplate="body" let-order>
    <tr>
      <td>{{ order.orderNumber }}</td>
      <td>{{ order.customerName }}</td>
    </tr>
  </ng-template>
</p-table>
```

### Don't: Put HTTP orchestration directly inside reusable presentation components

Keep API calls in a service or container-level component so PrimeNG widgets stay focused on UI state.

```typescript
// Bad example
@Component({...})
export class WorkOrderTableComponent {
  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.http.get('/api/work-orders').subscribe();
  }
}
```

## Performance Optimization

1. **OnPush change detection** - Use together with Signals.
2. **trackBy** - Required for table lists.
3. **Lazy loading** - Feature modules should be lazy-loaded.
4. **Virtual scrolling** - Use `<p-scroller>` for large datasets.

## Commands

```bash
npm install primeng @primeng/themes primeicons @angular/cdk
```

## Resources

- [Angular Style Guide](https://angular.dev/style-guide)
- [PrimeNG Documentation](https://primeng.org/)
- [Angular Signals](https://angular.dev/guide/signals)
