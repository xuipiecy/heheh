I’m sorry for the confusion, bro. I understand now—you want the `.ts` and `.html` files separated for each component, instead of having the template embedded in the `.ts` file using the `template` property. Let’s fix that by splitting the files into separate `.ts` and `.html` files for each component. I’ll provide the updated files for the components we worked on: `header`, `admin-dashboard`, `pending-merchant-requests`, and `pending-delivery-agent-requests`.

---

### Step 1: Split Files into `.ts` and `.html`

I’ll separate the TypeScript logic (`.ts`) and the HTML templates (`.html`) for each component. The `.ts` files will reference their respective `.html` templates using the `templateUrl` property.

#### Component 1: `header`

**`header.component.ts`**:
```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink, RouterLinkActive, Router, NavigationEnd } from '@angular/router';
import { AuthService } from '../../services/auth.service';
import { CartService } from '../../services/cart.service';
import { RoleService } from '../../services/role.service';
import { Subscription } from 'rxjs';

interface RoleRequest {
  id: number;
  userId: number;
  requestedRole: string;
  status: string;
  requestedAt: string;
  reviewedAt: string | null;
}

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
  isCustomer: boolean = false;
  isDeliveryAgent: boolean = false;
  isAdmin: boolean = false;
  selectedRole: string | null = null;
  roleRequest: RoleRequest | null = null;
  roleRequestMessage: string | null = null;
  errorMessage: string | null = null;
  showConfirmModal: boolean = false;
  showPendingModal: boolean = false;
  showStatusModal: boolean = false;
  showMessageModal: boolean = false;
  private authSubscription!: Subscription;

  constructor(
    public authService: AuthService,
    private router: Router,
    private cartService: CartService,
    private roleService: RoleService
  ) {
    this.updateCartCount();
  }

  ngOnInit() {
    this.authSubscription = this.authService.authState$.subscribe(isLoggedIn => {
      this.updateHeaderState(isLoggedIn);
    });

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

  getHomeRoute(): string {
    if (this.isMerchant) {
      return '/merchant-home';
    } else if (this.isDeliveryAgent) {
      return '/delivery-agent-home';
    } else if (this.isAdmin) {
      return '/admin-home';
    } else {
      return '/';
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

      const userRole = this.authService.getUserRole();
      this.isMerchant = userRole === 'Merchant';
      this.isCustomer = userRole === 'Customer';
      this.isDeliveryAgent = userRole === 'Delivery Agent';
      this.isAdmin = userRole === 'Admin';

      if (this.isCustomer) {
        this.cartService.cartUpdate$.subscribe(count => {
          this.cartItemCount = count;
        });
        this.updateCartCount();
      } else {
        this.cartItemCount = 0;
      }
    } else {
      this.firstName = null;
      this.isMerchant = false;
      this.isCustomer = false;
      this.isDeliveryAgent = false;
      this.isAdmin = false;
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

  openRoleRequestModal(role: string) {
    if (!this.authService.isLoggedIn()) {
      this.errorMessage = 'Please log in to submit a role request.';
      this.showMessageModal = true;
      console.log('Not logged in, showing message modal');
      return;
    }

    this.selectedRole = role;
    console.log(`Attempting to open role request modal for role: ${role}`);

    this.roleService.getMyRoleRequest().subscribe({
      next: (response) => {
        console.log('getMyRoleRequest response:', response);
        if (response && typeof response.success === 'boolean') {
          if (response.success && response.data) {
            // Existing request found
            this.roleRequest = response.data;
            console.log('Existing request found:', this.roleRequest);
            this.showPendingModal = true;
            console.log('showPendingModal set to true');
          } else {
            // No existing request, show confirm modal
            console.log('No existing request, showing confirm modal');
            this.roleRequest = null;
            this.showConfirmModal = true;
            console.log('showConfirmModal set to true');
          }
        } else {
          console.error('Unexpected response format:', response);
          this.errorMessage = 'Unexpected response from server. Please try again later.';
          this.showMessageModal = true;
          console.log('Showing error message modal due to unexpected response');
        }
      },
      error: (err) => {
        console.error('Error checking role requests:', err);
        this.errorMessage = err.message || 'Unable to check existing role requests. Please try again later.';
        this.showMessageModal = true;
        console.log('Error occurred, showing message modal');
      }
    });
  }

  confirmRoleRequest() {
    if (!this.selectedRole) {
      console.log('No selected role to confirm');
      return;
    }

    console.log(`Submitting role request for: ${this.selectedRole}`);
    this.roleService.submitRoleRequest({ requestedRole: this.selectedRole }).subscribe({
      next: (response) => {
        console.log('submitRoleRequest response:', response);
        this.showConfirmModal = false;
        console.log('Closed confirm modal after submission');
        if (response && typeof response.success === 'boolean') {
          if (response.success) {
            this.roleRequestMessage = `Your request to become a ${this.selectedRole} has been submitted successfully! Stay tuned for updates—we’ll notify you once your application is reviewed. 🚀`;
            this.errorMessage = null;
          } else {
            this.errorMessage = response.message || 'Failed to submit role request.';
            this.roleRequestMessage = null;
          }
          this.showMessageModal = true;
          console.log('Showing message modal after submission');
        } else {
          console.error('Unexpected response format:', response);
          this.errorMessage = 'Unexpected response from server. Please try again later.';
          this.roleRequestMessage = null;
          this.showMessageModal = true;
          console.log('Showing error message modal due to unexpected response');
        }
      },
      error: (err) => {
        console.error('Error submitting role request:', err);
        this.showConfirmModal = false;
        this.errorMessage = err.message || 'Failed to submit role request. Please try again later.';
        this.roleRequestMessage = null;
        this.showMessageModal = true;
        console.log('Error occurred, showing message modal');
      }
    });
  }

  openCheckRequestModal() {
    if (!this.authService.isLoggedIn()) {
      this.errorMessage = 'Please log in to check your role request status.';
      this.showMessageModal = true;
      console.log('Not logged in, showing message modal');
      return;
    }

    console.log('Attempting to open check request modal');
    this.roleService.getMyRoleRequest().subscribe({
      next: (response) => {
        console.log('getMyRoleRequest response for status check:', response);
        if (response && typeof response.success === 'boolean') {
          this.roleRequest = response.success ? response.data : null;
          this.roleRequestMessage = response.message || 'No message provided';
          this.showStatusModal = true;
          console.log('showStatusModal set to true');
        } else {
          console.error('Unexpected response format:', response);
          this.errorMessage = 'Unexpected response from server. Please try again later.';
          this.showMessageModal = true;
          console.log('Showing error message modal due to unexpected response');
        }
      },
      error: (err) => {
        console.error('Error checking role requests:', err);
        this.errorMessage = err.message || 'Failed to check role request status. Please try again later.';
        this.showMessageModal = true;
        console.log('Error occurred, showing message modal');
      }
    });
  }

  closeConfirmModal() {
    this.showConfirmModal = false;
    console.log('Closed confirm modal');
  }

  closePendingModal() {
    this.showPendingModal = false;
    console.log('Closed pending modal');
  }

  closeStatusModal() {
    this.showStatusModal = false;
    console.log('Closed status modal');
  }

  closeMessageModal() {
    this.showMessageModal = false;
    this.errorMessage = null;
    this.roleRequestMessage = null;
    console.log('Closed message modal');
  }
}
```

