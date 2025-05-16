Let’s make the necessary updates to the `HeaderComponent` to enhance the merchant experience while ensuring the existing logic for customers and other users remains unaffected. We’ll focus on updating the navbar dropdown to include merchant-specific options (as previously implemented) and refine the HTML structure to make it more intuitive and consistent for all user types. I’ll also ensure the `cartItemCount` updates in real-time by subscribing to the `cartService.cartUpdate$` observable (already implemented earlier).

### Changes to `header.component.ts`
We’ll add the `isMerchant` property to determine the user role and ensure the cart count updates in real-time for customers. The existing logic (e.g., dropdown closing, search, logout) will remain unchanged.

**Updated `header.component.ts`**:
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink, RouterLinkActive, Router, NavigationEnd } from '@angular/router';
import { AuthService } from '../../services/auth.service';
import { CartService } from '../../services/cart.service';

@Component({
  selector: 'app-header',
  standalone: true,
  imports: [CommonModule, RouterLink, RouterLinkActive],
  templateUrl: './header.component.html',
  styleUrls: ['./header.component.css']
})
export class HeaderComponent {
  searchQuery: string = '';
  isDropdownOpen: boolean = false;
  cartItemCount: number = 0;
  firstName: string | null = null;
  isMerchant: boolean = false; // Add property to identify merchant role

  constructor(public authService: AuthService, private router: Router, private cartService: CartService) {
    this.updateCartCount();
  }

  ngOnInit() {
    this.router.events.subscribe(event => {
      if (event instanceof NavigationEnd) {
        this.closeDropdown();
      }
    });

    if (this.authService.isLoggedIn()) {
      this.authService.viewProfile().subscribe({
        next: (response) => {
          if (response.success && response.data) {
            this.firstName = response.data.fullName.split(' ')[0];
          }
        },
        error: (err) => {
          console.error('Error fetching profile:', err);
          this.firstName = null;
        }
      });

      // Set user role
      this.isMerchant = this.authService.getUserRole() === 'Merchant';

      // Subscribe to cart updates for real-time cart count (for customers only)
      if (!this.isMerchant) {
        this.cartService.cartUpdate$.subscribe(count => {
          this.cartItemCount = count;
        });
      }
    }
  }

  search(event: Event) {
    const query = (event.target as HTMLInputElement).value;
    this.router.navigate(['/products'], { queryParams: { search: query } });
  }

  toggleDropdown(event?: Event) {
    if (event) {
      event.preventDefault();
    }
    this.isDropdownOpen = !this.isDropdownOpen;
  }

  closeDropdown() {
    this.isDropdownOpen = false;
  }

  logout() {
    this.authService.logout();
    this.isDropdownOpen = false;
  }

