Let’s address the issues you’ve raised and enhance the merchant experience in a professional manner. We’ll focus on creating a tailored experience for merchants that aligns with their role, ensuring the interface is sleek, functional, and professional. I’ll avoid childish elements and focus on a design that a professional developer would implement for a business application like EShoppingZone.

---

### Issues to Fix and Plan

1. **Merchant Should Not Have "Products" in Navbar**:
   - Remove the "Products" link from the navbar for merchants since they don’t need to browse products like customers do.
   - Adjust the navbar to focus on merchant-specific actions (e.g., manage products, dashboard).

2. **Home Page for Merchants Should Be Different**:
   - Currently, the home page (`HomeComponent`) is the same for all users, showing all products. For merchants, redirect them to the `MerchantDashboardComponent` as their home page instead of showing customer-focused content.

3. **Merchant Dashboard Needs to Be More Professional**:
   - Redesign the dashboard to display more relevant metrics (e.g., total products, total stock, products with low stock, total sales/orders if API available).
   - Fix the average rating calculation to exclude products with no reviews (i.e., `reviewCount == 0`).
   - Use a cleaner, more professional layout.

4. **Manage Products Enhancements**:
   - Add an "Add Product" button directly in `ManageProductsComponent`.
   - Display product images (using the `images` field from `ProductResponse`).
   - Allow merchants to click a product to view a detailed page (`MerchantProductDetailComponent`) with an option to edit from there.
   - Improve the layout to look more professional.

5. **Profile Dropdown - Remove Childish "Add Product" Option**:
   - Replace the "Add Product 🆕" option in the dropdown with a more professional label (e.g., "Create Product").
   - Ensure the dropdown aligns with a professional tone.

6. **Overall Professional Approach**:
   - Use consistent, professional terminology (e.g., avoid emojis in critical areas, use proper labels).
   - Focus on functionality and usability for a business user (merchant).

---

### Implementation

#### 1. Update Navbar to Remove "Products" for Merchants
We’ll modify the `HeaderComponent` to conditionally hide the "Products" link for merchants and update the dropdown labels to be more professional.

**Changes to `header.component.ts`**:
The logic remains largely the same, but we’ll ensure the `isMerchant` property is used to control visibility. No changes are needed here since `isMerchant` is already defined.

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
        <!-- Show Products and Deals only for non-merchants -->
        <li class="nav-item" *ngIf="!isMerchant">
          <a class="nav-link" [routerLink]="['/products']" routerLinkActive="active">Products</a>
        </li>
        <li class="nav-item" *ngIf="!isMerchant">
          <a class="nav-link" [routerLink]="['/deals']" routerLinkActive="active">Deals</a>
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

              <!-- Merchant-Specific Options -->
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/merchant-dashboard')">Dashboard</a>
              </li>
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/add-product')">Create Product</a>
              </li>
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/manage-products')">Manage Products</a>
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
- Removed the "Products" and "Deals" links for merchants using `*ngIf="!isMerchant"`.
- Hid the search bar for merchants since they don’t need to search for products.
- Updated the dropdown to use "Create Product" instead of "Add Product 🆕" for a more professional tone.
- Removed all emojis to maintain a professional look.

#### 2. Redirect Merchant Home Page to Dashboard
Currently, the home page (`HomeComponent`) shows all products, which is customer-focused. For merchants, we’ll redirect the root path (`/`) to `/merchant-dashboard`.

**Changes to `app.routes.ts`**:
We’ll add a route guard to redirect based on user role.

**New File: `role-redirect.guard.ts`**:
```typescript
import { Injectable } from '@angular/core';
import { CanActivate, Router } from '@angular/router';
import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root'
})
export class RoleRedirectGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}

  canActivate(): boolean {
    if (this.authService.isLoggedIn() && this.authService.getUserRole() === 'Merchant') {
      this.router.navigate(['/merchant-dashboard']);
      return false;
    }
    return true;
  }
}
```