**`header.component.html`**:
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-primary">
  <div class="container-fluid px-5">
    <!-- Brand -->
    <a class="navbar-brand fw-bold" [routerLink]="getHomeRoute()">EShoppingZone</a>
    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
      aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>

    <!-- Navbar Content -->
    <div class="collapse navbar-collapse" id="navbarNav">
      <!-- Navigation Links -->
      <ul class="navbar-nav me-auto mb-2 mb-lg-0 ms-3">
        <!-- Home Link (shown only for Guest and Customer; hidden for Merchant, Delivery Agent, and Admin) -->
        <li class="nav-item" *ngIf="!isMerchant && !isDeliveryAgent && !isAdmin">
          <a class="nav-link" [routerLink]="['/']" routerLinkActive="active">Home</a>
        </li>
        <!-- Guest and Customer Links -->
        <li class="nav-item" *ngIf="!isMerchant && !isDeliveryAgent && !isAdmin">
          <a class="nav-link" [routerLink]="['/products']" routerLinkActive="active">Products</a>
        </li>
        <li class="nav-item" *ngIf="!isMerchant && !isDeliveryAgent && !isAdmin">
          <a class="nav-link" [routerLink]="['/deals']" routerLinkActive="active">Deals</a>
        </li>
        <!-- Merchant Links -->
        <li class="nav-item" *ngIf="isMerchant">
          <a class="nav-link" [routerLink]="['/merchant-home']" routerLinkActive="active">Home</a>
        </li>
        <li class="nav-item" *ngIf="isMerchant">
          <a class="nav-link" [routerLink]="['/merchant-dashboard']" routerLinkActive="active">Dashboard</a>
        </li>
        <li class="nav-item" *ngIf="isMerchant">
          <a class="nav-link" [routerLink]="['/manage-products']" routerLinkActive="active">Manage Products</a>
        </li>
        <!-- Delivery Agent Links -->
        <li class="nav-item" *ngIf="isDeliveryAgent">
          <a class="nav-link" [routerLink]="['/delivery-agent-home']" routerLinkActive="active">Delivery Home</a>
        </li>
        <li class="nav-item" *ngIf="isDeliveryAgent">
          <a class="nav-link" [routerLink]="['/delivery-agent-dashboard']" routerLinkActive="active">Dashboard</a>
        </li>
        <!-- Admin Links -->
        <li class="nav-item" *ngIf="isAdmin">
          <a class="nav-link" [routerLink]="['/admin-home']" routerLinkActive="active">Home</a>
        </li>
        <li class="nav-item" *ngIf="isAdmin">
          <a class="nav-link" [routerLink]="['/admin-dashboard']" routerLinkActive="active">Dashboard</a>
        </li>
      </ul>

      <!-- Search Bar (Only for Guest and Customers) -->
      <div class="d-flex flex-grow-1 justify-content-center mx-3" *ngIf="!isMerchant && !isDeliveryAgent && !isAdmin">
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
            <a class="nav-link btn btn-outline-light login-signup-btn" [routerLink]="['/signup']">Signup</a>
          </li>
        </ng-container>

        <ng-template #loggedIn>
          <!-- Cart Link for Customers -->
          <li class="nav-item" *ngIf="isCustomer">
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
              <li *ngIf="isCustomer">
                <a class="dropdown-item" (click)="navigateTo('/manage-addresses')">Manage Addresses</a>
              </li>
              <li *ngIf="isCustomer">
                <a class="dropdown-item" (click)="navigateTo('/cart')">Place Order</a>
              </li>
              <li *ngIf="isCustomer">
                <a class="dropdown-item" (click)="navigateTo('/order-history')">Order History</a>
              </li>
              <li *ngIf="isCustomer">
                <a class="dropdown-item" (click)="openRoleRequestModal('Merchant')">Become a Merchant</a>
              </li>
              <li *ngIf="isCustomer">
                <a class="dropdown-item" (click)="openRoleRequestModal('DeliveryAgent')">Become a Delivery Agent</a>
              </li>
              <li *ngIf="isCustomer">
                <a class="dropdown-item" (click)="openCheckRequestModal()">Show Role Request Status</a>
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