  updateCartCount() {
    if (this.authService.isLoggedIn() && this.authService.getUserRole() === 'Customer') {
      this.cartService.getCart().subscribe({
        next: (response) => {
          if (response.success && response.data) {
            this.cartItemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0); // Use total quantity
          }
        },
        error: (err) => {
          console.error('Error fetching cart count:', err);
        }
      });
    }
  }

  navigateTo(route: string) {
    this.router.navigate([route]);
    this.closeDropdown();
  }
}
```

#### Explanation of Changes in `header.component.ts`
- **Added `isMerchant` Property**: Determines if the logged-in user is a merchant, used to conditionally render merchant-specific options in the dropdown.
- **Real-Time Cart Count**: Subscribed to `cartService.cartUpdate$` in `ngOnInit` to ensure the cart count updates in real-time for customers (already implemented earlier, just ensuring it’s here).
- **Preserved Existing Logic**: All existing methods (`search`, `toggleDropdown`, `closeDropdown`, `logout`, `navigateTo`) remain unchanged to avoid affecting previous functionality.
- **Updated `updateCartCount`**: Changed to use `reduce` to sum the quantities of items (already implemented earlier, just ensuring consistency).

### Changes to `header.component.html`
We’ll update the HTML to:
- Improve the dropdown structure for better usability.
- Add merchant-specific options ("Dashboard", "Add Product", "Manage Products") while keeping customer-specific options ("Cart", "Place Order", "Order History").
- Ensure the layout is clean and consistent for all user types.
- Add icons for visual clarity (using emoji placeholders since UI will be polished later).

**Updated `header.component.html`**:
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
  <div class="container-fluid px-5">
    <!-- Brand -->
    <a class="navbar-brand fw-bold" [routerLink]="['/']">EShoppingZone 🛍️</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
      aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>

    <!-- Navbar Content -->
    <div class="collapse navbar-collapse" id="navbarNav">
      <!-- Navigation Links -->
      <ul class="navbar-nav me-auto mb-2 mb-lg-0 ms-3">
        <li class="nav-item">
          <a class="nav-link" [routerLink]="['/']" routerLinkActive="active">Home 🏠</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" [routerLink]="['/products']" routerLinkActive="active">Products 📦</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" [routerLink]="['/deals']" routerLinkActive="active">Deals 🎉</a>
        </li>
      </ul>

      <!-- Search Bar -->
      <div class="d-flex flex-grow-1 justify-content-center mx-3">
        <div class="input-group w-50">
          <span class="input-group-text bg-light">
            <i class="bi bi-search"></i>
          </span>
          <input type="text" class="form-control" placeholder="Search for products..." (input)="search($event)">
        </div>
      </div>

      <!-- User Actions -->
      <div class="navbar-nav ms-auto">
        <ng-container *ngIf="!authService.isLoggedIn(); else loggedIn">
          <li class="nav-item">
            <a class="nav-link btn btn-outline-light me-2 login-signup-btn" [routerLink]="['/login']">Login 🔑</a>
          </li>
          <li class="nav-item">
            <a class="nav-link btn btn-outline-light login-signup-btn" [routerLink]="['/register']">Signup 📝</a>
          </li>
        </ng-container>

        <ng-template #loggedIn>
          <!-- Cart Link for Customers -->
          <li class="nav-item" *ngIf="!isMerchant">
            <a class="nav-link text-white" [routerLink]="['/cart']">
              Cart 🛒
              <span *ngIf="cartItemCount > 0" class="badge bg-danger ms-1">{{ cartItemCount }}</span>
            </a>
          </li>

          <!-- Profile Dropdown -->
          <li class="nav-item dropdown" (click)="toggleDropdown(); $event.preventDefault()">
            <a class="nav-link dropdown-toggle text-white" id="profileDropdown" role="button" aria-expanded="false">
              {{ firstName ? firstName : 'Profile' }} 👤
            </a>
            <ul class="dropdown-menu dropdown-menu-end" [ngClass]="{'show': isDropdownOpen}" aria-labelledby="profileDropdown">
              <!-- Common Options for All Users -->
              <li>
                <a class="dropdown-item" (click)="navigateTo('/profile')">Profile 👤</a>
              </li>
              <li>
                <a class="dropdown-item" (click)="navigateTo('/update-profile')">Update Profile ✏️</a>
              </li>

              <!-- Customer-Specific Options -->
              <li *ngIf="!isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/manage-addresses')">Manage Addresses 🏠</a>
              </li>
              <li *ngIf="!isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/cart')">Place Order 🛍️</a>
              </li>
              <li *ngIf="!isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/order-history')">Order History 📦</a>
              </li>

              <!-- Merchant-Specific Options -->
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/merchant-dashboard')">Dashboard 📊</a>
              </li>
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/add-product')">Add Product 🆕</a>
              </li>
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/manage-products')">Manage Products 📦</a>
              </li>

              <!-- Logout -->
              <li><hr class="dropdown-divider"></li>
              <li>
                <a class="dropdown-item" (click)="logout()">Logout 🚪</a>
              </li>
            </ul>
          </li>
        </ng-template>
      </div>
    </div>
  </div>
</nav>
```

#### Explanation of Changes in `header.component.html`
- **Structural Improvements**:
  - Wrapped navigation items (`Home`, `Products`, `Deals`) in `<li>` tags for consistency with Bootstrap’s navbar structure.
  - Moved the cart link and profile dropdown into a `<li>` structure under `navbar-nav` for better alignment.
  - Added `ms-auto` to the user actions section to align it to the right.
