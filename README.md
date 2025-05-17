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
