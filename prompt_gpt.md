You are a senior full-stack developer with expertise in PHP 8.2+, MariaDB 10.6+, MVC architecture, and modern web development.

Create a COMPLETE production-ready web application for a digital marketplace system.

========================================
📌 PROJECT OVERVIEW
========================================
Build a web-based digital marketplace system for selling:
- Game Accounts (username/password, game-specific data, account level, etc.)
- Game Keys (redeem codes, serial keys, CD keys)
- Digital Accounts (Netflix, Spotify, Disney+, YouTube Premium, etc.)

Complete User Flow:
1. Visitor browses products (homepage, categories, search)
2. Visitor registers account OR logs in
3. User tops up wallet (PromptPay QR, TrueWallet Gift, or Redeem Code)
4. User adds products to cart
5. User applies coupon (optional)
6. User checks out using wallet balance
7. System deducts wallet balance and creates order
8. User receives product (auto-delivery or manual delivery)
9. User views order history and downloaded products

CRITICAL BUSINESS LOGIC:
- This is a WALLET-BASED SYSTEM (users must top-up credit first, then use credit to purchase)
- NO direct payment gateway at checkout (no Stripe, PayPal, etc.)
- Wallet balance is stored in DATABASE (NOT session)
- All wallet transactions must be logged with type (top-up, purchase, refund)
- Stock must be checked and decremented atomically during purchase
- Auto-delivery products must be sent immediately after successful payment

========================================
🎨 DESIGN REQUIREMENTS
========================================
Color Scheme:
- Primary Background: #000000 (Black)
- Accent Color: #80ef25 (Lime Green)
- Secondary Accent: #2d2d2d (Dark Gray)
- Text Color: #ffffff (White)
- Error Color: #ef4444 (Red)
- Success Color: #22c55e (Green)

Typography:
- Headings: Kanit (Google Fonts) - weights: 400, 500, 600, 700
- Body: Sarabun (Google Fonts) - weights: 300, 400, 500, 600
- Font sizes: Base 16px, H1 36px, H2 30px, H3 24px

UI Framework:
- Bootstrap 5.3.x (latest stable)
- Bootstrap Icons for iconography
- Custom CSS in /public/assets/css/style.css

CSS Requirements:
- NO inline CSS styles in HTML
- All styles in external CSS file
- Use CSS variables for colors and fonts
- Add glow effects on hover for buttons and cards
- Smooth transitions (0.3s ease)
- Custom scrollbar styling

Layout Structure:
- Frontend: Navbar (top) + Main Content + Footer (bottom)
- Admin: Sidebar (left) + Main Content (right)
- Responsive: Mobile-first approach, hamburger menu on mobile

Animation Effects:
- Button hover: glow effect with box-shadow
- Card hover: slight lift and glow
- Page transitions: smooth fade-in
- Loading states: spinner animations

========================================
🧠 CORE FEATURES - DETAILED SPECIFICATIONS
========================================

👤 AUTHENTICATION SYSTEM:

Database Table: users
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- username (VARCHAR(20), UNIQUE, NOT NULL)
- password (VARCHAR(255), NOT NULL) - hashed with password_hash()
- email (VARCHAR(100), UNIQUE, NOT NULL)
- role (ENUM('user', 'admin'), DEFAULT 'user')
- wallet_balance (DECIMAL(10,2), DEFAULT 0.00)
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- updated_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)

Features:
1. Register:
   - Username: 6-20 characters, alphanumeric only
   - Password: minimum 6 characters
   - Confirm password: must match
   - Email: valid email format
   - Store password using password_hash() with PASSWORD_DEFAULT
   - Insert into users table with role='user', wallet_balance=0
   - Redirect to login page after successful registration
   - Show success/error messages using SweetAlert2

2. Login:
   - Username or email input
   - Password input
   - Verify credentials using password_verify()
   - Set session variables: $_SESSION['user_id'], $_SESSION['username'], $_SESSION['role']
   - Redirect to homepage after successful login
   - Remember me option (optional cookie)

3. Logout:
   - Destroy session
   - Clear all session variables
   - Redirect to homepage

4. Change Password:
   - Current password verification
   - New password (min 6 chars)
   - Confirm new password
   - Update password in database
   - Require re-login after password change

5. Session Management:
   - Check session on every protected page
   - Auto-logout after 30 minutes of inactivity
   - Regenerate session ID on login

🔐 SECURITY IMPLEMENTATION:

1. Database Security:
   - Use PDO prepared statements for ALL queries
   - NEVER concatenate user input into SQL
   - Use bindParam() or bindValue() for parameters
   - Enable PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION

2. Password Security:
   - Hash passwords using password_hash($password, PASSWORD_DEFAULT)
   - Verify using password_verify($input, $hash)
   - Never store plain text passwords

3. CSRF Protection:
   - Generate CSRF token on session start: $_SESSION['csrf_token']
   - Include token in all forms as hidden input
   - Verify token on form submission
   - Regenerate token after successful submission

4. XSS Protection:
   - Use htmlspecialchars() when outputting user data
   - Set Content-Security-Policy header
   - Validate and sanitize all inputs

5. SQL Injection Prevention:
   - Prepared statements ONLY
   - Validate data types before binding
   - Use whitelist validation for enum values

6. Cloudflare Turnstile:
   - Add Turnstile widget to register/login forms
   - Verify token server-side before processing
   - Enable/disable via admin settings
   - Store site key and secret key in settings table

========================================
💰 WALLET SYSTEM - DETAILED SPECIFICATIONS
========================================

Database Tables:

wallet_transactions:
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- user_id (INT, FOREIGN KEY users.id)
- type (ENUM('topup', 'purchase', 'refund', 'admin_adjust'))
- amount (DECIMAL(10,2), NOT NULL)
- balance_before (DECIMAL(10,2), NOT NULL)
- balance_after (DECIMAL(10,2), NOT NULL)
- description (VARCHAR(255))
- reference_id (VARCHAR(50)) - order_id or topup_id
- payment_method (VARCHAR(50)) - promptpay, truewallet, redeem, admin
- status (ENUM('pending', 'completed', 'failed', 'cancelled'), DEFAULT 'pending')
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

