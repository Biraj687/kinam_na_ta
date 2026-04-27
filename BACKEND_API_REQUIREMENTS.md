# Kinam Na Ta - Comprehensive Backend & API Specification

**Role:** Full-Stack Solution Architect  
**Project:** Kinam Na Ta (E-commerce Platform)  
**Version:** 2.0 (Detailed Per-Page Specification)

---

## I. Global Application Architecture

### 1. Authentication & Security
*   **Method:** JWT (JSON Web Tokens) with a 24-hour expiry.
*   **Headers:** `Authorization: Bearer <TOKEN>` required for all `/api/user/*` and `/api/orders/*` routes.
*   **State:** Auth status and User Profile (Retail/Business) must be globally available.

### 2. State Management Requirements
*   **Cart Persistence:** Backend-synced cart (Items should persist across devices).
*   **Global Search:** Single endpoint for real-time search suggestions.

---

## II. Detailed Page-by-Page Requirements

### 1. Home Page (`index.html`)
*   **Features:**
    *   Dynamic Category Menu: Fetch category hierarchy.
    *   Banner Slider: Fetch active promotional banners.
    *   SuperDeals: Fetch items with `active_discount > 0` and `deal_expiry > now`.
    *   Top Selections: Personalized product grid based on user history.
*   **Required Endpoints:**
    *   `GET /api/home/content`: Returns banners, active categories, and deal timers.
    *   `GET /api/products/super-deals`: Returns paginated list of discounted items.
    *   `GET /api/products/top-selections`: Returns personalized/featured products.

### 2. Product Detail (`product.html`, `product1.html`)
*   **Features:**
    *   **Variant Management:** Real-time stock check for Color/Size/Type combinations.
    *   **Wholesale Utility:** Tiered pricing display for verified Business users.
    *   **Review Dashboard:** Paginated reviews with image support and rating distribution.
*   **Wholesale Logic (B2B):**
    *   If `User.role === 'business'`, expose `wholesale_tiers` and `moq` (Minimum Order Quantity).
    *   Backend must validate that `cart_quantity >= moq` for business pricing to apply.
*   **Required Endpoints:**
    *   `GET /api/products/:id`: Full details, variants, images, and descriptions.
    *   `GET /api/products/:id/stock?variant_id=...`: Real-time stock status.
    *   `GET /api/products/:id/reviews`: List reviews (paginated).
    *   `POST /api/products/:id/reviews`: Submit review (Requires `Authorization`).

### 3. Shopping Cart (`cart.html`)
*   **Features:**
    *   Real-time subtotal/tax (13% VAT) calculation.
    *   Coupon validation.
    *   Persistence of selected shipping provider (Cargo for Bulk, Courier for Retail).
*   **Required Endpoints:**
    *   `GET /api/cart`: Current items and calculated totals.
    *   `POST /api/cart/add`: Add product/variant.
    *   `PATCH /api/cart/update/:id`: Update quantity.
    *   `DELETE /api/cart/remove/:id`: Remove item.
    *   `POST /api/cart/apply-coupon`: Validate and apply discount code.

### 4. Checkout Flow (`checkout.html`)
*   **Features:**
    *   Address Selection: Fetch saved addresses or create new.
    *   Payment Integration: eSewa, Khalti, ConnectIPS, and COD.
    *   Order Creation: Atomic transaction for order creation and stock decrement.
*   **Required Endpoints:**
    *   `GET /api/user/addresses`: List saved addresses.
    *   `POST /api/orders/initiate`: Create a pending order and return payment provider metadata.
    *   `POST /api/payments/verify`: Backend-to-backend verification of eSewa/Khalti signatures.

### 5. Account Management (`account.html`)
*   **Features:**
    *   **Order Tracking:** Status updates (Pending -> Packed -> Shipped -> Out for Delivery -> Delivered).
    *   **Address Book:** Full CRUD for shipping and billing addresses.
    *   **Security:** Password update and 2FA (Two-Factor Authentication) toggle.
*   **Required Endpoints:**
    *   `GET /api/user/orders`: List all orders with status.
    *   `GET /api/user/orders/:id`: Detailed view with tracking milestones.
    *   `POST /api/user/addresses`: Create new.
    *   `PUT /api/user/addresses/:id`: Update existing.
    *   `PATCH /api/user/security/2fa`: Toggle 2FA status.

### 6. Authentication & Business Onboarding (`signup.html`, `login.html`)
*   **Features:**
    *   Retail vs. Business registration toggle.
    *   Document upload for Business Verification (PAN/VAT).
*   **Required Endpoints:**
    *   `POST /api/auth/signup`: Handle registration data.
    *   `POST /api/auth/login`: Issue JWT.
    *   `POST /api/auth/business-verify`: Upload PAN/VAT docs for verification.

---

## III. Data Schemas (JSON)

### Order Object Example
```json
{
  "order_id": "KNT-881290",
  "status": "delivered",
  "total_amount": 32450.00,
  "currency": "NPR",
  "tracking": [
    { "status": "Order Placed", "time": "2024-04-12 10:00:00", "completed": true },
    { "status": "Shipped", "time": "2024-04-13 14:00:00", "completed": true },
    { "status": "Delivered", "time": "2024-04-14 11:00:00", "completed": true }
  ]
}
```

### Business Account Profile
```json
{
  "user_id": "u_991",
  "role": "business",
  "verification_status": "verified",
  "business_details": {
    "company_name": "KNT Retailers Ltd",
    "pan_number": "123456789",
    "vat_registered": true
  }
}
```

---

## IV. Nepal-Specific Technical Notes
1.  **VAT:** Standard 13% must be calculated on the subtotal.
2.  **Phone Numbers:** Must follow Nepal telecom format (`+977 98xxxxxxxx`).
3.  **Payment Signatures:** eSewa requires a base64 signature for transaction verification.

---

## V. Admin Dashboard API Requirements
The backend must provide endpoints for an administrative dashboard to manage site content.

### 1. Product & Inventory Management
*   `POST /api/admin/products`: Create new product.
*   `PUT /api/admin/products/:id`: Update details/stock.
*   `POST /api/admin/products/:id/images`: Upload product photography.

### 2. Order & User Management
*   `GET /api/admin/orders`: List all orders with advanced filtering (by status, date, amount).
*   `PATCH /api/admin/orders/:id/status`: Update tracking status.
*   `GET /api/admin/users/business-pending`: List business accounts awaiting PAN/VAT verification.
*   `PATCH /api/admin/users/:id/verify`: Approve/Reject business status.

---

## VI. Proposed Database Schema (Relational)

### 1. `Users` Table
*   `id`, `email`, `password_hash`, `role` (retail/business), `status` (active/pending), `2fa_enabled`, `created_at`.

### 2. `Products` Table
*   `id`, `name`, `description`, `base_price`, `stock_qty`, `category_id`, `moq`, `discount_rate`.

### 3. `Wholesale_Tiers` Table
*   `id`, `product_id`, `min_quantity`, `discounted_price`.

### 4. `Orders` Table
*   `id`, `user_id`, `total_amount`, `payment_status`, `order_status`, `shipping_address_id`.

---

## VII. Recommended Technical Stack
*   **Backend:** Node.js with Express or NestJS (for scalability).
*   **Database:** PostgreSQL (for complex relations like tiered pricing) or MongoDB.
*   **Cache:** Redis for session management and cart persistence.
*   **Media Storage:** AWS S3 or Cloudinary for product and review images.
*   **Payments:** Integration with eSewa and Khalti APIs.

**End of Specification**