**Changes to `app.routes.ts`**:
```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './components/home/home.component';
import { LoginComponent } from './components/login/login.component';
import { RegisterComponent } from './components/register/register.component';
import { AuthGuard } from './guards/auth.guard';
import { RoleRedirectGuard } from './guards/role-redirect.guard';
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
  { path: '', component: HomeComponent, canActivate: [RoleRedirectGuard] },
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

**Explanation**:
- Added `RoleRedirectGuard` to check if the user is a merchant. If so, redirects them to `/merchant-dashboard`.
- Applied the guard to the root path (`/`) so merchants are automatically taken to their dashboard instead of the customer-focused `HomeComponent`.

#### 3. Redesign Merchant Dashboard
We’ll redesign the `MerchantDashboardComponent` to be more professional, fix the average rating calculation, and add more relevant metrics (e.g., low stock products).

**Changes to `merchant-dashboard.component.ts`**:
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
  lowStockProducts: number = 0;
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
          this.lowStockProducts = response.data.filter(product => product.stock < 10).length; // Define low stock as < 10

          // Calculate average rating, excluding products with no reviews
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
      <div class="card shadow-sm border-0">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Total Products</h5>
          <p class="card-text display-5 fw-bold text-primary">{{ totalProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Total Stock</h5>
          <p class="card-text display-5 fw-bold text-primary">{{ totalStock }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Low Stock Products</h5>
          <p class="card-text display-5 fw-bold text-warning">{{ lowStockProducts }}</p>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-4">
      <div class="card shadow-sm border-0">
        <div class="card-body text-center">
          <h5 class="card-title text-muted">Average Product Rating</h5>
          <p class="card-text display-5 fw-bold text-success">{{ averageRating | number:'1.1-1' }}</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Quick Actions -->
  <div class="mt-5">
    <h4 class="fw-bold mb-3">Quick Actions</h4>
    <div class="d-flex gap-3">
      <a [routerLink]="['/add-product']" class="btn btn-primary">Create Product</a>
      <a [routerLink]="['/manage-products']" class="btn btn-outline-primary">Manage Products</a>
    </div>
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
.btn {
  border-radius: 5px;
}
```

**Explanation**:
- **Average Rating Fix**: Excluded products with `reviewCount == 0` when calculating the average rating.
- **New Metric**: Added `lowStockProducts` to show products with stock below 10, helping merchants identify items needing restocking.
- **Professional Design**:
  - Used a clean card layout with subtle shadows and hover effects.
  - Added color coding for metrics (blue for total products/stock, yellow for low stock, green for average rating).
  - Removed emojis and used professional labels (e.g., "Create Product" instead of "Add New Product 🆕").
  - Adjusted font sizes and spacing for a polished look.

#### 4. Enhance Manage Products
We’ll add an "Add Product" button, display product images, and allow merchants to click a product to view its details with an edit option.

**Changes to `manage-products.component.ts`**:
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

**Changes to `manage-products.component.html`**:
```html
<div class="container py-5 px-5">
  <div class="d-flex justify-content-between align-items-center mb-4">
    <h2 class="fw-bold">Manage Products</h2>
    <a [routerLink]="['/add-product']" class="btn btn-primary">Create Product</a>
  </div>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="products.length > 0; else noProducts">
    <div class="row">
      <div class="col-md-4 mb-4" *ngFor="let product of products">
        <div class="card shadow-sm border-0 h-100">
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
              <a [routerLink]="['/merchant-product-detail', product.id]" class="btn btn-outline-primary btn-sm">View Details</a>
              <a [routerLink]="['/update-product', product.id]" class="btn btn-outline-secondary btn-sm">Edit</a>
              <button class="btn btn-outline-danger btn-sm" (click)="deleteProduct(product.id)">Delete</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <ng-template #noProducts>
    <p class="text-muted text-center">No products found. Create a new product to get started.</p>
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
```

**Explanation**:
- **Add Product Button**: Added a "Create Product" button at the top right of the page.
- **Product Images**: Displayed the product image using the `images` field from `ProductResponse`. If no image exists, a placeholder is shown.
- **Grid Layout**: Used a Bootstrap grid (`col-md-4`) to display products in a responsive 3-column layout.
- **View Details Link**: Added a "View Details" button linking to a new `MerchantProductDetailComponent`.
- **Professional Design**: Used subtle shadows, hover effects, and smaller buttons (`btn-sm`) for a cleaner look.