redeem_codes:
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- code (VARCHAR(20), UNIQUE, NOT NULL) - Format: REDEEM-XXXXXX
- amount (DECIMAL(10,2), NOT NULL)
- is_used (BOOLEAN, DEFAULT FALSE)
- used_by (INT, NULL, FOREIGN KEY users.id)
- used_at (TIMESTAMP, NULL)
- created_by (INT, FOREIGN KEY users.id)
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- expires_at (TIMESTAMP, NULL)

Wallet Logic:
1. Top-up Flow:
   a. User selects top-up method
   b. For PromptPay: Generate QR, wait for callback
   c. For TrueWallet: Submit gift link URL
   d. For Redeem: Enter code
   e. On success: Create wallet_transactions record, update users.wallet_balance
   f. Use database transaction to ensure atomicity

2. Purchase Flow:
   a. Check if user has sufficient wallet balance
   b. Calculate total (cart items - coupon discount)
   c. Deduct from wallet_balance
   d. Create wallet_transactions record (type='purchase')
   e. Create order record
   f. Decrease product stock
   g. All operations in single database transaction

3. Balance Query:
   - ALWAYS query from database users.wallet_balance
   - NEVER store wallet balance in session
   - Refresh balance display via AJAX after transactions

4. Transaction History:
   - Show all wallet transactions for user
   - Filter by type, date range
   - Pagination (20 items per page)

========================================
📱 PAYMENT API INTEGRATIONS
========================================

1. PromptPay API (Reference: /PromptPay/ folder):
   API Endpoint: https://tmwallet.thaighost.net/api_pph.php
   
   Configuration (stored in settings table):
   - tmweasy_user: API username
   - tmweasy_password: API password
   - tmweasy_api_key: API key
   - con_id: Connection ID
   - prommpay_no: PromptPay number (mobile/ID card/E-Wallet)
   - prommpay_type: 01=Mobile, 02=ID Card, 03=E-Wallet
   - prommpay_name: Account holder name
   
   API Methods:
   - create_pay: Create payment with amount, ref1, con_id, ip
   - detail_pay: Get QR code and payment details
   - cancel: Cancel payment
   - check: Check payment status
   
   Integration Steps:
   1. User enters amount
   2. Call create_pay API
   3. Store id_pay in session
   4. Display QR code from detail_pay API
   5. Start countdown timer
   6. Poll check API every 3 seconds
   7. On success: Add wallet balance, create transaction
   8. Redirect to success page

2. TrueWallet Gift API (Reference: /TrueWallet-Gift/ folder):
   API Endpoint: https://tmwallet.thaighost.net/apiwallet.php
   
   Configuration (stored in settings table):
   - tmweasy_user: API username
   - tmweasy_password: API password
   - truewallet_mobile: TrueWallet mobile number
   
   API Methods:
   - check: Check API readiness
   - yes: Process gift link
   
   Integration Steps:
   1. User enters gift link URL (must contain 'ttp')
   2. Call yes API with transactionid=gift URL
   3. API returns Status='check_success' with Amount
   4. Add amount to user wallet balance
   5. Create wallet transaction record
   6. Show success message

3. Redeem Code System:
   - Admin generates codes with specified amount
   - Format: REDEEM-XXXXXX (6 random alphanumeric chars)
   - Codes are one-time use only
   - Can set expiration date
   - User enters code to redeem
   - System checks if code exists, not used, not expired
   - If valid: Add amount, mark as used, log transaction

========================================
🛒 PRODUCT SYSTEM - DETAILED SPECIFICATIONS
========================================

Database Tables:

categories:
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- name (VARCHAR(100), NOT NULL)
- slug (VARCHAR(100), UNIQUE, NOT NULL)
- description (TEXT)
- icon (VARCHAR(255)) - icon class or URL
- sort_order (INT, DEFAULT 0)
- is_active (BOOLEAN, DEFAULT TRUE)
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

products:
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- category_id (INT, FOREIGN KEY categories.id)
- name (VARCHAR(255), NOT NULL)
- slug (VARCHAR(255), UNIQUE, NOT NULL)
- description (TEXT) - HTML from QuillJS
- price (DECIMAL(10,2), NOT NULL)
- stock (INT, DEFAULT 0)
- images (JSON) - Array of image URLs
- youtube_url (VARCHAR(255)) - Optional YouTube embed
- delivery_method (ENUM('manual', 'auto'), DEFAULT 'manual')
- delivery_data (TEXT) - JSON for auto-delivery data
- is_active (BOOLEAN, DEFAULT TRUE)
- is_featured (BOOLEAN, DEFAULT FALSE)
- views (INT, DEFAULT 0)
- sales_count (INT, DEFAULT 0)
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- updated_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)

Product Features:

1. Category System:
   - Hierarchical categories (optional - flat for now)
   - Category page shows all products in category
   - Filter by category on product listing
   - Admin can add/edit/delete categories
   - Sort order for display

2. Product Listing:
   - Grid layout (responsive: 1 col mobile, 2 tablet, 3-4 desktop)
   - Product card shows:
     * First image (or placeholder)
     * Product name
     * Price
     * Stock status (In Stock / Out of Stock)
     * Add to Cart button (disabled if out of stock)
   - Pagination (12 products per page)
   - Sort by: Newest, Price Low-High, Price High-Low, Best Selling
   - Search by name
   - Filter by category

3. Product Detail Page:
   - Image gallery (slider for 2-4 images)
   - Product name
   - Category breadcrumb
   - Price
   - Stock indicator
   - Description (HTML from QuillJS)
   - YouTube embed (if provided)
   - Add to Cart button
   - Quantity selector
   - Related products (same category)
   - Increment view count on page load

4. Image Handling:
   - Store image URLs as JSON array in database
   - No file uploads - use external image URLs
   - Support URLs from: Imgur, Cloudinary, direct image links
   - Fallback placeholder image if no images
   - Lazy loading for images