- **Enhanced Dropdown**:
  - Separated options into sections: common options (Profile, Update Profile), customer-specific options (Manage Addresses, Place Order, Order History), and merchant-specific options (Dashboard, Add Product, Manage Products).
  - Added a divider (`<hr>`) before the "Logout" option for visual separation.
  - Removed the redundant "Manage Products" link outside the dropdown (already present in the dropdown for merchants).
- **Visual Clarity**:
  - Added emojis to each menu item for better visual distinction (e.g., 🏠 for Home, 📊 for Dashboard).
  - Updated the search placeholder to be more descriptive ("Search for products...").
- **Preserved Functionality**:
  - Kept all existing bindings and events (e.g., `(input)="search($event)"`, `(click)="navigateTo('/cart')"`) intact.
  - Ensured the cart badge (`cartItemCount`) and dropdown toggle (`isDropdownOpen`) work as before.

#### Test
1. **Login as Customer**:
   - The dropdown should show "Profile", "Update Profile", "Manage Addresses", "Place Order", "Order History", and "Logout".
   - Cart link should be visible with the item count.
2. **Login as Merchant**:
   - The dropdown should show "Profile", "Update Profile", "Dashboard", "Add Product", "Manage Products", and "Logout".
   - Cart link should not be visible.
3. **Not Logged In**:
   - Should see "Login" and "Signup" buttons.
4. **Navigation**:
   - Clicking any link (e.g., Home, Products) should close the dropdown.
   - Search functionality should work as before.

These changes enhance the navbar for merchants while preserving all previous functionality for customers and unauthenticated users. Let me know if you need further adjustments! 🚀





Let’s implement the merchant functionality for your EShoppingZone application, focusing on creating a seamless and professional experience for merchants. Since you’ve shared the necessary backend files (`ProductController`, `ProductService`, and DTOs), I’ll leverage those to build the merchant features on the frontend. I’ll create a merchant dashboard, update the navbar for merchants, and ensure the experience is distinct from customers by limiting product visibility to the merchant’s own products. I’ll also design the merchant experience to be intuitive and robust, aiming for a top-notch implementation.

### Plan for Merchant Implementation
1. **Navbar for Merchants**:
   - Update the `HeaderComponent` to show a merchant-specific dropdown with options like "Dashboard", "Add Product", "Manage Products", "Profile", and "Logout".
   - Display the merchant’s first name in the dropdown (already implemented for customers).
2. **Merchant Dashboard**:
   - Create a `MerchantDashboardComponent` to show key stats (e.g., total products, total stock, average rating across products).
   - Include quick links to "Add Product" and "Manage Products".
3. **Add Product**:
   - Create an `AddProductComponent` to allow merchants to add new products using the `POST /api/ProductController` endpoint.
4. **Manage Products**:
   - Create a `ManageProductsComponent` to list the merchant’s products (using `GET /api/ProductController/MerchantGetProdut`), with options to update or delete them.
5. **Update Product**:
   - Create an `UpdateProductComponent` to edit existing products using the `PUT /api/ProductController/UpdateProduct/{productId}` endpoint.
6. **Product Visibility**:
   - Ensure merchants only see their own products (already handled by `MerchantGetProdut` API) and not all products like customers do.

---

### Implementation

#### 1. Update Navbar for Merchants
We’ll modify the `HeaderComponent` to show a merchant-specific dropdown when the user is a merchant.

**Changes to `header.component.ts`**:
```typescript
// Add property to determine user role
isMerchant: boolean = false;

// Update ngOnInit to set user role
ngOnInit() {
  this.router.events.subscribe(event => {
    if (event instanceof NavigationEnd) {
      this.closeDropdown();
    }
  });

  if (this.authService.isLoggedIn()) {
    this.authService.viewProfile().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.firstName = response.data.fullName.split(' ')[0];
        }
      },
      error: (err) => {
        console.error('Error fetching profile:', err);
        this.firstName = null;
      }
    });

    this.cartService.cartUpdate$.subscribe(count => {
      this.cartItemCount = count;
    });

    this.cartService.getCart().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.cartItemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0);
        }
      }
    });

    // Set user role
    this.isMerchant = this.authService.getUserRole() === 'Merchant';
  }
}
```

