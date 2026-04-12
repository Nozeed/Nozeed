# AI PROMPT: Build Complete PHP Game Store Website

You are an expert PHP developer. Build a complete e-commerce website for selling digital products following ALL specifications below.

---

## REQUIREMENTS SUMMARY

- **Type**: PHP 8+ E-commerce for Digital Products (Netflix, Spotify, Steam, etc.)
- **Database**: MariaDB 10.3+ with PDO prepared statements
- **Language**: ALL content in Thai
- **Theme**: Dark mode with Lime Green (#80ef25) primary
- **Frontend**: Bootstrap 5 + jQuery + SweetAlert2 (MANDATORY for all alerts)
- **Fonts**: Kanit (headings), Sarabun (body)
- **Rich Text**: QuillJS for admin product descriptions
- **Security**: Cloudflare Turnstile for login/register
- **No File Uploads**: External image URLs only
- **URLs**: Clean URLs with .htaccess router
- **AJAX**: Real-time updates for cart, credit, search, orders

---

## STEP 1: DATABASE

### database.sql
```sql
CREATE DATABASE IF NOT EXISTS gamestore CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE gamestore;

CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    credit DECIMAL(10,2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP NULL,
    status ENUM('active', 'banned') DEFAULT 'active'
);

CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    icon VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id INT PRIMARY KEY AUTO_INCREMENT,
    category_id INT NOT NULL,
    name VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    image_url VARCHAR(255),
    gallery JSON,
    video_url VARCHAR(255),
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status ENUM('pending', 'processing', 'completed', 'cancelled') DEFAULT 'pending',
    delivery_type ENUM('manual', 'auto') NOT NULL,
    delivery_data TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE order_items (
    id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

CREATE TABLE cart (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (product_id) REFERENCES products(id),
    UNIQUE KEY unique_cart_item (user_id, product_id)
);

CREATE TABLE coupons (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(20) UNIQUE NOT NULL,
    discount_percent DECIMAL(5,2) NOT NULL,
    min_amount DECIMAL(10,2) DEFAULT 0.00,
    max_uses INT DEFAULT NULL,
    used_count INT DEFAULT 0,
    expires_at TIMESTAMP NULL,
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE redeem_codes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(20) UNIQUE NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    used_by INT NULL,
    used_at TIMESTAMP NULL,
    status ENUM('active', 'used') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (used_by) REFERENCES users(id)
);

CREATE TABLE topup_history (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    method ENUM('promptpay', 'truewallet', 'redeem') NOT NULL,
    transaction_id VARCHAR(100),
    status ENUM('pending', 'completed', 'failed') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE settings (
    id INT PRIMARY KEY AUTO_INCREMENT,
    setting_key VARCHAR(100) UNIQUE NOT NULL,
    setting_value TEXT,
    description TEXT,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE admin_logs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    admin_id INT NOT NULL,
    action VARCHAR(100) NOT NULL,
    details TEXT,
    ip_address VARCHAR(45),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (admin_id) REFERENCES users(id)
);
```

### seed.sql
```sql
INSERT INTO categories (name, slug, description) VALUES
('Netflix', 'netflix', 'บัญชี Netflix'),
('Spotify', 'spotify', 'บัญชี Spotify Premium'),
('Steam', 'steam', 'เกม Steam'),
('PlayStation', 'playstation', 'บัญชี PSN'),
('Xbox', 'xbox', 'Xbox Live'),
('เกมมือถือ', 'mobile-games', 'บัญชีเกมมือถือ');

INSERT INTO settings (setting_key, setting_value, description) VALUES
('site_name', 'GameStore', 'ชื่อเว็บ'),
('site_logo', '', 'โลโก้'),
('site_description', 'ร้านค้าเกมดิจิทัล', 'คำอธิบาย'),
('contact_info', 'Email: support@gamestore.com', 'ติดต่อ'),
('hero_title', 'ยินดีต้อนรับ', 'หัวข้อ Hero'),
('hero_subtitle', 'ร้านค้าเกมดิจิทัลที่ดีที่สุด', 'คำอธิบาย Hero'),
('hero_button_text', 'เลือกซื้อสินค้า', 'ปุ่ม Hero'),
('hero_images', '["https://via.placeholder.com/800x500/80ef25/0d0d0d?text=GameStore"]', 'รูป Hero'),
('promptpay_enabled', 'true', 'เปิด PromptPay'),
('promptpay_api_url', 'https://tmwallet.thaighost.net/apiwallet.php', 'API URL'),
('promptpay_username', '', 'Username'),
('promptpay_password', '', 'Password'),
('promptpay_number', '', 'เบอร์พร้อมเพย์'),
('promptpay_name', '', 'ชื่อบัญชี'),
('promptpay_type', '01', 'ประเภท'),
('truewallet_enabled', 'true', 'เปิด TrueWallet'),
('truewallet_api_url', 'https://tmwallet.thaighost.net/apiwallet.php', 'API URL'),
('truewallet_username', '', 'Username'),
('truewallet_password', '', 'Password'),
('truewallet_mobile', '', 'เบอร์มือถือ'),
('turnstile_enabled', 'false', 'เปิด Turnstile'),
('turnstile_site_key', '', 'Site Key'),
('turnstile_secret_key', '', 'Secret Key'),
('discord_webhook_url', '', 'Discord Webhook'),
('discord_new_order_enabled', 'true', 'แจ้งเตือนคำสั่งซื้อ'),
('discord_payment_enabled', 'true', 'แจ้งเตือนเงินเข้า');
```

---

## STEP 2: CONFIG FILES

### config/database.php
```php
<?php
session_start();

define('DB_HOST', 'localhost');
define('DB_NAME', 'gamestore');
define('DB_USER', 'root');
define('DB_PASS', '');
define('DB_CHARSET', 'utf8mb4');

try {
    $pdo = new PDO("mysql:host=" . DB_HOST . ";dbname=" . DB_NAME . ";charset=" . DB_CHARSET, DB_USER, DB_PASS);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
} catch (PDOException $e) {
    die("การเชื่อมต่อฐานข้อมูลล้มเหลว: " . $e->getMessage());
}

function getSetting($key, $default = '') {
    global $pdo;
    try {
        $stmt = $pdo->prepare("SELECT setting_value FROM settings WHERE setting_key = ?");
        $stmt->execute([$key]);
        $result = $stmt->fetch();
        return $result ? $result['setting_value'] : $default;
    } catch (PDOException $e) {
        return $default;
    }
}

function formatPrice($price) {
    return '฿' . number_format($price, 2);
}

function isLoggedIn() {
    return isset($_SESSION['user_id']);
}

function getUserCredit($userId) {
    global $pdo;
    try {
        $stmt = $pdo->prepare("SELECT credit FROM users WHERE id = ?");
        $stmt->execute([$userId]);
        $result = $stmt->fetch();
        return $result ? $result['credit'] : 0;
    } catch (PDOException $e) {
        return 0;
    }
}

function updateUserCredit($userId, $amount) {
    global $pdo;
    try {
        $stmt = $pdo->prepare("UPDATE users SET credit = credit + ? WHERE id = ?");
        return $stmt->execute([$amount, $userId]);
    } catch (PDOException $e) {
        return false;
    }
}

function generateOrderNumber() {
    return 'ORD' . date('Ymd') . str_pad(mt_rand(1, 9999), 4, '0', STR_PAD_LEFT);
}

function generateRedeemCode() {
    return 'REDEEM-' . strtoupper(substr(str_shuffle('0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'), 0, 6));
}

function getCartCount() {
    if (!isLoggedIn()) return 0;
    global $pdo;
    try {
        $stmt = $pdo->prepare("SELECT SUM(quantity) FROM cart WHERE user_id = ?");
        $stmt->execute([$_SESSION['user_id']]);
        return (int)($stmt->fetchColumn() ?: 0);
    } catch (PDOException $e) {
        return 0;
    }
}

function sendDiscordWebhook($message) {
    $url = getSetting('discord_webhook_url');
    if (!$url) return false;
    
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode(['content' => $message, 'username' => 'GameStore Bot']));
    curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    $result = curl_exec($ch);
    curl_close($ch);
    return $result;
}

function connect_api($url) {
    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0);
    curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 0);
    curl_setopt($ch, CURLOPT_USERAGENT, "Mozilla/5.0");
    curl_setopt($ch, CURLOPT_FOLLOWLOCATION, 1);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    $response = curl_exec($ch);
    curl_close($ch);
    return $response;
}

function get_client_ip() {
    if (!empty($_SERVER['HTTP_CLIENT_IP'])) return $_SERVER['HTTP_CLIENT_IP'];
    if (!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) return $_SERVER['HTTP_X_FORWARDED_FOR'];
    return $_SERVER['REMOTE_ADDR'];
}
?>
```

---

## STEP 3: CORE FILES

### .htaccess
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^([^/]+)/?$ index.php?page=$1 [L,QSA]
RewriteRule ^([^/]+)/([^/]+)/?$ index.php?page=$1&id=$2 [L,QSA]

<IfModule mod_headers.c>
    Header always set X-Content-Type-Options nosniff
    Header always set X-Frame-Options DENY
    Header always set X-XSS-Protection "1; mode=block"
</IfModule>

php_value upload_max_filesize 10M
php_value post_max_size 10M
```

### index.php (Router)
```php
<?php
require_once 'config/database.php';

$page = isset($_GET['page']) ? $_GET['page'] : 'home';
$id = isset($_GET['id']) ? $_GET['id'] : null;

$allowed = ['home', 'products', 'product', 'cart', 'checkout', 'login', 'register', 'profile', 'topup', 'orders', 'contact', 'logout'];

if (in_array($page, $allowed)) {
    require_once "pages/{$page}.php";
} else {
    require_once "pages/home.php";
}
?>
```

### includes/header.php
```php
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><?php echo getSetting('site_name', 'GameStore'); ?> - <?php echo isset($pageTitle) ? $pageTitle : 'ร้านค้าเกมดิจิทัล'; ?></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Kanit:wght@400;600;700&family=Sarabun:wght@400;500;600&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    <style>
        :root { --primary: #80ef25; --dark: #1a1a1a; --darker: #0d0d0d; }
        body { font-family: 'Sarabun', sans-serif; background: var(--dark); color: #fff; }
        h1, h2, h3, h4, h5, h6 { font-family: 'Kanit', sans-serif; font-weight: 600; }
        .btn-success { background: var(--primary); border-color: var(--primary); color: var(--darker); font-weight: 600; }
        .btn-success:hover { background: #6bc420; border-color: #6bc420; color: var(--darker); }
        .card { background: #2a2a2a; border: 1px solid #404040; transition: all 0.3s; }
        .card:hover { transform: translateY(-5px); box-shadow: 0 10px 30px rgba(128, 239, 37, 0.3); }
        .navbar { background: var(--darker) !important; }
        .text-success { color: var(--primary) !important; }
        .bg-success { background: var(--primary) !important; color: var(--darker) !important; }
        .form-control, .form-select { background: #2a2a2a; border: 1px solid #404040; color: #fff; }
        .form-control:focus { background: #2a2a2a; border-color: var(--primary); color: #fff; box-shadow: 0 0 0 0.25rem rgba(128, 239, 37, 0.25); }
        .hero-section { background: linear-gradient(135deg, var(--darker) 0%, var(--dark) 100%); min-height: 500px; display: flex; align-items: center; }
        .product-card img { height: 200px; object-fit: cover; transition: transform 0.3s; }
        .product-card:hover img { transform: scale(1.05); }
        .footer { background: var(--darker); }
    </style>
    <?php if (getSetting('turnstile_enabled') == 'true'): ?>
    <script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer></script>
    <?php endif; ?>
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark sticky-top">
        <div class="container">
            <a class="navbar-brand" href="index.php">
                <?php $logo = getSetting('site_logo');
                if ($logo && filter_var($logo, FILTER_VALIDATE_URL)) {
                    echo '<img src="' . htmlspecialchars($logo) . '" height="40">';
                } else {
                    echo '<span class="text-success">' . getSetting('site_name', 'GameStore') . '</span>';
                } ?>
            </a>
            <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navbarNav">
                <ul class="navbar-nav me-auto">
                    <li class="nav-item"><a class="nav-link" href="index.php">หน้าแรก</a></li>
                    <li class="nav-item"><a class="nav-link" href="products.php">สินค้า</a></li>
                    <li class="nav-item"><a class="nav-link" href="contact.php">ติดต่อ</a></li>
                </ul>
                <ul class="navbar-nav">
                    <?php if (isLoggedIn()): ?>
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"><?php echo htmlspecialchars($_SESSION['username']); ?></a>
                        <ul class="dropdown-menu dropdown-menu-dark">
                            <li><a class="dropdown-item" href="profile.php">โปรไฟล์</a></li>
                            <li><a class="dropdown-item" href="orders.php">ประวัติการสั่งซื้อ</a></li>
                            <li><a class="dropdown-item" href="topup.php">เติมเงิน</a></li>
                            <li><hr class="dropdown-divider"></li>
                            <li><a class="dropdown-item" href="logout.php">ออกจากระบบ</a></li>
                        </ul>
                    </li>
                    <?php else: ?>
                    <li class="nav-item"><a class="nav-link" href="login.php">เข้าสู่ระบบ</a></li>
                    <li class="nav-item"><a class="nav-link" href="register.php">สมัครสมาชิก</a></li>
                    <?php endif; ?>
                    <li class="nav-item">
                        <a class="nav-link position-relative" href="cart.php">
                            <i class="bi bi-cart3"></i>
                            <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-success cart-count"><?php echo getCartCount(); ?></span>
                        </a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    <?php if (isLoggedIn()): ?>
    <div class="bg-success text-dark text-center py-2">
        <small><strong>เครดิต: <span id="user-credit"><?php echo formatPrice(getUserCredit($_SESSION['user_id'])); ?></span></strong></small>
    </div>
    <?php endif; ?>
```

### includes/footer.php
```php
    <footer class="footer text-white py-4 mt-5">
        <div class="container">
            <div class="row">
                <div class="col-md-4">
                    <h5 class="text-success"><?php echo getSetting('site_name', 'GameStore'); ?></h5>
                    <p><?php echo getSetting('site_description', 'ร้านค้าเกมดิจิทัล'); ?></p>
                </div>
                <div class="col-md-4">
                    <h5 class="text-success">ลิงก์ด่วน</h5>
                    <ul class="list-unstyled">
                        <li><a href="index.php" class="text-white text-decoration-none">หน้าแรก</a></li>
                        <li><a href="products.php" class="text-white text-decoration-none">สินค้า</a></li>
                        <li><a href="contact.php" class="text-white text-decoration-none">ติดต่อ</a></li>
                    </ul>
                </div>
                <div class="col-md-4">
                    <h5 class="text-success">ติดต่อเรา</h5>
                    <p><?php echo nl2br(htmlspecialchars(getSetting('contact_info', 'ติดต่อเรา'))); ?></p>
                </div>
            </div>
            <hr class="border-secondary">
            <div class="text-center">
                <p>&copy; <?php echo date('Y'); ?> <?php echo getSetting('site_name', 'GameStore'); ?>. สงวนลิขสิทธิ์</p>
            </div>
        </div>
    </footer>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
    $(document).ready(function() {
        // AJAX: Add to cart
        $('.add-to-cart').click(function(e) {
            e.preventDefault();
            const id = $(this).data('product-id');
            const qty = $(this).data('quantity') || 1;
            $.ajax({
                url: 'api/cart.php',
                method: 'POST',
                data: { action: 'add', product_id: id, quantity: qty },
                dataType: 'json',
                success: function(r) {
                    if (r.success) {
                        Swal.fire({ title: 'สำเร็จ!', text: 'เพิ่มสินค้าในตะกร้าแล้ว', icon: 'success', confirmButtonText: 'ตกลง' });
                        updateCartCount();
                    } else {
                        Swal.fire({ title: 'ผิดพลาด!', text: r.message, icon: 'error', confirmButtonText: 'ตกลง' });
                    }
                }
            });
        });
        
        // AJAX: Cart count
        function updateCartCount() {
            $.ajax({
                url: 'api/cart.php',
                method: 'GET',
                data: { action: 'count' },
                dataType: 'json',
                success: function(r) {
                    $('.cart-count').text(r.count);
                }
            });
        }
        
        // AJAX: Real-time credit (30 sec)
        <?php if (isLoggedIn()): ?>
        setInterval(function() {
            $.ajax({
                url: 'api/auth.php',
                method: 'GET',
                data: { action: 'get_credit' },
                dataType: 'json',
                success: function(r) {
                    if (r.success) $('#user-credit').text(r.credit_formatted);
                }
            });
        }, 30000);
        <?php endif; ?>
        
        // AJAX: Apply coupon
        $('#apply-coupon').click(function() {
            const code = $('#coupon-code').val();
            if (!code) { Swal.fire({ title: 'ผิดพลาด!', text: 'กรุณากรอกรหัสคูปอง', icon: 'error', confirmButtonText: 'ตกลง' }); return; }
            $.ajax({
                url: 'api/checkout.php',
                method: 'POST',
                data: { action: 'apply_coupon', coupon_code: code },
                dataType: 'json',
                success: function(r) {
                    if (r.success) {
                        Swal.fire({ title: 'สำเร็จ!', text: 'ใช้คูปองสำเร็จ', icon: 'success', confirmButtonText: 'ตกลง' }).then(() => location.reload());
                    } else {
                        Swal.fire({ title: 'ผิดพลาด!', text: r.message, icon: 'error', confirmButtonText: 'ตกลง' });
                    }
                }
            });
        });
        
        // AJAX: Remove from cart
        $('.remove-from-cart').click(function(e) {
            e.preventDefault();
            const id = $(this).data('product-id');
            Swal.fire({
                title: 'ยืนยัน?', text: 'ลบสินค้าออกจากตะกร้า?', icon: 'warning', showCancelButton: true,
                confirmButtonText: 'ใช่', cancelButtonText: 'ยกเลิก'
            }).then((result) => {
                if (result.isConfirmed) {
                    $.ajax({
                        url: 'api/cart.php', method: 'POST',
                        data: { action: 'remove', product_id: id },
                        dataType: 'json', success: function(r) { if (r.success) location.reload(); }
                    });
                }
            });
        });
    });
    </script>
</body>
</html>
```

---

## STEP 4: FRONTEND PAGES

### pages/home.php
```php
<?php
$pageTitle = 'หน้าแรก';
require_once 'includes/header.php';

$heroImages = json_decode(getSetting('hero_images', '[]'), true);
if (empty($heroImages)) $heroImages = ['https://via.placeholder.com/800x500/80ef25/0d0d0d?text=GameStore'];

try {
    $totalProducts = $pdo->query("SELECT COUNT(*) FROM products WHERE status = 'active'")->fetchColumn();
    $totalUsers = $pdo->query("SELECT COUNT(*) FROM users WHERE status = 'active'")->fetchColumn();
    $totalOrders = $pdo->query("SELECT COUNT(*) FROM orders WHERE status = 'completed'")->fetchColumn();
    $totalRevenue = $pdo->query("SELECT SUM(total_amount) FROM orders WHERE status = 'completed'")->fetchColumn() ?: 0;
    $newProducts = $pdo->query("SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE p.status = 'active' ORDER BY p.created_at DESC LIMIT 8")->fetchAll();
} catch (PDOException $e) {
    $totalProducts = $totalUsers = $totalOrders = 0; $totalRevenue = 0; $newProducts = [];
}
?>
<section class="hero-section">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6">
                <h1 class="display-4 fw-bold text-success mb-4"><?php echo getSetting('hero_title', 'ยินดีต้อนรับสู่ GameStore'); ?></h1>
                <p class="lead mb-4"><?php echo getSetting('hero_subtitle', 'ร้านค้าเกมดิจิทัลที่ดีที่สุด'); ?></p>
                <a href="products.php" class="btn btn-success btn-lg"><?php echo getSetting('hero_button_text', 'เลือกซื้อสินค้า'); ?></a>
            </div>
            <div class="col-lg-6">
                <?php if (count($heroImages) > 1): ?>
                <div id="heroCarousel" class="carousel slide" data-bs-ride="carousel">
                    <div class="carousel-inner">
                        <?php foreach ($heroImages as $i => $img): ?>
                        <div class="carousel-item <?php echo $i === 0 ? 'active' : ''; ?>"><img src="<?php echo htmlspecialchars($img); ?>" class="d-block w-100 rounded"></div>
                        <?php endforeach; ?>
                    </div>
                    <button class="carousel-control-prev" type="button" data-bs-target="#heroCarousel" data-bs-slide="prev"><span class="carousel-control-prev-icon"></span></button>
                    <button class="carousel-control-next" type="button" data-bs-target="#heroCarousel" data-bs-slide="next"><span class="carousel-control-next-icon"></span></button>
                </div>
                <?php else: ?>
                <img src="<?php echo htmlspecialchars($heroImages[0]); ?>" class="img-fluid rounded">
                <?php endif; ?>
            </div>
        </div>
    </div>
</section>

<section class="py-5">
    <div class="container">
        <div class="row text-center">
            <div class="col-md-3 mb-4"><div class="card p-4"><h2 class="text-success"><?php echo $totalProducts; ?></h2><p>สินค้า</p></div></div>
            <div class="col-md-3 mb-4"><div class="card p-4"><h2 class="text-success"><?php echo $totalUsers; ?></h2><p>สมาชิก</p></div></div>
            <div class="col-md-3 mb-4"><div class="card p-4"><h2 class="text-success"><?php echo $totalOrders; ?></h2><p>คำสั่งซื้อ</p></div></div>
            <div class="col-md-3 mb-4"><div class="card p-4"><h2 class="text-success"><?php echo formatPrice($totalRevenue); ?></h2><p>ยอดขาย</p></div></div>
        </div>
    </div>
</section>

<section class="py-5">
    <div class="container">
        <h2 class="text-center mb-5 text-success">สินค้ามาใหม่</h2>
        <div class="row">
            <?php foreach ($newProducts as $p): ?>
            <div class="col-md-3 mb-4">
                <div class="card product-card h-100">
                    <img src="<?php echo $p['image_url'] ?: 'https://via.placeholder.com/300x200/2a2a2a/80ef25?text=No+Image'; ?>" class="card-img-top">
                    <div class="card-body">
                        <h5 class="card-title"><?php echo htmlspecialchars($p['name']); ?></h5>
                        <p class="text-muted small"><?php echo $p['category_name']; ?></p>
                        <h4 class="text-success"><?php echo formatPrice($p['price']); ?></h4>
                        <div class="d-flex justify-content-between align-items-center mt-3">
                            <span class="badge bg-<?php echo $p['stock'] > 0 ? 'success' : 'danger'; ?>"><?php echo $p['stock'] > 0 ? 'มีสินค้า (' . $p['stock'] . ')' : 'หมด'; ?></span>
                            <a href="product.php?id=<?php echo $p['id']; ?>" class="btn btn-sm btn-outline-success">ดูรายละเอียด</a>
                        </div>
                    </div>
                </div>
            </div>
            <?php endforeach; ?>
        </div>
        <div class="text-center mt-4"><a href="products.php" class="btn btn-success btn-lg">ดูทั้งหมด</a></div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/products.php
```php
<?php
$pageTitle = 'สินค้าทั้งหมด';
require_once 'includes/header.php';

$category = $_GET['category'] ?? '';
$search = $_GET['search'] ?? '';
$sort = $_GET['sort'] ?? 'newest';
$page = isset($_GET['page_num']) ? (int)$_GET['page_num'] : 1;
$perPage = 12; $offset = ($page - 1) * $perPage;

$params = []; $where = ["p.status = 'active'"];
if ($category) { $where[] = "c.slug = ?"; $params[] = $category; }
if ($search) { $where[] = "(p.name LIKE ? OR p.description LIKE ?)"; $params[] = "%$search%"; $params[] = "%$search%"; }
$whereClause = implode(" AND ", $where);

$sortOptions = ['newest' => 'p.created_at DESC', 'oldest' => 'p.created_at ASC', 'price_low' => 'p.price ASC', 'price_high' => 'p.price DESC', 'name_asc' => 'p.name ASC', 'name_desc' => 'p.name DESC'];
$orderBy = $sortOptions[$sort] ?? 'p.created_at DESC';

try {
    $sql = "SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE $whereClause ORDER BY $orderBy LIMIT $perPage OFFSET $offset";
    $stmt = $pdo->prepare($sql); $stmt->execute($params); $products = $stmt->fetchAll();
    $countSql = "SELECT COUNT(*) FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE $whereClause";
    $stmt = $pdo->prepare($countSql); $stmt->execute($params); $total = $stmt->fetchColumn(); $totalPages = ceil($total / $perPage);
    $categories = $pdo->query("SELECT * FROM categories ORDER BY name")->fetchAll();
} catch (PDOException $e) { $products = []; $total = 0; $totalPages = 0; $categories = []; }
?>
<section class="py-4">
    <div class="container">
        <div class="row mb-4">
            <div class="col-md-4 mb-2">
                <form method="GET" class="d-flex">
                    <input type="text" name="search" class="form-control me-2" placeholder="ค้นหา..." value="<?php echo htmlspecialchars($search); ?>">
                    <button type="submit" class="btn btn-success">ค้นหา</button>
                </form>
            </div>
            <div class="col-md-4 mb-2">
                <select class="form-select" onchange="location.href='?category='+this.value+'&search=<?php echo urlencode($search); ?>&sort=<?php echo $sort; ?>'">
                    <option value="">ทุกหมวดหมู่</option>
                    <?php foreach ($categories as $c): ?><option value="<?php echo $c['slug']; ?>" <?php echo $category === $c['slug'] ? 'selected' : ''; ?>><?php echo $c['name']; ?></option><?php endforeach; ?>
                </select>
            </div>
            <div class="col-md-4 mb-2">
                <select class="form-select" onchange="location.href='?category=<?php echo urlencode($category); ?>&search=<?php echo urlencode($search); ?>&sort='+this.value">
                    <option value="newest" <?php echo $sort === 'newest' ? 'selected' : ''; ?>>ใหม่ล่าสุด</option>
                    <option value="oldest" <?php echo $sort === 'oldest' ? 'selected' : ''; ?>>เก่าที่สุด</option>
                    <option value="price_low" <?php echo $sort === 'price_low' ? 'selected' : ''; ?>>ราคา: ต่ำ-สูง</option>
                    <option value="price_high" <?php echo $sort === 'price_high' ? 'selected' : ''; ?>>ราคา: สูง-ต่ำ</option>
                </select>
            </div>
        </div>
        <div class="row">
            <?php if (empty($products)): ?>
            <div class="col-12 text-center py-5"><h3 class="text-muted">ไม่พบสินค้า</h3></div>
            <?php else: foreach ($products as $p): ?>
            <div class="col-md-3 mb-4">
                <div class="card product-card h-100">
                    <img src="<?php echo $p['image_url'] ?: 'https://via.placeholder.com/300x200/2a2a2a/80ef25?text=No+Image'; ?>" class="card-img-top">
                    <div class="card-body">
                        <h5 class="card-title"><?php echo htmlspecialchars($p['name']); ?></h5>
                        <p class="text-muted small"><?php echo $p['category_name']; ?></p>
                        <h4 class="text-success"><?php echo formatPrice($p['price']); ?></h4>
                        <div class="d-flex justify-content-between align-items-center mt-3">
                            <span class="badge bg-<?php echo $p['stock'] > 0 ? 'success' : 'danger'; ?>"><?php echo $p['stock'] > 0 ? 'มีสินค้า' : 'หมด'; ?></span>
                            <div>
                                <a href="product.php?id=<?php echo $p['id']; ?>" class="btn btn-sm btn-outline-success">ดูรายละเอียด</a>
                                <?php if (isLoggedIn() && $p['stock'] > 0): ?>
                                <button class="btn btn-sm btn-success add-to-cart" data-product-id="<?php echo $p['id']; ?>"><i class="bi bi-cart-plus"></i></button>
                                <?php endif; ?>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <?php endforeach; endif; ?>
        </div>
        <?php if ($totalPages > 1): ?>
        <nav class="mt-4">
            <ul class="pagination justify-content-center">
                <?php for ($i = 1; $i <= $totalPages; $i++): ?>
                <li class="page-item <?php echo $i === $page ? 'active' : ''; ?>"><a class="page-link bg-dark text-white border-secondary" href="?page_num=<?php echo $i; ?>&category=<?php echo urlencode($category); ?>&search=<?php echo urlencode($search); ?>&sort=<?php echo $sort; ?>"><?php echo $i; ?></a></li>
                <?php endfor; ?>
            </ul>
        </nav>
        <?php endif; ?>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/product.php (Product Detail)
```php
<?php
$pageTitle = 'รายละเอียดสินค้า';
require_once 'includes/header.php';

$id = isset($_GET['id']) ? (int)$_GET['id'] : 0;
if (!$id) { header('Location: products.php'); exit; }

try {
    $stmt = $pdo->prepare("SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id WHERE p.id = ? AND p.status = 'active'");
    $stmt->execute([$id]); $product = $stmt->fetch();
    if (!$product) { header('Location: products.php'); exit; }
    $gallery = json_decode($product['gallery'] ?? '[]', true);
    $related = $pdo->prepare("SELECT * FROM products WHERE category_id = ? AND id != ? AND status = 'active' ORDER BY created_at DESC LIMIT 4");
    $related->execute([$product['category_id'], $id]); $relatedProducts = $related->fetchAll();
} catch (PDOException $e) { header('Location: products.php'); exit; }
?>
<section class="py-5">
    <div class="container">
        <nav aria-label="breadcrumb">
            <ol class="breadcrumb">
                <li class="breadcrumb-item"><a href="index.php" class="text-success">หน้าแรก</a></li>
                <li class="breadcrumb-item"><a href="products.php" class="text-success">สินค้า</a></li>
                <li class="breadcrumb-item active"><?php echo htmlspecialchars($product['name']); ?></li>
            </ol>
        </nav>
        <div class="row">
            <div class="col-md-6">
                <?php if (!empty($gallery)): ?>
                    <?php if (count($gallery) <= 4): ?>
                    <div class="mb-3"><img id="mainImage" src="<?php echo $gallery[0]; ?>" class="img-fluid rounded"></div>
                    <div class="row">
                        <?php foreach ($gallery as $img): ?>
                        <div class="col-3 mb-2"><img src="<?php echo $img; ?>" class="img-fluid rounded" onclick="document.getElementById('mainImage').src='<?php echo $img; ?>'" style="cursor:pointer"></div>
                        <?php endforeach; ?>
                    </div>
                    <?php else: ?>
                    <div id="productCarousel" class="carousel slide" data-bs-ride="carousel">
                        <div class="carousel-inner">
                            <?php foreach ($gallery as $i => $img): ?>
                            <div class="carousel-item <?php echo $i === 0 ? 'active' : ''; ?>"><img src="<?php echo $img; ?>" class="d-block w-100 rounded"></div>
                            <?php endforeach; ?>
                        </div>
                        <button class="carousel-control-prev" type="button" data-bs-target="#productCarousel" data-bs-slide="prev"><span class="carousel-control-prev-icon"></span></button>
                        <button class="carousel-control-next" type="button" data-bs-target="#productCarousel" data-bs-slide="next"><span class="carousel-control-next-icon"></span></button>
                    </div>
                    <?php endif; ?>
                <?php elseif ($product['image_url']): ?>
                <img src="<?php echo $product['image_url']; ?>" class="img-fluid rounded">
                <?php else: ?>
                <div class="bg-secondary d-flex align-items-center justify-content-center rounded" style="height:400px"><span class="text-white">ไม่มีรูปภาพ</span></div>
                <?php endif; ?>
            </div>
            <div class="col-md-6">
                <h1 class="mb-3"><?php echo htmlspecialchars($product['name']); ?></h1>
                <p class="text-muted">หมวดหมู่: <?php echo $product['category_name']; ?></p>
                <h2 class="text-success mb-4"><?php echo formatPrice($product['price']); ?></h2>
                <div class="mb-4">
                    <span class="badge bg-<?php echo $product['stock'] > 0 ? 'success' : 'danger'; ?> fs-6"><?php echo $product['stock'] > 0 ? 'มีสินค้า (' . $product['stock'] . ')' : 'หมดสินค้า'; ?></span>
                </div>
                <div class="mb-4"><h5>รายละเอียด</h5><div class="text-muted"><?php echo nl2br(htmlspecialchars($product['description'])); ?></div></div>
                <?php if ($product['video_url']): ?>
                <div class="mb-4"><h5>รีวิวสินค้า</h5><div class="ratio ratio-16x9"><iframe src="<?php echo $product['video_url']; ?>" allowfullscreen class="rounded"></iframe></div></div>
                <?php endif; ?>
                <?php if (isLoggedIn() && $product['stock'] > 0): ?>
                <div class="row align-items-center">
                    <div class="col-md-4">
                        <label>จำนวน</label>
                        <input type="number" class="form-control" id="qty" value="1" min="1" max="<?php echo $product['stock']; ?>">
                    </div>
                    <div class="col-md-8">
                        <label class="invisible">d</label>
                        <button class="btn btn-success w-100 add-to-cart" data-product-id="<?php echo $product['id']; ?>" data-quantity="#qty"><i class="bi bi-cart-plus"></i> เพิ่มลงตะกร้า</button>
                    </div>
                </div>
                <?php elseif (!isLoggedIn()): ?>
                <a href="login.php" class="btn btn-success w-100"><i class="bi bi-person"></i> เข้าสู่ระบบเพื่อซื้อ</a>
                <?php endif; ?>
            </div>
        </div>
    </div>
</section>

<?php if (!empty($relatedProducts)): ?>
<section class="py-5 bg-dark"><div class="container"><h3 class="text-center mb-4 text-success">สินค้าที่เกี่ยวข้อง</h3><div class="row">
    <?php foreach ($relatedProducts as $p): ?>
    <div class="col-md-3 mb-4">
        <div class="card h-100">
            <img src="<?php echo $p['image_url'] ?: 'https://via.placeholder.com/300x200/2a2a2a/80ef25?text=No+Image'; ?>" class="card-img-top">
            <div class="card-body">
                <h5><?php echo htmlspecialchars($p['name']); ?></h5>
                <h4 class="text-success"><?php echo formatPrice($p['price']); ?></h4>
                <a href="product.php?id=<?php echo $p['id']; ?>" class="btn btn-sm btn-outline-success">ดูรายละเอียด</a>
            </div>
        </div>
    </div>
    <?php endforeach; ?>
</div></div></section>
<?php endif; ?>
<?php require_once 'includes/footer.php'; ?>
```

### pages/cart.php
```php
<?php
$pageTitle = 'ตะกร้าสินค้า';
require_once 'includes/header.php';

if (!isLoggedIn()) { header('Location: login.php'); exit; }

try {
    $stmt = $pdo->prepare("SELECT c.*, p.name, p.price, p.image_url, p.stock FROM cart c LEFT JOIN products p ON c.product_id = p.id WHERE c.user_id = ? AND p.status = 'active'");
    $stmt->execute([$_SESSION['user_id']]); $cartItems = $stmt->fetchAll();
    
    $subtotal = 0;
    foreach ($cartItems as $item) $subtotal += $item['price'] * $item['quantity'];
    
    $discount = 0;
    $total = $subtotal - $discount;
} catch (PDOException $e) { $cartItems = []; $subtotal = $total = 0; }
?>
<section class="py-5">
    <div class="container">
        <h2 class="mb-4 text-success">ตะกร้าสินค้า</h2>
        <?php if (empty($cartItems)): ?>
        <div class="text-center py-5"><h3 class="text-muted">ตะกร้าว่างเปล่า</h3><a href="products.php" class="btn btn-success mt-3">เลือกซื้อสินค้า</a></div>
        <?php else: ?>
        <div class="row">
            <div class="col-md-8">
                <?php foreach ($cartItems as $item): ?>
                <div class="card mb-3">
                    <div class="row g-0">
                        <div class="col-md-2"><img src="<?php echo $item['image_url'] ?: 'https://via.placeholder.com/100/2a2a2a/80ef25?text=No+Image'; ?>" class="img-fluid rounded-start"></div>
                        <div class="col-md-10">
                            <div class="card-body">
                                <div class="row align-items-center">
                                    <div class="col-md-6"><h5 class="card-title"><?php echo htmlspecialchars($item['name']); ?></h5><p class="text-success"><?php echo formatPrice($item['price']); ?> / ชิ้น</p></div>
                                    <div class="col-md-3"><input type="number" class="form-control" value="<?php echo $item['quantity']; ?>" min="1" max="<?php echo $item['stock']; ?>" readonly></div>
                                    <div class="col-md-2"><h5 class="text-success"><?php echo formatPrice($item['price'] * $item['quantity']); ?></h5></div>
                                    <div class="col-md-1"><button class="btn btn-danger btn-sm remove-from-cart" data-product-id="<?php echo $item['product_id']; ?>"><i class="bi bi-trash"></i></button></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <?php endforeach; ?>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title text-success">สรุปรายการ</h5>
                        <div class="mb-3">
                            <label>รหัสคูปองส่วนลด</label>
                            <div class="input-group">
                                <input type="text" class="form-control" id="coupon-code" placeholder="กรอกรหัส">
                                <button class="btn btn-outline-success" id="apply-coupon">ใช้</button>
                            </div>
                        </div>
                        <hr>
                        <div class="d-flex justify-content-between"><span>ยอดรวม:</span><span><?php echo formatPrice($subtotal); ?></span></div>
                        <div class="d-flex justify-content-between"><span>ส่วนลด:</span><span class="text-danger">-<?php echo formatPrice($discount); ?></span></div>
                        <hr>
                        <div class="d-flex justify-content-between"><strong>รวมทั้งสิ้น:</strong><strong class="text-success"><?php echo formatPrice($total); ?></strong></div>
                        <a href="checkout.php" class="btn btn-success w-100 mt-3">ดำเนินการสั่งซื้อ</a>
                        <a href="products.php" class="btn btn-outline-secondary w-100 mt-2">เลือกซื้อต่อ</a>
                    </div>
                </div>
            </div>
        </div>
        <?php endif; ?>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/checkout.php
```php
<?php
$pageTitle = 'ชำระเงิน';
require_once 'includes/header.php';

if (!isLoggedIn()) { header('Location: login.php'); exit; }

try {
    $stmt = $pdo->prepare("SELECT c.*, p.name, p.price, p.stock FROM cart c LEFT JOIN products p ON c.product_id = p.id WHERE c.user_id = ? AND p.status = 'active'");
    $stmt->execute([$_SESSION['user_id']]); $cartItems = $stmt->fetchAll();
    if (empty($cartItems)) { header('Location: cart.php'); exit; }
    
    $subtotal = 0;
    foreach ($cartItems as $item) $subtotal += $item['price'] * $item['quantity'];
    $total = $subtotal;
    
    // Check coupon
    $discount = 0;
    if (isset($_SESSION['coupon_code'])) {
        $stmt = $pdo->prepare("SELECT * FROM coupons WHERE code = ? AND status = 'active' AND (max_uses IS NULL OR used_count < max_uses) AND (expires_at IS NULL OR expires_at > NOW())");
        $stmt->execute([$_SESSION['coupon_code']]); $coupon = $stmt->fetch();
        if ($coupon && $subtotal >= $coupon['min_amount']) {
            $discount = $subtotal * ($coupon['discount_percent'] / 100);
            $total = $subtotal - $discount;
        }
    }
    
    // Check credit
    $userCredit = getUserCredit($_SESSION['user_id']);
    $canCheckout = $userCredit >= $total;
    
    // Process checkout
    if ($_SERVER['REQUEST_METHOD'] === 'POST' && $canCheckout) {
        $pdo->beginTransaction();
        try {
            // Create order
            $orderNumber = generateOrderNumber();
            $stmt = $pdo->prepare("INSERT INTO orders (user_id, order_number, total_amount, delivery_type) VALUES (?, ?, ?, 'manual')");
            $stmt->execute([$_SESSION['user_id'], $orderNumber, $total]);
            $orderId = $pdo->lastInsertId();
            
            // Create order items
            foreach ($cartItems as $item) {
                $stmt = $pdo->prepare("INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?, ?, ?, ?)");
                $stmt->execute([$orderId, $item['product_id'], $item['quantity'], $item['price']]);
                // Reduce stock
                $pdo->prepare("UPDATE products SET stock = stock - ? WHERE id = ?")->execute([$item['quantity'], $item['product_id']]);
            }
            
            // Deduct credit
            updateUserCredit($_SESSION['user_id'], -$total);
            
            // Clear cart
            $pdo->prepare("DELETE FROM cart WHERE user_id = ?")->execute([$_SESSION['user_id']]);
            
            // Update coupon usage
            if (isset($_SESSION['coupon_code'])) {
                $pdo->prepare("UPDATE coupons SET used_count = used_count + 1 WHERE code = ?")->execute([$_SESSION['coupon_code']]);
                unset($_SESSION['coupon_code']);
            }
            
            $pdo->commit();
            
            // Discord notification
            sendDiscordWebhook("🔔 คำสั่งซื้อใหม่: {$orderNumber} - ฿{$total}");
            
            echo "<script>
                Swal.fire({ title: 'สำเร็จ!', text: 'สั่งซื้อสำเร็จ เลขที่คำสั่งซื้อ: {$orderNumber}', icon: 'success', confirmButtonText: 'ตกลง' })
                .then(() => window.location = 'orders.php');
            </script>";
            exit;
        } catch (Exception $e) {
            $pdo->rollBack();
            echo "<script>Swal.fire({ title: 'ผิดพลาด!', text: 'ไม่สามารถสั่งซื้อได้', icon: 'error' });</script>";
        }
    }
} catch (PDOException $e) { header('Location: cart.php'); exit; }
?>
<section class="py-5">
    <div class="container">
        <h2 class="mb-4 text-success">ชำระเงิน</h2>
        <div class="row">
            <div class="col-md-8">
                <div class="card mb-3">
                    <div class="card-body">
                        <h5 class="text-success">รายการสินค้า</h5>
                        <?php foreach ($cartItems as $item): ?>
                        <div class="d-flex justify-content-between align-items-center py-2 border-bottom">
                            <div><h6><?php echo htmlspecialchars($item['name']); ?></h6><small><?php echo $item['quantity']; ?> x <?php echo formatPrice($item['price']); ?></small></div>
                            <span><?php echo formatPrice($item['price'] * $item['quantity']); ?></span>
                        </div>
                        <?php endforeach; ?>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-body">
                        <h5 class="text-success">สรุปยอด</h5>
                        <div class="d-flex justify-content-between"><span>ยอดรวม:</span><span><?php echo formatPrice($subtotal); ?></span></div>
                        <?php if ($discount > 0): ?><div class="d-flex justify-content-between"><span>ส่วนลด:</span><span class="text-danger">-<?php echo formatPrice($discount); ?></span></div><?php endif; ?>
                        <hr>
                        <div class="d-flex justify-content-between"><strong>รวมทั้งสิ้น:</strong><strong class="text-success"><?php echo formatPrice($total); ?></strong></div>
                        <hr>
                        <div class="d-flex justify-content-between"><span>เครดิตของคุณ:</span><span><?php echo formatPrice($userCredit); ?></span></div>
                        <div class="d-flex justify-content-between"><span>เครดิตคงเหลือ:</span><span class="<?php echo $canCheckout ? 'text-success' : 'text-danger'; ?>"><?php echo formatPrice($userCredit - $total); ?></span></div>
                        
                        <?php if ($canCheckout): ?>
                        <form method="POST" class="mt-3">
                            <button type="submit" class="btn btn-success w-100">ยืนยันการสั่งซื้อ</button>
                        </form>
                        <?php else: ?>
                        <div class="alert alert-danger mt-3">เครดิตไม่เพียงพอ กรุณา<a href="topup.php">เติมเงิน</a></div>
                        <?php endif; ?>
                        <a href="cart.php" class="btn btn-outline-secondary w-100 mt-2">กลับไปตะกร้า</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/topup.php (Credit Top-up with 3 payment methods)
```php
<?php
$pageTitle = 'เติมเงิน';
require_once 'includes/header.php';

if (!isLoggedIn()) { header('Location: login.php'); exit; }

$error = ''; $success = '';

// Get topup history
$history = $pdo->prepare("SELECT * FROM topup_history WHERE user_id = ? ORDER BY created_at DESC LIMIT 10")->execute([$_SESSION['user_id']])->fetchAll();

// Process Redeem Code
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['redeem_code'])) {
    $code = strtoupper(trim($_POST['redeem_code']));
    $stmt = $pdo->prepare("SELECT * FROM redeem_codes WHERE code = ? AND status = 'active'");
    $stmt->execute([$code]); $redeem = $stmt->fetch();
    
    if ($redeem) {
        $pdo->beginTransaction();
        try {
            // Update redeem code
            $pdo->prepare("UPDATE redeem_codes SET status = 'used', used_by = ?, used_at = NOW() WHERE id = ?")->execute([$_SESSION['user_id'], $redeem['id']]);
            // Add credit
            updateUserCredit($_SESSION['user_id'], $redeem['amount']);
            // Add history
            $pdo->prepare("INSERT INTO topup_history (user_id, amount, method, transaction_id, status) VALUES (?, ?, 'redeem', ?, 'completed')")->execute([$_SESSION['user_id'], $redeem['amount'], $code]);
            
            $pdo->commit();
            
            // Discord notification
            if (getSetting('discord_payment_enabled') == 'true') {
                sendDiscordWebhook("💰 เติมเงินผ่านรหัส: {$code} - ฿{$redeem['amount']} - โดย {$_SESSION['username']}");
            }
            
            echo "<script>Swal.fire({ title: 'สำเร็จ!', text: 'เติมเงินสำเร็จ ฿{$redeem['amount']}', icon: 'success' }).then(()=>location.reload());</script>";
        } catch (Exception $e) { $pdo->rollBack(); $error = 'เกิดข้อผิดพลาด'; }
    } else { $error = 'รหัสไม่ถูกต้องหรือถูกใช้แล้ว'; }
}
?>
<section class="py-5">
    <div class="container">
        <h2 class="mb-4 text-success">เติมเงิน</h2>
        <div class="row">
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <div class="card-body text-center">
                        <h4 class="text-success"><i class="bi bi-qr-code"></i> พร้อมเพย์</h4>
                        <p>สแกน QR Code เพื่อเติมเงิน</p>
                        <form method="POST" action="api/topup.php">
                            <input type="hidden" name="method" value="promptpay">
                            <div class="mb-3">
                                <label>จำนวนเงิน</label>
                                <select name="amount" class="form-select">
                                    <option value="50">50 บาท</option>
                                    <option value="100" selected>100 บาท</option>
                                    <option value="200">200 บาท</option>
                                    <option value="500">500 บาท</option>
                                    <option value="1000">1,000 บาท</option>
                                </select>
                            </div>
                            <button type="submit" class="btn btn-success w-100">สร้าง QR Code</button>
                        </form>
                    </div>
                </div>
            </div>
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <div class="card-body text-center">
                        <h4 class="text-success"><i class="bi bi-wallet2"></i> TrueWallet</h4>
                        <p>ใช้ลิงก์ซองของขวัญ</p>
                        <form method="POST" action="api/topup.php">
                            <input type="hidden" name="method" value="truewallet">
                            <div class="mb-3">
                                <label>ลิงก์ซองของขวัญ</label>
                                <input type="url" name="gift_url" class="form-control" placeholder="https://gift.truemoney.com/..." required>
                            </div>
                            <button type="submit" class="btn btn-success w-100">ยืนยันการเติมเงิน</button>
                        </form>
                    </div>
                </div>
            </div>
            <div class="col-md-4 mb-4">
                <div class="card h-100">
                    <div class="card-body text-center">
                        <h4 class="text-success"><i class="bi bi-gift"></i> รหัสเติมเงิน</h4>
                        <p>ใช้รหัส REEDEM-XXXXXX</p>
                        <form method="POST">
                            <div class="mb-3">
                                <label>รหัสเติมเงิน</label>
                                <input type="text" name="redeem_code" class="form-control" placeholder="REDEEM-XXXXXX" required>
                            </div>
                            <button type="submit" class="btn btn-success w-100">เติมเงิน</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
        
        <?php if ($error): ?><div class="alert alert-danger"><?php echo $error; ?></div><?php endif; ?>
        
        <h4 class="mt-5 mb-3 text-success">ประวัติการเติมเงิน</h4>
        <div class="table-responsive">
            <table class="table table-dark">
                <thead><tr><th>วันที่</th><th>วิธีการ</th><th>จำนวน</th><th>สถานะ</th></tr></thead>
                <tbody>
                    <?php foreach ($history as $h): ?>
                    <tr>
                        <td><?php echo date('d/m/Y H:i', strtotime($h['created_at'])); ?></td>
                        <td><?php echo $h['method'] === 'promptpay' ? 'พร้อมเพย์' : ($h['method'] === 'truewallet' ? 'TrueWallet' : 'รหัสเติมเงิน'); ?></td>
                        <td><?php echo formatPrice($h['amount']); ?></td>
                        <td><span class="badge bg-<?php echo $h['status'] === 'completed' ? 'success' : ($h['status'] === 'pending' ? 'warning' : 'danger'); ?>"><?php echo $h['status']; ?></span></td>
                    </tr>
                    <?php endforeach; ?>
                </tbody>
            </table>
        </div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/orders.php (Order History)
```php
<?php
$pageTitle = 'ประวัติการสั่งซื้อ';
require_once 'includes/header.php';

if (!isLoggedIn()) { header('Location: login.php'); exit; }

try {
    $stmt = $pdo->prepare("SELECT o.*, COUNT(oi.id) as item_count FROM orders o LEFT JOIN order_items oi ON o.id = oi.order_id WHERE o.user_id = ? GROUP BY o.id ORDER BY o.created_at DESC");
    $stmt->execute([$_SESSION['user_id']]);
    $orders = $stmt->fetchAll();
} catch (PDOException $e) { $orders = []; }
?>
<section class="py-5">
    <div class="container">
        <h2 class="mb-4 text-success">ประวัติการสั่งซื้อ</h2>
        <?php if (empty($orders)): ?>
        <div class="text-center py-5"><h3 class="text-muted">ไม่มีรายการสั่งซื้อ</h3><a href="products.php" class="btn btn-success mt-3">เลือกซื้อสินค้า</a></div>
        <?php else: ?>
        <div class="table-responsive">
            <table class="table table-dark">
                <thead>
                    <tr><th>เลขที่คำสั่งซื้อ</th><th>จำนวนสินค้า</th><th>ยอดรวม</th><th>สถานะ</th><th>วันที่</th><th>จัดการ</th></tr>
                </thead>
                <tbody>
                    <?php foreach ($orders as $o): ?>
                    <tr>
                        <td><?php echo $o['order_number']; ?></td>
                        <td><?php echo $o['item_count']; ?> รายการ</td>
                        <td><?php echo formatPrice($o['total_amount']); ?></td>
                        <td>
                            <?php 
                            $statusText = ['pending' => 'รอดำเนินการ', 'processing' => 'กำลังจัดส่ง', 'completed' => 'จัดส่งแล้ว', 'cancelled' => 'ยกเลิก'][$o['status']] ?? $o['status'];
                            $badgeClass = ['pending' => 'warning', 'processing' => 'info', 'completed' => 'success', 'cancelled' => 'danger'][$o['status']] ?? 'secondary';
                            ?>
                            <span class="badge bg-<?php echo $badgeClass; ?>"><?php echo $statusText; ?></span>
                        </td>
                        <td><?php echo date('d/m/Y H:i', strtotime($o['created_at'])); ?></td>
                        <td>
                            <?php if ($o['status'] === 'completed' && $o['delivery_data']): ?>
                            <button class="btn btn-sm btn-success" onclick="viewOrderDetails('<?php echo htmlspecialchars($o['delivery_data']); ?>')"><i class="bi bi-eye"></i> ดูสินค้า</button>
                            <?php else: ?>
                            <a href="order_detail.php?id=<?php echo $o['id']; ?>" class="btn btn-sm btn-outline-success">รายละเอียด</a>
                            <?php endif; ?>
                        </td>
                    </tr>
                    <?php endforeach; ?>
                </tbody>
            </table>
        </div>
        <?php endif; ?>
    </div>
</section>
<script>
function viewOrderDetails(data) {
    Swal.fire({
        title: 'ข้อมูลสินค้า',
        html: '<textarea class="form-control bg-dark text-white" rows="5" readonly>' + data + '</textarea><button class="btn btn-success mt-2" onclick="navigator.clipboard.writeText(\'' + data.replace(/'/g, "\\'") + '\'); Swal.showValidationMessage(\'คัดลอกแล้ว!\')">คัดลอก</button>',
        showConfirmButton: false,
        showCloseButton: true
    });
}
</script>
<?php require_once 'includes/footer.php'; ?>
```

### pages/login.php
```php
<?php
$pageTitle = 'เข้าสู่ระบบ';
require_once 'includes/header.php';

$error = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = $_POST['username'] ?? '';
    $password = $_POST['password'] ?? '';
    
    // Turnstile check
    if (getSetting('turnstile_enabled') == 'true') {
        $token = $_POST['cf-turnstile-response'] ?? '';
        if (!$token) { $error = 'กรุณายืนยันว่าไม่ใช่หุ่นยนต์'; }
        else {
            $verify = file_get_contents('https://challenges.cloudflare.com/turnstile/v0/siteverify', false, stream_context_create([
                'http' => ['method' => 'POST', 'header' => 'Content-Type: application/x-www-form-urlencoded',
                'content' => http_build_query(['secret' => getSetting('turnstile_secret_key'), 'response' => $token])]
            ]));
            if (!json_decode($verify)->success) $error = 'การยืนยันล้มเหลว';
        }
    }
    
    if (!$error) {
        $stmt = $pdo->prepare("SELECT * FROM users WHERE (username = ? OR email = ?) AND status = 'active'");
        $stmt->execute([$username, $username]); $user = $stmt->fetch();
        if ($user && password_verify($password, $user['password'])) {
            $_SESSION['user_id'] = $user['id']; $_SESSION['username'] = $user['username']; $_SESSION['email'] = $user['email'];
            $pdo->prepare("UPDATE users SET last_login = NOW() WHERE id = ?")->execute([$user['id']]);
            echo "<script>Swal.fire({ title: 'สำเร็จ!', text: 'เข้าสู่ระบบสำเร็จ', icon: 'success' }).then(()=>location='index.php');</script>";
            exit;
        } else $error = 'ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง';
    }
}
?>
<section class="py-5">
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-6">
                <div class="card">
                    <div class="card-body">
                        <h3 class="text-center mb-4 text-success">เข้าสู่ระบบ</h3>
                        <?php if ($error): ?><div class="alert alert-danger"><?php echo $error; ?></div><?php endif; ?>
                        <form method="POST">
                            <div class="mb-3">
                                <label>ชื่อผู้ใช้หรืออีเมล</label>
                                <input type="text" name="username" class="form-control" required>
                            </div>
                            <div class="mb-3">
                                <label>รหัสผ่าน</label>
                                <input type="password" name="password" class="form-control" required>
                            </div>
                            <?php if (getSetting('turnstile_enabled') == 'true'): ?>
                            <div class="mb-3">
                                <div class="cf-turnstile" data-sitekey="<?php echo getSetting('turnstile_site_key'); ?>"></div>
                            </div>
                            <?php endif; ?>
                            <button type="submit" class="btn btn-success w-100">เข้าสู่ระบบ</button>
                        </form>
                        <hr>
                        <p class="text-center mb-0">ยังไม่มีบัญชี? <a href="register.php" class="text-success">สมัครสมาชิก</a></p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/register.php
```php
<?php
$pageTitle = 'สมัครสมาชิก';
require_once 'includes/header.php';

$error = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $email = trim($_POST['email'] ?? '');
    $password = $_POST['password'] ?? '';
    $confirm = $_POST['confirm_password'] ?? '';
    
    // Validation
    if (strlen($username) < 6 || strlen($username) > 20 || !preg_match('/^[a-zA-Z0-9_]+$/', $username)) $error = 'ชื่อผู้ใช้ต้อง 6-20 ตัวอักษร (a-z, 0-9, _)';
    elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) $error = 'รูปแบบอีเมลไม่ถูกต้อง';
    elseif (strlen($password) < 6) $error = 'รหัสผ่านต้องอย่างน้อย 6 ตัวอักษร';
    elseif ($password !== $confirm) $error = 'รหัสผ่านไม่ตรงกัน';
    else {
        // Turnstile
        if (getSetting('turnstile_enabled') == 'true') {
            $token = $_POST['cf-turnstile-response'] ?? '';
            if (!$token) $error = 'กรุณายืนยันว่าไม่ใช่หุ่นยนต์';
            else {
                $verify = file_get_contents('https://challenges.cloudflare.com/turnstile/v0/siteverify', false, stream_context_create([
                    'http' => ['method' => 'POST', 'header' => 'Content-Type: application/x-www-form-urlencoded',
                    'content' => http_build_query(['secret' => getSetting('turnstile_secret_key'), 'response' => $token])]
                ]));
                if (!json_decode($verify)->success) $error = 'การยืนยันล้มเหลว';
            }
        }
        
        if (!$error) {
            // Check exists
            $stmt = $pdo->prepare("SELECT id FROM users WHERE username = ? OR email = ?");
            $stmt->execute([$username, $email]);
            if ($stmt->fetch()) $error = 'ชื่อผู้ใช้หรืออีเมลนี้มีอยู่แล้ว';
            else {
                // Create user
                $hash = password_hash($password, PASSWORD_DEFAULT);
                $stmt = $pdo->prepare("INSERT INTO users (username, email, password) VALUES (?, ?, ?)");
                if ($stmt->execute([$username, $email, $hash])) {
                    echo "<script>Swal.fire({ title: 'สำเร็จ!', text: 'สมัครสมาชิกสำเร็จ', icon: 'success' }).then(()=>location='login.php');</script>";
                    exit;
                } else $error = 'เกิดข้อผิดพลาด';
            }
        }
    }
}
?>
<section class="py-5">
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-6">
                <div class="card">
                    <div class="card-body">
                        <h3 class="text-center mb-4 text-success">สมัครสมาชิก</h3>
                        <?php if ($error): ?><div class="alert alert-danger"><?php echo $error; ?></div><?php endif; ?>
                        <form method="POST">
                            <div class="mb-3"><label>ชื่อผู้ใช้ (6-20 ตัวอักษร)</label><input type="text" name="username" class="form-control" required minlength="6" maxlength="20"></div>
                            <div class="mb-3"><label>อีเมล</label><input type="email" name="email" class="form-control" required></div>
                            <div class="mb-3"><label>รหัสผ่าน (6+ ตัวอักษร)</label><input type="password" name="password" class="form-control" required minlength="6"></div>
                            <div class="mb-3"><label>ยืนยันรหัสผ่าน</label><input type="password" name="confirm_password" class="form-control" required></div>
                            <?php if (getSetting('turnstile_enabled') == 'true'): ?>
                            <div class="mb-3"><div class="cf-turnstile" data-sitekey="<?php echo getSetting('turnstile_site_key'); ?>"></div></div>
                            <?php endif; ?>
                            <button type="submit" class="btn btn-success w-100">สมัครสมาชิก</button>
                        </form>
                        <hr><p class="text-center mb-0">มีบัญชีแล้ว? <a href="login.php" class="text-success">เข้าสู่ระบบ</a></p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/contact.php
```php
<?php
$pageTitle = 'ติดต่อเรา';
require_once 'includes/header.php';
?>
<section class="py-5">
    <div class="container">
        <h2 class="mb-4 text-success">ติดต่อเรา</h2>
        <div class="row">
            <div class="col-md-6">
                <div class="card">
                    <div class="card-body">
                        <h5 class="text-success">ข้อมูลติดต่อ</h5>
                        <p><?php echo nl2br(htmlspecialchars(getSetting('contact_info', 'ติดต่อเราสำหรับข้อมูลเพิ่มเติม'))); ?></p>
                    </div>
                </div>
            </div>
            <div class="col-md-6">
                <div class="card">
                    <div class="card-body">
                        <h5 class="text-success">ส่งข้อความถึงเรา</h5>
                        <form onsubmit="event.preventDefault(); Swal.fire({ title: 'สำเร็จ!', text: 'ส่งข้อความแล้ว', icon: 'success' });">
                            <div class="mb-3"><label>ชื่อ</label><input type="text" class="form-control" required></div>
                            <div class="mb-3"><label>อีเมล</label><input type="email" class="form-control" required></div>
                            <div class="mb-3"><label>ข้อความ</label><textarea class="form-control" rows="4" required></textarea></div>
                            <button type="submit" class="btn btn-success w-100">ส่งข้อความ</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

### pages/logout.php
```php
<?php
session_start();
session_destroy();
header('Location: login.php');
exit;
?>
```

### pages/profile.php
```php
<?php
$pageTitle = 'โปรไฟล์';
require_once 'includes/header.php';

if (!isLoggedIn()) { header('Location: login.php'); exit; }

$success = ''; $error = '';

// Get user data
$user = $pdo->prepare("SELECT * FROM users WHERE id = ?")->execute([$_SESSION['user_id']])->fetch();

// Update profile
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['update_profile'])) {
    $email = trim($_POST['email'] ?? '');
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) $error = 'รูปแบบอีเมลไม่ถูกต้อง';
    else {
        $stmt = $pdo->prepare("UPDATE users SET email = ? WHERE id = ?");
        if ($stmt->execute([$email, $_SESSION['user_id']])) {
            $_SESSION['email'] = $email; $success = 'อัพเดตโปรไฟล์สำเร็จ';
        } else $error = 'เกิดข้อผิดพลาด';
    }
}

