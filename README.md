Got it — we’ll make your merchant dashboard clean, modern, and better than the competition.

---

### ✅ Here's the Plan to Build a Killer Merchant Dashboard:

#### 🔹 Step 1: Merchant Navbar with Profile Dropdown

We'll include:

* "Dashboard", "Add Product", "Manage Products", "Orders"
* A profile dropdown with "My Profile", "Change Password", "Logout"

#### 🔹 Step 2: Dashboard Layout

We'll have a clean layout with:

* **Quick Stats** (Total Products, Total Orders, Total Revenue)
* Recent Orders
* Product Performance

#### 🔹 Step 3: Routing Setup

Each tab in the nav will route to:

* `/merchant/dashboard`
* `/merchant/products`
* `/merchant/products/add`
* `/merchant/orders`
* `/merchant/profile`

#### 🔹 Step 4: Guard (Merchant Only)

We'll reuse auth guard and extend it if needed to check for merchant role.

---

### ✅ Step-by-Step: Starting with Merchant Navbar Component

#### 🧩 1. Merchant Navbar (`merchant-navbar.component.html`)

```html
<nav class="bg-white shadow-md px-6 py-3 flex justify-between items-center">
  <a routerLink="/merchant/dashboard" class="text-2xl font-bold text-indigo-600">MerchantPanel</a>

  <div class="flex items-center space-x-6">
    <a routerLink="/merchant/dashboard" class="hover:text-indigo-600">Dashboard</a>
    <a routerLink="/merchant/products" class="hover:text-indigo-600">Products</a>
    <a routerLink="/merchant/products/add" class="hover:text-indigo-600">Add Product</a>
    <a routerLink="/merchant/orders" class="hover:text-indigo-600">Orders</a>

    <!-- Profile Dropdown -->
    <div class="relative" (mouseenter)="showDropdown = true" (mouseleave)="showDropdown = false">
      <button class="flex items-center space-x-2 text-gray-800">
        <i class="fas fa-user-circle text-xl"></i>
        <span>Profile</span>
      </button>
      <div *ngIf="showDropdown" class="absolute right-0 mt-2 w-48 bg-white shadow-md rounded z-10">
        <a routerLink="/merchant/profile" class="block px-4 py-2 hover:bg-gray-100">My Profile</a>
        <a routerLink="/merchant/change-password" class="block px-4 py-2 hover:bg-gray-100">Change Password</a>
        <a (click)="logout()" class="block px-4 py-2 text-red-500 hover:bg-red-50">Logout</a>
      </div>
    </div>
  </div>
</nav>
```

---

#### 🧠 2. `merchant-navbar.component.ts`

```ts
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-merchant-navbar',
  templateUrl: './merchant-navbar.component.html',
  standalone: true,
})
export class MerchantNavbarComponent {
  showDropdown = false;

  constructor(private router: Router) {}

  logout() {
    localStorage.removeItem('token');
    this.router.navigate(['/login']);
  }
}
```

---

### ✅ Up Next

Let’s build the **merchant dashboard page** with:

* Total Products
* Total Orders
* Revenue
* Recent Orders Table

Do you already have:

* Merchant API endpoints like `GetOrdersByMerchantId`, `GetAllMerchantProducts`, etc.?
* Or should I mock them for now and build the structure?

Once you confirm that, we’ll move to designing the **dashboard component with responsive cards and table**, then proceed step-by-step.