**Changes to `header.component.html`**:
```html
<!-- Update dropdown to show merchant-specific options -->
<div class="nav-item dropdown" (click)="toggleDropdown(); $event.preventDefault()">
  <a class="nav-link dropdown-toggle text-white" id="profileDropdown" role="button" aria-expanded="false">
    {{ firstName ? firstName : 'Profile' }}
  </a>
  <ul class="dropdown-menu" [ngClass]="{'show': isDropdownOpen}">
    <li *ngIf="isMerchant">
      <a class="dropdown-item" (click)="navigateTo('/merchant-dashboard')">Dashboard 📊</a>
    </li>
    <li *ngIf="isMerchant">
      <a class="dropdown-item" (click)="navigateTo('/add-product')">Add Product 🆕</a>
    </li>
    <li *ngIf="isMerchant">
      <a class="dropdown-item" (click)="navigateTo('/manage-products')">Manage Products 📦</a>
    </li>
    <li>
      <a class="dropdown-item" (click)="navigateTo('/profile')">Profile 👤</a>
    </li>
    <li *ngIf="!isMerchant">
      <a class="dropdown-item" (click)="navigateTo('/cart')">Cart 🛒</a>
    </li>
    <li>
      <a class="dropdown-item" (click)="logout()">Logout 🚪</a>
    </li>
  </ul>
</div>
```

**Explanation**:
- Added `isMerchant` to determine if the logged-in user is a merchant.
- Updated the dropdown to show merchant-specific options ("Dashboard", "Add Product", "Manage Products") when `isMerchant` is true.
- Kept "Profile" and "Logout" for all users, but hid "Cart" for merchants since they don’t need it.

#### 2. Create Merchant Dashboard
The dashboard will display key stats and provide quick navigation for merchants.

**New File: `merchant-dashboard.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-merchant-dashboard',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './merchant-dashboard.component.html',
  styleUrls: ['./merchant-dashboard.component.css']
})
export class MerchantDashboardComponent implements OnInit {
  totalProducts: number = 0;
  totalStock: number = 0;
  averageRating: number = 0;
  errorMessage: string | null = null;

  constructor(private productService: ProductService) {}

  ngOnInit() {
    this.loadStats();
  }

  loadStats() {
    this.productService.getMerchantProducts().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.totalProducts = response.data.length;
          this.totalStock = response.data.reduce((sum, product) => sum + product.stock, 0);
          const totalRating = response.data.reduce((sum, product) => sum + product.averageRating, 0);
          this.averageRating = response.data.length > 0 ? totalRating / response.data.length : 0;
        } else {
          this.errorMessage = response.message || 'No products found';
        }
      },
      error: (err) => {
        this.errorMessage = 'Error loading stats';
        console.error(err);
      }
    });
  }
}
```

**New File: `merchant-dashboard.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Merchant Dashboard 📊</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>

  <div class="row">
    <div class="col-md-4 mb-4">
      <div class="card shadow text-center">
        <div class="card-body">
          <h5 class="card-title">Total Products</h5>
          <p class="card-text display-4">{{ totalProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow text-center">
        <div class="card-body">
          <h5 class="card-title">Total Stock</h5>
          <p class="card-text display-4">{{ totalStock }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow text-center">
        <div class="card-body">
          <h5 class="card-title">Average Rating</h5>
          <p class="card-text display-4">{{ averageRating | number:'1.1-1' }}</p>
        </div>
      </div>
    </div>
  </div>

  <div class="mt-4">
    <h4>Quick Actions</h4>
    <div class="d-flex gap-3">
      <a [routerLink]="['/add-product']" class="btn btn-primary">Add New Product 🆕</a>
      <a [routerLink]="['/manage-products']" class="btn btn-secondary">Manage Products 📦</a>
    </div>
  </div>
</div>
```

**New File: `merchant-dashboard.component.css`**:
```css
.card {
  border-radius: 10px;
}
.display-4 {
  font-size: 2.5rem;
  font-weight: bold;
}
```