// Change password
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['change_password'])) {
    $current = $_POST['current_password'] ?? '';
    $new = $_POST['new_password'] ?? '';
    $confirm = $_POST['confirm_password'] ?? '';
    
    if (strlen($new) < 6) $error = 'รหัสผ่านใหม่ต้องอย่างน้อย 6 ตัวอักษร';
    elseif ($new !== $confirm) $error = 'รหัสผ่านไม่ตรงกัน';
    elseif (!password_verify($current, $user['password'])) $error = 'รหัสผ่านปัจจุบันไม่ถูกต้อง';
    else {
        $hash = password_hash($new, PASSWORD_DEFAULT);
        if ($pdo->prepare("UPDATE users SET password = ? WHERE id = ?")->execute([$hash, $_SESSION['user_id']])) $success = 'เปลี่ยนรหัสผ่านสำเร็จ';
        else $error = 'เกิดข้อผิดพลาด';
    }
}
?>
<section class="py-5">
    <div class="container">
        <h2 class="mb-4 text-success">โปรไฟล์ของฉัน</h2>
        <?php if ($success): ?><div class="alert alert-success"><?php echo $success; ?></div><?php endif; ?>
        <?php if ($error): ?><div class="alert alert-danger"><?php echo $error; ?></div><?php endif; ?>
        <div class="row">
            <div class="col-md-6">
                <div class="card mb-4">
                    <div class="card-body">
                        <h5 class="text-success">ข้อมูลส่วนตัว</h5>
                        <form method="POST">
                            <input type="hidden" name="update_profile" value="1">
                            <div class="mb-3"><label>ชื่อผู้ใช้</label><input type="text" class="form-control" value="<?php echo htmlspecialchars($user['username']); ?>" disabled></div>
                            <div class="mb-3"><label>อีเมล</label><input type="email" name="email" class="form-control" value="<?php echo htmlspecialchars($user['email']); ?>" required></div>
                            <button type="submit" class="btn btn-success">บันทึก</button>
                        </form>
                    </div>
                </div>
            </div>
            <div class="col-md-6">
                <div class="card">
                    <div class="card-body">
                        <h5 class="text-success">เปลี่ยนรหัสผ่าน</h5>
                        <form method="POST">
                            <input type="hidden" name="change_password" value="1">
                            <div class="mb-3"><label>รหัสผ่านปัจจุบัน</label><input type="password" name="current_password" class="form-control" required></div>
                            <div class="mb-3"><label>รหัสผ่านใหม่</label><input type="password" name="new_password" class="form-control" required minlength="6"></div>
                            <div class="mb-3"><label>ยืนยันรหัสผ่านใหม่</label><input type="password" name="confirm_password" class="form-control" required></div>
                            <button type="submit" class="btn btn-success">เปลี่ยนรหัสผ่าน</button>
                        </form>
                    </div>
                </div>
            </div>
        </div>
        <div class="card mt-4">
            <div class="card-body">
                <h5 class="text-success">ข้อมูลบัญชี</h5>
                <p><strong>เครดิตปัจจุบัน:</strong> <?php echo formatPrice($user['credit']); ?></p>
                <p><strong>สมัครสมาชิก:</strong> <?php echo date('d/m/Y', strtotime($user['created_at'])); ?></p>
                <p><strong>เข้าสู่ระบบล่าสุด:</strong> <?php echo $user['last_login'] ? date('d/m/Y H:i', strtotime($user['last_login'])) : 'ไม่มีข้อมูล'; ?></p>
                <a href="topup.php" class="btn btn-success">เติมเงิน</a>
                <a href="orders.php" class="btn btn-outline-success">ประวัติการสั่งซื้อ</a>
            </div>
        </div>
    </div>
