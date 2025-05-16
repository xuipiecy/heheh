Let’s tackle the fixes and implement the profile page for all user types, focusing on functionality first since UI improvements will be addressed later. I’ll use the backend APIs you provided (`AuthController`, `OrderController`, etc.) to ensure everything aligns with your existing setup. I’ll also address the specific issues you mentioned (profile dropdown behavior, cart item count, star ratings, and adding reviews) while implementing the profile page.

---

### Fixes

#### 1. Profile Dropdown Should Close After Other Operations
The profile dropdown in `HeaderComponent` should close when the user performs other actions (e.g., login, clicking links, or navigating). We’ll modify the `HeaderComponent` to close the dropdown on navigation or other events.

**Changes to `header.component.ts`**:
```typescript
// Add a method to close the dropdown
closeDropdown() {
  this.isDropdownOpen = false;
}

// Update toggleDropdown to close on other actions
toggleDropdown(event?: Event) {
  if (event) {
    event.preventDefault();
  }
  this.isDropdownOpen = !this.isDropdownOpen;
}

// Update navigateTo to close dropdown
navigateTo(route: string) {
  this.router.navigate([route]);
  this.closeDropdown(); // Already exists, ensure it’s called
}

// Add a listener for route changes to close dropdown
ngOnInit() {
  this.router.events.subscribe(event => {
    if (event instanceof NavigationEnd) {
      this.closeDropdown(); // Close dropdown on any navigation
    }
  });
}
```

**Explanation**:
- Added `closeDropdown` to explicitly close the dropdown.
- Subscribed to router events (`NavigationEnd`) to close the dropdown whenever navigation occurs (e.g., clicking on a product, logging in, or navigating to another page).
- Ensured `navigateTo` closes the dropdown (already present but reinforced here).

#### 2. Profile Dropdown Should Show First Name Instead of "User 1, User 2"
We need to fetch the user’s full name from the `viewProfile` API and display the first name in the dropdown instead of the user ID.

**Changes to `auth.service.ts`**:
Ensure `viewProfile` is correctly implemented (already exists but confirming):
```typescript
viewProfile(): Observable<ResponseDTO<ProfileResponse>> {
  return this.http.get<ResponseDTO<ProfileResponse>>(`${this.baseUrl}/api/Auth/ViewProfile`);
}
```

**Changes to `header.component.ts`**:
```typescript
// Add property for first name
firstName: string | null = null;

// Update ngOnInit to fetch user profile and set first name
ngOnInit() {
  this.router.events.subscribe(event => {
    if (event instanceof NavigationEnd) {
      this.closeDropdown();
    }
  });

  // Fetch user profile to get first name
  if (this.authService.isLoggedIn()) {
    this.authService.viewProfile().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.firstName = response.data.fullName.split(' ')[0]; // Extract first name
        }
      },
      error: (err) => {
        console.error('Error fetching profile:', err);
        this.firstName = null;
      }
    });
  }
}
```

**Changes to `header.component.html`**:
```html
<div class="nav-item dropdown" (click)="toggleDropdown(); $event.preventDefault()">
  <a class="nav-link dropdown-toggle text-white" id="profileDropdown" role="button" aria-expanded="false">
    {{ firstName ? firstName : 'Profile' }}
  </a>
  <!-- Rest of the dropdown remains unchanged -->
</div>
```

**Explanation**:
- Added `firstName` property to store the user’s first name.
- In `ngOnInit`, fetched the user profile using `viewProfile` and extracted the first name from `fullName`.
- Updated the dropdown label to display `firstName` instead of the user ID.

#### 3. Cart Item Count on Navbar Should Update Immediately
The cart item count on the navbar should reflect changes immediately after adding items, without requiring a refresh. We’ll modify the `CartService` to emit updates and ensure the `HeaderComponent` listens for these changes.