#### 3. Add Product
Allow merchants to add new products using the `POST /api/ProductController` endpoint.

**Update `product.service.ts`** (if not already present):
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface ProductRequest {
  name: string;
  type?: string;
  category?: string;
  price: number;
  stock: number;
  description?: string;
  images?: string;
  specifications?: string;
}

export interface UpdateProductRequest {
  name?: string;
  type?: string;
  category?: string;
  price: number;
  stock: number;
  description?: string;
  images?: string;
  specifications?: string;
}

export interface ProductResponse {
  id: number;
  name: string;
  type?: string;
  category?: string;
  price: number;
  stock: number;
  averageRating: number;
  reviewCount: number;
  description?: string;
  images?: string;
  specifications?: string;
}

export interface ResponseDTO<T> {
  success: boolean;
  message: string;
  data?: T;
}

@Injectable({
  providedIn: 'root'
})
export class ProductService {
  private baseUrl = 'https://api.eshoppingzone.com';

  constructor(private http: HttpClient) {}

  addProduct(productRequest: ProductRequest): Observable<ResponseDTO<ProductResponse>> {
    return this.http.post<ResponseDTO<ProductResponse>>(`${this.baseUrl}/api/ProductController`, productRequest);
  }

  getMerchantProducts(): Observable<ResponseDTO<ProductResponse[]>> {
    return this.http.get<ResponseDTO<ProductResponse[]>>(`${this.baseUrl}/api/ProductController/MerchantGetProdut`);
  }

  getProduct(productId: number): Observable<ResponseDTO<ProductResponse>> {
    return this.http.get<ResponseDTO<ProductResponse>>(`${this.baseUrl}/api/ProductController/GetProduct/${productId}`);
  }

  getAllProducts(): Observable<ResponseDTO<ProductResponse[]>> {
    return this.http.get<ResponseDTO<ProductResponse[]>>(`${this.baseUrl}/api/ProductController/GetAllProducts`);
  }

  updateProduct(productId: number, updateRequest: UpdateProductRequest): Observable<ResponseDTO<ProductResponse>> {
    return this.http.put<ResponseDTO<ProductResponse>>(`${this.baseUrl}/api/ProductController/UpdateProduct/${productId}`, updateRequest);
  }

  deleteProduct(productId: number): Observable<ResponseDTO<ProductResponse>> {
    return this.http.delete<ResponseDTO<ProductResponse>>(`${this.baseUrl}/api/ProductController/DeleteProduct/${productId}`);
  }
}
```

**New File: `add-product.component.ts`**:
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { ProductService, ProductRequest } from '../../services/product.service';
import { Router } from '@angular/router';

@Component({
  selector: 'app-add-product',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './add-product.component.html',
  styleUrls: ['./add-product.component.css']
})
export class AddProductComponent {
  product: ProductRequest = {
    name: '',
    type: '',
    category: '',
    price: 0,
    stock: 0,
    description: '',
    images: '',
    specifications: ''
  };
  errorMessage: string | null = null;
  successMessage: string | null = null;

  constructor(private productService: ProductService, private router: Router) {}

  addProduct() {
    this.productService.addProduct(this.product).subscribe({
      next: (response) => {
        if (response.success) {
          this.successMessage = 'Product added successfully!';
          this.errorMessage = null;
          setTimeout(() => {
            this.router.navigate(['/manage-products']);
          }, 2000);
        } else {
          this.errorMessage = response.message || 'Failed to add product';
          this.successMessage = null;
        }
      },
      error: (err) => {
        this.errorMessage = 'Error adding product';
        console.error(err);
      }
    });
  }
}
```