</section>
<?php require_once 'includes/footer.php'; ?>
```

---

## STEP 5: API FILES

### api/cart.php
```php
<?php
header('Content-Type: application/json');
require_once '../config/database.php';

$action = $_POST['action'] ?? $_GET['action'] ?? '';

switch ($action) {
    case 'add':
        if (!isLoggedIn()) { echo json_encode(['success' => false, 'message' => 'กรุณาเข้าสู่ระบบ']); exit; }
        $productId = (int)($_POST['product_id'] ?? 0); $qty = (int)($_POST['quantity'] ?? 1);
        if ($productId <= 0 || $qty <= 0) { echo json_encode(['success' => false, 'message' => 'ข้อมูลไม่ถูกต้อง']); exit; }
        
        $product = $pdo->prepare("SELECT stock FROM products WHERE id = ? AND status = 'active'")->execute([$productId])->fetch();
        if (!$product) { echo json_encode(['success' => false, 'message' => 'ไม่พบสินค้า']); exit; }
        if ($product['stock'] < $qty) { echo json_encode(['success' => false, 'message' => 'สินค้าไม่เพียงพอ']); exit; }
        
        $cartItem = $pdo->prepare("SELECT id, quantity FROM cart WHERE user_id = ? AND product_id = ?")->execute([$_SESSION['user_id'], $productId])->fetch();
        if ($cartItem) {
            $newQty = $cartItem['quantity'] + $qty;
            if ($newQty > $product['stock']) { echo json_encode(['success' => false, 'message' => 'สินค้าไม่เพียงพอ']); exit; }
            $pdo->prepare("UPDATE cart SET quantity = ? WHERE id = ?")->execute([$newQty, $cartItem['id']]);
        } else $pdo->prepare("INSERT INTO cart (user_id, product_id, quantity) VALUES (?, ?, ?)") ->execute([$_SESSION['user_id'], $productId, $qty]);
        
        echo json_encode(['success' => true, 'message' => 'เพิ่มสินค้าแล้ว']);
        break;
        
    case 'remove':
        if (!isLoggedIn()) { echo json_encode(['success' => false, 'message' => 'กรุณาเข้าสู่ระบบ']); exit; }
        $productId = (int)($_POST['product_id'] ?? 0);
        $pdo->prepare("DELETE FROM cart WHERE user_id = ? AND product_id = ?")->execute([$_SESSION['user_id'], $productId]);
        echo json_encode(['success' => true, 'message' => 'ลบสินค้าแล้ว']);
        break;
        
    case 'count':
        if (!isLoggedIn()) { echo json_encode(['success' => true, 'count' => 0]); exit; }
        $count = $pdo->prepare("SELECT SUM(quantity) FROM cart WHERE user_id = ?")->execute([$_SESSION['user_id']])->fetchColumn() ?: 0;
        echo json_encode(['success' => true, 'count' => (int)$count]);
        break;
        
    default: echo json_encode(['success' => false, 'message' => 'Invalid']);
}
?>
```

### api/auth.php
```php
<?php
header('Content-Type: application/json');
require_once '../config/database.php';

