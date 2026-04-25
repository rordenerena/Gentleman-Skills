---
name: angular-primeng
description: Best practices for Angular 21 + PrimeNG enterprise application development. Suitable for enterprise-grade applications such as MES and ERP. Covers component design, state management, performance optimization, and PrimeNG component usage conventions.
source: custom
updated: 2025-01-16
---

# Angular 21 + PrimeNG Development Guidelines

## Applicable Scenarios

- MES Manufacturing Execution Systems
- ERP Enterprise Resource Planning
- Admin management systems
- Data-intensive applications

## Core Principles

### 1. Project Structure

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

### 2. PrimeNG Configuration (`app.config.ts`)

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

### 3. PrimeNG Component Reference

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

### 4. Signals State Management (Angular 21)

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

### 5. Service Layer Design

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

### 6. API Response Format

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

### 7. Form Handling

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

### 8. Table Best Practices

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

### 9. Drawer Sidebar Pattern

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

### 10. Error Handling

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

## Prohibited Practices

1. **No fallback mock data** - If the API fails, show an error and do not use fake data.
2. **No emoji** - Do not use emoji in code, comments, or UI.
3. **No Simplified Chinese** - When Chinese text is needed, use Traditional Chinese (Taiwan terminology).
4. **No single file over 500 lines** - Split files when they exceed this limit.

## Performance Optimization

1. **OnPush change detection** - Use together with Signals.
2. **trackBy** - Required for table lists.
3. **Lazy loading** - Feature modules should be lazy-loaded.
4. **Virtual scrolling** - Use `<p-scroller>` for large datasets.

## Reference Resources

- [Angular Style Guide](https://angular.dev/style-guide)
- [PrimeNG Documentation](https://primeng.org/)
- [Angular Signals](https://angular.dev/guide/signals)