**Changes to `cart.service.ts`**:
```typescript
import { BehaviorSubject } from 'rxjs';

// Add a BehaviorSubject to emit cart updates
private cartUpdateSubject = new BehaviorSubject<number>(0);
cartUpdate$ = this.cartUpdateSubject.asObservable();

// Update methods to emit cart updates
addToCart(productId: number, quantity: number): Observable<ResponseDTO<CartResponse>> {
  return this.http.post<ResponseDTO<CartResponse>>(`${this.baseUrl}/api/Cart/AddToCart`, { productId, quantity }).pipe(
    tap(response => {
      if (response.success && response.data) {
        const itemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0);
        this.cartUpdateSubject.next(itemCount);
      }
    })
  );
}

updateCartItem(itemId: number, updateRequest: { quantity: number }): Observable<ResponseDTO<CartResponse>> {
  return this.http.put<ResponseDTO<CartResponse>>(`${this.baseUrl}/api/Cart/UpdateCartItem/${itemId}`, updateRequest).pipe(
    tap(response => {
      if (response.success && response.data) {
        const itemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0);
        this.cartUpdateSubject.next(itemCount);
      }
    })
  );
}

removeFromCart(itemId: number): Observable<ResponseDTO<CartResponse>> {
  return this.http.delete<ResponseDTO<CartResponse>>(`${this.baseUrl}/api/Cart/RemoveFromCart/${itemId}`).pipe(
    tap(response => {
      if (response.success && response.data) {
        const itemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0);
        this.cartUpdateSubject.next(itemCount);
      }
    })
  );
}

clearCart(): Observable<ResponseDTO<CartResponse>> {
  return this.http.delete<ResponseDTO<CartResponse>>(`${this.baseUrl}/api/Cart/ClearCart`).pipe(
    tap(response => {
      if (response.success && response.data) {
        this.cartUpdateSubject.next(0); // Reset to 0
      }
    })
  );
}
```

**Changes to `header.component.ts`**:
```typescript
// Add property for cart item count
cartItemCount: number = 0;

// Update ngOnInit to subscribe to cart updates
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

    // Subscribe to cart updates
    this.cartService.cartUpdate$.subscribe(count => {
      this.cartItemCount = count;
    });

    // Initial cart load to set count
    this.cartService.getCart().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.cartItemCount = response.data.items.reduce((sum, item) => sum + item.quantity, 0);
        }
      }
    });
  }
}
```

**Changes to `header.component.html`**:
```html
<!-- Update cart link to show item count -->
<li class="nav-item">
  <a class="nav-link text-white" [routerLink]="['/cart']">
    Cart 🛒
    <span *ngIf="cartItemCount > 0" class="badge bg-danger ms-1">{{ cartItemCount }}</span>
  </a>
</li>
```

**Explanation**:
- Added a `BehaviorSubject` in `CartService` to emit updates whenever the cart changes (add, update, remove, clear).
- In `HeaderComponent`, subscribed to `cartUpdate$` to update `cartItemCount` in real-time.
- On initial load, fetched the cart to set the initial count.
- Updated the navbar to display the cart item count as a badge.

#### 4. Fix Star Rating Display
The star rating should initially show 5 empty stars, then fill them in gold based on the average rating (e.g., for 3.72, fill 3 stars fully and 72% of the 4th star). This will be applied in components like `ProductDetailComponent` where ratings are displayed.

**Changes to `product-detail.component.ts`**:
```typescript
// Add properties for star rating
filledStars: number = 0;
partialStarPercentage: number = 0;

// Add method to calculate star rating
calculateStarRating(avgRating: number) {
  this.filledStars = Math.floor(avgRating); // Fully filled stars
  const fractionalPart = avgRating - this.filledStars; // Fractional part for partial star
  this.partialStarPercentage = Math.round(fractionalPart * 100); // Convert to percentage
}
```

**Changes to `product-detail.component.html`**:
```html
<!-- Update star rating display -->
<div class="star-rating">
  <span *ngFor="let star of [1, 2, 3, 4, 5]; let i = index" class="star">
    <span *ngIf="i < filledStars" class="filled">★</span>
    <span *ngIf="i === filledStars && partialStarPercentage > 0" class="partial" [style.width.%]="partialStarPercentage">★</span>
    <span *ngIf="i >= filledStars && (i !== filledStars || partialStarPercentage === 0)">☆</span>
  </span>
</div>
```