$action = $_GET['action'] ?? '';

switch ($action) {
    case 'get_credit':
        if (!isLoggedIn()) { echo json_encode(['success' => false]); exit; }
        $credit = getUserCredit($_SESSION['user_id']);
        echo json_encode(['success' => true, 'credit' => $credit, 'credit_formatted' => formatPrice($credit)]);
        break;
    default: echo json_encode(['success' => false]);
}
?>
```

### api/checkout.php
```php
<?php
header('Content-Type: application/json');
session_start();
require_once '../config/database.php';

$action = $_POST['action'] ?? '';

switch ($action) {
    case 'apply_coupon':
        if (!isLoggedIn()) { echo json_encode(['success' => false, 'message' => 'กรุณาเข้าสู่ระบบ']); exit; }
        $code = strtoupper(trim($_POST['coupon_code'] ?? ''));
        
        $coupon = $pdo->prepare("SELECT * FROM coupons WHERE code = ? AND status = 'active' AND (max_uses IS NULL OR used_count < max_uses) AND (expires_at IS NULL OR expires_at > NOW())")->execute([$code])->fetch();
        
        if (!$coupon) { echo json_encode(['success' => false, 'message' => 'รหัสคูปองไม่ถูกต้องหรือหมดอายุ']); exit; }
        
        // Check min amount
        $cartTotal = $pdo->prepare("SELECT SUM(p.price * c.quantity) FROM cart c LEFT JOIN products p ON c.product_id = p.id WHERE c.user_id = ?")->execute([$_SESSION['user_id']])->fetchColumn() ?: 0;
        if ($cartTotal < $coupon['min_amount']) { echo json_encode(['success' => false, 'message' => 'ยอดสั่งซื้อขั้นต่ำ ' . formatPrice($coupon['min_amount'])]); exit; }
        
        $_SESSION['coupon_code'] = $code;
        echo json_encode(['success' => true, 'message' => 'ใช้คูปองสำเร็จ', 'discount_percent' => $coupon['discount_percent']]);
        break;
        
    default: echo json_encode(['success' => false, 'message' => 'Invalid']);
}
?>
```

### api/topup.php
```php
<?php
header('Content-Type: application/json');
session_start();
require_once '../config/database.php';