**New File: `add-product.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Add New Product 🆕</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="successMessage" class="alert alert-success" role="alert">
    {{ successMessage }}
  </div>
  <div class="card shadow">
    <div class="card-body">
      <form (ngSubmit)="addProduct()">
        <div class="mb-3">
          <label for="name" class="form-label">Product Name</label>
          <input type="text" class="form-control" id="name" [(ngModel)]="product.name" name="name" required maxlength="100">
        </div>
        <div class="mb-3">
          <label for="type" class="form-label">Type (Optional)</label>
          <input type="text" class="form-control" id="type" [(ngModel)]="product.type" name="type" maxlength="50">
        </div>
        <div class="mb-3">
          <label for="category" class="form-label">Category (Optional)</label>
          <input type="text" class="form-control" id="category" [(ngModel)]="product.category" name="category" maxlength="50">
        </div>
        <div class="mb-3">
          <label for="price" class="form-label">Price</label>
          <input type="number" class="form-control" id="price" [(ngModel)]="product.price" name="price" required min="0.01" step="0.01">
        </div>
        <div class="mb-3">
          <label for="stock" class="form-label">Stock</label>
          <input type="number" class="form-control" id="stock" [(ngModel)]="product.stock" name="stock" required min="0">
        </div>
        <div class="mb-3">
          <label for="description" class="form-label">Description (Optional)</label>
          <textarea class="form-control" id="description" [(ngModel)]="product.description" name="description" maxlength="5000"></textarea>
        </div>
        <div class="mb-3">
          <label for="images" class="form-label">Images URL (Optional)</label>
          <input type="text" class="form-control" id="images" [(ngModel)]="product.images" name="images">
        </div>
        <div class="mb-3">
          <label for="specifications" class="form-label">Specifications (Optional)</label>
          <input type="text" class="form-control" id="specifications" [(ngModel)]="product.specifications" name="specifications">
        </div>
        <button type="submit" class="btn btn-primary w-100">Add Product</button>
      </form>
    </div>
  </div>
</div>
```

**New File: `add-product.component.css`**:
```css
.card {
  border-radius: 10px;
}
.form-control {
  border-radius: 5px;
}
```

#### 4. Manage Products
Allow merchants to view, update, and delete their products using `GET /api/ProductController/MerchantGetProdut`, `PUT /api/ProductController/UpdateProduct/{productId}`, and `DELETE /api/ProductController/DeleteProduct/{productId}`.

**New File: `manage-products.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { RouterLink } from '@angular/router';

@Component({
  selector: 'app-manage-products',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './manage-products.component.html',
  styleUrls: ['./manage-products.component.css']
})
export class ManageProductsComponent implements OnInit {
  products: ProductResponse[] = [];
  errorMessage: string | null = null;

  constructor(private productService: ProductService) {}

  ngOnInit() {
    this.loadProducts();
  }

  loadProducts() {
    this.productService.getMerchantProducts().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.products = response.data;
        } else {
          this.errorMessage = response.message || 'No products found';
        }
      },
      error: (err) => {
        this.errorMessage = 'Error loading products';
        console.error(err);
      }
    });
  }

  deleteProduct(productId: number) {
    if (confirm('Are you sure you want to delete this product?')) {
      this.productService.deleteProduct(productId).subscribe({
        next: (response) => {
          if (response.success) {
            this.products = this.products.filter(p => p.id !== productId);
          } else {
            this.errorMessage = response.message || 'Failed to delete product';
          }
        },
        error: (err) => {
          this.errorMessage = 'Error deleting product';
          console.error(err);
        }
      });
    }
  }
}
```

**New File: `manage-products.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Manage Products 📦</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="products.length > 0; else noProducts">
    <div class="card mb-4 shadow-sm" *ngFor="let product of products">
      <div class="card-body">
        <h5 class="card-title">{{ product.name }}</h5>
        <p class="card-text">Price: {{ product.price | currency }}</p>
        <p class="card-text">Stock: {{ product.stock }}</p>
        <p class="card-text">Category: {{ product.category || 'Not specified' }}</p>
        <p class="card-text">Average Rating: {{ product.averageRating | number:'1.1-1' }} ({{ product.reviewCount }} reviews)</p>
        <div class="d-flex gap-2">
          <a [routerLink]="['/update-product', product.id]" class="btn btn-primary">Update</a>
          <button class="btn btn-danger" (click)="deleteProduct(product.id)">Delete</button>
        </div>
      </div>
    </div>
  </div>
  <ng-template #noProducts>
    <p class="text-muted text-center">No products found. Add a new product to get started!</p>
  </ng-template>
</div>
```