**Changes to `product-detail.component.css`**:
```css
.star-rating {
  display: inline-flex;
  font-size: 1.5rem;
  color: #ccc; /* Empty star color */
}
.star {
  position: relative;
  display: inline-block;
}
.filled, .partial {
  color: gold; /* Filled star color */
  position: absolute;
  top: 0;
  left: 0;
  overflow: hidden;
}
.partial {
  white-space: nowrap;
}
```

**Explanation**:
- Added logic to calculate the number of fully filled stars (`filledStars`) and the percentage for the partial star (`partialStarPercentage`).
- In the template, looped through 5 stars, showing a filled star (★) for fully filled stars, a partial star with a percentage width for the fractional part, and an empty star (☆) for the rest.
- Styled the stars to show gold for filled portions and gray for empty portions.

#### 5. Add Option to Add Review with Star Rating
We’ll add a review section in `ProductDetailComponent` where customers can type a review, rate the product (clicking stars to set a rating), and submit it. The star rating input will allow users to click on a star to set the rating (e.g., clicking the 3rd star sets a 3/5 rating).

**Backend Setup**:
First, we need a backend API to handle reviews. Since this wasn’t provided, I’ll assume a new `ReviewController` with a `POST /api/Review/AddReview` endpoint.

**New File: `ReviewController.cs`** (Backend):
```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;
using System.Threading.Tasks;
using EShoppingZone.Interfaces;
using EShoppingZone.DTOs;

namespace EShoppingZone.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    [Authorize(Roles = "Customer")]
    public class ReviewController : ControllerBase
    {
        private readonly IReviewService _service;

        public ReviewController(IReviewService service)
        {
            _service = service;
        }

        [HttpPost("AddReview")]
        public async Task<IActionResult> AddReview([FromBody] ReviewRequest reviewRequest)
        {
            if (!ModelState.IsValid)
            {
                var errors = ModelState.Values
                    .SelectMany(v => v.Errors)
                    .Select(e => e.ErrorMessage)
                    .ToList();
                return BadRequest("Validation errors: " + string.Join("; ", errors));
            }
            var profileId = int.Parse(User.FindFirst(ClaimTypes.NameIdentifier).Value);
            var response = await _service.AddReviewAsync(profileId, reviewRequest);
            if (response.Success)
            {
                return Ok(response);
            }
            return BadRequest(response);
        }
    }
}
```

**New File: `IReviewService.cs`** (Backend):
```csharp
using System.Threading.Tasks;
using EShoppingZone.DTOs;

namespace EShoppingZone.Interfaces
{
    public interface IReviewService
    {
        Task<ResponseDTO<ReviewResponse>> AddReviewAsync(int profileId, ReviewRequest reviewRequest);
    }
}
```

**New File: `ReviewService.cs`** (Backend):
```csharp
using System.Threading.Tasks;
using AutoMapper;
using EShoppingZone.Data;
using EShoppingZone.DTOs;
using EShoppingZone.Models;

namespace EShoppingZone.Services
{
    public class ReviewService : IReviewService
    {
        private readonly EShoppingZoneDBContext _context;
        private readonly IMapper _mapper;

        public ReviewService(EShoppingZoneDBContext context, IMapper mapper)
        {
            _context = context;
            _mapper = mapper;
        }

        public async Task<ResponseDTO<ReviewResponse>> AddReviewAsync(int profileId, ReviewRequest reviewRequest)
        {
            var product = await _context.Products.FindAsync(reviewRequest.ProductId);
            if (product == null)
            {
                return new ResponseDTO<ReviewResponse>
                {
                    Success = false,
                    Message = "Product not found"
                };
            }

            var review = new Review
            {
                ProductId = reviewRequest.ProductId,
                CustomerId = profileId,
                Rating = reviewRequest.Rating,
                Comment = reviewRequest.Comment,
                ReviewDate = DateTime.UtcNow
            };

            _context.Reviews.Add(review);
            await _context.SaveChangesAsync();

            var response = _mapper.Map<ReviewResponse>(review);
            return new ResponseDTO<ReviewResponse>
            {
                Success = true,
                Message = "Review added successfully",
                Data = response
            };
        }
    }
}
```

