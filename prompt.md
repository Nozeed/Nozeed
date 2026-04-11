# Prompt: Game Key/ID/Account Selling Website

## Role Definition
You are an experienced web developer and designer with extensive experience in building e-commerce systems. You will build a complete game key/ID/account selling website from scratch.

## Technology Stack
- **PHP**: 8.1+
- **Database**: MariaDB 10.3+
- **Database Connection**: PDO with prepared statements (MANDATORY - no raw queries)
- **Deployment**: Must work easily on real web hosting

## Website Flow
1. Browse products (guest can view)
2. Login/Register / Member area
3. Purchase products
4. Receive products (view in purchase history)

## Design Specifications

### Language Requirement
- **Website Language**: 100% Thai language for ALL user-facing text
- All buttons, labels, menus, messages, alerts, forms, and content must be in Thai
- Database content (product names, descriptions, categories, etc.) must be in Thai
- Admin dashboard interface must be in Thai
- Error messages and success messages must be in Thai
- Code comments and variable names remain in English

### Color Scheme & Design System

#### 1. Color System
- **Primary Color**: Dark Lime Green: rgb(128, 239, 37)
  - Used for: CTA buttons (Buy Now, Add to Cart), Highlight/Active state, Progress bar, Badge
- **Secondary Colors**:
  - Deep Charcoal Black: #121212 → Main background (makes green pop)
  - Soft White: #F5F5F5 → Text on dark background / Cards
  - Muted Gray: #9E9E9E → Secondary text
- **Accent Colors**:
  - Electric Green Glow: #A8FF60 → Hover / Glow effect
  - Warning Orange: #FF8C42 → Flash sale / urgency
  - Error Red: #FF4D4F

#### 2. Mood & Style
- Style: Modern + Tech + Clean
- Feel: Fresh, Fast shopping, Slight futuristic

#### 3. Typography (Google Fonts)
- **Headings**: Kanit font
  - Weights: Bold / Semi-Bold
  - H1: 32-40px
  - H2: 24-28px
- **Body**: Sarabun font
  - Weights: Regular / Medium
  - Body text: 14-16px
- Import Google Fonts in header

