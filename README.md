**It’s 11:51 PM IST on Monday, May 19, 2025.**

I’m sorry for the formatting issues in my previous response, bro. I understand that some files might have been merged or not properly separated, which can make it hard to read and implement. Let me fix that by clearly separating each file, ensuring proper formatting, and providing the updated code for all the components we worked on. I’ll also ensure that each file is complete and properly closed before starting the next one. Let’s get this sorted properly this time!

---

### Recap of the Issues and Fixes

Based on your last message, we addressed the following:

1. **Customer Role Request Flow**:
   - Fixed the dialog boxes for "Become a Merchant", "Become a Delivery Agent", and "Show Role Request Status" in `header.component.ts`.
   - Ensured the `getMyRoleRequest` API response is handled correctly to show the appropriate modals (confirm, pending, or status).

2. **Admin Features**:
   - Updated `admin-dashboard.component.ts` to display total, Merchant, and Delivery Agent pending requests with a chart.
   - Updated `pending-merchant-requests.component.ts` and `pending-delivery-agent-requests.component.ts` to filter requests and allow the Admin to approve or reject them.

3. **Customer Role Request Status**:
   - Ensured the "Show Role Request Status" option in `header.component.ts` displays the correct status modal.

The issue now is that the files in my previous response were not properly formatted or separated. Let’s provide each file clearly, with proper formatting, and ensure there are no overlapping code blocks.

---

### Step 1: Provide Updated Files with Proper Formatting

I’ll provide each file separately, with clear headings, and ensure they are properly formatted and complete. I’ll also include comments to indicate where changes were made.

#### File 1: `header.component.ts`

This file handles the Customer role request flow and status check. We updated the `openRoleRequestModal` and `openCheckRequestModal` methods to fix the dialog box issues.

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

---

#### File 2: `admin-dashboard.component.ts`

This file displays the Admin Dashboard with total, Merchant, and Delivery Agent pending request counts, along with a chart.

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
  template: `
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
  `,
  styles: [`
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
  `]
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

---

#### File 3: `pending-merchant-requests.component.ts`

This file displays pending Merchant requests and allows the Admin to approve or reject them.

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
  template: `
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
  `,
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

---

#### File 4: `pending-delivery-agent-requests.component.ts`

This file displays pending Delivery Agent requests and allows the Admin to approve or reject them.

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
  template: `
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
  `,
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

---

### Step 2: Testing Scenarios

Let’s test all scenarios to ensure everything works as expected:

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

**It’s 11:58 PM IST on Monday, May 19, 2025.**

- **Fixed Formatting Issues**:
  - Provided each file (`header.component.ts`, `admin-dashboard.component.ts`, `pending-merchant-requests.component.ts`, `pending-delivery-agent-requests.component.ts`) separately with proper formatting.
  - Ensured no overlapping code blocks and proper closure of each file.
- **Previous Fixes Retained**:
  - Customer role request flow: Dialog boxes for "Become a Merchant", "Become a Delivery Agent", and "Show Role Request Status" are fixed.
  - Admin features: Dashboard displays counts and chart; pending requests are filtered, and Admin can approve/reject them.
- **Next Steps**:
  - Test the updated code in your project.
  - If there are any more issues (e.g., UI adjustments, additional features), let me know, and I’ll fix them immediately.

I’ve ensured the files are now properly formatted and separated, bro. Please test everything thoroughly. We’ll get this finalized by tomorrow! 🚀 Let me know if you need any more help.