**New File: `ReviewRequest.cs`** (Backend):
```csharp
using System.ComponentModel.DataAnnotations;

namespace EShoppingZone.DTOs
{
    public class ReviewRequest
    {
        [Required]
        public int ProductId { get; set; }

        [Range(1, 5)]
        public int Rating { get; set; }

        [StringLength(500)]
        public string Comment { get; set; } = string.Empty;
    }
}
```

**New File: `ReviewResponse.cs`** (Backend):
```csharp
namespace EShoppingZone.DTOs
{
    public class ReviewResponse
    {
        public int Id { get; set; }
        public int ProductId { get; set; }
        public int CustomerId { get; set; }
        public int Rating { get; set; }
        public string Comment { get; set; } = string.Empty;
        public DateTime ReviewDate { get; set; }
    }
}
```

**New File: `review.service.ts`** (Frontend):
```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface ReviewRequest {
  productId: number;
  rating: number;
  comment: string;
}

export interface ReviewResponse {
  id: number;
  productId: number;
  customerId: number;
  rating: number;
  comment: string;
  reviewDate: string;
}

export interface ResponseDTO<T> {
  success: boolean;
  message: string;
  data: T;
}

@Injectable({
  providedIn: 'root'
})
export class ReviewService {
  private baseUrl = 'https://api.eshoppingzone.com';

  constructor(private http: HttpClient) {}

  addReview(reviewRequest: ReviewRequest): Observable<ResponseDTO<ReviewResponse>> {
    return this.http.post<ResponseDTO<ReviewResponse>>(`${this.baseUrl}/api/Review/AddReview`, reviewRequest);
  }
}
```

**Changes to `product-detail.component.ts`**:
```typescript
import { ReviewService, ReviewRequest } from '../../services/review.service';

// Add properties for review submission
newReview: string = '';
selectedRating: number = 0;
reviewSubmitted: boolean = false;

// Add methods for review submission and star rating
setRating(rating: number) {
  this.selectedRating = rating;
}

submitReview() {
  if (!this.authService.isLoggedIn() || this.authService.getUserRole() !== 'Customer') {
    this.errorMessage = 'Please log in as a customer to submit a review';
    return;
  }

  const reviewRequest: ReviewRequest = {
    productId: this.product.id,
    rating: this.selectedRating,
    comment: this.newReview
  };

  this.reviewService.addReview(reviewRequest).subscribe({
    next: (response) => {
      if (response.success) {
        this.reviewSubmitted = true;
        this.newReview = '';
        this.selectedRating = 0;
        setTimeout(() => {
          this.reviewSubmitted = false;
        }, 3000);
      } else {
        this.errorMessage = response.message || 'Failed to submit review';
      }
    },
    error: (err) => {
      this.errorMessage = 'Error submitting review';
      console.error(err);
    }
  });
}
```

**Changes to `product-detail.component.html`**:
```html
<!-- Add review submission section -->
<div class="mt-4">
  <h5>Submit a Review ✍️</h5>
  <div *ngIf="reviewSubmitted" class="alert alert-success">
    Review submitted successfully! ✅
  </div>
  <div *ngIf="errorMessage" class="alert alert-danger">
    {{ errorMessage }}
  </div>
  <div class="mb-3">
    <label for="rating" class="form-label">Rating:</label>
    <div class="star-rating-input">
      <span *ngFor="let star of [1, 2, 3, 4, 5]; let i = index" class="star" (click)="setRating(i + 1)">
        <span *ngIf="i < selectedRating" class="filled">★</span>
        <span *ngIf="i >= selectedRating">☆</span>
      </span>
    </div>
  </div>
  <div class="mb-3">
    <label for="reviewText" class="form-label">Your Review:</label>
    <textarea class="form-control" id="reviewText" [(ngModel)]="newReview" rows="3" maxlength="500"></textarea>
  </div>
  <button class="btn btn-primary" (click)="submitReview()" [disabled]="selectedRating === 0 || !newReview">Submit Review</button>
</div>
```