#### 5. Create Merchant Product Detail Page
Create a new `MerchantProductDetailComponent` to allow merchants to view detailed product information and edit if needed.

**New File: `merchant-product-detail.component.ts`**:
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

  constructor(
    private productService: ProductService,
    private route: ActivatedRoute
  ) {}

  ngOnInit() {
    const productId = Number(this.route.snapshot.paramMap.get('id'));
    this.loadProduct(productId);
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
}
```

**New File: `merchant-product-detail.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">Product Details</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>
  <div *ngIf="product" class="card shadow-sm border-0">
    <div class="row g-0">
      <div class="col-md-4">
        <img *ngIf="product.images" [src]="product.images" class="img-fluid rounded-start" alt="{{ product.name }}" style="height: 100%; object-fit: cover;">
        <div *ngIf="!product.images" class="bg-light d-flex align-items-center justify-content-center" style="height: 100%;">
          <span class="text-muted">No Image</span>
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
</div>
```

**New File: `merchant-product-detail.component.css`**:
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
```

**Add Route in `app.routes.ts`**:
```typescript
import { Routes } from '@angular/router';
import { HomeComponent } from './components/home/home.component';
import { LoginComponent } from './components/login/login.component';
import { RegisterComponent } from './components/register/register.component';
import { AuthGuard } from './guards/auth.guard';
import { RoleRedirectGuard } from './guards/role-redirect.guard';
import { ProductDetailComponent } from './components/product-detail/product-detail.component';
import { CartComponent } from './components/cart/cart.component';
import { ProfileComponent } from './components/profile/profile.component';
import { UpdateProfileComponent } from './components/update-profile/update-profile.component';
import { ManageAddressesComponent } from './components/manage-addresses/manage-addresses.component';
import { MerchantDashboardComponent } from './components/merchant-dashboard/merchant-dashboard.component';
import { AddProductComponent } from './components/add-product/add-product.component';
import { ManageProductsComponent } from './components/manage-products/manage-products.component';
import { UpdateProductComponent } from './components/update-product/update-product.component';
import { MerchantProductDetailComponent } from './components/merchant-product-detail/merchant-product-detail.component';

export const routes: Routes = [
  { path: '', component: HomeComponent, canActivate: [RoleRedirectGuard] },
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
  { path: 'merchant-product-detail/:id', component: MerchantProductDetailComponent, canActivate: [AuthGuard] },
  { path: '**', redirectTo: '' }
];
```

**Explanation**:
- Created `MerchantProductDetailComponent` to display detailed product information.
- Included an image on the left and product details on the right for a professional layout.
- Added buttons to edit the product or return to the manage products page.

---

### Summary of Changes
- **Navbar**:
  - Removed "Products" and "Deals" links for merchants.
  - Hid the search bar for merchants.
  - Updated the dropdown to use professional labels ("Create Product").
- **Home Page**:
  - Redirected merchants to `/merchant-dashboard` using `RoleRedirectGuard`.
- **Merchant Dashboard**:
  - Fixed average rating to exclude products with no reviews.
  - Added "Low Stock Products" metric.
  - Redesigned with a professional card layout, color-coded metrics, and no emojis.
- **Manage Products**:
  - Added a "Create Product" button.
  - Displayed product images.
  - Used a grid layout for a cleaner presentation.
  - Added a "View Details" link to navigate to the new `MerchantProductDetailComponent`.
- **Merchant Product Detail**:
  - Created a detailed view for products with an option to edit.
- **Professional Tone**:
  - Removed all emojis and used professional terminology throughout.

---

### Test
1. **Login as Merchant**:
   - Navbar should only show "Home" (redirects to dashboard).
   - Dropdown should show "Profile", "Update Profile", "Dashboard", "Create Product", "Manage Products", "Logout".
2. **Merchant Dashboard**:
   - Should show total products, total stock, low stock products, and average rating (excluding unreviewed products).
   - Layout should look clean and professional.
3. **Manage Products**:
   - Should display products in a grid with images.
   - Should have a "Create Product" button at the top.
   - Clicking "View Details" should take you to the product detail page.