<!-- Confirmation Modal -->
<div class="modal-backdrop" *ngIf="showConfirmModal">
  <div class="modal-content">
    <div class="modal-header">
      <h5>Confirm Role Change Request</h5>
      <button type="button" class="close" (click)="closeConfirmModal()">
        <span>×</span>
      </button>
    </div>
    <div class="modal-body">
      <p>Are you sure you want to apply to become a {{ selectedRole }}?</p>
      <p>This action will change your role upon approval.</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="closeConfirmModal()">Cancel</button>
      <button type="button" class="btn btn-primary" (click)="confirmRoleRequest()">Confirm</button>
    </div>
  </div>
</div>

<!-- Pending Request Modal -->
<div class="modal-backdrop" *ngIf="showPendingModal">
  <div class="modal-content">
    <div class="modal-header warning">
      <h5>Pending Role Request</h5>
      <button type="button" class="close" (click)="closePendingModal()">
        <span>×</span>
      </button>
    </div>
    <div class="modal-body">
      <p>You already have a pending request to become a {{ roleRequest?.requestedRole }}.</p>
      <p>Requested on: {{ roleRequest?.requestedAt | date:'medium' }}</p>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="closePendingModal()">Close</button>
    </div>
  </div>
</div>

<!-- Role Request Status Modal -->
<div class="modal-backdrop" *ngIf="showStatusModal">
  <div class="modal-content">
    <div class="modal-header info">
      <h5>Role Request Status</h5>
      <button type="button" class="close" (click)="closeStatusModal()">
        <span>×</span>
      </button>
    </div>
    <div class="modal-body">
      <div *ngIf="roleRequest; else noRequest">
        <p>Your request to become a <strong>{{ roleRequest.requestedRole }}</strong> is <strong>{{ roleRequest.status }}</strong>.</p>
        <p><small>Requested on: {{ roleRequest.requestedAt | date:'medium' }}</small></p>
        <p *ngIf="roleRequest.reviewedAt"><small>Reviewed on: {{ roleRequest.reviewedAt | date:'medium' }}</small></p>
      </div>
      <ng-template #noRequest>
        <p>You do not have any pending role requests.</p>
      </ng-template>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="closeStatusModal()">Close</button>
    </div>
  </div>