**Changes to `product-detail.component.css`**:
```css
.star-rating-input {
  display: inline-flex;
  font-size: 1.5rem;
  color: #ccc; /* Empty star color */
  cursor: pointer;
}
.star-rating-input .star {
  position: relative;
  display: inline-block;
}
.star-rating-input .filled {
  color: gold; /* Filled star color */
  position: absolute;
  top: 0;
  left: 0;
}
```

**Explanation**:
- Added a backend `ReviewController` and `ReviewService` to handle review submissions.
- Created a frontend `ReviewService` to interact with the new API.
- In `ProductDetailComponent`, added logic to allow customers to select a rating by clicking stars (1 to 5) and type a review.
- The star rating input updates `selectedRating` when a star is clicked (e.g., clicking the 3rd star sets `selectedRating` to 3).
- Added a form to submit the review, with a success message shown for 3 seconds after submission.

---

### Implement Profile Page
The profile page will display user details (using `ViewProfile` API), order history (for customers), and a list of reviews (for customers). It will be accessible to all user types, but certain sections will be customer-specific.

**New File: `profile.component.ts`**:
```typescript
import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AuthService } from '../../services/auth.service';
import { OrderService, OrderResponse } from '../../services/order.service';

@Component({
  selector: 'app-profile',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './profile.component.html',
  styleUrls: ['./profile.component.css']
})
export class ProfileComponent implements OnInit {
  profile: any = null;
  orders: OrderResponse[] = [];
  errorMessage: string | null = null;
  isCustomer: boolean = false;

  constructor(
    private authService: AuthService,
    private orderService: OrderService
  ) {}

  ngOnInit() {
    this.loadProfile();
    this.isCustomer = this.authService.getUserRole() === 'Customer';
    if (this.isCustomer) {
      this.loadOrders();
    }
  }

  loadProfile() {
    this.authService.viewProfile().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.profile = response.data;
        } else {
          this.errorMessage = response.message || 'Failed to load profile';
        }
      },
      error: (err) => {
        this.errorMessage = 'Error loading profile';
        console.error(err);
      }
    });
  }

  loadOrders() {
    this.orderService.getAllOrders().subscribe({
      next: (response) => {
        if (response.success && response.data) {
          this.orders = response.data;
        } else {
          this.errorMessage = response.message || 'No orders found';
        }
      },
      error: (err) => {
        this.errorMessage = 'Error loading orders';
        console.error(err);
      }
    });
  }

  cancelOrder(orderId: number) {
    this.orderService.cancelOrder(orderId).subscribe({
      next: (response) => {
        if (response.success && response.data) {
          const order = this.orders.find(o => o.id === orderId);
          if (order) {
            order.status = response.data.status;
          }
        } else {
          this.errorMessage = response.message || 'Failed to cancel order';
        }
      },
      error: (err) => {
        this.errorMessage = 'Error cancelling order';
        console.error(err);
      }
    });
  }
}
```