$method = $_POST['method'] ?? '';

switch ($method) {
    case 'promptpay':
        if (!isLoggedIn()) { echo json_encode(['success' => false, 'message' => 'กรุณาเข้าสู่ระบบ']); exit; }
        $amount = (int)($_POST['amount'] ?? 0);
        if ($amount <= 0) { echo json_encode(['success' => false, 'message' => 'จำนวนเงินไม่ถูกต้อง']); exit; }
        
        // Create pending topup
        $pdo->prepare("INSERT INTO topup_history (user_id, amount, method, status) VALUES (?, ?, 'promptpay', 'pending')")->execute([$_SESSION['user_id'], $amount]);
        $topupId = $pdo->lastInsertId();
        
        // Call TMW API
        $apiUrl = getSetting('promptpay_api_url');
        $username = getSetting('promptpay_username');
        $password = getSetting('promptpay_password');
        $conId = getSetting('promptpay_con_id', '1');
        
        $callUrl = "{$apiUrl}?username={$username}&password={$password}&amount={$amount}&ref1={$_SESSION['user_id']}&con_id={$conId}&ip=" . get_client_ip() . "&method=create_pay";
        $response = connect_api($callUrl);
        $result = json_decode($response, true);
        
        if ($result && $result['status'] == '1') {
            // Update with transaction info
            $pdo->prepare("UPDATE topup_history SET transaction_id = ? WHERE id = ?")->execute([$result['id_pay'], $topupId]);
            echo json_encode(['success' => true, 'qr_image' => $result['qr_image_base64'], 'amount' => $result['amount_check'] / 100, 'id_pay' => $result['id_pay']]);
        } else {
            echo json_encode(['success' => false, 'message' => $result['msg'] ?? 'สร้าง QR ล้มเหลว']);
        }
        break;
        
    case 'truewallet':
        if (!isLoggedIn()) { echo json_encode(['success' => false, 'message' => 'กรุณาเข้าสู่ระบบ']); exit; }
        $giftUrl = trim($_POST['gift_url'] ?? '');
        if (!preg_match('/gift\.truemoney\.com/', $giftUrl)) { echo json_encode(['success' => false, 'message' => 'ลิงก์ไม่ถูกต้อง']); exit; }
        
        // Extract transaction ID from URL
        $tranId = basename(parse_url($giftUrl, PHP_URL_PATH));
        
        // Call TMW API
        $apiUrl = getSetting('truewallet_api_url');
        $username = getSetting('truewallet_username');
        $password = getSetting('truewallet_password');
        $mobile = getSetting('truewallet_mobile');
        
        $callUrl = "{$apiUrl}?username={$username}&password={$password}&tmemail={$mobile}&transactionid={$tranId}&clientip=" . urlencode(get_client_ip()) . "&ref1={$_SESSION['user_id']}&action=yes&json=1";
        $response = connect_api($callUrl);
        $result = json_decode($response, true);
        
        if ($result && $result['Status'] == 'check_success') {
            $amount = $result['Amount'];
            
            $pdo->beginTransaction();
            try {
                $pdo->prepare("INSERT INTO topup_history (user_id, amount, method, transaction_id, status) VALUES (?, ?, 'truewallet', ?, 'completed')")->execute([$_SESSION['user_id'], $amount, $tranId]);
                updateUserCredit($_SESSION['user_id'], $amount);
                $pdo->commit();
                
                if (getSetting('discord_payment_enabled') == 'true') {
                    sendDiscordWebhook("💰 เติมเงิน TrueWallet: ฿{$amount} - โดย {$_SESSION['username']}");
                }
                
                echo json_encode(['success' => true, 'message' => "เติมเงินสำเร็จ ฿{$amount}", 'amount' => $amount]);
            } catch (Exception $e) {
                $pdo->rollBack();
                echo json_encode(['success' => false, 'message' => 'เกิดข้อผิดพลาด']);
            }
        } else {
            echo json_encode(['success' => false, 'message' => $result['Msg'] ?? 'ตรวจสอบลิงก์ล้มเหลว']);
        }
        break;
        
    default: echo json_encode(['success' => false, 'message' => 'Invalid method']);
}
?>
```

### api/products.php
```php
<?php
header('Content-Type: application/json');
require_once '../config/database.php';