</div>

<!-- Message Modal -->
<div class="modal-backdrop" *ngIf="showMessageModal">
  <div class="modal-content">
    <div class="modal-header" [ngClass]="{'success': roleRequestMessage, 'error': errorMessage}">
      <h5>{{ roleRequestMessage ? 'Request Submitted' : 'Error' }}</h5>
      <button type="button" class="close" (click)="closeMessageModal()">
        <span>×</span>
      </button>
    </div>
    <div class="modal-body">
      <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
        {{ errorMessage }}
      </div>
      <div *ngIf="roleRequestMessage" class="alert alert-success" role="alert">
        {{ roleRequestMessage }}
      </div>
    </div>
    <div class="modal-footer">
      <button type="button" class="btn btn-secondary" (click)="closeMessageModal()">Close</button>
    </div>
  </div>
</div>
```

---

#### Component 2: `admin-dashboard`

**`admin-dashboard.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RoleService } from '../../services/role.service';
import { RouterLink } from '@angular/router';

interface RoleRequest {
  id: number;
  userId: number;
  requestedRole: string;
  status: string;
  requestedAt: string;
  reviewedAt: string | null;
  user: {
    id: number;
    userName: string;
    email: string;
    phoneNumber: string;
  } | null;
}

@Component({
  selector: 'app-admin-dashboard',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './admin-dashboard.component.html',
  styleUrls: ['./admin-dashboard.component.css']
})
export class AdminDashboardComponent implements OnInit {
  pendingRequests: RoleRequest[] = [];
  totalPendingRequests: number = 0;
  merchantPendingRequests: number = 0;
  deliveryAgentPendingRequests: number = 0;

  constructor(private roleService: RoleService) {}

  ngOnInit() {
    this.loadPendingRequests();
  }