5. Delivery Methods:
   
   a. Manual Delivery:
      - Admin must manually deliver product after order
      - Order status: "Pending Delivery"
      - Admin enters delivery data and marks as "Delivered"
      - User sees delivery data after status change
   
   b. Auto Delivery:
      - Product data stored in delivery_data field (JSON)
      - Automatically sent to user immediately after successful payment
      - Order status: "Delivered" instantly
      - delivery_data format:
        {
          "type": "account",
          "username": "user123",
          "password": "pass123",
          "notes": "Additional info"
        }
        OR
        {
          "type": "key",
          "keys": ["KEY1", "KEY2", "KEY3"]
        }

6. Stock Management:
   - Stock decremented on successful purchase
   - Stock cannot go below 0
   - Product shows "Out of Stock" when stock = 0
   - Add to Cart disabled when out of stock
   - Admin can adjust stock manually
   - Low stock alert (optional: when stock < 5)

========================================
🧾 ORDER SYSTEM - DETAILED SPECIFICATIONS
========================================

Database Tables:

orders:
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- order_number (VARCHAR(20), UNIQUE, NOT NULL) - Format: ORD-YYYYMMDD-XXXXX
- user_id (INT, FOREIGN KEY users.id)
- total_amount (DECIMAL(10,2), NOT NULL)
- discount_amount (DECIMAL(10,2), DEFAULT 0.00)
- coupon_code (VARCHAR(50), NULL)
- final_amount (DECIMAL(10,2), NOT NULL)
- status (ENUM('pending', 'completed', 'cancelled', 'refunded'), DEFAULT 'pending')
- delivery_status (ENUM('pending_delivery', 'delivered'), DEFAULT 'pending_delivery')
- payment_method (VARCHAR(50)) - 'wallet'
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
- updated_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)

order_items:
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- order_id (INT, FOREIGN KEY orders.id)
- product_id (INT, FOREIGN KEY products.id)
- product_name (VARCHAR(255), NOT NULL)
- product_price (DECIMAL(10,2), NOT NULL)
- quantity (INT, DEFAULT 1)
- subtotal (DECIMAL(10,2), NOT NULL)
- delivery_data (TEXT) - JSON (for auto-delivery)
- delivery_status (ENUM('pending_delivery', 'delivered'), DEFAULT 'pending_delivery')
- delivery_data_sent (TEXT) - JSON (actual data sent to user)
- delivered_at (TIMESTAMP, NULL)

Cart System:

1. Cart Storage:
   - Store cart in session: $_SESSION['cart']
   - Format: Array of items with product_id, quantity
   - Cart persists across page loads
   - Cart cleared after successful checkout

2. Cart Operations:
   a. Add to Cart:
      - Check if product exists and in stock
      - Check if already in cart (increment quantity)
      - Validate quantity (max 10 per product)
      - Update cart via AJAX
      - Show success message via SweetAlert2
   
   b. Remove from Cart:
      - Remove item from cart array
      - Update via AJAX
   
   c. Update Quantity:
      - Increment/decrement quantity
      - Validate against stock
      - Update via AJAX
   
   d. Get Cart:
      - Fetch product details for each cart item
      - Calculate subtotal
      - Display cart page with:
        * Product list with images
        * Quantity controls
        * Remove buttons
        * Subtotal per item
        * Total amount
        * Coupon input
        * Checkout button

3. Coupon System:
   
   Database Table: coupons
   - id (INT, AUTO_INCREMENT, PRIMARY KEY)
   - code (VARCHAR(50), UNIQUE, NOT NULL)
   - discount_percent (DECIMAL(5,2), NOT NULL) - e.g., 10.00 for 10%
   - max_uses (INT, NULL) - NULL for unlimited
   - used_count (INT, DEFAULT 0)
   - valid_from (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
   - valid_until (TIMESTAMP, NULL)
   - is_active (BOOLEAN, DEFAULT TRUE)
   - created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)
   
   Coupon Logic:
   - User enters coupon code at checkout
   - Validate: exists, active, within date range, not exceeded max uses
   - Apply percentage discount to cart total
   - Show discounted amount
   - One coupon per order
   - Increment used_count after successful order

4. Checkout Process:
   a. User reviews cart
   b. Applies coupon (optional)
   c. Clicks checkout
   d. System validates:
      - User is logged in
      - Cart is not empty
      - All products in stock
      - User has sufficient wallet balance
   e. Create database transaction:
      - Lock user row for update
      - Check wallet balance again
      - Deduct wallet balance
      - Create order record
      - Create order_items records
      - Create wallet transaction (type='purchase')
      - Decrement product stock
      - For auto-delivery items: set delivery_data and status='delivered'
   f. Commit transaction
   g. Clear cart
   h. Send Discord webhook notification
   i. Redirect to order success page
   j. Show order details and delivery data

5. Order History:
   - User can view all their orders
   - Show order number, date, total, status
   - Click to view order details
   - Order details page shows:
     * Order information
     * All items with delivery status
     * If delivered: show delivery data with copy button
     * Download/copy functionality for account info or keys
   - Filter by status
   - Pagination (10 orders per page)

6. Order Status Flow:
   - Pending: Order created, payment processed
   - Completed: All items delivered (manual delivery)
   - Cancelled: Order cancelled by admin
   - Refunded: Refunded by admin (wallet credited back)

========================================
🎟️ COUPON SYSTEM - ADMIN FEATURES
========================================

Admin Coupon Management:
1. Create Coupon:
   - Generate random code or enter custom code
   - Set discount percentage (1-100%)
   - Set maximum uses (optional, unlimited if NULL)
   - Set validity period (from/until dates)
   - Toggle active status

2. Edit Coupon:
   - Modify all fields except code
   - Change active status

3. Delete Coupon:
   - Soft delete (set is_active=FALSE)
   - Or hard delete if never used

4. View Coupons:
   - List all coupons with details
   - Show usage statistics
   - Filter by active status

