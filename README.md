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