  loadPendingRequests() {
    this.roleService.getPendingRequests().subscribe({
      next: (response) => {
        console.log('getPendingRequests response:', response);
        if (response && typeof response.success === 'boolean') {
          if (response.success && response.data) {
            this.pendingRequests = response.data;
            this.totalPendingRequests = this.pendingRequests.length;
            this.merchantPendingRequests = this.pendingRequests.filter(req => req.requestedRole === 'Merchant').length;
            this.deliveryAgentPendingRequests = this.pendingRequests.filter(req => req.requestedRole === 'DeliveryAgent').length;
            console.log('Pending requests loaded:', this.pendingRequests);
          } else {
            console.log('No pending requests found');
            this.pendingRequests = [];
            this.totalPendingRequests = 0;
            this.merchantPendingRequests = 0;
            this.deliveryAgentPendingRequests = 0;
          }
        } else {
          console.error('Unexpected response format:', response);
          this.pendingRequests = [];
          this.totalPendingRequests = 0;
          this.merchantPendingRequests = 0;
          this.deliveryAgentPendingRequests = 0;
        }
      },
      error: (err) => {
        console.error('Error fetching pending requests:', err);
        this.pendingRequests = [];
        this.totalPendingRequests = 0;
        this.merchantPendingRequests = 0;
        this.deliveryAgentPendingRequests = 0;
      }
    });
  }
}
```

**`admin-dashboard.component.html`**:
```html
<div class="container py-5">
  <h2 class="mb-4">Admin Dashboard</h2>

  <!-- Overview Cards -->
  <div class="row mb-5">
    <div class="col-md-4 mb-3">
      <div class="card text-center shadow-sm">
        <div class="card-header bg-primary text-white">
          <h5 class="card-title mb-0">Total Pending Requests</h5>
        </div>
        <div class="card-body">
          <p class="display-4">{{ totalPendingRequests }}</p>
          <a [routerLink]="['/pending-merchant-requests']" class="btn btn-outline-primary">View All Requests</a>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-3">
      <div class="card text-center shadow-sm">
        <div class="card-header bg-warning text-dark">
          <h5 class="card-title mb-0">Pending Merchant Requests</h5>
        </div>
        <div class="card-body">
          <p class="display-4">{{ merchantPendingRequests }}</p>
          <a [routerLink]="['/pending-merchant-requests']" class="btn btn-outline-warning">View Merchant Requests</a>
        </div>
      </div>
    </div>
    <div class="col-md-4 mb-3">
      <div class="card text-center shadow-sm">
        <div class="card-header bg-info text-white">
          <h5 class="card-title mb-0">Pending Delivery Agent Requests</h5>
        </div>
        <div class="card-body">
          <p class="display-4">{{ deliveryAgentPendingRequests }}</p>
          <a [routerLink]="['/pending-delivery-agent-requests']" class="btn btn-outline-info">View Delivery Agent Requests</a>
        </div>
      </div>
    </div>
  </div>

  <!-- Chart Section -->
  <div class="row mb-5">
    <div class="col-12">
      <div class="card shadow-sm">
        <div class="card-header bg-light">
          <h5 class="card-title mb-0">Pending Requests Distribution</h5>
        </div>
        <div class="card-body">
          <div class="chart-container">
            <div *ngIf="totalPendingRequests > 0">
              <chartjs [config]="{
                'type': 'pie',
                'data': {
                  'labels': ['Merchant Requests', 'Delivery Agent Requests'],
                  'datasets': [{
                    'label': 'Pending Requests',
                    'data': [merchantPendingRequests, deliveryAgentPendingRequests],
                    'backgroundColor': ['#ffc107', '#17a2b8'],
                    'borderColor': ['#fff', '#fff'],
                    'borderWidth': 1
                  }]
                },
                'options': {
                  'responsive': true,
                  'maintainAspectRatio': false,
                  'plugins': {
                    'legend': {
                      'position': 'top',
                      'labels': {
                        'color': '#333'
                      }
                    },
                    'title': {
                      'display': true,
                      'text': 'Pending Requests Distribution',
                      'color': '#333',
                      'font': {
                        'size': 16
                      }
                    }
                  }
                }
              }"></chartjs>
            </div>
            <p *ngIf="totalPendingRequests === 0" class="text-center text-muted">No pending requests to display.</p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Quick Actions -->
  <div class="row">
    <div class="col-md-6 mb-3">
      <div class="card quick-action-card shadow-sm">
        <div class="card-body text-center">
          <h5 class="card-title">Review Merchant Requests</h5>
          <p class="card-text">Approve or reject pending Merchant applications.</p>
          <a [routerLink]="['/pending-merchant-requests']" class="btn btn-primary">Go to Requests</a>
        </div>
      </div>
    </div>
    <div class="col-md-6 mb-3">
      <div class="card quick-action-card shadow-sm">
        <div class="card-body text-center">
          <h5 class="card-title">Review Delivery Agent Requests</h5>
          <p class="card-text">Approve or reject pending Delivery Agent applications.</p>
          <a [routerLink]="['/pending-delivery-agent-requests']" class="btn btn-primary">Go to Requests</a>
        </div>
      </div>
    </div>
  </div>