5. Coupon Analytics:
   - Total uses per coupon
   - Total discount given
   - Most popular coupons

========================================
📊 HOMEPAGE - DETAILED SPECIFICATIONS
========================================

Database Table: homepage_settings
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- hero_button_text (VARCHAR(100))
- hero_button_url (VARCHAR(255))
- hero_images (JSON) - Array of 2 image URLs
- stats_enabled (BOOLEAN, DEFAULT TRUE)
- new_products_count (INT, DEFAULT 8)

Homepage Sections:

1. Hero Section:
   - Full-width banner
   - Background image slider (2 images, auto-rotate every 5 seconds)
   - Overlay text with headline
   - CTA button (customizable text and URL)
   - Images editable via admin
   - Button text and URL editable via admin

2. Statistics Section:
   - Display 3-4 statistics cards:
     * Total Users
     * Total Products
     * Total Orders
     * Total Sales (THB)
   - Update in real-time via AJAX
   - Can be enabled/disabled via admin

3. New Products Section:
   - Grid of newest products (8 by default)
   - Sort by created_at DESC
   - Same card layout as product listing
   - "View All Products" button
   - Count configurable via admin

4. Featured Categories (Optional):
   - Show 4-6 featured categories
   - Category icon and name
   - Click to view category page

5. Footer:
   - Copyright info
   - Social media links
   - Quick links (Home, Products, Contact)
   - Contact info (phone, email)

========================================
📞 CONTACT PAGE - DETAILED SPECIFICATIONS
========================================

Database Table: contact_settings
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- title (VARCHAR(255))
- content (TEXT) - HTML content
- phone (VARCHAR(50))
- email (VARCHAR(100))
- facebook_url (VARCHAR(255))
- line_url (VARCHAR(255))
- address (TEXT)
- updated_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP)

Contact Page Features:
- Page title (editable via admin)
- Rich text content (editable via admin with QuillJS)
- Contact information display:
  * Phone number
  * Email
  * Facebook link
  * LINE link
  * Address
- Social media buttons
- All fields editable via admin panel
- No contact form (informational only)

========================================
🔔 DISCORD WEBHOOK - DETAILED SPECIFICATIONS
========================================

Database Table: discord_settings
- id (INT, AUTO_INCREMENT, PRIMARY KEY)
- webhook_url (VARCHAR(500))
- enabled (BOOLEAN, DEFAULT FALSE)
- notify_topup (BOOLEAN, DEFAULT TRUE)
- notify_order (BOOLEAN, DEFAULT TRUE)
- bot_name (VARCHAR(100), DEFAULT 'Store Bot')
- bot_avatar_url (VARCHAR(255))

Webhook Notifications:

1. Wallet Top-up Notification:
   - Trigger: Successful wallet top-up
   - Embed fields:
     * Title: "💰 Wallet Top-up"
     * Username
     * Amount
     * Payment Method
     * New Balance
     * Timestamp
   - Color: Green (0x22c55e)

2. New Order Notification:
   - Trigger: Successful order creation
   - Embed fields:
     * Title: "🛒 New Order"
     * Order Number
     * Username
     * Total Amount
     * Items Count
     * Timestamp
   - Color: Blue (0x3b82f6)

3. Webhook Implementation:
   - Use cURL to send POST request to Discord webhook URL
   - Send as JSON embed
   - Handle errors gracefully (log if webhook fails)
   - Can be enabled/disabled per event type
   - Customizable bot name and avatar

4. Admin Configuration:
   - Set webhook URL
   - Enable/disable webhook entirely
   - Enable/disable per event type
   - Set bot name and avatar

========================================
⚙️ ADMIN PANEL - DETAILED SPECIFICATIONS
========================================

Admin Authentication:
- Separate login from frontend (/admin/login)
- Only users with role='admin' can access
- Check role on every admin page
- Redirect to admin login if not authenticated
- Session timeout: 60 minutes

Admin Layout:
- Sidebar navigation (left, fixed)
- Main content area (right)
- Responsive: Hamburger menu on mobile
- Dark theme matching frontend

Admin Navigation Menu:
1. Dashboard (/admin)
2. Users (/admin/users)
3. Products (/admin/products)
4. Categories (/admin/categories)
5. Orders (/admin/orders)
6. Coupons (/admin/coupons)
7. Redeem Codes (/admin/redeem-codes)
8. Wallet Transactions (/admin/wallet)
9. Website Settings (/admin/settings)
10. Payment API Config (/admin/payment-api)
11. Discord Webhook (/admin/discord)
12. Contact Page (/admin/contact)
13. Logout (/admin/logout)

Admin Dashboard:
- Statistics cards:
  * Total Users
  * Total Products
  * Total Orders
  * Total Revenue
  * Pending Orders
  * Low Stock Products
- Recent orders table (last 10)
- Recent wallet top-ups (last 10)
- Quick action buttons

Admin CRUD Operations:

1. Users Management:
   - List all users with pagination
   - Search by username/email
   - View user details
   - Edit user (role, wallet balance)
   - Adjust wallet balance (admin_adjust transaction)
   - Delete user (soft delete recommended)
   - Ban/unban user (add is_banned field to users table)

2. Products Management:
   - List all products with pagination
   - Search by name
   - Filter by category
   - Add new product:
     * Name, slug (auto-generated)
     * Category selection
     * Price
     * Stock
     * Images (comma-separated URLs)
     * Description (QuillJS editor)
     * YouTube URL (optional)
     * Delivery method (manual/auto)
     * Delivery data (JSON textarea for auto-delivery)
     * Featured checkbox
     * Active status
   - Edit product (all fields)
   - Delete product
   - Bulk actions (delete, activate, deactivate)

3. Categories Management:
   - List all categories
   - Add category:
     * Name, slug (auto-generated)
     * Description
     * Icon (class or URL)
     * Sort order
     * Active status
   - Edit category
   - Delete category (check if products exist)