#### 4. Dark Background Theme
- **Mandatory**: Website must use dark background theme
- Main background: Deep Charcoal Black (#121212)
- Cards: Dark gray/black with soft shadows
- Text: Soft White (#F5F5F5) for primary, Muted Gray (#9E9E9E) for secondary
- Spacing: Generous spacing for premium feel

#### 5. Layout Structure

**Homepage:**
- Hero Banner: Black background + green gradient, large CTA "Shop Now" button
- Category Grid: Cards with green glow on hover
- Flash Sale Section: Orange color + countdown timer

**Product Card Design:**
- Background: Dark gray/black
- Border-radius: 12px
- Shadow: Soft + green glow on hover
- Elements: Product image, product name, price (green), Add to Cart button (green), Wishlist button (outline)

#### 6. Navigation Bar
- Background: Black (#121212)
- Logo: Green color
- Menu: Home, Shop, Categories, Contact
- Interaction: Hover → green + underline animation

#### 7. Buttons
- **Primary Button**: Lime green background, black text, hover with glow effect, scale 1.05
- **Secondary Button**: Green outline, hover → fill green

#### 8. Effects & Micro-interactions
- Hover Card: Lift + shadow
- Button: Pulse animation
- Loading: Green running line
- Add to cart: Bounce animation

#### 9. Mobile Design
- Sticky bottom bar: Home / Cart / Profile
- Large "Buy Now" button full width
- Swipe gesture for product browsing

#### 10. UX Strategy
- Green = Buy / Safe
- Orange = Urgency
- Include: Number of buyers, Stock remaining

Add animations and hover effects throughout the website
Modern, clean UI with smooth transitions

### Frameworks & Libraries
- **Bootstrap**: v5.3.8 (CDN)
- **Custom CSS**: custom.css for styling overrides
- **Rich Text Editor**: QuillJS (for Admin product details)
- **Alerts**: SweetAlert2 (CDN) - MANDATORY for ALL notifications, no other alert methods allowed
- **Font**: Google Fonts (Kanit for headings, Sarabun for body)

## Routing System
- **Query String / GET Parameters** only (e.g., index.php?page=products)
- **NO Router system** - use simple query parameters
- **CLEAN URLs**: Use `.htaccess` with `mod_rewrite` to hide `index.php` from URLs
- Example: `/products` should rewrite to `index.php?page=products`
- Example: `/login` should rewrite to `index.php?page=login`
- **ALL links in the website must use clean URLs** (e.g., `/products`, `/login`, `/cart`)
- **NO `index.php?page=` in any URLs displayed to users

## Authentication System

### User Auth Features
- Login
- Register
- Change Password
- Logout
- **NO** forgot password feature
- **NO** email verification on registration

### Validation Rules
- **Username**: 6-20 characters, alphanumeric only
- **Password**: 6+ characters
- **Email**: Valid email format filter
- **Confirm Password**: Must match password

### Cloudflare Turnstile
- Integrate Cloudflare Turnstile for spam protection on login/register
- Configurable in Admin Dashboard (enable/disable, set site key, secret key)
- **Default**: Disabled

### Admin Auth
- **Separate login system** from user authentication
- Admin and user login are completely separate
- Admin uses different credentials table

## Layout Structure

### Main Website
- **Shared Header** across all main pages
- **Shared Footer** across all main pages
- **IMPORTANT**: Header files MUST use `__DIR__` for all includes (e.g., `require_once __DIR__ . '/../config/functions.php'`)
- **NO relative paths** like `../config/functions.php` without `__DIR__`
- This ensures correct file resolution regardless of include context

### Admin Dashboard
- **Separate Sidebar** layout (not shared with main website)
- Complete separation from main website UI

### Navbar Structure (Main Website)
**Left Side:**
- Logo (PNG image)
- Site name (next to logo)
- Home page link (`/`)
- Products page link (`/products`)

**Right Side:**
- Search bar (action=`/products`)
- Balance display (requires login - fetch from database, NOT session)
- Member menu (Dropdown - shows when logged in)
- Cart link (`/cart`)
- All links must use **clean URLs** (NO `index.php?page=`)

## Database & Credit System

### Credit System
- **NO separate wallet system**
- Credit stored directly in `users` table field `credit`
- Always fetch credit from database (never use session)

### Database Structure
- Separate tables for Admin and User
- Create seed file with sample data
- Separate seed file from main schema

## Payment Systems (3 Methods for TOPUP ONLY)

**IMPORTANT**: These payment methods are ONLY for topping up credit. They CANNOT be used directly during checkout. Users must:
1. Top up credit first using these methods
2. Then use their credit balance to purchase products

**Checkout ONLY accepts**: Credit balance (deducted from user account)

### 1. PromptPay Payment
**API Reference**: Study files in `tmweasyapi-example/API-PromptPay/`

**Implementation Details:**
- API URL: https://tmwallet.thaighost.net/api_pph.php
- Config variables (set in Admin):
  - tmweasy_user (from tmweasyapi.com login)
  - tmweasy_password (from tmweasyapi.com login)
  - tmweasy_api_key (from tmweasyapi.com API key)
  - con_id (from tmweasyapi.com)
  - prommpay_no (PromptPay ID - numbers only)
  - prommpay_type (01=Mobile, 02=ID Card, 03=E-Wallet)
  - prommpay_name (account name)

**Flow:**
1. User enters amount → Create payment via API (method=create_pay)
2. Get id_pay → Store in database (status=pending)
3. Call API detail_pay → Display QR code (base64), amount, countdown (10 minutes)
4. Polling every 3 seconds via AJAX to check payment status
5. On success → Update user credit, send Discord webhook notification
6. On timeout → Show cancel button → Call API cancel

**Webhook Handler:**
- Create webhook endpoint to receive payment callbacks
- Verify signature: MD5(data + ":" + api_key)
- Check timestamp (must be within 30 seconds)
- Check idempotency (prevent duplicate processing)
- Update user credit on successful payment

### 2. TrueWallet Gift Link
**API Reference**: Study files in `tmweasyapi-example/Api-TrueWallet-Gift/`

**Implementation Details:**
- API URL: https://tmwallet.thaighost.net/apiwallet.php
- Config variables (set in Admin):
  - tmweasy_user (from tmweasyapi.com login)
  - tmweasy_password (from tmweasyapi.com login)
  - truewallet_mobile (TrueWallet number - digits only)
  - truewallet_name (TrueWallet account name)

**Flow:**
1. User enters gift URL (must contain "ttp")
2. Validate URL format
3. Check if URL already used (prevent duplicate)
4. Call API with gift URL
5. On success (Status=check_success) → Update user credit, send Discord webhook
6. On fail → Show error message from API

### 3. Redeem Code
**Implementation Details:**
- Single-use codes created by Admin
- Format: REDEEM-XXXXXX (6 characters: numbers + letters mixed)
- Admin can create codes with specific credit amount
- User redeems code → Credit added → Code marked as used
- Prevent reuse of codes

## Product System

### Product Display
- **Card layout** with beautiful design
- Category filtering available
- Bootstrap carousel for product gallery (max 4 images)
- If single image → show as cover image
- If multiple images (gallery mode) → use Bootstrap carousel

### Product Details
- Use **QuillJS Rich Text Editor** in Admin for product descriptions
- Support YouTube video embed in product details
- User only needs to paste YouTube watch URL (e.g., https://www.youtube.com/watch?v=eVTXPUF4Oz4)
- System extracts video ID and creates embed iframe

### Product Data Structure
- Product name
- Category
- Price
- Stock quantity
- Description (Rich Text)
- Cover image URL (external link - NO file uploads)
- Gallery images (up to 4 external URLs)
- YouTube video URL (optional)
- Status (active/inactive)
- **Delivery Type** (manual/auto)
- **Product Keys/Accounts** (for auto delivery - comma-separated list)

## Delivery System

### Delivery Types

**1. Auto Delivery**
- Products are delivered immediately after successful payment
- System automatically assigns product keys/accounts from the product's key pool
- No admin intervention required
- Best for: Game keys, digital codes, account credentials

**2. Manual Delivery**
- Products require admin approval before delivery
- Admin must manually send product to user after purchase
- Order status shows "pending delivery" until admin sends
- Admin can add delivery notes/credentials
- Best for: Custom services, manual processing, verification-required items

### Auto Delivery Implementation
- Store product keys/accounts in `products` table (comma-separated)
- On successful payment:
  - Check if product has available keys
  - Assign first available key to order
  - Remove key from product pool
  - Update order status to "completed"
  - Send notification to user
- If no keys available: Show error, prevent purchase

### Manual Delivery Implementation
- On successful payment:
  - Create order with status "pending_delivery"
 - Notify admin of new manual delivery order
- Admin workflow:
  - View orders with "pending_delivery" status
  - Click "Send Product" button
  - Enter delivery content (keys, credentials, instructions)
  - Submit to complete delivery
  - Update order status to "completed"
  - Send notification to user

### Delivery Display in Purchase History
- **Auto Delivery**: Show delivered key/account immediately
- **Manual Delivery**: Show "Pending Delivery" status until admin sends
- After manual delivery: Show delivered content
- Both types: Allow copy/download of delivered content

## Shopping Cart System

### Cart Features
- Add products to cart
- Remove products from cart
- Update quantity
- **Coupon discount system** (percentage-based)
- Coupons managed in Admin (create, edit, delete, set discount %)

### Real-time Updates
- Cart count in navbar must update via AJAX
- Cart page must update via AJAX
- No page reloads for cart operations

## Order System

### Purchase Flow
1. User adds products to cart
2. Applies coupon (optional)
3. Proceeds to checkout
4. **Credit Check**: Verify user has sufficient credit balance
   - If insufficient: Show error "เครดิตไม่เพียงพอ" with link to `/topup` page, prevent checkout
   - If sufficient: Proceed with credit deduction
5. **Credit Payment Only**: Deduct credit from user balance immediately
6. Order created:
   - If auto delivery: status "completed", product delivered immediately
   - If manual delivery: status "pending_delivery", await admin approval
7. Discord webhook notification sent

### Topup First Requirement
- **MANDATORY**: Users must top up credit BEFORE purchasing
- **NO direct payment during checkout**: PromptPay/TrueWallet can ONLY be used in topup page, NOT during checkout
- Checkout flow:
  - Check user credit balance
  - If credit >= total: Allow purchase, deduct credit immediately
  - If credit < total: Show "เครดิตไม่เพียงพอ" error with "เติมเงิน" button
  - Redirect insufficient balance users to `/topup` page
- Payment methods (PromptPay/TrueWallet/Redeem) are ONLY available on topup page
- Users must complete topup → return to checkout → complete purchase with credit

### Purchase History
- User can view all past orders
- Show order details: products, total, date, status
- **Auto Delivery**: Display delivered keys/accounts immediately
- **Manual Delivery**: Show "Pending Delivery" until admin sends
- After delivery: Display delivered content
- Downloadable/copyable product keys/credentials

## Homepage Design

### Hero Section
- **Carousel/slider** with 3 background images
- Images configurable in Admin
- Smooth transitions between slides
- Call-to-action buttons

### Latest Products
- Display latest products in card layout below hero section
- Show product image, name, price

### Recently Sold
- Display recently sold products (small cards)
- Less prominent than latest products
- Shows what others are buying

### Popup Promotion
- **Modal popup** for promotions/news
- Shows as image only
- Close button to dismiss
- Configurable in Admin (enable/disable, set image URL)
- Example: https://img2.pic.in.th/image667c3634ae8751a8.png

## Admin Dashboard

### Dashboard Analytics
You must analyze and include all necessary admin features:
- Total users count
- Total products count
- Total orders count
- Total revenue
- Today's revenue
- Recent orders list
- Recent registrations
- Low stock alerts
- Popular products

### Admin Sections (CRUD Operations)

**1. Products Management**
- Create, Read, Update, Delete products
- Category management
- Stock management
- Image URL management (external links only)
- Rich text editor for descriptions
- YouTube URL handling
- **Delivery Type selection** (manual/auto)
- **Product Keys/Accounts management** (for auto delivery - add/remove keys)

**2. Orders Management**
- View all orders
- Order details
- Order status management
- Search/filter orders
- **Manual Delivery Queue**:
  - View all orders with "pending_delivery" status
  - Send product to user (enter delivery content)
  - Add delivery notes
  - Mark as completed
- **Auto Delivery Monitoring**:
  - View auto-delivered orders
  - Check key inventory status
  - Low key alerts

**3. Users Management**
- View all users
- Edit user details
- **Change user email** (users cannot change their own email)
- **Change user password** (users cannot change their own password in profile)
- Manage user credit
- Ban/unban users

**4. Coupons Management**
- Create discount coupons
- Set discount percentage
- Set usage limit (optional)
- Enable/disable coupons
- View coupon usage statistics

**5. Redeem Codes Management**
- Generate new redeem codes
- Set credit amount per code
- View all codes
- Mark codes as used
- View code usage history

**6. Payment Settings**
- PromptPay API configuration
- TrueWallet API configuration
- Enable/disable payment methods

**7. Cloudflare Turnstile Settings**
- Enable/disable Turnstile
- Set Site Key
- Set Secret Key

**8. Discord Webhook Settings**
- Set Discord webhook URL
- Enable/disable notifications for:
  - Payment received
  - New order placed
- Test webhook button

**9. Site Settings**
- Site name
- Logo URL
- Hero section images (3 URLs)
- Popup promotion (enable/disable, image URL)
- Footer social media links:
  - Line
  - Facebook
  - Discord
  - Email

**10. Categories Management**
- Create, edit, delete categories
- Category ordering

## Footer Design
- **Left side**: Copyright text
- **Right side**: Social media icons (Line, Facebook, Discord, Email)
- All social links configurable in Admin

## File Upload Policy
- **STRICTLY NO FILE UPLOADS**
- Use external image links only (Imgur, Cloudinary, etc.)
- All image inputs are URL fields

## Real-time Updates (AJAX)
The following must update in real-time without page reload:
- Cart count in navbar
- Cart page operations (add/remove/update)
- Credit balance display (fetch from database via AJAX)
- Payment status polling (PromptPay)

## File Structure
Create a clean, organized file structure:

```
/
├── config/
│   ├── config.php           # Database & general config
│   ├── functions.php        # Common functions
│   └── constants.php        # Constants
├── includes/
│   ├── header.php           # Main website header
│   ├── footer.php           # Main website footer
│   ├── navbar.php           # Navbar component
│   ├── admin-header.php     # Admin header
│   ├── admin-sidebar.php    # Admin sidebar
│   └── admin-footer.php     # Admin footer
├── assets/
│   ├── css/
│   │   └── custom.css       # Custom styles
│   ├── js/
│   │   ├── main.js          # Main JavaScript
│   │   ├── ajax.js          # AJAX functions
│   │   └── admin.js         # Admin JavaScript
│   └── images/              # (empty - use external URLs)
├── pages/
│   ├── home.php             # Homepage
│   ├── products.php         # Products listing
│   ├── product-detail.php   # Product detail
│   ├── cart.php             # Shopping cart
│   ├── checkout.php         # Checkout
│   ├── login.php            # User login
│   ├── register.php         # User registration
│   ├── profile.php          # User profile
│   ├── orders.php           # Purchase history
│   ├── topup.php            # Top-up/Select Type for top-up/credit page
│   ├── topup-promptpay.php  # PromptPay payment
│   ├── topup-gift.php       # TrueWallet gift
│   ├── topup-redeem.php     # Redeem code
│   └── redeem.php           # Redeem code processing
├── admin/
│   ├── index.php            # Admin dashboard
│   ├── login.php            # Admin login
│   ├── products.php         # Products management
│   ├── orders.php           # Orders management
│   ├── users.php            # Users management
│   ├── coupons.php          # Coupons management
│   ├── redeem-codes.php     # Redeem codes management
│   ├── settings.php         # Payment & general settings
│   ├── categories.php       # Categories management
│   └── webhook-promptpay.php # PromptPay webhook
├── api/
│   ├── cart.php             # Cart AJAX endpoints
│   ├── search.php           # Search AJAX
│   └── check-payment.php    # Payment status check
├── database/
│   ├── schema.sql           # Database schema
│   └── seed.sql             # Sample data seed
├── index.php                # Main entry point
├── .htaccess                # URL rewriting rules
└── prompt.md                # This file
└── README.md                # Project documentation
```

## Stepwise Development Process

You MUST follow this stepwise process to ensure correct implementation:

### Step 1: Database Schema
- Create complete database schema
- Include all tables: users, admins, products, categories, orders, order_items, coupons, redeem_codes, site_settings, payments
- **Add delivery-related fields**:
  - `products` table: delivery_type (enum: 'manual', 'auto'), product_keys (text - comma-separated)
  - `orders` table: delivery_status (enum: 'pending', 'delivered', 'pending_delivery'), delivery_content (text)
  - `order_items` table: assigned_key (text - for auto delivery)
- Use proper data types, indexes, foreign keys
- Add seed data file with sample data

### Step 2: Configuration & Functions
- Create config.php with database connection (PDO)
- Create functions.php with common functions
- Create constants.php
- **Verify**: Test database connection

### Step 3: Authentication System
- Create user registration (with validation, Turnstile)
- Create user login (with Turnstile)
- Create admin login (separate from user)
- Create logout functionality
- Create password change (user)
- **Verify**: Test all auth flows, check validation, verify Turnstile integration

### Step 4: Basic Layout
- Create header.php, footer.php, navbar.php
- Create admin-header.php, admin-sidebar.php, admin-footer.php
- Apply Bootstrap and custom CSS
- **Verify**: Check layout rendering, responsive design

### Step 5: Homepage
- Create hero section with carousel
- Create latest products section
- Create recently sold section
- Create popup promotion modal
- **Verify**: Check carousel functionality, popup display

### Step 6: Product System
- Create products listing page with category filter
- Create product detail page
- Implement Bootstrap carousel for product gallery
- Implement YouTube embed functionality
- Display delivery type on product page (Auto/Manual badge)
- **Verify**: Check product display, carousel, video embed, delivery type display

### Step 7: Admin - Products
- Create products CRUD in admin
- Integrate QuillJS for rich text
- Test category management
- **Add delivery type selection** (manual/auto)
- **Add product keys management** for auto delivery (add/remove keys from pool)
- **Verify**: Test all product operations, image handling, delivery type, key management

### Step 8: Shopping Cart
- Create cart page
- Implement AJAX cart operations
- Implement coupon system
- **Verify**: Test add/remove/update, coupon application, real-time updates

### Step 9: Order System
- Create checkout flow
- Create order processing with delivery type logic:
  - Auto delivery: Assign key immediately, mark as completed
  - Manual delivery: Mark as pending_delivery
- Create purchase history page with delivery status display
- Display delivered products
- **Verify**: Test complete purchase flow for both delivery types, order history display

### Step 10: Payment Systems
- Implement PromptPay payment (study API example)
- Implement TrueWallet gift (study API example)
- Implement Redeem code system
- Create webhook handler for PromptPay
- **Verify**: Test all payment methods, webhook notifications

### Step 11: Admin Dashboard
- Create dashboard with analytics
- Create orders management
- **Add Manual Delivery Queue**:
  - List pending_delivery orders
  - Send product form with delivery content
  - Complete delivery action
- Create users management (with email/password edit)
- Create coupons management
- Create redeem codes management
- Create payment settings
- Create Turnstile settings
- Create Discord webhook settings
- Create site settings
- **Verify**: Test all admin features, settings persistence, manual delivery workflow

### Step 12: Discord Integration
- Implement Discord webhook for payment notifications
- Implement Discord webhook for order notifications
- Test webhook functionality
- **Verify**: Verify webhook delivery, message format

### Step 13: Real-time Updates
- Implement AJAX for cart operations
- Implement AJAX for credit balance
- Implement polling for PromptPay payment
- **Verify**: Test all real-time updates, no page reloads

### Step 14: Final Testing & Debugging
- Test complete user flow from registration to purchase
- Test all admin features
- Check all validations
- Verify security (SQL injection, XSS, CSRF)
- Test on different browsers
- **Verify**: Comprehensive testing, fix any bugs
- create README.md

### Step 15: Routing System Implementation
- Create index.php as the main entry point with proper routing
- Create `.htaccess` with mod_rewrite rules to hide `index.php`
- Implement clean URL routing (e.g., `/products` rewrites to `index.php?page=products`)
- Ensure ALL pages are accessible through clean URLs:
  - home (`/`)
  - products (`/products`)
  - product-detail (`/product-detail?id=X`)
  - cart (`/cart`)
  - checkout (`/checkout`)
  - login (`/login`)
  - register (`/register`)
  - profile (`/profile`)
  - orders (`/orders`)
  - topup (`/topup`)
  - topup-promptpay (`/topup-promptpay`)
  - topup-gift (`/topup-gift`)
  - topup-redeem (`/topup-redeem`)
  - redeem (`/redeem`)
  - Admin pages (`/admin/xxx.php`)
- **ALL links in website must use clean URLs** (NO `index.php?page=` in user-facing links)
- Verify all routes work correctly
- Test navigation between all pages
- **Verify**: Complete routing system, all pages accessible with clean URLs

### Step 16: File Structure Verification
- Verify all files exist according to the structure below
- Check all directories are created
- Ensure no missing files
- Validate file paths are correct
- **Verify**: Complete file structure matches specification

### Step 17: Syntax & Error Checking
- Check all PHP files for syntax errors
- Verify all variables are correctly named and used
- Check all database queries use prepared statements
- Verify all AJAX endpoints work correctly
- **Verify**: No syntax errors, all variables match, clean code

## Critical Requirements

### Security
- **ALL database queries MUST use PDO prepared statements**
- No raw SQL queries
- Validate all user inputs
- Sanitize all outputs
- Protect against XSS, CSRF, SQL injection

### Validation
- Username: 6-20 characters, alphanumeric
- Password: 6+ characters
- Email: Valid email format
- Confirm password: Must match
- All fields must have proper validation

### No File Uploads
- Use external image URLs only
- No file upload forms
- No server-side file processing

### SweetAlert2
- **MANDATORY**: All alerts must use SweetAlert2
- No JavaScript alert()
- No Bootstrap alerts
- Consistent alert styling

### Real-time Updates
- Cart operations via AJAX
- Credit balance via AJAX
- Payment status via polling
- No page reloads for these operations

### Admin Separation
- Admin and user authentication completely separate
- Admin uses different layout (sidebar)
- Admin cannot access user areas
- Users cannot access admin areas

## Debugging & Testing Requirements

For each step, you must:
1. Test the implementation thoroughly
2. Verify all variables are correctly named and used
3. Check for syntax errors
4. Verify database queries work correctly
5. Test edge cases
6. Document any issues found
7. Fix issues before proceeding to next step

## Final Deliverables

1. Complete, working website
2. Database schema (schema.sql)
3. Seed data (seed.sql)
4. Clean, organized file structure
5. All features working as specified
6. No syntax errors
7. All validations working
8. All payment methods functional
9. Admin dashboard complete
10. Discord webhook notifications working
11. Real-time updates working
12. Responsive design
13. Dark lime green theme with animations

## Important Notes

- Use Thai language for all user-facing text
- Use English for code comments and variable names
- Follow PHP best practices
- Write clean, readable code
- Add comments for complex logic
- Test everything before final delivery
- Verify all requirements are met

---

**Begin development following the stepwise process above. Complete each step fully before moving to the next. Test thoroughly at each step.**
