Let’s address the changes you need for your EShoppingZone project, focusing on the merchant experience in a professional and efficient way. We’ll modify the navbar, enhance the merchant dashboard, create a new merchant home page, and adjust the product details view to include an image gallery and reviews. I’ll ensure the updates are quick, professional, and align with your existing codebase, reusing components where possible.

---

### Changes to Address
1. **Navbar Updates**:
   - Move "Dashboard" and "Manage Products" to the navbar for merchants.
   - Remove "Create Product" (now "Add Product") from the dropdown.

2. **Merchant Dashboard Enhancements**:
   - Make "Total Products", "Total Stock", "Low Stock Products", and "Average Product Rating" clickable.
   - **Total Products**: Redirect to the `ManageProductsComponent` (already implemented).
   - **Total Stock**: Show a table with product names and their stock.
   - **Low Stock Products**: Define low stock as `< 100`, show products with low stock and their stock count.
   - **Average Product Rating**: Show product names, their average ratings, or "No ratings so far" if unrated.

3. **Merchant Home Page**:
   - Create a new home page for merchants with a welcoming banner.
   - Move "Quick Actions" from the dashboard to the home page, rename "Create Product" to "Add Product", and add more options.

4. **Product Details View**:
   - Remove the "View Details" button in `ManageProductsComponent`, make the product card clickable to navigate to `MerchantProductDetailComponent`.
   - In `MerchantProductDetailComponent`, add an image gallery (similar to the customer’s `ProductDetailComponent`) and display ratings with reviews.

---

### Implementation

#### 1. Update Navbar (`HeaderComponent`)
We’ll move "Dashboard" and "Manage Products" to the navbar for merchants and remove "Add Product" from the dropdown.

**Changes to `header.component.html`**:
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
  <div class="container-fluid px-5">
    <!-- Brand -->
    <a class="navbar-brand fw-bold" [routerLink]="['/']">EShoppingZone</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
      aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>

    <!-- Navbar Content -->
    <div class="collapse navbar-collapse" id="navbarNav">
      <!-- Navigation Links -->
      <ul class="navbar-nav me-auto mb-2 mb-lg-0 ms-3">
        <li class="nav-item">
          <a class="nav-link" [routerLink]="['/']" routerLinkActive="active">Home</a>
        </li>
        <!-- Customer Links -->
        <li class="nav-item" *ngIf="!isMerchant">
          <a class="nav-link" [routerLink]="['/products']" routerLinkActive="active">Products</a>
        </li>
        <li class="nav-item" *ngIf="!isMerchant">
          <a class="nav-link" [routerLink]="['/deals']" routerLinkActive="active">Deals</a>
        </li>
        <!-- Merchant Links -->
        <li class="nav-item" *ngIf="isMerchant">
          <a class="nav-link" [routerLink]="['/merchant-dashboard']" routerLinkActive="active">Dashboard</a>
        </li>
        <li class="nav-item" *ngIf="isMerchant">
          <a class="nav-link" [routerLink]="['/manage-products']" routerLinkActive="active">Manage Products</a>
        </li>
      </ul>

      <!-- Search Bar (Only for non-merchants) -->
      <div class="d-flex flex-grow-1 justify-content-center mx-3" *ngIf="!isMerchant">
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
            <a class="nav-link btn btn-outline-light me-2 login-signup-btn" [routerLink]="['/login']">Login</a>
          </li>
          <li class="nav-item">
            <a class="nav-link btn btn-outline-light login-signup-btn" [routerLink]="['/register']">Signup</a>
          </li>
        </ng-container>

        <ng-template #loggedIn>
          <!-- Cart Link for Customers -->
          <li class="nav-item" *ngIf="!isMerchant">
            <a class="nav-link text-white" [routerLink]="['/cart']">
              Cart
              <span *ngIf="cartItemCount > 0" class="badge bg-danger ms-1">{{ cartItemCount }}</span>
            </a>
          </li>

          <!-- Profile Dropdown -->
          <li class="nav-item dropdown" (click)="toggleDropdown(); $event.preventDefault()">
            <a class="nav-link dropdown-toggle text-white" id="profileDropdown" role="button" aria-expanded="false">
              {{ firstName ? firstName : 'Profile' }}
            </a>
            <ul class="dropdown-menu dropdown-menu-end" [ngClass]="{'show': isDropdownOpen}" aria-labelledby="profileDropdown">
              <!-- Common Options for All Users -->
              <li>
                <a class="dropdown-item" (click)="navigateTo('/profile')">Profile</a>
              </li>
              <li>
                <a class="dropdown-item" (click)="navigateTo('/update-profile')">Update Profile</a>
              </li>

              <!-- Customer-Specific Options -->
              <li *ngIf="!isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/manage-addresses')">Manage Addresses</a>
              </li>
              <li *ngIf="!isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/cart')">Place Order</a>
              </li>
              <li *ngIf="!isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/order-history')">Order History</a>
              </li>

              <!-- Logout -->
              <li><hr class="dropdown-divider"></li>
              <li>
                <a class="dropdown-item" (click)="logout()">Logout</a>
              </li>
            </ul>
          </li>
        </ng-template>
      </div>
    </div>
  </div>