4. Orders Management:
   - List all orders with pagination
   - Filter by status
   - Search by order number or username
   - View order details:
     * Order information
     * Items list
     * Delivery status per item
     * For manual delivery: enter delivery data and mark as delivered
   - Change order status
   - Cancel order (refund wallet)
   - Export order to CSV (optional)

5. Coupons Management:
   - List all coupons
   - Add coupon (see Coupon System section)
   - Edit coupon
   - Delete coupon
   - View usage statistics

6. Redeem Codes Management:
   - List all codes
   - Generate new codes:
     * Batch generation (generate multiple at once)
     * Set amount per code
     * Set expiration (optional)
   - View code status (used/unused)
   - Delete codes
   - Export codes to CSV

7. Wallet Transactions:
   - List all transactions
   - Filter by type, user, date range
   - View transaction details
   - Manual adjustment (add/deduct user balance)

8. Website Settings:
   - Site name
   - Site description
   - Site logo URL
   - Hero section settings
   - Statistics section toggle
   - New products count

9. Payment API Configuration:
   - PromptPay settings:
     * TMW Easy API credentials
     * PromptPay number
     * PromptPay type
     * Account name
     * Enable/disable
   - TrueWallet settings:
     * TMW Easy API credentials
     * TrueWallet mobile number
     * Enable/disable

10. Discord Webhook:
    - Webhook URL
    - Enable/disable
    - Notification toggles
    - Bot name and avatar

11. Contact Page:
    - Title
    - Content (QuillJS editor)
    - Phone, email
    - Social media links
    - Address

Admin UI Requirements:
- Use Bootstrap for layout
- SweetAlert2 for confirmations
- DataTables for tables (optional, or use Bootstrap tables)
- QuillJS for rich text editors
- Form validation
- Loading states for async operations

========================================
⚡ AJAX / REAL-TIME FEATURES - DETAILED SPECIFICATIONS
========================================

AJAX Endpoints (all in /public/api/):

1. Cart Operations:
   POST /api/cart/add
   - Input: product_id, quantity
   - Output: success/error, cart count
   
   POST /api/cart/remove
   - Input: product_id
   - Output: success/error, cart count
   
   POST /api/cart/update
   - Input: product_id, quantity
   - Output: success/error, cart count, subtotal
   
   GET /api/cart
   - Output: cart items, total amount
   
   POST /api/cart/clear
   - Output: success

2. Wallet Operations:
   GET /api/wallet/balance
   - Output: current balance (from database)
   - Called after top-up or purchase
   
   POST /api/wallet/topup/promptpay/create
   - Input: amount
   - Output: payment_id, qr_code, timeout
   
   POST /api/wallet/topup/promptpay/check
   - Input: payment_id
   - Output: status, amount (if success)
   
   POST /api/wallet/topup/truewallet
   - Input: gift_url
   - Output: success/error, amount
   
   POST /api/wallet/redeem
   - Input: code
   - Output: success/error, amount, new_balance

3. Coupon Operations:
   POST /api/coupon/apply
   - Input: coupon_code
   - Output: success/error, discount_amount, new_total
   
   POST /api/coupon/remove
   - Output: success, original_total

4. Order Operations:
   GET /api/order/status/:order_id
   - Output: order status, delivery_status
   
   POST /api/order/cancel/:order_id
   - Output: success/error

5. Product Operations:
   GET /api/product/stock/:product_id
   - Output: stock_count, in_stock (boolean)
   
   GET /api/product/search
   - Input: query
   - Output: array of matching products

AJAX Implementation Guidelines:
- Use fetch() API or jQuery $.ajax()
- Return JSON responses
- Include HTTP status codes (200 for success, 400 for errors)
- Handle errors gracefully with SweetAlert2
- Show loading spinners during requests
- Validate inputs on server-side
- Use CSRF tokens for POST requests
- Rate limit API endpoints (optional but recommended)

========================================
🧩 TECHNICAL REQUIREMENTS - DETAILED SPECIFICATIONS
========================================

Server Requirements:
- PHP 8.2 or higher
- MariaDB 10.6 or higher
- Apache with mod_rewrite enabled
- PDO extension enabled
- JSON extension enabled
- cURL extension enabled
- MBString extension enabled

Database Requirements:
- Use PDO for all database connections
- Prepared statements ONLY (never concatenate queries)
- Use UTF-8 character set (utf8mb4)
- Enable foreign key constraints
- Use InnoDB engine for all tables
- Create indexes on frequently queried columns

PHP Coding Standards:
- Use PSR-12 coding style
- Use namespaces for classes
- Use type hints where possible
- Use strict_types declare in files
- Error reporting: E_ALL in development, 0 in production
- Use try-catch for database operations
- Log errors to file (not display to users)

URL Routing:
- Clean URLs using .htaccess
- Router class to handle routes
- Route format: /controller/action/params
- Example routes:
  * / → HomeController::index()
  * /products → ProductController::index()
  * /product/123 → ProductController::view(123)
  * /cart → CartController::index()
  * /checkout → CheckoutController::process()
  * /admin → AdminController::dashboard()
  * /admin/products → AdminProductController::index()

.htaccess Configuration:
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

No File Uploads:
- All images must be external URLs
- Support: Imgur, Cloudinary, direct image links
- Validate URL format
- Check if image is accessible (optional)
- Use placeholder images if URLs are invalid

========================================
📚 EXTERNAL LIBRARIES - DETAILED SPECIFICATIONS
========================================

CDN Links (use latest stable versions):

1. Bootstrap 5.3.x:
   CSS: https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css
   JS: https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js
   Icons: https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.1/font/bootstrap-icons.css

2. SweetAlert2:
   CSS/JS: https://cdn.jsdelivr.net/npm/sweetalert2@11/dist/sweetalert2.all.min.js

3. QuillJS:
   CSS: https://cdn.quilljs.com/1.3.6/quill.snow.css
   JS: https://cdn.quilljs.com/1.3.6/quill.min.js

4. Google Fonts:
   Kanit: https://fonts.googleapis.com/css2?family=Kanit:wght@400;500;600;700&display=swap
   Sarabun: https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600&display=swap