4. **Merchant Product Detail**:
   - Should show all product details with an image.
   - Should have an "Edit Product" button linking to `/update-product/:id`.

This implementation provides a professional and merchant-focused experience. Let me know if you’d like further refinements! 🚀



Let’s address the issues and make the necessary updates to the `HeaderComponent` to ensure a seamless and professional experience. The main problems are:

1. **Header Not Updating After Login**: The header (profile name, navbar options) doesn’t update immediately after login; a refresh is required. This is because the `ngOnInit` logic doesn’t re-run after login, and there’s no mechanism to react to authentication state changes.
2. **Merchant Navbar Options**: The navbar still shows customer-specific options (e.g., "Products", "Deals", search bar) for merchants until refreshed. We’ll ensure these are hidden for merchants as previously implemented, but we’ll make it reactive.
3. **Professional Tone**: We’ll remove the emojis and refine the dropdown labels (e.g., replace "Add Product 🆕" with "Create Product") to align with a professional design.

I’ll also assume you’re using the latest `header.component.ts` from our previous interactions (with `isMerchant` and `cartService.cartUpdate$`), but I’ll ensure it’s reactive to login/logout events.

---

### Changes to `header.component.ts`
We need to make the header reactive to authentication state changes. We’ll use a `BehaviorSubject` in `AuthService` to emit login/logout events and subscribe to it in `HeaderComponent`. This will ensure the header updates immediately after login without requiring a refresh.

**Update `auth.service.ts` (Add Authentication State Subject)**:
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, BehaviorSubject } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private baseUrl = 'https://api.eshoppingzone.com';
  private authState = new BehaviorSubject<boolean>(this.isLoggedIn());
  authState$ = this.authState.asObservable();

  constructor(private http: HttpClient) {}

  // Existing methods...

  login(loginRequest: any): Observable<any> {
    return this.http.post<any>(`${this.baseUrl}/api/Auth/Login`, loginRequest).pipe(
      tap(response => {
        if (response.success && response.data) {
          localStorage.setItem('token', response.data.token);
          localStorage.setItem('role', response.data.role);
          this.authState.next(true); // Emit login event
        }
      })
    );
  }

  logout() {
    localStorage.removeItem('token');
    localStorage.removeItem('role');
    this.authState.next(false); // Emit logout event
  }

  isLoggedIn(): boolean {
    return !!localStorage.getItem('token');
  }

  getUserRole(): string | null {
    return localStorage.getItem('role');
  }

  viewProfile(): Observable<any> {
    return this.http.get<any>(`${this.baseUrl}/api/Auth/ViewProfile`);
  }
}
```

**Updated `header.component.ts`**:
```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink, RouterLinkActive, Router, NavigationEnd } from '@angular/router';
import { AuthService } from '../../services/auth.service';
import { CartService } from '../../services/cart.service';
import { Subscription } from 'rxjs';

@Component({
  selector: 'app-header',
  standalone: true,
  imports: [CommonModule, RouterLink, RouterLinkActive],
  templateUrl: './header.component.html',
  styleUrls: ['./header.component.css']
})
export class HeaderComponent implements OnInit, OnDestroy {
  searchQuery: string = '';
  isDropdownOpen: boolean = false;
  cartItemCount: number = 0;
  firstName: string | null = null;
  isMerchant: boolean = false;
  private authSubscription!: Subscription;

  constructor(
    public authService: AuthService,
    private router: Router,
    private cartService: CartService
  ) {
    this.updateCartCount();
  }

  ngOnInit() {
    // Subscribe to auth state changes
    this.authSubscription = this.authService.authState$.subscribe(isLoggedIn => {
      this.updateHeaderState(isLoggedIn);
    });

    // Subscribe to navigation events to close dropdown
    this.router.events.subscribe(event => {
      if (event instanceof NavigationEnd) {
        this.closeDropdown();
      }
    });
  }

  ngOnDestroy() {
    if (this.authSubscription) {
      this.authSubscription.unsubscribe();
    }
  }