**New File: `profile.component.html`**:
```html
<div class="container py-5 px-5">
  <h2 class="fw-bold mb-4">My Profile 👤</h2>
  <div *ngIf="errorMessage" class="alert alert-danger" role="alert">
    {{ errorMessage }}
  </div>

  <!-- Profile Details -->
  <div *ngIf="profile" class="card shadow mb-4">
    <div class="card-body">
      <h4 class="card-title">{{ profile.fullName }}</h4>
      <p><strong>Email:</strong> {{ profile.email }}</p>
      <p><strong>Mobile Number:</strong> {{ profile.mobileNumber }}</p>
      <p><strong>About:</strong> {{ profile.about || 'Not provided' }}</p>
      <p><strong>Gender:</strong> {{ profile.gender || 'Not specified' }}</p>
      <p><strong>Date of Birth:</strong> {{ profile.dateOfBirth | date:'mediumDate' }}</p>
      <p><strong>Email Verified:</strong> {{ profile.emailConfirmed ? 'Yes ✅' : 'No ❌' }}</p>
    </div>
  </div>

  <!-- Order History (Customer Only) -->
  <div *ngIf="isCustomer">
    <h3 class="fw-bold mb-3">Order History 📦</h3>
    <div *ngIf="orders.length > 0; else noOrders">
      <div class="card mb-4 shadow-sm" *ngFor="let order of orders">
        <div class="card-body">
          <h5 class="card-title">Order #{{ order.id }} - {{ order.status }}</h5>
          <p class="card-text">Placed on: {{ order.orderDate | date:'medium' }}</p>
          <p class="card-text">Total: {{ order.totalPrice | currency }}</p>
          <h6>Items:</h6>
          <ul class="list-group mb-3">
            <li class="list-group-item" *ngFor="let item of order.items">
              {{ item.productName }} - {{ item.quantity }} x {{ item.price / item.quantity | currency }} = {{ item.price | currency }}
            </li>
          </ul>
          <button class="btn btn-danger" (click)="cancelOrder(order.id)" [disabled]="order.status !== 'Placed'">Cancel Order</button>
        </div>
      </div>
    </div>
    <ng-template #noOrders>
      <p class="text-muted text-center">No orders found.</p>
    </ng-template>
  </div>
</div>
```

**New File: `profile.component.css`**:
```css
.card {
  border-radius: 10px;
}
.list-group-item {
  border-radius: 5px;
}
```

**Ensure Route Exists in `app.routes.ts`**:
The route for `/profile` already exists (from previous implementations):
```typescript
{ path: 'profile', component: ProfileComponent, canActivate: [AuthGuard] }
```

---

### Summary of Changes
- **Profile Dropdown Closing**:
  - Added `closeDropdown` and router event subscription to close the dropdown on navigation.
- **Profile Dropdown First Name**:
  - Fetched `fullName` from `viewProfile` and displayed the first name in the dropdown.
- **Cart Item Count Update**:
  - Used a `BehaviorSubject` in `CartService` to emit real-time cart updates.
  - Subscribed to updates in `HeaderComponent` to reflect the cart item count immediately.
- **Star Rating Fix**:
  - Updated `ProductDetailComponent` to display 5 empty stars initially, filling them based on the average rating (e.g., 3.72 fills 3 stars fully and 72% of the 4th star).
- **Add Review Option**:
  - Added a backend `ReviewController` and `ReviewService` to handle review submissions.
  - Implemented a review form in `ProductDetailComponent` with a star rating input (click to set rating) and a text area for comments.
- **Profile Page**:
  - Created `ProfileComponent` to display user details (for all users) and order history (for customers only).
  - Used `viewProfile` and `getAllOrders` APIs to fetch data.

---

### Test
1. **Profile Dropdown**:
   - Log in, open the dropdown—it should show the first name.
   - Navigate to another page (e.g., click a product)—dropdown should close.
2. **Cart Item Count**:
   - Add items to the cart—the navbar should immediately show the updated count.
3. **Star Rating**:
   - On the product detail page, check the star rating—if the average is 3.72, 3 stars should be fully gold, and the 4th star should be 72% gold.
4. **Add Review**:
   - On the product detail page, as a customer, type a review, click stars to set a rating (e.g., click 3rd star for 3/5), and submit—should show a success message.
5. **Profile Page**:
   - Navigate to `/profile`—should show user details.
   - If a customer, should also show order history with cancel options for "Placed" orders.

This addresses all the fixes and implements the profile page. We’ll tackle UI improvements later as planned. Let me know if you need further adjustments! 🚀