5. jQuery (optional, for AJAX):
   https://code.jquery.com/jquery-3.7.1.min.js

Library Usage Guidelines:
- Load CSS in <head>
- Load JS at end of <body>
- Use defer attribute for non-critical JS
- Minimize external dependencies
- Consider bundling for production (optional)

========================================
🌐 LANGUAGE & LOCALIZATION
========================================

Language: Thai (TH)

All UI text must be in Thai language:
- Navigation menu items
- Form labels and placeholders
- Button text
- Error and success messages
- Page titles and headings
- Product descriptions (admin can enter in any language)
- Admin panel interface

Common Thai Translations:
- Home: หน้าแรก
- Products: สินค้า
- Cart: ตะกร้าสินค้า
- Checkout: ชำระเงิน
- Login: เข้าสู่ระบบ
- Register: สมัครสมาชิก
- Logout: ออกจากระบบ
- Wallet: วอลเล็ต
- Top-up: เติมเงิน
- Orders: คำสั่งซื้อ
- Contact: ติดต่อเรา
- Admin: ผู้ดูแลระบบ
- Price: ราคา
- Stock: สต็อก
- Add to Cart: เพิ่มลงตะกร้า
- Buy Now: ซื้อทันที
- Search: ค้นหา
- Category: หมวดหมู่
- Success: สำเร็จ
- Error: ผิดพลาด
- Loading: กำลังโหลด...

========================================
📁 PROJECT STRUCTURE - DETAILED FILE TREE
========================================

Complete directory structure:

/
├── public/
│   ├── index.php (entry point, loads bootstrap)
│   ├── .htaccess (URL rewriting)
│   ├── assets/
│   │   ├── css/
│   │   │   └── style.css (custom styles)
│   │   ├── js/
│   │   │   ├── main.js (frontend JS)
│   │   │   ├── cart.js (cart functionality)
│   │   │   ├── wallet.js (wallet functionality)
│   │   │   └── admin.js (admin panel JS)
│   │   └── images/ (placeholder images)
│   └── api/ (AJAX endpoints)
│       ├── cart.php
│       ├── wallet.php
│       ├── coupon.php
│       ├── order.php
│       └── product.php
├── app/
│   ├── config/
│   │   ├── database.php (PDO connection)
│   │   ├── config.php (general settings)
│   │   └── constants.php (constants)
│   ├── core/
│   │   ├── Router.php (routing class)
│   │   ├── Controller.php (base controller)
│   │   ├── Model.php (base model)
│   │   ├── View.php (view rendering)
│   │   ├── Session.php (session management)
│   │   └── Security.php (CSRF, XSS protection)
│   ├── controllers/
│   │   ├── HomeController.php
│   │   ├── AuthController.php
│   │   ├── ProductController.php
│   │   ├── CartController.php
│   │   ├── CheckoutController.php
│   │   ├── OrderController.php
│   │   ├── WalletController.php
│   │   └── ContactController.php
│   ├── models/
│   │   ├── User.php
│   │   ├── Product.php
│   │   ├── Category.php
│   │   ├── Order.php
│   │   ├── Coupon.php
│   │   ├── WalletTransaction.php
│   │   └── RedeemCode.php
│   ├── views/
│   │   ├── layouts/
│   │   │   ├── main.php (frontend layout)
│   │   │   └── admin.php (admin layout)
│   │   ├── home/
│   │   ├── auth/
│   │   ├── products/
│   │   ├── cart/
│   │   ├── orders/
│   │   ├── wallet/
│   │   └── contact/
│   └── admin/
│       ├── controllers/
│       │   ├── AdminController.php
│       │   ├── AdminUserController.php
│       │   ├── AdminProductController.php
│       │   ├── AdminCategoryController.php
│       │   ├── AdminOrderController.php
│       │   ├── AdminCouponController.php
│       │   ├── AdminRedeemController.php
│       │   ├── AdminWalletController.php
│       │   └── AdminSettingsController.php
│       └── views/
│           ├── dashboard/
│           ├── users/
│           ├── products/
│           ├── categories/
│           ├── orders/
│           ├── coupons/
│           ├── redeem-codes/
│           ├── wallet/
│           └── settings/
├── database/
│   ├── schema.sql (empty database structure)
│   └── seed.sql (sample data)
├── logs/
│   └── error.log (error logging)
├── .htaccess (root redirect to public)
├── README.md (installation guide)
└── LICENSE (MIT License)

File Naming Conventions:
- Controllers: PascalCase (HomeController.php)
- Models: PascalCase (User.php)
- Views: lowercase with dashes (product-detail.php)
- CSS/JS: lowercase with dashes (style.css, main.js)
- Classes: PascalCase
- Methods: camelCase
- Variables: camelCase
- Constants: UPPER_SNAKE_CASE

========================================
🗄️ DATABASE SCHEMA - DETAILED SPECIFICATIONS
========================================

Complete Database Schema:

```sql
-- Database: digital_marketplace
-- Character Set: utf8mb4
-- Collation: utf8mb4_unicode_ci

-- Users Table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(20) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    role ENUM('user', 'admin') DEFAULT 'user',
    wallet_balance DECIMAL(10,2) DEFAULT 0.00,
    is_banned BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_username (username),
    INDEX idx_email (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Categories Table
CREATE TABLE categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    icon VARCHAR(255),
    sort_order INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_slug (slug),
    INDEX idx_active (is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Products Table
CREATE TABLE products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    category_id INT,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    images JSON,
    youtube_url VARCHAR(255),
    delivery_method ENUM('manual', 'auto') DEFAULT 'manual',
    delivery_data TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    is_featured BOOLEAN DEFAULT FALSE,
    views INT DEFAULT 0,
    sales_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL,
    INDEX idx_slug (slug),
    INDEX idx_category (category_id),
    INDEX idx_active (is_active),
    INDEX idx_featured (is_featured)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Orders Table
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(20) UNIQUE NOT NULL,
    user_id INT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) DEFAULT 0.00,
    coupon_code VARCHAR(50),
    final_amount DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'completed', 'cancelled', 'refunded') DEFAULT 'pending',
    delivery_status ENUM('pending_delivery', 'delivered') DEFAULT 'pending_delivery',
    payment_method VARCHAR(50) DEFAULT 'wallet',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_order_number (order_number),
    INDEX idx_user (user_id),
    INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Order Items Table
CREATE TABLE order_items (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT,
    product_name VARCHAR(255) NOT NULL,
    product_price DECIMAL(10,2) NOT NULL,
    quantity INT DEFAULT 1,
    subtotal DECIMAL(10,2) NOT NULL,
    delivery_data TEXT,
    delivery_status ENUM('pending_delivery', 'delivered') DEFAULT 'pending_delivery',
    delivery_data_sent TEXT,
    delivered_at TIMESTAMP NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE SET NULL,
    INDEX idx_order (order_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Wallet Transactions Table
CREATE TABLE wallet_transactions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    type ENUM('topup', 'purchase', 'refund', 'admin_adjust') NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    balance_before DECIMAL(10,2) NOT NULL,
    balance_after DECIMAL(10,2) NOT NULL,
    description VARCHAR(255),
    reference_id VARCHAR(50),
    payment_method VARCHAR(50),
    status ENUM('pending', 'completed', 'failed', 'cancelled') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    INDEX idx_user (user_id),
    INDEX idx_type (type),
    INDEX idx_reference (reference_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Coupons Table
CREATE TABLE coupons (
    id INT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(50) UNIQUE NOT NULL,
    discount_percent DECIMAL(5,2) NOT NULL,
    max_uses INT NULL,
    used_count INT DEFAULT 0,
    valid_from TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    valid_until TIMESTAMP NULL,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_code (code),
    INDEX idx_active (is_active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Redeem Codes Table
CREATE TABLE redeem_codes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(20) UNIQUE NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    is_used BOOLEAN DEFAULT FALSE,
    used_by INT NULL,
    used_at TIMESTAMP NULL,
    created_by INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NULL,
    FOREIGN KEY (used_by) REFERENCES users(id) ON DELETE SET NULL,
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_code (code),
    INDEX idx_used (is_used)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Homepage Settings Table
CREATE TABLE homepage_settings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    hero_button_text VARCHAR(100) DEFAULT 'ซื้อสินค้าเลย',
    hero_button_url VARCHAR(255) DEFAULT '/products',
    hero_images JSON,
    stats_enabled BOOLEAN DEFAULT TRUE,
    new_products_count INT DEFAULT 8
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Contact Settings Table
CREATE TABLE contact_settings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) DEFAULT 'ติดต่อเรา',
    content TEXT,
    phone VARCHAR(50),
    email VARCHAR(100),
    facebook_url VARCHAR(255),
    line_url VARCHAR(255),
    address TEXT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Discord Settings Table
CREATE TABLE discord_settings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    webhook_url VARCHAR(500),
    enabled BOOLEAN DEFAULT FALSE,
    notify_topup BOOLEAN DEFAULT TRUE,
    notify_order BOOLEAN DEFAULT TRUE,
    bot_name VARCHAR(100) DEFAULT 'Store Bot',
    bot_avatar_url VARCHAR(255)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Website Settings Table
CREATE TABLE website_settings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    site_name VARCHAR(100) DEFAULT 'Digital Store',
    site_description VARCHAR(255),
    site_logo_url VARCHAR(255),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Payment API Settings Table
CREATE TABLE payment_api_settings (
    id INT AUTO_INCREMENT PRIMARY KEY,
    promptpay_enabled BOOLEAN DEFAULT FALSE,
    promptpay_tmweasy_user VARCHAR(100),
    promptpay_tmweasy_password VARCHAR(100),
    promptpay_tmweasy_api_key VARCHAR(100),
    promptpay_con_id VARCHAR(100),
    promptpay_number VARCHAR(50),
    promptpay_type VARCHAR(10) DEFAULT '01',
    promptpay_name VARCHAR(100),
    truewallet_enabled BOOLEAN DEFAULT FALSE,
    truewallet_tmweasy_user VARCHAR(100),
    truewallet_tmweasy_password VARCHAR(100),
    truewallet_mobile VARCHAR(20)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

Seed Data Requirements:
- 1 admin user (username: admin, password: admin123)
- 3-5 sample categories
- 10-15 sample products
- 2-3 sample coupons
- 5-10 sample redeem codes
- Default homepage settings
- Default contact settings
- Default website settings

========================================
📦 OUTPUT REQUIREMENTS - WHAT TO GENERATE
========================================

Generate COMPLETE, FUNCTIONAL CODE:

1. Core System Files:
   - /public/index.php (entry point, bootstrap)
   - /public/.htaccess (URL rewriting)
   - /.htaccess (root redirect)
   - /app/config/database.php (PDO connection class)
   - /app/config/config.php (configuration constants)
   - /app/config/constants.php (application constants)

2. Core Classes:
   - /app/core/Router.php (routing logic)
   - /app/core/Controller.php (base controller with view loading)
   - /app/core/Model.php (base model with database methods)
   - /app/core/View.php (view rendering)
   - /app/core/Session.php (session management, flash messages)
   - /app/core/Security.php (CSRF token generation/validation, XSS protection)

3. Frontend Controllers:
   - HomeController.php (homepage)
   - AuthController.php (register, login, logout, change password)
   - ProductController.php (listing, detail, search)
   - CartController.php (add, remove, update, view)
   - CheckoutController.php (process checkout)
   - OrderController.php (order history, order detail)
   - WalletController.php (top-up methods, transaction history)
   - ContactController.php (contact page)

4. Frontend Models:
   - User.php (authentication, wallet operations)
   - Product.php (CRUD, stock management)
   - Category.php (CRUD)
   - Order.php (CRUD, order processing)
   - Coupon.php (validation, usage tracking)
   - WalletTransaction.php (transaction logging)
   - RedeemCode.php (generation, validation)

5. Frontend Views:
   - /app/views/layouts/main.php (navbar, footer, head)
   - /app/views/home/index.php
   - /app/views/auth/register.php
   - /app/views/auth/login.php
   - /app/views/auth/change-password.php
   - /app/views/products/index.php (listing)
   - /app/views/products/detail.php (single product)
   - /app/views/cart/index.php
   - /app/views/checkout/index.php
   - /app/views/checkout/success.php
   - /app/views/orders/index.php (history)
   - /app/views/orders/detail.php
   - /app/views/wallet/index.php (balance, top-up methods)
   - /app/views/wallet/topup-promptpay.php
   - /app/views/wallet/topup-truewallet.php
   - /app/views/wallet/transactions.php
   - /app/views/contact/index.php

6. Admin Controllers:
   - AdminController.php (dashboard, authentication)
   - AdminUserController.php (user management)
   - AdminProductController.php (product CRUD)
   - AdminCategoryController.php (category CRUD)
   - AdminOrderController.php (order management, delivery)
   - AdminCouponController.php (coupon CRUD)
   - AdminRedeemController.php (redeem code generation/management)
   - AdminWalletController.php (wallet transaction viewing, manual adjustments)
   - AdminSettingsController.php (website, payment API, discord, contact settings)

7. Admin Views:
   - /app/admin/views/layouts/admin.php (sidebar, admin layout)
   - /app/admin/views/dashboard/index.php
   - /app/admin/views/users/index.php, create.php, edit.php
   - /app/admin/views/products/index.php, create.php, edit.php
   - /app/admin/views/categories/index.php, create.php, edit.php
   - /app/admin/views/orders/index.php, detail.php
   - /app/admin/views/coupons/index.php, create.php, edit.php
   - /app/admin/views/redeem-codes/index.php, generate.php
   - /app/admin/views/wallet/index.php
   - /app/admin/views/settings/website.php
   - /app/admin/views/settings/payment-api.php
   - /app/admin/views/settings/discord.php
   - /app/admin/views/settings/contact.php
   - /app/admin/views/auth/login.php

8. AJAX API Endpoints:
   - /public/api/cart.php (add, remove, update, clear)
   - /public/api/wallet.php (balance, topup methods, redeem)
   - /public/api/coupon.php (apply, remove)
   - /public/api/order.php (status check)
   - /public/api/product.php (stock check, search)

9. Frontend Assets:
   - /public/assets/css/style.css (custom dark theme, lime green accents)
   - /public/assets/js/main.js (global JS, SweetAlert2 config)
   - /public/assets/js/cart.js (cart AJAX operations)
   - /public/assets/js/wallet.js (wallet AJAX operations)
   - /public/assets/js/admin.js (admin panel JS)

10. Database Files:
    - /database/schema.sql (complete database structure)
    - /database/seed.sql (sample data)

11. Documentation:
    - README.md with:
      * Project description
      * Requirements (PHP, MariaDB, extensions)
      * Installation steps
      * Configuration guide
      * Directory structure explanation
      * How to run the application
      * Troubleshooting common issues
    - INSTALLATION.md (detailed installation guide)
    - API_DOCUMENTATION.md (AJAX API endpoints documentation)

Code Quality Requirements:
- NO syntax errors
- All PHP files must be syntactically correct
- Proper error handling with try-catch
- Comments for complex logic
- Consistent coding style (PSR-12)
- Security best practices implemented
- Database queries use prepared statements
- CSRF protection on all forms
- XSS protection on all outputs

========================================
❗ CRITICAL RULES - MUST FOLLOW
========================================

1. CSS Rules:
   - NO inline CSS styles in HTML files
   - ALL styles must be in /public/assets/css/style.css
   - Use CSS variables for colors and fonts
   - Bootstrap classes allowed, custom styles in external CSS

2. JavaScript Alerts:
   - NO native alert(), confirm(), prompt()
   - ALL alerts must use SweetAlert2 ONLY
   - Configure SweetAlert2 in main.js
   - Use for success, error, confirmation messages

3. Email System:
   - NO email verification system
   - NO password reset via email
   - NO email notifications (use Discord webhook instead)

4. Wallet System:
   - Wallet balance MUST be queried from database users.wallet_balance
   - NEVER store wallet balance in session
   - Refresh balance via AJAX after every transaction
   - Log all wallet transactions

5. Code Quality:
   - Fully functional, production-ready code
   - NO pseudo-code or placeholders
   - All features must be implemented
   - No "TODO" comments - implement everything
   - Code must be immediately runnable

6. Deployment:
   - Ready to deploy on shared hosting (cPanel, etc.)
   - No special server configuration required
   - Standard Apache + PHP + MariaDB setup
   - No Docker or special dependencies

7. Security:
   - ALL database queries MUST use prepared statements
   - NO SQL injection vulnerabilities
   - CSRF protection on ALL forms
   - XSS protection on ALL outputs
   - Password hashing with password_hash()

8. File Organization:
   - Follow the exact directory structure specified
   - Use correct naming conventions
   - Separate frontend and admin clearly
   - Keep API endpoints in /public/api/

9. Language:
   - ALL UI text in Thai language
   - Admin panel in Thai
   - Error messages in Thai
   - Success messages in Thai

10. Database:
    - Use the exact schema provided
    - Include all tables with correct columns
    - Set proper foreign key constraints
    - Use InnoDB engine
    - Use utf8mb4 character set

========================================
🚀 GENERATION INSTRUCTIONS
========================================

Generate the complete application following ALL specifications above.

Start with:
1. Database schema (schema.sql)
2. Core classes (Router, Controller, Model, etc.)
3. Configuration files
4. Frontend controllers and views
5. Admin controllers and views
6. AJAX API endpoints
7. Assets (CSS, JS)
8. Documentation (README.md)

Ensure:
- Every feature is fully implemented
- Code is error-free and follows best practices
- Security measures are in place
- Thai language is used throughout
- The application is ready to deploy

Generate the complete codebase now, file by file, with complete implementations.