**New File: `manage-products.component.css`**:
```css
.card {
  border-radius: 10px;
}
```

#### 5. Update Product
Allow merchants to edit their products.

**New File: `update-product.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { ProductService, UpdateProductRequest, ProductResponse } from '../../services/product.service';
import { ActivatedRoute, Router } from '@angular/router';

@Component({
  selector: 'app-update-product',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './update-product.component.html',
  styleUrls: ['./update-product.component.css']
})
export class UpdateProductComponent implements OnInit {
  productId: number = 0;
  product: UpdateProductRequest = {
    name: '',
    type: '',
    category: '',
    price: 0,
    stock: 0,
    description: '',
    images: '',
    specifications: ''
  };
  errorMessage: string | null = null;
  successMessage: string | null = null;

  constructor(
    private productService: ProductService,
    private route: ActivatedRoute,
    private router: Router
  ) {}

  ngOnInit() {
    this.productId = Number(this.route.snapshot.paramMap.get('id'));
    this.loadProduct();
  }

  loadProduct() {
    this.productService.getProduct(this.productId).subscribe({
      next: (response) => {
        if (response.success && response.data) {
          const product = response.data;
          this.product = {
            name: product.name,
            type: product.type,
            category: product.category,
            price: product.price,
            stock: product.stock,
            description: product.description,
            images: product.images,
            specifications: product.specifications
          };
        } else {
          this.errorMessage = response.message || 'Product not found';
        }
      },
      error: (err) => {
        this.errorMessage = 'Error loading product';
        console.error(err);
      }
    });
  }

  updateProduct() {
    this.productService.updateProduct(this.productId, this.product).subscribe({
      next: (response) => {
        if (response.success) {
          this.successMessage = 'Product updated successfully!';
          this.errorMessage = null;
          setTimeout(() => {
            this.router.navigate(['/manage-products']);
          }, 2000);
        } else {
          this.errorMessage = response.message || 'Failed to update product';
          this.successMessage = null;
        }
      },
      error: (err) => {
        this.errorMessage = 'Error updating product';
        console.error(err);
      }
    });
  }
}
```

**New File: `update-product.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Update Product ✏️</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="successMessage" class="alert alert-success" role="alert">
    {{ successMessage }}
  </div>
  <div class="card shadow">
    <div class="card-body">
      <form (ngSubmit)="updateProduct()">
        <div class="mb-3">
          <label for="name" class="form-label">Product Name</label>
          <input type="text" class="form-control" id="name" [(ngModel)]="product.name" name="name" maxlength="100">
        </div>
        <div class="mb-3">
          <label for="type" class="form-label">Type</label>
          <input type="text" class="form-control" id="type" [(ngModel)]="product.type" name="type" maxlength="50">
        </div>
        <div class="mb-3">
          <label for="category" class="form-label">Category</label>
          <input type="text" class="form-control" id="category" [(ngModel)]="product.category" name="category" maxlength="50">
        </div>
        <div class="mb-3">
          <label for="price" class="form-label">Price</label>
          <input type="number" class="form-control" id="price" [(ngModel)]="product.price" name="price" required min="0.01" step="0.01">
        </div>
        <div class="mb-3">
          <label for="stock" class="form-label">Stock</label>
          <input type="number" class="form-control" id="stock" [(ngModel)]="product.stock" name="stock" required min="0">
        </div>
        <div class="mb-3">
          <label for="description" class="form-label">Description</label>
          <textarea class="form-control" id="description" [(ngModel)]="product.description" name="description" maxlength="500"></textarea>
        </div>
        <div class="mb-3">
          <label for="images" class="form-label">Images URL</label>
          <input type="text" class="form-control" id="images" [(ngModel)]="product.images" name="images">
        </div>
        <div class="mb-3">
          <label for="specifications" class="form-label">Specifications</label>
          <input type="text" class="form-control" id="specifications" [(ngModel)]="product.specifications" name="specifications" maxlength="500">
        </div>
        <button type="submit" class="btn btn-primary w-100">Update Product</button>
      </form>
    </div>
  </div>
</div>
```

