# Frontend — Angular

UI text below is English placeholder copy — use the project's UI language from `project-description.md`. Use Syncfusion components where they add value (grids, date pickers, dialogs); plain markup keeps this example short.

## ApiService — `core/services/api.service.ts` (created once)

```ts
import { HttpClient, HttpParams } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';

type QueryParams = Record<string, string | number | boolean>;

/** Single entry point for HTTP calls. Feature services use this, never HttpClient directly. */
@Injectable({ providedIn: 'root' })
export class ApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = environment.apiUrl; // e.g. https://localhost:5001/api/v1

  get<T>(path: string, params?: QueryParams): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${path}`, { params: new HttpParams({ fromObject: params ?? {} }) });
  }

  post<T>(path: string, body: unknown): Observable<T> {
    return this.http.post<T>(`${this.baseUrl}/${path}`, body);
  }

  put<T>(path: string, body: unknown): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}/${path}`, body);
  }

  patch<T>(path: string, body: unknown): Observable<T> {
    return this.http.patch<T>(`${this.baseUrl}/${path}`, body);
  }
}
```

## Model — `features/<area>/products/product.model.ts`

Plain interfaces and types, not classes.

```ts
export type ProductStatus = 'Draft' | 'Published' | 'Archived';

export interface Product {
  id: string;
  name: string;
  price: number;
  status: ProductStatus;
  createdOn: string;
}

export interface PagedResult<T> {
  items: T[];
  page: number;
  pageSize: number;
  totalCount: number;
}
```

## Feature service — `features/<area>/products/product.service.ts`

```ts
import { Injectable, inject } from '@angular/core';
import { ApiService } from '../../../core/services/api.service';
import { PagedResult, Product } from './product.model';

@Injectable({ providedIn: 'root' })
export class ProductService {
  private readonly api = inject(ApiService);

  list(page = 1, pageSize = 20) {
    return this.api.get<PagedResult<Product>>('products', { page, pageSize });
  }

  publish(id: string) {
    return this.api.post<Product>(`products/${id}/publish`, {});
  }
}
```

## Page — `features/<area>/products/pages/product-list/product-list.page.ts`

```ts
import { CurrencyPipe } from '@angular/common';
import { ChangeDetectionStrategy, Component, DestroyRef, computed, inject, signal } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { ToastService } from '../../../../../shared/services/toast.service';
import { Product } from '../../product.model';
import { ProductService } from '../../product.service';

@Component({
  selector: 'app-product-list',
  imports: [CurrencyPipe],
  templateUrl: './product-list.page.html',
  styleUrl: './product-list.page.scss',
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class ProductListPage {
  private readonly products = inject(ProductService);
  private readonly toast = inject(ToastService);
  private readonly destroyRef = inject(DestroyRef);

  protected readonly items = signal<Product[]>([]);
  protected readonly loading = signal(true);
  protected readonly error = signal<string | null>(null);
  protected readonly isEmpty = computed(() => !this.loading() && !this.error() && this.items().length === 0);

  constructor() {
    this.load();
  }

  protected load(): void {
    this.loading.set(true);
    this.error.set(null);

    this.products
      .list()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe({
        next: (result) => {
          this.items.set(result.items);
          this.loading.set(false);
        },
        error: () => {
          this.error.set('Could not load products.');
          this.loading.set(false);
        },
      });
  }

  protected publish(product: Product): void {
    this.products
      .publish(product.id)
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe({
        next: (updated) => {
          this.items.update((list) => list.map((p) => (p.id === updated.id ? updated : p)));
          this.toast.success('Product published.');
        },
        error: () => this.toast.error('Could not publish the product.'),
      });
  }
}
```

## Template — `product-list.page.html`

```html
<h1>Products</h1>

@if (loading()) {
  <p role="status">Loading…</p>
} @else if (error()) {
  <div role="alert">
    <p>{{ error() }}</p>
    <button type="button" (click)="load()">Try again</button>
  </div>
} @else if (isEmpty()) {
  <p>No products yet.</p>
} @else {
  <ul aria-label="Products">
    @for (product of items(); track product.id) {
      <li>
        <span>{{ product.name }}</span>
        <!-- Amount as returned by the API; currency/locale from the project settings -->
        <span>{{ product.price | currency }}</span>
        @if (product.status === 'Draft') {
          <button type="button" (click)="publish(product)" [attr.aria-label]="'Publish ' + product.name">Publish</button>
        }
      </li>
    }
  </ul>
}
```

## Route — `features/<area>/<area>.routes.ts`

```ts
export const AREA_ROUTES: Routes = [
  {
    path: 'products',
    canActivate: [roleGuard('Partner')], // The project's role guard
    loadComponent: () => import('./products/pages/product-list/product-list.page').then((m) => m.ProductListPage),
  },
];
```

## Component test (Vitest) — `product-list.page.spec.ts`

```ts
import { TestBed } from '@angular/core/testing';
import { of, throwError } from 'rxjs';
import { describe, expect, it, vi } from 'vitest';
import { ToastService } from '../../../../../shared/services/toast.service';
import { ProductService } from '../../product.service';
import { ProductListPage } from './product-list.page';

function setup(list: ReturnType<typeof vi.fn>) {
  TestBed.configureTestingModule({
    imports: [ProductListPage],
    providers: [
      { provide: ProductService, useValue: { list, publish: vi.fn() } },
      { provide: ToastService, useValue: { success: vi.fn(), error: vi.fn() } },
    ],
  });
  const fixture = TestBed.createComponent(ProductListPage);
  fixture.detectChanges();
  return fixture;
}

describe('ProductListPage', () => {
  it('shows the empty state when the API returns no products', async () => {
    const fixture = setup(vi.fn(() => of({ items: [], page: 1, pageSize: 20, totalCount: 0 })));
    await fixture.whenStable();

    expect(fixture.nativeElement.textContent).toContain('No products yet.');
  });

  it('shows the error state when the API fails', async () => {
    const fixture = setup(vi.fn(() => throwError(() => new Error('500'))));
    await fixture.whenStable();

    expect(fixture.nativeElement.querySelector('[role="alert"]')).not.toBeNull();
  });
});
```