</nav>
```

**Explanation**:
- Added "Dashboard" and "Manage Products" to the navbar for merchants using `*ngIf="isMerchant"`.
- Removed the "Add Product" option from the dropdown (previously labeled "Create Product").

#### 2. Enhance Merchant Dashboard
We’ll make the stats clickable and display relevant data when clicked. We’ll also remove the "Quick Actions" section since it’s moving to the home page.

**Changes to `merchant-dashboard.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { RouterLink, Router } from '@angular/router';

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
  lowStockProducts: number = 0;
  averageRating: number = 0;
  errorMessage: string | null = null;
  products: ProductResponse[] = [];
  showTotalStock: boolean = false;
  showLowStock: boolean = false;
  showAverageRating: boolean = false;

  constructor(private productService: ProductService, private router: Router) {}

  ngOnInit() {
    this.loadStats();
  }

  loadStats() {
    this.productService.getMerchantProducts().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.products = response.data;
          this.totalProducts = response.data.length;
          this.totalStock = response.data.reduce((sum, product) => sum + product.stock, 0);
          this.lowStockProducts = response.data.filter(product => product.stock < 100).length; // Low stock < 100

          const reviewedProducts = response.data.filter(product => product.reviewCount > 0);
          const totalRating = reviewedProducts.reduce((sum, product) => sum + product.averageRating, 0);
          this.averageRating = reviewedProducts.length > 0 ? totalRating / reviewedProducts.length : 0;
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

  onTotalProductsClick() {
    this.router.navigate(['/manage-products']);
  }

  onTotalStockClick() {
    this.showTotalStock = !this.showTotalStock;
    this.showLowStock = false;
    this.showAverageRating = false;
  }

  onLowStockClick() {
    this.showLowStock = !this.showLowStock;
    this.showTotalStock = false;
    this.showAverageRating = false;
  }

  onAverageRatingClick() {
    this.showAverageRating = !this.showAverageRating;
    this.showTotalStock = false;
    this.showLowStock = false;
  }
}
```

**Changes to `merchant-dashboard.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Merchant Dashboard</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>

  <!-- Stats Overview -->
  <div class="row">
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onTotalProductsClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Total Products</h5>
          <p class="card-text display-5 fw-bold text-primary">{{ totalProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onTotalStockClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Total Stock</h5>
          <p class="card-text display-5 fw-bold text-primary">{{ totalStock }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onLowStockClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Low Stock Products</h5>
          <p class="card-text display-5 fw-bold text-warning">{{ lowStockProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onAverageRatingClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Average Product Rating</h5>
          <p class="card-text display-5 fw-bold text-success">{{ averageRating | number:'1.1-1' }}</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Total Stock Details -->
  <div *ngIf="showTotalStock" class="mt-4">
    <h4 class="fw-bold mb-3">Total Stock Details</h4>
    <table class="table table-bordered table-hover shadow-sm">
      <thead class="table-primary">
        <tr>
          <th>Product Name</th>
          <th>Stock</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let product of products">
          <td>{{ product.name }}</td>
          <td>{{ product.stock }}</td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Low Stock Details -->
  <div *ngIf="showLowStock" class="mt-4">
    <h4 class="fw-bold mb-3">Low Stock Products (Stock < 100)</h4>
    <table class="table table-bordered table-hover shadow-sm">
      <thead class="table-warning">
        <tr>
          <th>Product Name</th>
          <th>Stock Remaining</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let product of products | filterLowStock">
          <td>{{ product.name }}</td>
          <td>{{ product.stock }}</td>
        </tr>
        <tr *ngIf="(products | filterLowStock).length === 0">
          <td colspan="2" class="text-center text-muted">No products with low stock.</td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Average Rating Details -->
  <div *ngIf="showAverageRating" class="mt-4">
    <h4 class="fw-bold mb-3">Product Ratings</h4>
    <table class="table table-bordered table-hover shadow-sm">
      <thead class="table-success">
        <tr>
          <th>Product Name</th>
          <th>Average Rating</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let product of products">
          <td>{{ product.name }}</td>
          <td>{{ product.reviewCount > 0 ? (product.averageRating | number:'1.1-1') : 'No ratings so far' }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
```

**Changes to `merchant-dashboard.component.css`**:
```css
.card {
  border-radius: 8px;
  transition: transform 0.2s;
}
.card:hover {
  transform: translateY(-5px);
}
.display-5 {
  font-size: 2.2rem;
}
.text-primary {
  color: #007bff !important;
}
.text-warning {
  color: #ffca2c !important;
}
.text-success {
  color: #28a745 !important;
}
.clickable {
  cursor: pointer;
}
.table {
  border-radius: 8px;
  overflow: hidden;
}
```

**New Pipe for Low Stock Filtering**:
Create a new pipe to filter low stock products.

**`filter-low-stock.pipe.ts`**:
```typescript
import { Pipe, PipeTransform } from '@angular/core';
import { ProductResponse } from '../services/product.service';

@Pipe({
  name: 'filterLowStock',
  standalone: true
})
export class FilterLowStockPipe implements PipeTransform {
  transform(products: ProductResponse[]): ProductResponse[] {
    return products.filter(product => product.stock < 100);
  }
}
```

**Update `MerchantDashboardComponent` Imports**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { RouterLink, Router } from '@angular/router';
import { FilterLowStockPipe } from '../../pipes/filter-low-stock.pipe';

@Component({
  selector: 'app-merchant-dashboard',
  standalone: true,
  imports: [CommonModule, RouterLink, FilterLowStockPipe],
  templateUrl: './merchant-dashboard.component.html',
  styleUrls: ['./merchant-dashboard.component.css']
})
```

**Explanation**:
- **Clickable Stats**:
  - Added `(click)` handlers to each card to toggle visibility of details or navigate.
  - "Total Products" navigates to `/manage-products`.
  - "Total Stock" shows a table with product names and stock.
  - "Low Stock Products" (defined as `< 100`) shows a table of products with low stock.
  - "Average Product Rating" shows a table with product names and ratings, displaying "No ratings so far" for unrated products.
- **Professional Design**:
  - Used Bootstrap tables with color-coded headers (blue for total stock, yellow for low stock, green for ratings).
  - Added shadows and rounded corners for a polished look.
- **Removed Quick Actions**:
  - Removed the "Quick Actions" section since it’s moving to the home page.

#### 3. Create Merchant Home Page
We’ll create a new `MerchantHomeComponent` as the landing page for merchants after login, with a banner and quick actions.

**New File: `merchant-home.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-merchant-home',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './merchant-home.component.html',
  styleUrls: ['./merchant-home.component.css']
})
export class MerchantHomeComponent implements OnInit {
  firstName: string | null = null;

  constructor(private authService: AuthService) {}

  ngOnInit() {
    this.authService.viewProfile().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.firstName = response.data.fullName.split(' ')[0];
        }
      },
      error: (err) => {
        console.error('Error fetching profile:', err);
      }
    });
  }
}
```

**New File: `merchant-home.component.html`**:
```html
<div class="container-fluid py-5 px-5">
  <!-- Banner -->
  <div class="banner text-center text-white py-5 rounded" style="background: linear-gradient(135deg, #007bff, #00c4b4);">
    <h1 class="display-4 fw-bold">Welcome Back, {{ firstName ? firstName : 'Merchant' }}!</h1>
    <p class="lead">Manage your products and grow your business with EShoppingZone.</p>
  </div>

  <!-- Quick Actions -->
  <div class="mt-5">
    <h4 class="fw-bold mb-3">Quick Actions</h4>
    <div class="d-flex flex-wrap gap-3">
      <a [routerLink]="['/add-product']" class="btn btn-primary">Add Product</a>
      <a [routerLink]="['/manage-products']" class="btn btn-outline-primary">Manage Products</a>
      <a [routerLink]="['/merchant-dashboard']" class="btn btn-outline-primary">View Dashboard</a>
      <a [routerLink]="['/profile']" class="btn btn-outline-primary">Update Profile</a>
    </div>
  </div>
</div>
```

**New File: `merchant-home.component.css`**:
```css
.banner {
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.btn {
  border-radius: 5px;
  padding: 0.75rem 1.5rem;
}
```

**Update `app.routes.ts`**:
```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './components/home/home.component';
import { MerchantHomeComponent } from './components/merchant-home/merchant-home.component';
import { ProductsComponent } from './components/product/product.component';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';
import { DealsComponent } from './components/deals/deals.component';
import { LoginComponent } from './components/login/login.component';
import { SignupComponent } from './components/signup/signup.component';
import { ProfileComponent } from './components/profile/profile.component';
import { CartComponent } from './components/cart/cart.component';
import { ManageProductsComponent } from './components/manage-products/manage-products.component';
import { MerchantDashboardComponent } from './components/merchant-dashboard/merchant-dashboard.component';
import { AddProductComponent } from './components/add-product/add-product.component';
import { UpdateProductComponent } from './components/update-product/update-product.component';
import { UpdateProfileComponent } from './components/update-profile/update-profile.component';
import { ManageAddressesComponent } from './components/manage-addresses/manage-addresses.component';
import { MerchantProductDetailComponent } from './components/merchant-product-detail/merchant-product-detail.component';
import { authGuard } from './guards/auth/auth.guard';
import { RoleGuard } from './guards/role/role.guard';

export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: HomeComponent, canActivate: [RoleGuard] },
  { path: 'merchant-home', component: MerchantHomeComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } },
  { path: 'products', component: ProductsComponent },
  { path: 'product/:id', component: ProductDetailComponent },
  { path: 'deals', component: DealsComponent },
  { path: 'login', component: LoginComponent },
  { path: 'signup', component: SignupComponent },
  { path: 'profile', component: ProfileComponent, canActivate: [authGuard] },
  { path: 'update-profile', component: UpdateProfileComponent, canActivate: [authGuard] },
  { path: 'manage-addresses', component: ManageAddressesComponent, canActivate: [authGuard] },
  { path: 'cart', component: CartComponent, canActivate: [authGuard], data: { expectedRole: 'Customer' }, canActivateChild: [RoleGuard] },
  { path: 'merchant-dashboard', component: MerchantDashboardComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } },
  { path: 'add-product', component: AddProductComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } },
  { path: 'manage-products', component: ManageProductsComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } },
  { path: 'update-product/:id', component: UpdateProductComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } },
  { path: 'merchant-product-detail/:id', component: MerchantProductDetailComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } },
  { path: '**', redirectTo: '/home' }
];
```

**Update `RoleGuard` to Redirect to `MerchantHomeComponent`**:
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
import { AuthService } from '../../services/auth.service';

@Injectable({
  providedIn: 'root'
})
export class RoleGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    const expectedRole = route.data['expectedRole'];
    const userRole = this.authService.getUserRole();
    const isLoggedIn = this.authService.isLoggedIn();

    // Redirect merchants to merchant-home if they access the root path or /home
    if ((state.url === '/' || state.url === '/home') && isLoggedIn && userRole === 'Merchant') {
      this.router.navigate(['/merchant-home']);
      return false;
    }

    if (expectedRole) {
      if (isLoggedIn && userRole === expectedRole) {
        return true;
      }
      this.router.navigate(['/home']);
      return false;
    }

    return true;
  }
}
```

**Explanation**:
- **New Merchant Home Page**:
  - Created `MerchantHomeComponent` with a welcoming banner using a gradient background.
  - Moved "Quick Actions" from the dashboard, renamed "Create Product" to "Add Product", and added more options ("View Dashboard", "Update Profile").
- **Route Updates**:
  - Added `/merchant-home` route, protected with `authGuard` and `RoleGuard`.
  - Updated `RoleGuard` to redirect merchants to `/merchant-home` instead of `/merchant-dashboard`, making it the new landing page.
- **Professional Design**:
  - Used a gradient banner with a shadow for a vibrant look.
  - Styled buttons with proper spacing and rounded corners.

#### 4. Update Manage Products (Make Products Clickable)
We’ll remove the "View Details" button and make the entire product card clickable to navigate to `MerchantProductDetailComponent`.

**Changes to `manage-products.component.html`**:
```html
<div class="container py-5 px-5">
  <div class="d-flex justify-content-between align-items-center mb-4">
    <h2 class="fw-bold">Manage Products</h2>
    <a [routerLink]="['/add-product']" class="btn btn-primary">Add Product</a>
  </div>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="products.length > 0; else noProducts">
    <div class="row">
      <div class="col-md-4 mb-4" *ngFor="let product of products">
        <div class="card shadow-sm border-0 h-100 clickable" [routerLink]="['/merchant-product-detail', product.id]">
          <img *ngIf="product.images" [src]="product.images" class="card-img-top" alt="{{ product.name }}" style="height: 200px; object-fit: cover;">
          <div *ngIf="!product.images" class="card-img-top bg-light d-flex align-items-center justify-content-center" style="height: 200px;">
            <span class="text-muted">No Image</span>
          </div>
          <div class="card-body">
            <h5 class="card-title">{{ product.name }}</h5>
            <p class="card-text text-muted">Price: {{ product.price | currency }}</p>
            <p class="card-text text-muted">Stock: {{ product.stock }}</p>
            <p class="card-text text-muted">Category: {{ product.category || 'Not specified' }}</p>
            <p class="card-text text-muted">Rating: {{ product.averageRating | number:'1.1-1' }} ({{ product.reviewCount }} reviews)</p>
            <div class="d-flex gap-2">
              <a [routerLink]="['/update-product', product.id]" class="btn btn-outline-secondary btn-sm" (click)="$event.stopPropagation()">Edit</a>
              <button class="btn btn-outline-danger btn-sm" (click)="deleteProduct(product.id); $event.stopPropagation()">Delete</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <ng-template #noProducts>
    <p class="text-muted text-center">No products found. Add a new product to get started.</p>
  </ng-template>
</div>
```

**Changes to `manage-products.component.css`**:
```css
.card {
  border-radius: 8px;
  transition: transform 0.2s;
}
.card:hover {
  transform: translateY(-5px);
}
.card-img-top {
  border-top-left-radius: 8px;
  border-top-right-radius: 8px;
}
.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}
.clickable {
  cursor: pointer;
}
```

**Explanation**:
- **Made Product Card Clickable**:
  - Added `[routerLink]="['/merchant-product-detail', product.id]"` to the card `div`.
  - Removed the "View Details" button.
- **Prevent Button Clicks from Triggering Navigation**:
  - Added `(click)="$event.stopPropagation()"` to the "Edit" and "Delete" buttons to prevent their clicks from navigating to the product detail page.
- **Updated "Create Product" to "Add Product"**:
  - Changed the button label at the top to "Add Product".

#### 5. Update Merchant Product Detail Page (Add Image Gallery and Reviews)
We’ll enhance `MerchantProductDetailComponent` to include an image gallery (similar to the customer’s `ProductDetailComponent`) and display ratings with reviews. I’ll assume `ProductResponse` includes an array of images and reviews, and we’ll fetch reviews via a new API call.

**Assumed API Service Update**:
Add a method to `ProductService` to fetch reviews (assuming your backend supports this).

**`product.service.ts`** (Add Method):
```typescript
getProductReviews(productId: number): Observable<any> {
  return this.http.get<any>(`${this.baseUrl}/api/Product/${productId}/reviews`);
}
```

**Update `MerchantProductDetailComponent`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { ActivatedRoute, RouterLink } from '@angular/router';

@Component({
  selector: 'app-merchant-product-detail',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './merchant-product-detail.component.html',
  styleUrls: ['./merchant-product-detail.component.css']
})
export class MerchantProductDetailComponent implements OnInit {
  product: ProductResponse | null = null;
  errorMessage: string | null = null;
  reviews: any[] = [];
  currentImageIndex: number = 0;

  constructor(
    private productService: ProductService,
    private route: ActivatedRoute
  ) {}

  ngOnInit() {
    const productId = Number(this.route.snapshot.paramMap.get('id'));
    this.loadProduct(productId);
    this.loadReviews(productId);
  }

  loadProduct(productId: number) {
    this.productService.getProduct(productId).subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.product = response.data;
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

  loadReviews(productId: number) {
    this.productService.getProductReviews(productId).subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.reviews = response.data;
        }
      },
      error: (err) => {
        console.error('Error loading reviews:', err);
      }
    });
  }

  changeImage(direction: number) {
    if (this.product && this.product.images) {
      const images = this.product.images.split(','); // Assuming images are comma-separated
      this.currentImageIndex = (this.currentImageIndex + direction + images.length) % images.length;
    }
  }
}
```

**Update `merchant-product-detail.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Product Details</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="product" class="card shadow-sm border-0">
    <div class="row g-0">
      <div class="col-md-4">
        <div class="position-relative">
          <img *ngIf="product.images" [src]="product.images.split(',')[currentImageIndex]" class="img-fluid rounded-start" alt="{{ product.name }}" style="height: 100%; object-fit: cover;">
          <div *ngIf="!product.images" class="bg-light d-flex align-items-center justify-content-center" style="height: 100%;">
            <span class="text-muted">No Image</span>
          </div>
          <button *ngIf="product.images && product.images.split(',').length > 1" class="btn btn-outline-secondary position-absolute start-0 top-50 translate-middle-y" (click)="changeImage(-1)">&lt;</button>
          <button *ngIf="product.images && product.images.split(',').length > 1" class="btn btn-outline-secondary position-absolute end-0 top-50 translate-middle-y" (click)="changeImage(1)">&gt;</button>
        </div>
      </div>
      <div class="col-md-8">
        <div class="card-body">
          <h3 class="card-title">{{ product.name }}</h3>
          <p class="card-text"><strong>Price:</strong> {{ product.price | currency }}</p>
          <p class="card-text"><strong>Stock:</strong> {{ product.stock }}</p>
          <p class="card-text"><strong>Category:</strong> {{ product.category || 'Not specified' }}</p>
          <p class="card-text"><strong>Type:</strong> {{ product.type || 'Not specified' }}</p>
          <p class="card-text"><strong>Average Rating:</strong> {{ product.averageRating | number:'1.1-1' }} ({{ product.reviewCount }} reviews)</p>
          <p class="card-text"><strong>Description:</strong> {{ product.description || 'No description provided' }}</p>
          <p class="card-text"><strong>Specifications:</strong> {{ product.specifications || 'No specifications provided' }}</p>
          <div class="mt-4">
            <a [routerLink]="['/update-product', product.id]" class="btn btn-primary me-2">Edit Product</a>
            <a [routerLink]="['/manage-products']" class="btn btn-outline-secondary">Back to Manage Products</a>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Reviews Section -->
  <div class="mt-5" *ngIf="product">
    <h4 class="fw-bold mb-3">Customer Reviews</h4>
    <div *ngIf="reviews.length > 0; else noReviews">
      <div class="card mb-3 shadow-sm" *ngFor="let review of reviews">
        <div class="card-body">
          <div class="d-flex justify-content-between">
            <h6 class="card-title">{{ review.userName || 'Anonymous' }}</h6>
            <span class="text-warning">{{ '★'.repeat(review.rating) }}{{ '☆'.repeat(5 - review.rating) }}</span>
          </div>
          <p class="card-text">{{ review.comment || 'No comment provided' }}</p>
          <small class="text-muted">{{ review.date | date:'medium' }}</small>
        </div>
      </div>
    </div>
    <ng-template #noReviews>
      <p class="text-muted">No reviews yet for this product.</p>
    </ng-template>
  </div>
</div>
```

**Update `merchant-product-detail.component.css`**:
```css
.card {
  border-radius: 8px;
}
.img-fluid {
  border-top-left-radius: 8px;
  border-bottom-left-radius: 8px;
}
.card-body {
  padding: 2rem;
}
.position-relative {
  height: 100%;
}
.btn-outline-secondary {
  background-color: rgba(255, 255, 255, 0.8);
}
```

**Explanation**:
- **Image Gallery**:
  - Reused the image gallery logic from `ProductDetailComponent`, assuming `images` is a comma-separated string of URLs.
  - Added navigation buttons to cycle through images.
- **Reviews**:
  - Added a reviews section below the product details, fetching reviews via a new `getProductReviews` method.
  - Displayed each review with the user’s name, rating (as stars), comment, and date.
- **Professional Design**:
  - Used cards for reviews with shadows and star ratings for a polished look.

---

### Summary of Changes
- **Navbar**:
  - Added "Dashboard" and "Manage Products" to the navbar for merchants.
  - Removed "Add Product" from the dropdown.
- **Merchant Dashboard**:
  - Made stats clickable with detailed views (tables for stock, low stock, ratings).
  - Defined low stock as `< 100`.
  - Removed "Quick Actions".
- **Merchant Home Page**:
  - Created `MerchantHomeComponent` with a welcoming banner and quick actions.
  - Renamed "Create Product" to "Add Product" and added more options.
  - Updated `RoleGuard` to redirect merchants to `/merchant-home`.
- **Manage Products**:
  - Made product cards clickable, removed "View Details".
  - Updated labels to "Add Product".
- **Merchant Product Detail**:
  - Added an image gallery and reviews section, reusing logic from the customer view.

These changes provide a professional and efficient merchant experience. Let’s test quickly:
- Log in as a merchant: You should land on `/merchant-home` with a banner and quick actions.
- Check the navbar: Should show "Home", "Dashboard", "Manage Products".
- In the dashboard, click each stat to see the respective details.
- In `ManageProducts`, click a product to see the detailed view with an image gallery and reviews.

Let me know if you need to adjust anything further! 🚀



Let’s address your concern about avoiding the creation of a separate `filter-low-stock.pipe.ts` file and instead find an alternative within the existing components. We’ll revisit the last response, identify the changes made, and modify the approach for the low stock filter to keep everything in the `MerchantDashboardComponent`. Then, I’ll summarize the changes from the last response and highlight what needs to be adjusted.

---

### Addressing the Separate Pipe File
In the last response, I suggested creating a `FilterLowStockPipe` to filter products with stock less than 100 for the "Low Stock Products" section in the `MerchantDashboardComponent`. You don’t want to create a separate file, so let’s explore an alternative: we can move the filtering logic directly into the `MerchantDashboardComponent` by either adding a method or a computed property to filter low stock products.

#### Alternative to `FilterLowStockPipe`
Instead of a pipe, we’ll add a method in `MerchantDashboardComponent` to filter low stock products and use it directly in the template. This keeps the logic within the component and avoids creating a new file.

---

### Revisiting the Last Response: Changes Made
Let’s break down the changes from the last response to understand what was done and what needs to be adjusted:

1. **Navbar Updates (`HeaderComponent`)**:
   - Moved "Dashboard" and "Manage Products" to the navbar for merchants.
   - Removed "Add Product" from the dropdown.

2. **Merchant Dashboard Enhancements (`MerchantDashboardComponent`)**:
   - Made stats clickable ("Total Products", "Total Stock", "Low Stock Products", "Average Product Rating").
   - **Total Products**: Navigates to `/manage-products`.
   - **Total Stock**: Shows a table with product names and stock.
   - **Low Stock Products**: Shows products with stock < 100 (this is where the pipe was used).
   - **Average Product Rating**: Shows product names and average ratings, with "No ratings so far" for unrated products.
   - Removed the "Quick Actions" section.

3. **Merchant Home Page (`MerchantHomeComponent`)**:
   - Created a new `MerchantHomeComponent` with a banner and quick actions.
   - Moved "Quick Actions" from the dashboard to this page, renamed "Create Product" to "Add Product", and added more options ("View Dashboard", "Update Profile").
   - Updated `app.routes.ts` to include the `/merchant-home` route.
   - Updated `RoleGuard` to redirect merchants to `/merchant-home`.

4. **Manage Products (`ManageProductsComponent`)**:
   - Made product cards clickable to navigate to `MerchantProductDetailComponent`.
   - Removed the "View Details" button.
   - Updated labels to "Add Product".

5. **Merchant Product Detail (`MerchantProductDetailComponent`)**:
   - Added an image gallery (similar to `ProductDetailComponent`).
   - Added a reviews section with ratings and comments.
   - Assumed a new `getProductReviews` method in `ProductService`.

6. **Filter Low Stock Pipe**:
   - Created `filter-low-stock.pipe.ts` to filter products with stock < 100.
   - Imported the pipe into `MerchantDashboardComponent`.

Since you’ve copied the files up to the creation of the `filter-low-stock.pipe.ts`, the pipe hasn’t been implemented yet, and we need to adjust the `MerchantDashboardComponent` to handle the low stock filtering without a separate pipe.

---

### Changes Needed: Adjusting `MerchantDashboardComponent`
We’ll modify `MerchantDashboardComponent` to include a method for filtering low stock products and update the template to use this method instead of the pipe. Here’s how we’ll do it:

#### Update `MerchantDashboardComponent` (Remove Pipe Dependency)
**Changes to `merchant-dashboard.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { RouterLink, Router } from '@angular/router';

@Component({
  selector: 'app-merchant-dashboard',
  standalone: true,
  imports: [CommonModule, RouterLink], // Removed FilterLowStockPipe
  templateUrl: './merchant-dashboard.component.html',
  styleUrls: ['./merchant-dashboard.component.css']
})
export class MerchantDashboardComponent implements OnInit {
  totalProducts: number = 0;
  totalStock: number = 0;
  lowStockProducts: number = 0;
  averageRating: number = 0;
  errorMessage: string | null = null;
  products: ProductResponse[] = [];
  showTotalStock: boolean = false;
  showLowStock: boolean = false;
  showAverageRating: boolean = false;

  constructor(private productService: ProductService, private router: Router) {}

  ngOnInit() {
    this.loadStats();
  }

  loadStats() {
    this.productService.getMerchantProducts().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.products = response.data;
          this.totalProducts = response.data.length;
          this.totalStock = response.data.reduce((sum, product) => sum + product.stock, 0);
          this.lowStockProducts = this.getLowStockProducts().length; // Use method to calculate low stock count

          const reviewedProducts = response.data.filter(product => product.reviewCount > 0);
          const totalRating = reviewedProducts.reduce((sum, product) => sum + product.averageRating, 0);
          this.averageRating = reviewedProducts.length > 0 ? totalRating / reviewedProducts.length : 0;
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

  // Method to filter low stock products
  getLowStockProducts(): ProductResponse[] {
    return this.products.filter(product => product.stock < 100);
  }

  onTotalProductsClick() {
    this.router.navigate(['/manage-products']);
  }

  onTotalStockClick() {
    this.showTotalStock = !this.showTotalStock;
    this.showLowStock = false;
    this.showAverageRating = false;
  }

  onLowStockClick() {
    this.showLowStock = !this.showLowStock;
    this.showTotalStock = false;
    this.showAverageRating = false;
  }

  onAverageRatingClick() {
    this.showAverageRating = !this.showAverageRating;
    this.showTotalStock = false;
    this.showLowStock = false;
  }
}
```

**Changes to `merchant-dashboard.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Merchant Dashboard</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>

  <!-- Stats Overview -->
  <div class="row">
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onTotalProductsClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Total Products</h5>
          <p class="card-text display-5 fw-bold text-primary">{{ totalProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onTotalStockClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Total Stock</h5>
          <p class="card-text display-5 fw-bold text-primary">{{ totalStock }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onLowStockClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Low Stock Products</h5>
          <p class="card-text display-5 fw-bold text-warning">{{ lowStockProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0 clickable" (click)="onAverageRatingClick()">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Average Product Rating</h5>
          <p class="card-text display-5 fw-bold text-success">{{ averageRating | number:'1.1-1' }}</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Total Stock Details -->
  <div *ngIf="showTotalStock" class="mt-4">
    <h4 class="fw-bold mb-3">Total Stock Details</h4>
    <table class="table table-bordered table-hover shadow-sm">
      <thead class="table-primary">
        <tr>
          <th>Product Name</th>
          <th>Stock</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let product of products">
          <td>{{ product.name }}</td>
          <td>{{ product.stock }}</td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Low Stock Details -->
  <div *ngIf="showLowStock" class="mt-4">
    <h4 class="fw-bold mb-3">Low Stock Products (Stock < 100)</h4>
    <table class="table table-bordered table-hover shadow-sm">
      <thead class="table-warning">
        <tr>
          <th>Product Name</th>
          <th>Stock Remaining</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let product of getLowStockProducts()"> <!-- Use method instead of pipe -->
          <td>{{ product.name }}</td>
          <td>{{ product.stock }}</td>
        </tr>
        <tr *ngIf="getLowStockProducts().length === 0">
          <td colspan="2" class="text-center text-muted">No products with low stock.</td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Average Rating Details -->
  <div *ngIf="showAverageRating" class="mt-4">
    <h4 class="fw-bold mb-3">Product Ratings</h4>
    <table class="table table-bordered table-hover shadow-sm">
      <thead class="table-success">
        <tr>
          <th>Product Name</th>
          <th>Average Rating</th>
        </tr>
      </thead>
      <tbody>
        <tr *ngFor="let product of products">
          <td>{{ product.name }}</td>
          <td>{{ product.reviewCount > 0 ? (product.averageRating | number:'1.1-1') : 'No ratings so far' }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
```

**Explanation of Changes**:
- **Added `getLowStockProducts` Method**:
  - Created a method in the component to filter products with stock < 100.
  - Used this method in `loadStats` to calculate `lowStockProducts` and in the template to display the low stock table.
- **Removed Pipe Dependency**:
  - Removed the import of `FilterLowStockPipe` from the component’s `imports`.
  - Replaced the pipe usage (`products | filterLowStock`) with the method call (`getLowStockProducts()`).
- **Preserved Functionality**:
  - The low stock section still shows products with stock < 100, and the table displays the product name and remaining stock.

---

### Things to Change from the Last Response
Since you’ve copied files up to the creation of the pipe, the remaining changes from the last response (after the pipe creation) are still relevant, but they don’t depend on the pipe. Let’s summarize those changes and ensure they’re applied correctly:

#### 1. **Create `MerchantHomeComponent`**
You need to create the `MerchantHomeComponent` as the new landing page for merchants. This includes:
- **File Creation**:
  - `merchant-home.component.ts`
  - `merchant-home.component.html`
  - `merchant-home.component.css`
- **Details**:
  - A banner welcoming the merchant.
  - Quick actions moved from the dashboard, with "Add Product" instead of "Create Product", and additional options like "View Dashboard" and "Update Profile".
- **Status**: These files haven’t been created yet in your project, so you’ll need to copy them as provided in the last response.

#### 2. **Update `app.routes.ts`**
You need to add the `/merchant-home` route and ensure it’s protected for merchants.
- **Change**:
  - Add `{ path: 'merchant-home', component: MerchantHomeComponent, canActivate: [authGuard, RoleGuard], data: { expectedRole: 'Merchant' } }`.
- **Status**: You’ll need to update your `app.routes.ts` with this route.

#### 3. **Update `RoleGuard`**
Modify `RoleGuard` to redirect merchants to `/merchant-home` instead of `/merchant-dashboard`.
- **Change**:
  - Update the redirect condition to navigate to `/merchant-home`.
- **Status**: You’ll need to apply this change to your `role.guard.ts`.

#### 4. **Update `ManageProductsComponent`**
Make product cards clickable and remove the "View Details" button.
- **Files**:
  - `manage-products.component.html`
  - `manage-products.component.css`
- **Status**: You’ll need to apply these changes to your `ManageProductsComponent`.

#### 5. **Update `MerchantProductDetailComponent`**
Add the image gallery and reviews section.
- **Files**:
  - `merchant-product-detail.component.ts`
  - `merchant-product-detail.component.html`
  - `merchant-product-detail.component.css`
- **Additional Requirement**:
  - Assumed a `getProductReviews` method in `ProductService`. You’ll need to ensure your backend supports this endpoint or adjust the reviews fetching logic accordingly.
- **Status**: You’ll need to apply these changes to your `MerchantProductDetailComponent`.

#### 6. **Remove Pipe Dependency in `MerchantDashboardComponent`**
This is the change we just made above, so you’re already set here.

---

### Summary of Actions
You’ve copied the changes up to the creation of the pipe, so the following steps remain:

1. **Apply the Updated `MerchantDashboardComponent`**:
   - Use the modified `merchant-dashboard.component.ts` and `.html` above to handle low stock filtering without a pipe.
   - No changes are needed for `merchant-dashboard.component.css` since it remains the same.

2. **Create `MerchantHomeComponent`**:
   - Copy the `merchant-home.component.ts`, `.html`, and `.css` files from the last response.
   - These files create the new landing page for merchants with a banner and quick actions.

3. **Update `app.routes.ts`**:
   - Add the `/merchant-home` route as shown in the last response.
   - Ensure all other routes remain consistent with your project.

4. **Update `RoleGuard`**:
   - Modify `role.guard.ts` to redirect merchants to `/merchant-home` as shown in the last response.

5. **Update `ManageProductsComponent`**:
   - Apply the changes to `manage-products.component.html` and `.css` to make product cards clickable and remove the "View Details" button.

6. **Update `MerchantProductDetailComponent`**:
   - Apply the changes to `merchant-product-detail.component.ts`, `.html`, and `.css` to add the image gallery and reviews.
   - Ensure your `ProductService` has a method to fetch reviews, or adjust the logic if your backend doesn’t support this.

---

### Test Scenarios
Let’s think about how to verify these changes work as expected:
- **Merchant Dashboard**:
  - Click "Low Stock Products"—it should show a table of products with stock < 100 using the new `getLowStockProducts` method.
- **Merchant Home Page**:
  - Log in as a merchant—you should land on `/merchant-home` with a banner and quick actions.
- **Navbar**:
  - Verify that "Dashboard" and "Manage Products" appear in the navbar for merchants.
- **Manage Products**:
  - Click a product card—it should navigate to the product detail page.
- **Merchant Product Detail**:
  - Verify the image gallery and reviews section display correctly.

Does this approach make sense, and are you ready to apply these remaining changes and test them? 🚀




Thank you for sharing the `ProductDetailComponent` code! Let’s analyze how reviews and ratings are handled in this component so we can apply the same functionality to the merchant side in `MerchantProductDetailComponent`. We’ll also examine the star-filling issue in the rating display, propose a fix (as requested to address at the end), and ensure the merchant’s rating display matches the customer’s implementation.

---

### Step 1: Analyze `ProductDetailComponent` for Ratings and Reviews
Let’s break down how `ProductDetailComponent` handles ratings and reviews, focusing on the relevant parts:

#### **Key Components in `ProductDetailComponent`**
1. **Services Used**:
   - `RatingService`: Fetches ratings for a product and user details for each rating.
   - `ReviewService`: Submits new reviews (though we’re only focusing on displaying ratings for the merchant, not submitting them).

2. **Rating Fetching Logic (`loadRatings`)**:
   - The `loadRatings` method calls `ratingService.getRatingsByProduct(productId)` to fetch ratings.
   - For each rating, it fetches the user’s details using `ratingService.getUserDetails(rating.userProfileId)` to get the reviewer’s name.
   - The ratings are stored in `ratings: any[]`, with each rating object enhanced to include `reviewerName`.

   ```typescript
   loadRatings(productId: number) {
     this.ratingService.getRatingsByProduct(productId).subscribe({
       next: (response) => {
         if (response.success && response.data) {
           const Apiratings = response.data;
           Apiratings.forEach(rating => {
             this.ratingService.getUserDetails(rating.userProfileId).subscribe({
               next: (userResponse) => {
                 if (userResponse != null) {
                   this.ratings.push({ ...rating, reviewerName: userResponse.fullName });
                 }
               },
               error: (err) => {
                 this.ratings.push({ ...rating, reviewerName: 'Unknown User' });
               }
             });
           });
         }
       },
       error: (err) => {
         console.error('Error fetching ratings:', err);
       }
     });
   }
   ```

3. **Rating Display in Template**:
   - The template displays each rating with stars, the review text, the reviewer’s name, and the date.
   - Stars are rendered using a loop with Bootstrap icons (`bi-star-fill` for filled stars, `bi-star` for empty stars).

   ```html
   <div *ngFor="let rating of ratings">
     <div class="d-flex align-items-center mb-2">
       <ng-container *ngFor="let i of [1,2,3,4,5]; let index = index">
         <i class="bi bi-star-fill text-warning me-1" *ngIf="rating.starRating >= index + 1"></i>
         <i class="bi bi-star text-warning me-1" *ngIf="rating.starRating <= index"></i>
       </ng-container>
     </div>
     <p *ngIf="rating.review" class="mb-2">{{ rating.review }}</p>
     <p class="text-muted small">BY {{ rating.reviewerName }}, {{ formatDate(rating.createdAt) }}</p>
   </div>
   ```

4. **Average Rating Display**:
   - The average rating is displayed using `product.averageRating`, with stars rendered using `filledStars` and `partialStarPercentage`.
   - The `calculateStarRating` method computes how many stars to fill fully and the percentage for a partial star.

   ```typescript
   calculateStarRating(avgRating: number) {
     this.filledStars = Math.floor(avgRating);
     const fractionalPart = avgRating - this.filledStars;
     this.partialStarPercentage = Math.round(fractionalPart * 100);
   }
   ```

   ```html
   <div class="star-rating">
     <span *ngFor="let star of [1, 2, 3, 4, 5]; let i = index" class="star">
       <span *ngIf="i < filledStars" class="filled">★</span>
       <span *ngIf="i === filledStars && partialStarPercentage > 0" class="partial" [style.width.%]="partialStarPercentage">★</span>
       <span *ngIf="i >= filledStars && (i !== filledStars || partialStarPercentage === 0)">☆</span>
     </span>
   </div>
   ```

5. **Star-Filling Issue**:
   - The issue in the star-filling code lies in the conditions and rendering logic for partial stars:
     - The condition `rating.starRating <= index` in the reviews section doesn’t handle partial stars (e.g., a rating of 3.5 shows 3 filled stars and 2 empty stars, missing the 0.5 partial star).
     - The `calculateStarRating` method for the average rating computes `partialStarPercentage`, but the template logic has a flaw: the partial star condition doesn’t always render correctly due to overlapping conditions.
   - We’ll fix this at the end, as requested, but let’s note the issue: the partial star rendering needs adjustment to handle fractional ratings more accurately.

---

### Step 2: Apply Rating Display to `MerchantProductDetailComponent`
We need to update `MerchantProductDetailComponent` to fetch and display ratings and reviews using `RatingService`, matching the customer-side implementation. The merchant should see the same rating and review display as the customer, including the average rating with stars and the list of reviews with stars, reviewer names, and dates.

#### **Updated `MerchantProductDetailComponent`**
**`merchant-product-detail.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ProductService, ProductResponse } from '../../services/product.service';
import { RatingService, ProfileDetails } from '../../services/rating.service'; // Import RatingService
import { ActivatedRoute, RouterLink } from '@angular/router';

@Component({
  selector: 'app-merchant-product-detail',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './merchant-product-detail.component.html',
  styleUrls: ['./merchant-product-detail.component.css']
})
export class MerchantProductDetailComponent implements OnInit {
  product: ProductResponse | null = null;
  ratings: any[] = []; // Match the structure in ProductDetailComponent
  errorMessage: string | null = null;
  currentImageIndex: number = 0;
  filledStars: number = 0; // For average rating stars
  partialStarPercentage: number = 0;

  constructor(
    private productService: ProductService,
    private ratingService: RatingService, // Inject RatingService
    private route: ActivatedRoute
  ) {}

  ngOnInit() {
    const productId = Number(this.route.snapshot.paramMap.get('id'));
    this.loadProduct(productId);
    this.loadRatings(productId);
    if (this.product?.averageRating) {
      this.calculateStarRating(this.product.averageRating);
    }
  }

  loadProduct(productId: number) {
    this.productService.getProduct(productId).subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.product = response.data;
          if (this.product.averageRating) {
            this.calculateStarRating(this.product.averageRating); // Calculate stars for average rating
          }
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

  loadRatings(productId: number) {
    this.ratingService.getRatingsByProduct(productId).subscribe({
      next: (response) => {
        if (response.success && response.data) {
          const Apiratings = response.data;
          Apiratings.forEach(rating => {
            this.ratingService.getUserDetails(rating.userProfileId).subscribe({
              next: (userResponse) => {
                if (userResponse != null) {
                  this.ratings.push({ ...rating, reviewerName: userResponse.fullName });
                }
              },
              error: (err) => {
                console.error(`Error fetching user details for user ${rating.userProfileId}:`, err);
                this.ratings.push({ ...rating, reviewerName: 'Unknown User' });
              }
            });
          });
        }
      },
      error: (err) => {
        console.error('Error fetching ratings:', err);
      }
    });
  }

  calculateStarRating(avgRating: number) {
    this.filledStars = Math.floor(avgRating);
    const fractionalPart = avgRating - this.filledStars;
    this.partialStarPercentage = Math.round(fractionalPart * 100);
  }

  changeImage(direction: number) {
    if (this.product && this.product.images) {
      const images = this.product.images.split(',');
      this.currentImageIndex = (this.currentImageIndex + direction + images.length) % images.length;
    }
  }

  formatDate(dateString: string): string {
    const date = new Date(dateString);
    return date.toLocaleString('en-US', { month: 'short', day: 'numeric', year: 'numeric', hour: 'numeric', minute: 'numeric', hour12: true });
  }
}
```

**`merchant-product-detail.component.html`** (Updated to match customer display):
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Product Details</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="product" class="card shadow-sm border-0">
    <div class="row g-0">
      <div class="col-md-4">
        <div class="position-relative">
          <img *ngIf="product.images" [src]="product.images.split(',')[currentImageIndex]" class="img-fluid rounded-start" alt="{{ product.name }}" style="height: 100%; object-fit: cover;">
          <div *ngIf="!product.images" class="bg-light d-flex align-items-center justify-content-center" style="height: 100%;">
            <span class="text-muted">No Image</span>
          </div>
          <button *ngIf="product.images && product.images.split(',').length > 1" class="btn btn-outline-secondary position-absolute start-0 top-50 translate-middle-y" (click)="changeImage(-1)">Previous</button>
          <button *ngIf="product.images && product.images.split(',').length > 1" class="btn btn-outline-secondary position-absolute end-0 top-50 translate-middle-y" (click)="changeImage(1)">Next</button>
        </div>
      </div>
      <div class="col-md-8">
        <div class="card-body">
          <h3 class="card-title">{{ product.name }}</h3>
          <div class="d-flex align-items-center mb-2">
            <div class="star-rating">
              <span *ngFor="let star of [1, 2, 3, 4, 5]; let i = index" class="star">
                <span *ngIf="i < filledStars" class="filled">★</span>
                <span *ngIf="i === filledStars && partialStarPercentage > 0" class="partial" [style.width.%]="partialStarPercentage">★</span>
                <span *ngIf="i >= filledStars && (i !== filledStars || partialStarPercentage === 0)">☆</span>
              </span>
            </div>
            <span class="ms-2">({{ product.averageRating | number:'1.1-1' }} / 5, {{ product.reviewCount }} reviews)</span>
          </div>
          <p class="card-text"><strong>Price:</strong> {{ product.price | currency }}</p>
          <p class="card-text"><strong>Stock:</strong> {{ product.stock }}</p>
          <p class="card-text"><strong>Category:</strong> {{ product.category || 'Not specified' }}</p>
          <p class="card-text"><strong>Type:</strong> {{ product.type || 'Not specified' }}</p>
          <p class="card-text"><strong>Description:</strong> {{ product.description || 'No description provided' }}</p>
          <p class="card-text"><strong>Specifications:</strong> {{ product.specifications || 'No specifications provided' }}</p>
          <div class="mt-4">
            <a [routerLink]="['/update-product', product.id]" class="btn btn-primary me-2">Edit Product</a>
            <a [routerLink]="['/manage-products']" class="btn btn-outline-secondary">Back to Manage Products</a>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Ratings and Reviews Section -->
  <div class="mt-5" *ngIf="product">
    <h3 class="fw-bold mb-3">Ratings & Reviews</h3>
    <div class="mb-4">
      <h5>Average Rating: {{ product.averageRating | number:'1.1-1' }} / 5 ({{ product.reviewCount }} reviews)</h5>
    </div>
    <div *ngIf="ratings.length > 0; else noReviews">
      <div class="border rounded p-3 mb-3" *ngFor="let rating of ratings">
        <div class="d-flex align-items-center mb-2">
          <ng-container *ngFor="let i of [1,2,3,4,5]; let index = index">
            <i class="bi bi-star-fill text-warning me-1" *ngIf="rating.starRating >= index + 1"></i>
            <i class="bi bi-star text-warning me-1" *ngIf="rating.starRating <= index"></i>
          </ng-container>
        </div>
        <p *ngIf="rating.review" class="mb-2">{{ rating.review }}</p>
        <p class="text-muted small">BY {{ rating.reviewerName }}, {{ formatDate(rating.createdAt) }}</p>
      </div>
    </div>
    <ng-template #noReviews>
      <p class="text-muted">No reviews yet for this product.</p>
    </ng-template>
  </div>
</div>
```

**`merchant-product-detail.component.css`** (Unchanged from last response):
```css
.card {
  border-radius: 8px;
}
.img-fluid {
  border-top-left-radius: 8px;
  border-bottom-left-radius: 8px;
}
.card-body {
  padding: 2rem;
}
.position-relative {
  height: 100%;
}
.btn-outline-secondary {
  background-color: rgba(255, 255, 255, 0.8);
}
```

**Explanation**:
- **Reused `RatingService`**:
  - Imported `RatingService` and used its `getRatingsByProduct` and `getUserDetails` methods to fetch ratings and reviewer names, exactly as done in `ProductDetailComponent`.
- **Added Average Rating Display**:
  - Added `filledStars` and `partialStarPercentage` properties and the `calculateStarRating` method to display the average rating with stars, matching the customer side.
- **Updated Template**:
  - Added the "Ratings & Reviews" section to display the average rating and list of reviews, using the same HTML structure as `ProductDetailComponent`.
  - The star rendering for individual reviews uses Bootstrap icons (`bi-star-fill` and `bi-star`), consistent with the customer view.
- **Reused `formatDate`**:
  - Copied the `formatDate` method to format the review date in the same way as the customer view.

---

### Step 3: Fix the Star-Filling Issue
Now, let’s address the star-filling issue in both `ProductDetailComponent` and `MerchantProductDetailComponent`. The issue is in two places:
1. **Average Rating Stars**:
   - The `calculateStarRating` method computes `filledStars` and `partialStarPercentage`, but the template logic has overlapping conditions, causing partial stars to render incorrectly.
2. **Individual Review Stars**:
   - The condition `rating.starRating <= index` doesn’t account for partial stars (e.g., a rating of 3.5 shows 3 filled stars and 2 empty stars, missing the partial star).

#### **Fixing the Star Rendering**
We’ll fix both components by:
- Adjusting the template logic for individual review stars to handle partial ratings.
- Simplifying the average rating star rendering to avoid overlapping conditions.

**Updated Star Rendering Logic for Both Components**:
1. **Individual Review Stars**:
   - We’ll modify the star rendering to show filled stars for whole numbers, a partial star for fractional ratings, and empty stars for the rest.
2. **Average Rating Stars**:
   - Fix the template conditions to ensure the partial star renders correctly.

#### **Update for `ProductDetailComponent`**
**`product-detail.component.ts`** (Add Fix in `loadRatings`):
```typescript
loadRatings(productId: number) {
  this.ratingService.getRatingsByProduct(productId).subscribe({
    next: (response) => {
      if (response.success && response.data) {
        const Apiratings = response.data;
        Apiratings.forEach(rating => {
          this.ratingService.getUserDetails(rating.userProfileId).subscribe({
            next: (userResponse) => {
              if (userResponse != null) {
                // Calculate star display for each rating
                const filledStars = Math.floor(rating.starRating);
                const partialStarPercentage = (rating.starRating - filledStars) * 100;
                this.ratings.push({ ...rating, reviewerName: userResponse.fullName, filledStars, partialStarPercentage });
              }
            },
            error: (err) => {
              console.error(`Error fetching user details for user ${rating.userProfileId}:`, err);
              const filledStars = Math.floor(rating.starRating);
              const partialStarPercentage = (rating.starRating - filledStars) * 100;
              this.ratings.push({ ...rating, reviewerName: 'Unknown User', filledStars, partialStarPercentage });
            }
          });
        });
      }
    },
    error: (err) => {
      console.error('Error fetching ratings:', err);
    }
  });
}
```

**`product-detail.component.html`** (Fix Star Rendering):
Replace the "Ratings & Reviews" section with:
```html
<div class="mt-5">
  <h3 class="fw-bold mb-3">Ratings & Reviews</h3>
  <div class="mb-4">
    <h5>Average Rating: {{ product!.averageRating.toFixed(1) }} / 5 ({{ product!.reviewCount }} reviews)</h5>
    <div class="star-rating">
      <span *ngFor="let star of [1, 2, 3, 4, 5]; let i = index" class="star">
        <span *ngIf="i < filledStars" class="filled">★</span>
        <span *ngIf="i === filledStars && partialStarPercentage > 0" class="partial" [style.clip-path]="'inset(0 ' + (100 - partialStarPercentage) + '% 0 0)'">★</span>
        <span *ngIf="i > filledStars || (i === filledStars && partialStarPercentage === 0)">☆</span>
      </span>
    </div>
  </div>
  <div *ngIf="ratings.length > 0; else noReviews">
    <div class="border rounded p-3 mb-3" *ngFor="let rating of ratings">
      <div class="d-flex align-items-center mb-2">
        <span *ngFor="let i of [1, 2, 3, 4, 5]; let index = index" class="star">
          <i class="bi bi-star-fill text-warning me-1" *ngIf="index < rating.filledStars"></i>
          <i class="bi bi-star-fill text-warning me-1" *ngIf="index === rating.filledStars && rating.partialStarPercentage > 0" [style.clip-path]="'inset(0 ' + (100 - rating.partialStarPercentage) + '% 0 0)'"></i>
          <i class="bi bi-star text-warning me-1" *ngIf="index >= rating.filledStars && (index !== rating.filledStars || rating.partialStarPercentage === 0)"></i>
        </span>
      </div>
      <p *ngIf="rating.review" class="mb-2">{{ rating.review }}</p>
      <p class="text-muted small">BY {{ rating.reviewerName }}, {{ formatDate(rating.createdAt) }}</p>
    </div>
  </div>
  <ng-template #noReviews>
    <p class="text-muted">No reviews yet for this product.</p>
  </ng-template>
</div>
```

#### **Update for `MerchantProductDetailComponent`**
**`merchant-product-detail.component.ts`** (Add Fix in `loadRatings`):
```typescript
loadRatings(productId: number) {
  this.ratingService.getRatingsByProduct(productId).subscribe({
    next: (response) => {
      if (response.success && response.data) {
        const Apiratings = response.data;
        Apiratings.forEach(rating => {
          this.ratingService.getUserDetails(rating.userProfileId).subscribe({
            next: (userResponse) => {
              if (userResponse != null) {
                const filledStars = Math.floor(rating.starRating);
                const partialStarPercentage = (rating.starRating - filledStars) * 100;
                this.ratings.push({ ...rating, reviewerName: userResponse.fullName, filledStars, partialStarPercentage });
              }
            },
            error: (err) => {
              console.error(`Error fetching user details for user ${rating.userProfileId}:`, err);
              const filledStars = Math.floor(rating.starRating);
              const partialStarPercentage = (rating.starRating - filledStars) * 100;
              this.ratings.push({ ...rating, reviewerName: 'Unknown User', filledStars, partialStarPercentage });
            }
          });
        });
      }
    },
    error: (err) => {
      console.error('Error fetching ratings:', err);
    }
  });
}
```

**`merchant-product-detail.component.html`** (Fix Star Rendering):
Update the "Ratings & Reviews" section:
```html
<div class="mt-5" *ngIf="product">
  <h3 class="fw-bold mb-3">Ratings & Reviews</h3>
  <div class="mb-4">
    <h5>Average Rating: {{ product.averageRating | number:'1.1-1' }} / 5 ({{ product.reviewCount }} reviews)</h5>
    <div class="star-rating">
      <span *ngFor="let star of [1, 2, 3, 4, 5]; let i = index" class="star">
        <span *ngIf="i < filledStars" class="filled">★</span>
        <span *ngIf="i === filledStars && partialStarPercentage > 0" class="partial" [style.clip-path]="'inset(0 ' + (100 - partialStarPercentage) + '% 0 0)'">★</span>
        <span *ngIf="i > filledStars || (i === filledStars && partialStarPercentage === 0)">☆</span>
      </span>
    </div>
  </div>
  <div *ngIf="ratings.length > 0; else noReviews">
    <div class="border rounded p-3 mb-3" *ngFor="let rating of ratings">
      <div class="d-flex align-items-center mb-2">
        <span *ngFor="let i of [1, 2, 3, 4, 5]; let index = index" class="star">
          <i class="bi bi-star-fill text-warning me-1" *ngIf="index < rating.filledStars"></i>
          <i class="bi bi-star-fill text-warning me-1" *ngIf="index === rating.filledStars && rating.partialStarPercentage > 0" [style.clip-path]="'inset(0 ' + (100 - rating.partialStarPercentage) + '% 0 0)'"></i>
          <i class="bi bi-star text-warning me-1" *ngIf="index > rating.filledStars || (index === rating.filledStars && rating.partialStarPercentage === 0)"></i>
        </span>
      </div>
      <p *ngIf="rating.review" class="mb-2">{{ rating.review }}</p>
      <p class="text-muted small">BY {{ rating.reviewerName }}, {{ formatDate(rating.createdAt) }}</p>
    </div>
  </div>
  <ng-template #noReviews>
    <p class="text-muted">No reviews yet for this product.</p>
  </ng-template>
</div>
```

**Explanation of Fix**:
- **Individual Review Stars**:
  - Added `filledStars` and `partialStarPercentage` to each rating object when pushing to the `ratings` array.
  - Updated the template to render filled stars, a partial star (using `clip-path` to show a portion of the star), and empty stars.
- **Average Rating Stars**:
  - Fixed the conditions to avoid overlap: a star is either fully filled, partially filled, or empty, with no overlapping rendering.
  - Used `clip-path` instead of `width` for partial stars to ensure better rendering across browsers.
- **Consistency**:
  - Both components now use the same star rendering logic for consistency.

---

### Summary of Changes
The `MerchantProductDetailComponent` now displays ratings and reviews exactly like the customer side, using `RatingService`. The star-filling issue has been fixed in both components by improving the rendering logic for partial stars. The remaining changes from the previous responses (e.g., creating `MerchantHomeComponent`, updating `app.routes.ts`, `RoleGuard`, and `ManageProductsComponent`) are still needed and unchanged.

**Test Scenarios**:
- **Merchant Product Detail**:
  - Check if ratings and reviews display with correct star rendering (e.g., a 3.5 rating shows 3 filled stars and a half-filled star).
- **Customer Product Detail**:
  - Verify the star rendering fix works for both average and individual ratings.

Are you ready to apply these updates and test the functionality? 🚀