  updateHeaderState(isLoggedIn: boolean) {
    if (isLoggedIn) {
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

      this.isMerchant = this.authService.getUserRole() === 'Merchant';

      if (!this.isMerchant) {
        this.cartService.cartUpdate$.subscribe(count => {
          this.cartItemCount = count;
        });
        this.updateCartCount();
      } else {
        this.cartItemCount = 0; // Reset cart count for merchants
      }
    } else {
      this.firstName = null;
      this.isMerchant = false;
      this.cartItemCount = 0;
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
            this.cartItemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0);
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
- **Reactive Auth State**:
  - Added a `BehaviorSubject` in `AuthService` (`authState`) to emit login/logout events.
  - Subscribed to `authState$` in `HeaderComponent` to update the header state (`firstName`, `isMerchant`, `cartItemCount`) whenever the user logs in or out.
- **Lifecycle Hooks**:
  - Implemented `OnDestroy` to unsubscribe from `authState$` and prevent memory leaks.
- **State Reset**:
  - When the user logs out, reset `firstName`, `isMerchant`, and `cartItemCount` to their default values.
- **Preserved Existing Logic**:
  - All existing methods (`search`, `toggleDropdown`, `closeDropdown`, `logout`, `navigateTo`, `updateCartCount`) remain unchanged.

---

### Changes to `header.component.html`
We’ll update the HTML to:
- Hide "Products", "Deals", and the search bar for merchants (already implemented, but ensuring it’s reactive).
- Remove all emojis for a professional look.
- Update dropdown labels to be more professional (e.g., "Create Product" instead of "Add Product 🆕").

**Updated `header.component.html`**:
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
        <!-- Show Products and Deals only for non-merchants -->
        <li class="nav-item" *ngIf="!isMerchant">
          <a class="nav-link" [routerLink]="['/products']" routerLinkActive="active">Products</a>
        </li>
        <li class="nav-item" *ngIf="!isMerchant">
          <a class="nav-link" [routerLink]="['/deals']" routerLinkActive="active">Deals</a>
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

              <!-- Merchant-Specific Options -->
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/merchant-dashboard')">Dashboard</a>
              </li>
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/add-product')">Create Product</a>
              </li>
              <li *ngIf="isMerchant">
                <a class="dropdown-item" (click)="navigateTo('/manage-products')">Manage Products</a>
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

#### Explanation of Changes in `header.component.html`
- **Professional Tone**:
  - Removed all emojis (e.g., 🛍️, 🏠, 📦, 🎉, 🔑, 📝, 🛒, 👤, ✏️, 📊, 🆕, 🚪) to align with a professional design.
  - Updated the "Add Product 🆕" label to "Create Product".
- **Reactive Display**:
  - Ensured `isMerchant` updates reactively after login, so the navbar options (e.g., "Products", "Deals", search bar) are hidden for merchants immediately.
  - The `authService.isLoggedIn()` check is now reactive due to the `authState$` subscription.
- **Consistency**:
  - Kept the structure consistent with previous updates (e.g., hiding "Products" and "Deals" for merchants, showing "Cart" only for customers).

---

### Test
1. **Login as Customer**:
   - Immediately after login, the header should update to show the profile name (`firstName`), "Cart" link with item count, and customer-specific dropdown options ("Manage Addresses", "Place Order", "Order History").
   - Navbar should show "Home", "Products", and "Deals".
2. **Login as Merchant**:
   - Immediately after login, the header should update to show the profile name, and merchant-specific dropdown options ("Dashboard", "Create Product", "Manage Products").
   - "Products", "Deals", and search bar should be hidden.
   - "Cart" link should not be visible.
3. **Logout**:
   - After logout, the header should immediately show "Login" and "Signup" buttons.
4. **No Refresh Needed**:
   - All changes (profile name, navbar options) should occur without refreshing the page.

---

### Summary of Fixes
- **Header Not Updating After Login**:
  - Added a `BehaviorSubject` in `AuthService` to emit auth state changes.
  - Subscribed to `authState$` in `HeaderComponent` to update the header reactively.
- **Merchant Navbar Options**:
  - Ensured `isMerchant` updates reactively, hiding customer-specific options for merchants immediately after login.
- **Professional Design**:
  - Removed all emojis and updated labels (e.g., "Create Product") for a professional tone.

This should resolve the issues and provide a polished, reactive header experience. Let me know if you need further adjustments! 🚀