</div>
```

**`admin-dashboard.component.css`**:
```css
.card {
  border-radius: 8px;
}
.card-header {
  border-bottom: none;
}
.display-4 {
  font-size: 2.5rem;
  color: #333;
}
.quick-action-card {
  transition: transform 0.2s;
}
.quick-action-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}
.chart-container {
  position: relative;
  height: 300px;
  width: 100%;
}
```

---

#### Component 3: `pending-merchant-requests`

**`pending-merchant-requests.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RoleService } from '../../services/role.service';

interface RoleRequest {
  id: number;
  userId: number;
  requestedRole: string;
  status: string;
  requestedAt: string;
  reviewedAt: string | null;
  user: {
    id: number;
    userName: string;
    email: string;
    phoneNumber: string;
  } | null;
}

@Component({
  selector: 'app-pending-merchant-requests',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './pending-merchant-requests.component.html',
  styleUrls: ['./pending-merchant-requests.component.css']
})
export class PendingMerchantRequestsComponent implements OnInit {
  requests: RoleRequest[] = [];

  constructor(private roleService: RoleService) {}

  ngOnInit() {
    this.loadPendingRequests();
  }

  loadPendingRequests() {
    this.roleService.getPendingRequests().subscribe({
      next: (response) => {
        console.log('getPendingRequests response for Merchant:', response);
        if (response && typeof response.success === 'boolean') {
          if (response.success && response.data) {
            this.requests = response.data.filter(req => req.requestedRole === 'Merchant');
            console.log('Filtered Merchant requests:', this.requests);
          } else {
            console.log('No Merchant requests found');
            this.requests = [];
          }
        } else {
          console.error('Unexpected response format:', response);
          this.requests = [];
        }
      },
      error: (err) => {
        console.error('Error fetching pending requests:', err);
        this.requests = [];
      }
    });
  }

  reviewRequest(requestId: number, status: string) {
    console.log(`Reviewing request ID ${requestId} with status ${status}`);
    this.roleService.reviewRequest({ requestId, status }).subscribe({
      next: (response) => {
        console.log('reviewRequest response:', response);
        if (response && typeof response.success === 'boolean' && response.success) {
          this.requests = this.requests.filter(req => req.id !== requestId);
          console.log('Request reviewed and removed from list');
        } else {
          console.error('Failed to review request:', response?.message || 'Unknown error');
        }
      },
      error: (err) => {
        console.error('Error reviewing request:', err);
      }
    });
  }
}
```

**`pending-merchant-requests.component.html`**:
```html
<div class="container py-5">
  <h2>Pending Merchant Requests</h2>
  <div class="mt-4">
    <div *ngIf="requests.length > 0; else noRequests">
      <table class="table table-bordered">
        <thead>
          <tr>
            <th>User</th>
            <th>Email</th>
            <th>Requested At</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let request of requests">
            <td>{{ request.user?.userName || 'N/A' }}</td>
            <td>{{ request.user?.email || 'N/A' }}</td>
            <td>{{ request.requestedAt | date:'medium' }}</td>
            <td>
              <button class="btn btn-success me-2" (click)="reviewRequest(request.id, 'Approved')">Approve</button>
              <button class="btn btn-danger" (click)="reviewRequest(request.id, 'Rejected')">Reject</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <ng-template #noRequests>
      <p>No pending Merchant requests found.</p>
    </ng-template>
  </div>
</div>
```

---

#### Component 4: `pending-delivery-agent-requests`

**`pending-delivery-agent-requests.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RoleService } from '../../services/role.service';

interface RoleRequest {
  id: number;
  userId: number;
  requestedRole: string;
  status: string;
  requestedAt: string;
  reviewedAt: string | null;
  user: {
    id: number;
    userName: string;
    email: string;
    phoneNumber: string;
  } | null;
}

@Component({
  selector: 'app-pending-delivery-agent-requests',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './pending-delivery-agent-requests.component.html',
  styleUrls: ['./pending-delivery-agent-requests.component.css']
})
export class PendingDeliveryAgentRequestsComponent implements OnInit {
  requests: RoleRequest[] = [];

  constructor(private roleService: RoleService) {}

  ngOnInit() {
    this.loadPendingRequests();
  }