**New File: `update-product.component.css`**:
```css
.card {
  border-radius: 10px;
}
.form-control {
  border-radius: 5px;
}
```

#### 6. Update Routes
Add routes for the new components in `app.routes.ts`.

**Changes to `app.routes.ts`**:
```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './components/home/home.component';
import { LoginComponent } from './components/login/login.component';
import { RegisterComponent } from './components/register/register.component';
import { AuthGuard } from './guards/auth.guard';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';
import { CartComponent } from './components/cart/cart.component';
import { ProfileComponent } from './components/profile/profile.component';
import { UpdateProfileComponent } from './components/update-profile/update-profile.component';
import { ManageAddressesComponent } from './components/manage-addresses/manage-addresses.component';
import { MerchantDashboardComponent } from './components/merchant-dashboard/merchant-dashboard.component';
import { AddProductComponent } from './components/add-product/add-product.component';
import { ManageProductsComponent } from './components/manage-products/manage-products.component';
import { UpdateProductComponent } from './components/update-product/update-product.component';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'login', component: LoginComponent },
  { path: 'register', component: RegisterComponent },
  { path: 'product/:id', component: ProductDetailComponent },
  { path: 'cart', component: CartComponent, canActivate: [AuthGuard] },
  { path: 'profile', component: ProfileComponent, canActivate: [AuthGuard] },
  { path: 'update-profile', component: UpdateProfileComponent, canActivate: [AuthGuard] },
  { path: 'manage-addresses', component: ManageAddressesComponent, canActivate: [AuthGuard] },
  { path: 'merchant-dashboard', component: MerchantDashboardComponent, canActivate: [AuthGuard] },
  { path: 'add-product', component: AddProductComponent, canActivate: [AuthGuard] },
  { path: 'manage-products', component: ManageProductsComponent, canActivate: [AuthGuard] },
  { path: 'update-product/:id', component: UpdateProductComponent, canActivate: [AuthGuard] },
  { path: '**', redirectTo: '' }
];
```

#### 7. Product Visibility for Merchants
Merchants should only see their own products, which is already handled by the `GET /api/ProductController/MerchantGetProdut` endpoint used in `ManageProductsComponent` and `MerchantDashboardComponent`. Customers, on the other hand, use `GET /api/ProductController/GetAllProducts` in `HomeComponent` to see all products. This distinction is already implemented correctly.

---

### Summary of Implementation
- **Navbar for Merchants**:
  - Updated `HeaderComponent` to show a merchant-specific dropdown with "Dashboard", "Add Product", "Manage Products", "Profile", and "Logout".
  - Hid the "Cart" option for merchants.
- **Merchant Dashboard**:
  - Created `MerchantDashboardComponent` to display stats (total products, total stock, average rating) and quick links to add and manage products.
- **Add Product**:
  - Created `AddProductComponent` to allow merchants to add new products with a form matching the `ProductRequest` DTO.
- **Manage Products**:
  - Created `ManageProductsComponent` to list the merchant’s products with options to update or delete them.
- **Update Product**:
  - Created `UpdateProductComponent` to allow merchants to edit existing products.
- **Product Visibility**:
  - Ensured merchants only see their own products via `MerchantGetProdut` API, while customers see all products via `GetAllProducts`.

---

### Test
1. **Login as Merchant**:
   - Log in with a merchant account.
   - The navbar dropdown should show "Dashboard", "Add Product", "Manage Products", "Profile", and "Logout".
2. **Merchant Dashboard**:
   - Navigate to `/merchant-dashboard`—should show total products, total stock, and average rating.
   - Click "Add New Product" or "Manage Products" to navigate to those pages.
3. **Add Product**:
   - Go to `/add-product`, fill in the form, and submit—should redirect to `/manage-products` after success.
4. **Manage Products**:
   - Go to `/manage-products`—should list only the merchant’s products.
   - Click "Update" to edit a product, or "Delete" to remove it (with confirmation).
5. **Update Product**:
   - Go to `/update-product/:id`, edit the product details, and submit—should redirect to `/manage-products` after success.

This implementation provides a robust and merchant-focused experience, leveraging your backend APIs effectively. Let me know if you’d like to refine anything further! 🚀