$action = $_GET['action'] ?? '';

switch ($action) {
    case 'search':
        $query = $_GET['query'] ?? '';
        if (strlen($query) < 2) { echo json_encode(['products' => []]); exit; }
        
        $stmt = $pdo->prepare("SELECT id, name, price, image_url FROM products WHERE status = 'active' AND (name LIKE ? OR description LIKE ?) LIMIT 10");
        $stmt->execute(["%$query%", "%$query%"]);
        $products = $stmt->fetchAll();
        
        foreach ($products as &$p) $p['price'] = formatPrice($p['price']);
        
        echo json_encode(['products' => $products]);
        break;
        
    default: echo json_encode(['products' => []]);
}
?>
```

---

## STEP 6: ADMIN PANEL

### admin/index.php (Dashboard)
```php
<?php
session_start();
require_once '../config/database.php';
if (!isLoggedIn() || $_SESSION['user_id'] != 1) { header('Location: ../login.php'); exit; }

$pageTitle = 'แดชบอร์ด';
$stats = [
    'products' => $pdo->query("SELECT COUNT(*) FROM products")->fetchColumn(),
    'users' => $pdo->query("SELECT COUNT(*) FROM users")->fetchColumn(),
    'orders' => $pdo->query("SELECT COUNT(*) FROM orders")->fetchColumn(),
    'revenue' => $pdo->query("SELECT SUM(total_amount) FROM orders WHERE status = 'completed'")->fetchColumn() ?: 0
];
$recentOrders = $pdo->query("SELECT o.*, u.username FROM orders o LEFT JOIN users u ON o.user_id = u.id ORDER BY o.created_at DESC LIMIT 10")->fetchAll();
?>
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title><?php echo $pageTitle; ?></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    <style>
        body { font-family: 'Sarabun', sans-serif; background: #1a1a1a; color: #fff; }
        .sidebar { background: #0d0d0d; min-height: 100vh; }
        .sidebar .nav-link { color: #fff; }
        .sidebar .nav-link:hover, .sidebar .nav-link.active { color: #80ef25; }
        .card { background: #2a2a2a; border: 1px solid #404040; }
        .text-success { color: #80ef25 !important; }
    </style>
</head>
<body>
    <div class="container-fluid">
        <div class="row">
            <nav class="col-md-2 sidebar p-3">
                <h4 class="text-success mb-4">Admin</h4>
                <ul class="nav flex-column">
                    <li class="nav-item"><a class="nav-link active" href="index.php"><i class="bi bi-speedometer2"></i> แดชบอร์ด</a></li>
                    <li class="nav-item"><a class="nav-link" href="products.php"><i class="bi bi-box"></i> สินค้า</a></li>
                    <li class="nav-item"><a class="nav-link" href="categories.php"><i class="bi bi-tags"></i> หมวดหมู่</a></li>
                    <li class="nav-item"><a class="nav-link" href="orders.php"><i class="bi bi-cart3"></i> คำสั่งซื้อ</a></li>
                    <li class="nav-item"><a class="nav-link" href="users.php"><i class="bi bi-people"></i> สมาชิก</a></li>
                    <li class="nav-item"><a class="nav-link" href="coupons.php"><i class="bi bi-ticket-perforated"></i> คูปอง</a></li>
                    <li class="nav-item"><a class="nav-link" href="redeem_codes.php"><i class="bi bi-gift"></i> รหัสเติมเงิน</a></li>
                    <li class="nav-item"><a class="nav-link" href="settings.php"><i class="bi bi-gear"></i> ตั้งค่า</a></li>
                    <li class="nav-item mt-3"><a class="nav-link text-danger" href="../logout.php"><i class="bi bi-box-arrow-right"></i> ออก</a></li>
                </ul>
            </nav>
            <main class="col-md-10 p-4">
                <h2 class="mb-4 text-success">แดชบอร์ด</h2>
                <div class="row mb-4">
                    <div class="col-md-3"><div class="card p-3 text-center"><h3 class="text-success"><?php echo $stats['products']; ?></h3><p>สินค้า</p></div></div>
                    <div class="col-md-3"><div class="card p-3 text-center"><h3 class="text-success"><?php echo $stats['users']; ?></h3><p>สมาชิก</p></div></div>
                    <div class="col-md-3"><div class="card p-3 text-center"><h3 class="text-success"><?php echo $stats['orders']; ?></h3><p>คำสั่งซื้อ</p></div></div>
                    <div class="col-md-3"><div class="card p-3 text-center"><h3 class="text-success"><?php echo formatPrice($stats['revenue']); ?></h3><p>รายได้</p></div></div>
                </div>
                <div class="card">
                    <div class="card-header"><h5 class="mb-0 text-success">คำสั่งซื้อล่าสุด</h5></div>
                    <div class="card-body">
                        <table class="table table-dark">
                            <thead><tr><th>เลขที่</th><th>ลูกค้า</th><th>ยอด</th><th>สถานะ</th><th>วันที่</th></tr></thead>
                            <tbody>
                                <?php foreach ($recentOrders as $o): ?>
                                <tr>
                                    <td><a href="order_view.php?id=<?php echo $o['id']; ?>" class="text-success"><?php echo $o['order_number']; ?></a></td>
                                    <td><?php echo $o['username']; ?></td>
                                    <td><?php echo formatPrice($o['total_amount']); ?></td>
                                    <td><span class="badge bg-<?php echo $o['status'] === 'completed' ? 'success' : 'warning'; ?>"><?php echo $o['status']; ?></span></td>
                                    <td><?php echo date('d/m/Y H:i', strtotime($o['created_at'])); ?></td>
                                </tr>
                                <?php endforeach; ?>
                            </tbody>
                        </table>
                    </div>
                </div>
            </main>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### admin/products.php (Product Management with QuillJS)
```php
<?php
session_start();
require_once '../config/database.php';
if (!isLoggedIn() || $_SESSION['user_id'] != 1) { header('Location: ../login.php'); exit; }

$pageTitle = 'จัดการสินค้า';
$action = $_GET['action'] ?? 'list';

// Delete
if ($action === 'delete' && isset($_GET['id'])) {
    $pdo->prepare("DELETE FROM products WHERE id = ?")->execute([$_GET['id']]);
    header('Location: products.php'); exit;
}

// Add/Edit
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $id = $_POST['id'] ?? null;
    $name = trim($_POST['name'] ?? '');
    $slug = strtolower(trim(preg_replace('/[^a-zA-Z0-9]+/', '-', $name), '-'));
    $category_id = (int)($_POST['category_id'] ?? 0);
    $price = (float)($_POST['price'] ?? 0);
    $stock = (int)($_POST['stock'] ?? 0);
    $description = $_POST['description'] ?? '';
    $image_url = trim($_POST['image_url'] ?? '');
    $video_url = trim($_POST['video_url'] ?? '');
    $status = $_POST['status'] ?? 'active';
    
    // Handle gallery (multiple URLs, comma separated)
    $gallery = [];
    if (!empty($_POST['gallery'])) {
        $gallery = array_map('trim', explode("\n", $_POST['gallery']));
        $gallery = array_filter($gallery);
    }
    $galleryJson = json_encode(array_values($gallery));
    
    if ($id) {
        $pdo->prepare("UPDATE products SET name=?, slug=?, category_id=?, price=?, stock=?, description=?, image_url=?, gallery=?, video_url=?, status=? WHERE id=?")
            ->execute([$name, $slug, $category_id, $price, $stock, $description, $image_url, $galleryJson, $video_url, $status, $id]);
    } else {
        $pdo->prepare("INSERT INTO products (name, slug, category_id, price, stock, description, image_url, gallery, video_url, status) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)")
            ->execute([$name, $slug, $category_id, $price, $stock, $description, $image_url, $galleryJson, $video_url, $status]);
    }
    header('Location: products.php'); exit;
}

// Get data
$categories = $pdo->query("SELECT * FROM categories ORDER BY name")->fetchAll();
$products = $pdo->query("SELECT p.*, c.name as category_name FROM products p LEFT JOIN categories c ON p.category_id = c.id ORDER BY p.created_at DESC")->fetchAll();
$editProduct = null;
if ($action === 'edit' && isset($_GET['id'])) {
    $editProduct = $pdo->prepare("SELECT * FROM products WHERE id = ?")->execute([$_GET['id']])->fetch();
}
?>
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title><?php echo $pageTitle; ?></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    <link href="https://cdn.quilljs.com/1.3.6/quill.snow.css" rel="stylesheet">
    <style>
        body { font-family: 'Sarabun', sans-serif; background: #1a1a1a; color: #fff; }
        .sidebar { background: #0d0d0d; min-height: 100vh; }
        .sidebar .nav-link { color: #fff; }
        .sidebar .nav-link:hover, .sidebar .nav-link.active { color: #80ef25; }
        .card { background: #2a2a2a; border: 1px solid #404040; }
        .text-success { color: #80ef25 !important; }
        .btn-success { background: #80ef25; border-color: #80ef25; color: #0d0d0d; }
        .quill-editor { background: #fff; color: #000; }
    </style>
</head>
<body>
    <div class="container-fluid">
        <div class="row">
            <nav class="col-md-2 sidebar p-3">
                <h4 class="text-success mb-4">Admin</h4>
                <ul class="nav flex-column">
                    <li class="nav-item"><a class="nav-link" href="index.php"><i class="bi bi-speedometer2"></i> แดชบอร์ด</a></li>
                    <li class="nav-item"><a class="nav-link active" href="products.php"><i class="bi bi-box"></i> สินค้า</a></li>
                    <li class="nav-item"><a class="nav-link" href="categories.php"><i class="bi bi-tags"></i> หมวดหมู่</a></li>
                    <li class="nav-item"><a class="nav-link" href="orders.php"><i class="bi bi-cart3"></i> คำสั่งซื้อ</a></li>
                    <li class="nav-item"><a class="nav-link" href="users.php"><i class="bi bi-people"></i> สมาชิก</a></li>
                    <li class="nav-item"><a class="nav-link" href="coupons.php"><i class="bi bi-ticket-perforated"></i> คูปอง</a></li>
                    <li class="nav-item"><a class="nav-link" href="redeem_codes.php"><i class="bi bi-gift"></i> รหัสเติมเงิน</a></li>
                    <li class="nav-item"><a class="nav-link" href="settings.php"><i class="bi bi-gear"></i> ตั้งค่า</a></li>
                    <li class="nav-item mt-3"><a class="nav-link text-danger" href="../logout.php"><i class="bi bi-box-arrow-right"></i> ออก</a></li>
                </ul>
            </nav>
            <main class="col-md-10 p-4">
                <?php if ($action === 'add' || $action === 'edit'): ?>
                <h2 class="mb-4 text-success"><?php echo $action === 'edit' ? 'แก้ไข' : 'เพิ่ม'; ?>สินค้า</h2>
                <div class="card">
                    <div class="card-body">
                        <form method="POST">
                            <?php if ($editProduct): ?><input type="hidden" name="id" value="<?php echo $editProduct['id']; ?>"><?php endif; ?>
                            <div class="row">
                                <div class="col-md-6 mb-3">
                                    <label>ชื่อสินค้า</label>
                                    <input type="text" name="name" class="form-control" value="<?php echo $editProduct['name'] ?? ''; ?>" required>
                                </div>
                                <div class="col-md-6 mb-3">
                                    <label>หมวดหมู่</label>
                                    <select name="category_id" class="form-select" required>
                                        <?php foreach ($categories as $c): ?>
                                        <option value="<?php echo $c['id']; ?>" <?php echo ($editProduct['category_id'] ?? '') == $c['id'] ? 'selected' : ''; ?>><?php echo $c['name']; ?></option>
                                        <?php endforeach; ?>
                                    </select>
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-md-4 mb-3">
                                    <label>ราคา (บาท)</label>
                                    <input type="number" name="price" class="form-control" step="0.01" value="<?php echo $editProduct['price'] ?? ''; ?>" required>
                                </div>
                                <div class="col-md-4 mb-3">
                                    <label>สต็อก</label>
                                    <input type="number" name="stock" class="form-control" value="<?php echo $editProduct['stock'] ?? '0'; ?>" required>
                                </div>
                                <div class="col-md-4 mb-3">
                                    <label>สถานะ</label>
                                    <select name="status" class="form-select">
                                        <option value="active" <?php echo ($editProduct['status'] ?? '') === 'active' ? 'selected' : ''; ?>>ใช้งาน</option>
                                        <option value="inactive" <?php echo ($editProduct['status'] ?? '') === 'inactive' ? 'selected' : ''; ?>>ไม่ใช้งาน</option>
                                    </select>
                                </div>
                            </div>
                            <div class="mb-3">
                                <label>รูปภาพหลัก (URL)</label>
                                <input type="url" name="image_url" class="form-control" value="<?php echo $editProduct['image_url'] ?? ''; ?>" placeholder="https://example.com/image.jpg">
                            </div>
                            <div class="mb-3">
                                <label>รูปภาพเพิ่มเติม (URL หนึ่งบรรทัดต่อรูป)</label>
                                <textarea name="gallery" class="form-control" rows="3" placeholder="https://example.com/img1.jpg
https://example.com/img2.jpg"><?php 
                                    if ($editProduct && $editProduct['gallery']) {
                                        $gallery = json_decode($editProduct['gallery'], true);
                                        echo implode("\n", $gallery);
                                    }
                                ?></textarea>
                            </div>
                            <div class="mb-3">
                                <label>คลิปรีวิว YouTube (embed URL)</label>
                                <input type="url" name="video_url" class="form-control" value="<?php echo $editProduct['video_url'] ?? ''; ?>" placeholder="https://www.youtube.com/embed/xxxxx">
                            </div>
                            <div class="mb-3">
                                <label>รายละเอียด (ใช้ QuillJS Editor)</label>
                                <div id="editor" class="quill-editor" style="height: 200px;"></div>
                                <input type="hidden" name="description" id="description">
                            </div>
                            <button type="submit" class="btn btn-success">บันทึก</button>
                            <a href="products.php" class="btn btn-secondary">ยกเลิก</a>
                        </form>
                    </div>
                </div>
                <?php else: ?>
                <div class="d-flex justify-content-between align-items-center mb-4">
                    <h2 class="text-success">จัดการสินค้า</h2>
                    <a href="products.php?action=add" class="btn btn-success"><i class="bi bi-plus"></i> เพิ่มสินค้า</a>
                </div>
                <div class="card">
                    <div class="card-body">
                        <div class="table-responsive">
                            <table class="table table-dark">
                                <thead><tr><th>ID</th><th>รูป</th><th>ชื่อ</th><th>หมวดหมู่</th><th>ราคา</th><th>สต็อก</th><th>สถานะ</th><th>จัดการ</th></tr></thead>
                                <tbody>
                                    <?php foreach ($products as $p): ?>
                                    <tr>
                                        <td><?php echo $p['id']; ?></td>
                                        <td><img src="<?php echo $p['image_url'] ?: 'https://via.placeholder.com/50'; ?>" height="50"></td>
                                        <td><?php echo htmlspecialchars($p['name']); ?></td>
                                        <td><?php echo $p['category_name']; ?></td>
                                        <td><?php echo formatPrice($p['price']); ?></td>
                                        <td><?php echo $p['stock']; ?></td>
                                        <td><span class="badge bg-<?php echo $p['status'] === 'active' ? 'success' : 'secondary'; ?>"><?php echo $p['status'] === 'active' ? 'ใช้งาน' : 'ไม่ใช้งาน'; ?></span></td>
                                        <td>
                                            <a href="products.php?action=edit&id=<?php echo $p['id']; ?>" class="btn btn-sm btn-warning"><i class="bi bi-pencil"></i></a>
                                            <a href="products.php?action=delete&id=<?php echo $p['id']; ?>" class="btn btn-sm btn-danger" onclick="return confirm('ยืนยันการลบ?')"><i class="bi bi-trash"></i></a>
                                        </td>
                                    </tr>
                                    <?php endforeach; ?>
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
                <?php endif; ?>
            </main>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://cdn.quilljs.com/1.3.6/quill.js"></script>
    <script>
    var quill = new Quill('#editor', { theme: 'snow' });
    <?php if ($editProduct && $editProduct['description']): ?>
    quill.root.innerHTML = <?php echo json_encode($editProduct['description']); ?>;
    <?php endif; ?>
    document.querySelector('form').onsubmit = function() {
        document.getElementById('description').value = quill.root.innerHTML;
    };
    </script>
</body>
</html>
```

### admin/settings.php (General Settings)
```php
<?php
session_start();
require_once '../config/database.php';
if (!isLoggedIn() || $_SESSION['user_id'] != 1) { header('Location: ../login.php'); exit; }

$pageTitle = 'ตั้งค่า';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    foreach ($_POST as $key => $value) {
        if (strpos($key, 'setting_') === 0) {
            $settingKey = substr($key, 8);
            $stmt = $pdo->prepare("INSERT INTO settings (setting_key, setting_value) VALUES (?, ?) ON DUPLICATE KEY UPDATE setting_value = ?");
            $stmt->execute([$settingKey, $value, $value]);
        }
    }
    echo "<script>alert('บันทึกการตั้งค่าสำเร็จ');</script>";
}

$settings = [];
$stmt = $pdo->query("SELECT * FROM settings");
while ($row = $stmt->fetch()) $settings[$row['setting_key']] = $row['setting_value'];
?>
<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <title><?php echo $pageTitle; ?></title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.0/font/bootstrap-icons.css">
    <style>
        body { font-family: 'Sarabun', sans-serif; background: #1a1a1a; color: #fff; }
        .sidebar { background: #0d0d0d; min-height: 100vh; }
        .sidebar .nav-link { color: #fff; }
        .sidebar .nav-link:hover, .sidebar .nav-link.active { color: #80ef25; }
        .card { background: #2a2a2a; border: 1px solid #404040; }
        .text-success { color: #80ef25 !important; }
        .btn-success { background: #80ef25; border-color: #80ef25; color: #0d0d0d; }
        .nav-tabs .nav-link { color: #fff; }
        .nav-tabs .nav-link.active { background: #80ef25; color: #0d0d0d; }
    </style>
</head>
<body>
    <div class="container-fluid">
        <div class="row">
            <nav class="col-md-2 sidebar p-3">
                <h4 class="text-success mb-4">Admin</h4>
                <ul class="nav flex-column">
                    <li class="nav-item"><a class="nav-link" href="index.php"><i class="bi bi-speedometer2"></i> แดชบอร์ด</a></li>
                    <li class="nav-item"><a class="nav-link" href="products.php"><i class="bi bi-box"></i> สินค้า</a></li>
                    <li class="nav-item"><a class="nav-link" href="categories.php"><i class="bi bi-tags"></i> หมวดหมู่</a></li>
                    <li class="nav-item"><a class="nav-link" href="orders.php"><i class="bi bi-cart3"></i> คำสั่งซื้อ</a></li>
                    <li class="nav-item"><a class="nav-link" href="users.php"><i class="bi bi-people"></i> สมาชิก</a></li>
                    <li class="nav-item"><a class="nav-link" href="coupons.php"><i class="bi bi-ticket-perforated"></i> คูปอง</a></li>
                    <li class="nav-item"><a class="nav-link" href="redeem_codes.php"><i class="bi bi-gift"></i> รหัสเติมเงิน</a></li>
                    <li class="nav-item"><a class="nav-link active" href="settings.php"><i class="bi bi-gear"></i> ตั้งค่า</a></li>
                    <li class="nav-item mt-3"><a class="nav-link text-danger" href="../logout.php"><i class="bi bi-box-arrow-right"></i> ออก</a></li>
                </ul>
            </nav>
            <main class="col-md-10 p-4">
                <h2 class="mb-4 text-success">ตั้งค่าระบบ</h2>
                
                <ul class="nav nav-tabs mb-4">
                    <li class="nav-item"><a class="nav-link active" data-bs-toggle="tab" href="#general">ทั่วไป</a></li>
                    <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#payment">การชำระเงิน</a></li>
                    <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#security">ความปลอดภัย</a></li>
                    <li class="nav-item"><a class="nav-link" data-bs-toggle="tab" href="#notifications">การแจ้งเตือน</a></li>
                </ul>
                
                <form method="POST">
                    <div class="tab-content">
                        <div class="tab-pane active" id="general">
                            <div class="card">
                                <div class="card-body">
                                    <h5 class="text-success mb-3">ตั้งค่าทั่วไป</h5>
                                    <div class="mb-3"><label>ชื่อเว็บไซต์</label><input type="text" name="setting_site_name" class="form-control" value="<?php echo $settings['site_name'] ?? 'GameStore'; ?>"></div>
                                    <div class="mb-3"><label>โลโก้ (URL หรือข้อความ)</label><input type="text" name="setting_site_logo" class="form-control" value="<?php echo $settings['site_logo'] ?? ''; ?>" placeholder="เว้นว่างเพื่อใช้ชื่อเว็บ"></div>
                                    <div class="mb-3"><label>คำอธิบายเว็บ</label><input type="text" name="setting_site_description" class="form-control" value="<?php echo $settings['site_description'] ?? ''; ?>"></div>
                                    <div class="mb-3"><label>ข้อมูลติดต่อ</label><textarea name="setting_contact_info" class="form-control" rows="3"><?php echo $settings['contact_info'] ?? ''; ?></textarea></div>
                                    <hr><h6 class="text-success">หน้าแรก (Hero Section)</h6>
                                    <div class="mb-3"><label>หัวข้อ</label><input type="text" name="setting_hero_title" class="form-control" value="<?php echo $settings['hero_title'] ?? ''; ?>"></div>
                                    <div class="mb-3"><label>คำอธิบาย</label><input type="text" name="setting_hero_subtitle" class="form-control" value="<?php echo $settings['hero_subtitle'] ?? ''; ?>"></div>
                                    <div class="mb-3"><label>ข้อความปุ่ม</label><input type="text" name="setting_hero_button_text" class="form-control" value="<?php echo $settings['hero_button_text'] ?? ''; ?>"></div>
                                    <div class="mb-3"><label>รูปภาพ Hero (JSON array)</label><textarea name="setting_hero_images" class="form-control" rows="3"><?php echo $settings['hero_images'] ?? '[]'; ?></textarea></div>
                                </div>
                            </div>
                        </div>
                        
                        <div class="tab-pane" id="payment">
                            <div class="card">
                                <div class="card-body">
                                    <h5 class="text-success mb-3">PromptPay (TMW API)</h5>
                                    <div class="mb-3"><div class="form-check form-switch"><input class="form-check-input" type="checkbox" name="setting_promptpay_enabled" value="true" <?php echo ($settings['promptpay_enabled'] ?? '') === 'true' ? 'checked' : ''; ?>><label class="form-check-label">เปิดใช้งาน PromptPay</label></div></div>
                                    <div class="mb-3"><label>API URL</label><input type="url" name="setting_promptpay_api_url" class="form-control" value="<?php echo $settings['promptpay_api_url'] ?? 'https://tmwallet.thaighost.net/apiwallet.php'; ?>"></div>
                                    <div class="row">
                                        <div class="col-md-6 mb-3"><label>Username</label><input type="text" name="setting_promptpay_username" class="form-control" value="<?php echo $settings['promptpay_username'] ?? ''; ?>"></div>
                                        <div class="col-md-6 mb-3"><label>Password</label><input type="password" name="setting_promptpay_password" class="form-control" value="<?php echo $settings['promptpay_password'] ?? ''; ?>"></div>
                                    </div>
                                    <div class="row">
                                        <div class="col-md-6 mb-3"><label>เบอร์พร้อมเพย์</label><input type="text" name="setting_promptpay_number" class="form-control" value="<?php echo $settings['promptpay_number'] ?? ''; ?>"></div>
                                        <div class="col-md-6 mb-3"><label>ชื่อบัญชี</label><input type="text" name="setting_promptpay_name" class="form-control" value="<?php echo $settings['promptpay_name'] ?? ''; ?>"></div>
                                    </div>
                                    <div class="mb-3"><label>ประเภท (01=มือถือ, 02=บัตรปชช, 03=E-Wallet)</label><input type="text" name="setting_promptpay_type" class="form-control" value="<?php echo $settings['promptpay_type'] ?? '01'; ?>"></div>
                                    
                                    <hr><h5 class="text-success mb-3">TrueWallet Gift (TMW API)</h5>
                                    <div class="mb-3"><div class="form-check form-switch"><input class="form-check-input" type="checkbox" name="setting_truewallet_enabled" value="true" <?php echo ($settings['truewallet_enabled'] ?? '') === 'true' ? 'checked' : ''; ?>><label class="form-check-label">เปิดใช้งาน TrueWallet</label></div></div>
                                    <div class="mb-3"><label>API URL</label><input type="url" name="setting_truewallet_api_url" class="form-control" value="<?php echo $settings['truewallet_api_url'] ?? 'https://tmwallet.thaighost.net/apiwallet.php'; ?>"></div>
                                    <div class="row">
                                        <div class="col-md-6 mb-3"><label>Username</label><input type="text" name="setting_truewallet_username" class="form-control" value="<?php echo $settings['truewallet_username'] ?? ''; ?>"></div>
                                        <div class="col-md-6 mb-3"><label>Password</label><input type="password" name="setting_truewallet_password" class="form-control" value="<?php echo $settings['truewallet_password'] ?? ''; ?>"></div>
                                    </div>
                                    <div class="mb-3"><label>เบอร์มือถือ TrueWallet</label><input type="text" name="setting_truewallet_mobile" class="form-control" value="<?php echo $settings['truewallet_mobile'] ?? ''; ?>"></div>
                                </div>
                            </div>
                        </div>
                        
                        <div class="tab-pane" id="security">
                            <div class="card">
                                <div class="card-body">
                                    <h5 class="text-success mb-3">Cloudflare Turnstile</h5>
                                    <div class="mb-3"><div class="form-check form-switch"><input class="form-check-input" type="checkbox" name="setting_turnstile_enabled" value="true" <?php echo ($settings['turnstile_enabled'] ?? '') === 'true' ? 'checked' : ''; ?>><label class="form-check-label">เปิดใช้งาน Turnstile (สมัคร/เข้าสู่ระบบ)</label></div></div>
                                    <div class="mb-3"><label>Site Key</label><input type="text" name="setting_turnstile_site_key" class="form-control" value="<?php echo $settings['turnstile_site_key'] ?? ''; ?>"></div>
                                    <div class="mb-3"><label>Secret Key</label><input type="password" name="setting_turnstile_secret_key" class="form-control" value="<?php echo $settings['turnstile_secret_key'] ?? ''; ?>"></div>
                                </div>
                            </div>
                        </div>
                        
                        <div class="tab-pane" id="notifications">
                            <div class="card">
                                <div class="card-body">
                                    <h5 class="text-success mb-3">Discord Webhook</h5>
                                    <div class="mb-3"><label>Webhook URL</label><input type="url" name="setting_discord_webhook_url" class="form-control" value="<?php echo $settings['discord_webhook_url'] ?? ''; ?>"></div>
                                    <div class="mb-3"><div class="form-check form-switch"><input class="form-check-input" type="checkbox" name="setting_discord_new_order_enabled" value="true" <?php echo ($settings['discord_new_order_enabled'] ?? '') === 'true' ? 'checked' : ''; ?>><label class="form-check-label">แจ้งเตือนเมื่อมีคำสั่งซื้อใหม่</label></div></div>
                                    <div class="mb-3"><div class="form-check form-switch"><input class="form-check-input" type="checkbox" name="setting_discord_payment_enabled" value="true" <?php echo ($settings['discord_payment_enabled'] ?? '') === 'true' ? 'checked' : ''; ?>><label class="form-check-label">แจ้งเตือนเมื่อมีเงินเข้า</label></div></div>
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <div class="mt-4">
                        <button type="submit" class="btn btn-success btn-lg">บันทึกการตั้งค่า</button>
                    </div>
                </form>
            </main>
        </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

---

## STEP 7: WEBHOOK HANDLERS

### api/webhook/promptpay.php
```php
<?php
// Webhook for PromptPay payment confirmation
require_once '../../config/database.php';

$id_pay = $_GET['id_pay'] ?? '';
$amount = $_GET['amount'] ?? 0;
$user_id = $_GET['ref1'] ?? 0;

if (!$id_pay || !$amount || !$user_id) {
    http_response_code(400);
    echo "Invalid parameters";
    exit;
}

// Verify with TMW API
$apiUrl = getSetting('promptpay_api_url');
$username = getSetting('promptpay_username');
$password = getSetting('promptpay_password');
$conId = getSetting('promptpay_con_id', '1');

$verifyUrl = "{$apiUrl}?username={$username}&password={$password}&con_id={$conId}&id_pay={$id_pay}&method=detail_pay";
$response = connect_api($verifyUrl);
$result = json_decode($response, true);

if ($result && $result['status'] == '1' && $result['pay_ok'] == '1') {
    $amount = $result['amount_check'] / 100; // Convert from satang
    
    // Update topup history
    $stmt = $pdo->prepare("UPDATE topup_history SET status = 'completed' WHERE transaction_id = ? AND user_id = ?");
    $stmt->execute([$id_pay, $user_id]);
    
    // Add credit
    updateUserCredit($user_id, $amount);
    
    // Discord notification
    if (getSetting('discord_payment_enabled') == 'true') {
        sendDiscordWebhook("💰 เติมเงิน PromptPay สำเร็จ: ฿{$amount} - User ID: {$user_id}");
    }
    
    echo "OK";
} else {
    http_response_code(400);
    echo "Payment not confirmed";
}
?>
```

---

## STEP 8: README.md

### README.md
```markdown
# GameStore - PHP Digital Products E-commerce

ระบบร้านค้าออนไลน์สำหรับขายสินค้าดิจิทัล (Netflix, Spotify, Steam, etc.)

## ความต้องการ

- PHP 8.0+
- MariaDB 10.3+
- ส่วนขยาย PHP: PDO, curl, json

## การติดตั้ง

1. อัปโหลดไฟล์ทั้งหมดไปยังเซิร์ฟเวอร์
2. สร้างฐานข้อมูล MariaDB
3. รันไฟล์ `database.sql` เพื่อสร้างตาราง
4. รันไฟล์ `seed.sql` เพื่อเพิ่มข้อมูลเริ่มต้น
5. แก้ไข `config/database.php` ด้วยข้อมูลฐานข้อมูลของคุณ
6. สร้างบัญชีแอดมิน (user_id = 1 จะเป็นแอดมินอัตโนมัติ)
7. เข้าสู่ระบบแอดมินและตั้งค่า API การชำระเงิน

## การใช้งาน

### สำหรับลูกค้า
1. สมัครสมาชิก / เข้าสู่ระบบ
2. เลือกซื้อสินค้า เพิ่มลงตะกร้า
3. เติมเงินเข้ากระเป๋า (พร้อมเพย์ / TrueWallet / รหัสเติมเงิน)
4. ชำระเงินด้วยเครดิต
5. รอรับสินค้า (Manual/Auto)

### สำหรับแอดมิน
- URL: `/admin/`
- User ID 1 เป็นแอดมินโดยอัตโนมัติ
- จัดการสินค้า, หมวดหมู่, คำสั่งซื้อ, สมาชิก, คูปอง, รหัสเติมเงิน, ตั้งค่าระบบ

## ระบบชำระเงิน

### 1. PromptPay (QR Code)
- ใช้ TMW Easy API
- สร้าง QR Code ให้ลูกค้าสแกนจ่าย
- ระบบตรวจสอบอัตโนมัติ

### 2. TrueWallet Gift
- ลูกค้าส่งลิงก์ซองของขวัญ
- ระบบตรวจสอบและเติมเงินอัตโนมัติ

### 3. Redeem Code
- แอดมินสร้างรหัส REEDEM-XXXXXX
- ลูกค้านำรหัสมาใช้เติมเงิน

## ความปลอดภัย

- PDO Prepared Statements (ป้องกัน SQL Injection)
- Password Hashing (bcrypt)
- Cloudflare Turnstile (ป้องกันบอท)
- XSS Protection
- CSRF Protection (ควรเพิ่ม token ในการปรับปรุง)

## การปรับแต่ง

- ธีม: แก้ไข CSS ใน `includes/header.php`
- สีหลัก: #80ef25 (Lime Green)
- ฟอนต์: Kanit (หัวข้อ), Sarabun (เนื้อหา)

## การสนับสนุน

ติดต่อผู้พัฒนาสำหรับความช่วยเหลือ
```

---

## FINAL CHECKLIST

- [x] Database schema (database.sql + seed.sql)
- [x] Config files (database.php with helper functions)
- [x] Core files (.htaccess, index.php, header.php, footer.php)
- [x] Frontend pages (home, products, product detail, cart, checkout, topup, orders, login, register, profile, contact)
- [x] API endpoints (cart, auth, checkout, topup, products)
- [x] Admin panel (dashboard, products with QuillJS, settings)
- [x] Payment systems (PromptPay, TrueWallet, Redeem Code)
- [x] Webhook handlers
- [x] AJAX real-time features
- [x] SweetAlert2 for all alerts
- [x] Thai language throughout
- [x] Dark theme with Lime Green
- [x] Cloudflare Turnstile integration
- [x] Discord webhook notifications
- [x] README.md

---

## INSTRUCTIONS FOR AI

1. Create ALL files exactly as specified above
2. Ensure ALL user-facing text is in Thai
3. Use SweetAlert2 for ALL alerts/notifications
4. Implement ALL AJAX features as specified
5. Follow the exact color scheme (Dark + Lime Green #80ef25)
6. Use Bootstrap 5 classes, NO inline styles
7. Use PDO prepared statements for ALL database queries
8. Ensure responsive design works on mobile
9. Test ALL functionality before completing
10. Include ALL files in the file structure

This is a COMPLETE specification for a professional PHP e-commerce website.