  loadPendingRequests() {
    this.roleService.getPendingRequests().subscribe({
      next: (response) => {
        console.log('getPendingRequests response for Delivery Agent:', response);
        if (response && typeof response.success === 'boolean') {
          if (response.success && response.data) {
            this.requests = response.data.filter(req => req.requestedRole === 'DeliveryAgent');
            console.log('Filtered Delivery Agent requests:', this.requests);
          } else {
            console.log('No Delivery Agent requests found');
            this.requests = [];
          }
        } else {
          console.error('Unexpected response format:', response);
          this.requests = [];
        }
      },
      error: (err) => {
        console.error('Error fetching pending requests:', err);
        this.requests = [];
      }
    });
  }

  reviewRequest(requestId: number, status: string) {
    console.log(`Reviewing request ID ${requestId} with status ${status}`);
    this.roleService.reviewRequest({ requestId, status }).subscribe({
      next: (response) => {
        console.log('reviewRequest response:', response);
        if (response && typeof response.success === 'boolean' && response.success) {
          this.requests = this.requests.filter(req => req.id !== requestId);
          console.log('Request reviewed and removed from list');
        } else {
          console.error('Failed to review request:', response?.message || 'Unknown error');
        }
      },
      error: (err) => {
        console.error('Error reviewing request:', err);
      }
    });
  }
}
```

**`pending-delivery-agent-requests.component.html`**:
```html
<div class="container py-5">
  <h2>Pending Delivery Agent Requests</h2>
  <div class="mt-4">
    <div *ngIf="requests.length > 0; else noRequests">
      <table class="table table-bordered">
        <thead>
          <tr>
            <th>User</th>
            <th>Email</th>
            <th>Requested At</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr *ngFor="let request of requests">
            <td>{{ request.user?.userName || 'N/A' }}</td>
            <td>{{ request.user?.email || 'N/A' }}</td>
            <td>{{ request.requestedAt | date:'medium' }}</td>
            <td>
              <button class="btn btn-success me-2" (click)="reviewRequest(request.id, 'Approved')">Approve</button>
              <button class="btn btn-danger" (click)="reviewRequest(request.id, 'Rejected')">Reject</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <ng-template #noRequests>
      <p>No pending Delivery Agent requests found.</p>
    </ng-template>
  </div>
</div>
```

---

### Step 2: Testing Scenarios

Let’s test all scenarios to ensure everything works as expected with the separated files:

- **Customer**:
  - Log in as a Customer.
  - Click "Become a Merchant":
    - If no existing request: See confirm modal, click "Confirm", request should be submitted.
    - If existing request: See pending modal.
  - Click "Become a Delivery Agent": Same behavior.
  - Click "Show Role Request Status": See status modal with request details.
- **Admin**:
  - Log in as an Admin.
  - Go to `/admin-dashboard`:
    - See updated counts and chart.
  - Go to `/pending-merchant-requests` and `/pending-delivery-agent-requests`:
    - See filtered requests.
    - Approve or reject requests, and they should disappear from the list.

---

### Step 3: Summary

**It’s 12:55 AM IST on Tuesday, May 20, 2025.**

- **Separated `.ts` and `.html` Files**:
  - `header.component.ts` and `header.component.html`
  - `admin-dashboard.component.ts` and `admin-dashboard.component.html`
  - `pending-merchant-requests.component.ts` and `pending-merchant-requests.component.html`
  - `pending-delivery-agent-requests.component.ts` and `pending-delivery-agent-requests.component.html`
- **Previous Fixes Retained**:
  - Customer role request flow: Dialog boxes for "Become a Merchant", "Become a Delivery Agent", and "Show Role Request Status" are fixed.
  - Admin features: Dashboard displays counts and chart; pending requests are filtered, and Admin can approve/reject them.
- **Next Steps**:
  - Place these files in your project’s appropriate directories (e.g., `src/app/components/header`, `src/app/components/admin-dashboard`, etc.).
  - Test thoroughly to ensure everything works as expected.

I’ve separated the `.ts` and `.html` files for each component, bro. Please test them in your project, and let me know if you need any more adjustments or if there’s anything else to fix. Let’s get this done! 🚀
