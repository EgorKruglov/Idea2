~~~~sql
-- Create the database
CREATE DATABASE IF NOT EXISTS ecommerce_store;
USE ecommerce_store;

-- Users/Customers table
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

-- User addresses
CREATE TABLE user_addresses (
    address_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    address_line1 VARCHAR(100) NOT NULL,
    address_line2 VARCHAR(100),
    city VARCHAR(50) NOT NULL,
    state VARCHAR(50) NOT NULL,
    postal_code VARCHAR(20) NOT NULL,
    country VARCHAR(50) NOT NULL DEFAULT 'United States',
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Product categories
CREATE TABLE categories (
    category_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT,
    parent_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_id) REFERENCES categories(category_id) ON DELETE SET NULL
);

-- Products table
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    category_id INT,
    sku VARCHAR(50) UNIQUE,
    stock_quantity INT NOT NULL DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(category_id) ON DELETE SET NULL
);

-- Product images
CREATE TABLE product_images (
    image_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    image_url VARCHAR(255) NOT NULL,
    alt_text VARCHAR(100),
    is_primary BOOLEAN DEFAULT FALSE,
    display_order INT DEFAULT 0,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE
);

-- Product reviews
CREATE TABLE product_reviews (
    review_id INT AUTO_INCREMENT PRIMARY KEY,
    product_id INT NOT NULL,
    user_id INT NOT NULL,
    rating TINYINT NOT NULL CHECK (rating BETWEEN 1 AND 5),
    title VARCHAR(100),
    comment TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Shopping carts
CREATE TABLE carts (
    cart_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Cart items
CREATE TABLE cart_items (
    cart_item_id INT AUTO_INCREMENT PRIMARY KEY,
    cart_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL DEFAULT 1 CHECK (quantity > 0),
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (cart_id) REFERENCES carts(cart_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE,
    UNIQUE KEY (cart_id, product_id)
);

-- Orders table
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    shipping_address_id INT NOT NULL,
    payment_method VARCHAR(50),
    tracking_number VARCHAR(100),
    notes TEXT,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (shipping_address_id) REFERENCES user_addresses(address_id)
);

-- Order items
CREATE TABLE order_items (
    order_item_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL,
    subtotal DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Payments table
CREATE TABLE payments (
    payment_id INT AUTO_INCREMENT PRIMARY KEY,
    order_id INT NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    payment_method VARCHAR(50) NOT NULL,
    transaction_id VARCHAR(100),
    status ENUM('pending', 'completed', 'failed', 'refunded') DEFAULT 'pending',
    payment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

-- Coupons/discounts
CREATE TABLE coupons (
    coupon_id INT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(20) NOT NULL UNIQUE,
    discount_type ENUM('percentage', 'fixed') NOT NULL,
    discount_value DECIMAL(10, 2) NOT NULL,
    min_order_amount DECIMAL(10, 2) DEFAULT 0,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP NOT NULL,
    max_uses INT,
    current_uses INT DEFAULT 0,
    is_active BOOLEAN DEFAULT TRUE
);

-- Orders with coupons
CREATE TABLE order_coupons (
    order_id INT NOT NULL,
    coupon_id INT NOT NULL,
    discount_amount DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (order_id, coupon_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (coupon_id) REFERENCES coupons(coupon_id)
);

-- Wishlists
CREATE TABLE wishlists (
    wishlist_id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    name VARCHAR(50) DEFAULT 'My Wishlist',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Wishlist items
CREATE TABLE wishlist_items (
    wishlist_item_id INT AUTO_INCREMENT PRIMARY KEY,
    wishlist_id INT NOT NULL,
    product_id INT NOT NULL,
    added_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (wishlist_id) REFERENCES wishlists(wishlist_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE CASCADE,
    UNIQUE KEY (wishlist_id, product_id)
);

-- Create indexes for better performance
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_cart_items_cart ON cart_items(cart_id);
CREATE INDEX idx_product_reviews_product ON product_reviews(product_id);

Key Features of This Database Design:
User Management: Tables for users and their addresses
Product Catalog: Products with categories, images, and reviews
Shopping Cart: Cart functionality with items
Order Processing: Complete order management with items and payments
Discount System: Coupons and promotions
Wishlists: For saving favorite products
Relationships: Proper foreign key relationships between tables
Timestamps: Created_at and updated_at fields for tracking changes
Indexes: Added for better query performance

~~~~

Простой PHP-сервер для интернет-магазина
Вот базовый код сервера для интернет-магазина с использованием PHP и MySQL. Этот код включает:
Подключение к базе данных
Основные API endpoints для работы с товарами, корзиной и пользователями
Простую систему маршрутизации

~~~~php
Файл index.php (главный входной point)
php
<?php
header("Content-Type: application/json; charset=UTF-8");
require_once 'config.php';
require_once 'Database.php';
require_once 'Product.php';
require_once 'User.php';
require_once 'Cart.php';

// Подключение к БД
$db = new Database(DB_HOST, DB_NAME, DB_USER, DB_PASS);

// Получаем метод и URI запроса
$method = $_SERVER['REQUEST_METHOD'];
$uri = parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH);
$uri = explode('/', trim($uri, '/'));

// Простая маршрутизация
switch ($uri[0]) {
    case 'products':
        $product = new Product($db);
        handleProductsRequest($method, $uri, $product);
        break;
    case 'users':
        $user = new User($db);
        handleUsersRequest($method, $uri, $user);
        break;
    case 'cart':
        $cart = new Cart($db);
        handleCartRequest($method, $uri, $cart);
        break;
    default:
        http_response_code(404);
        echo json_encode(['message' => 'Endpoint not found']);
        break;
}

// Обработка запросов к товарам
function handleProductsRequest($method, $uri, $product) {
    $id = isset($uri[1]) ? (int)$uri[1] : null;
    
    switch ($method) {
        case 'GET':
            if ($id) {
                // Получить один товар
                $result = $product->getProduct($id);
            } else {
                // Получить список товаров
                $result = $product->getProducts();
            }
            break;
        case 'POST':
            // Создать новый товар (для админа)
            $data = json_decode(file_get_contents('php://input'), true);
            $result = $product->createProduct($data);
            break;
        case 'PUT':
            // Обновить товар (для админа)
            $data = json_decode(file_get_contents('php://input'), true);
            $result = $product->updateProduct($id, $data);
            break;
        case 'DELETE':
            // Удалить товар (для админа)
            $result = $product->deleteProduct($id);
            break;
        default:
            http_response_code(405);
            $result = ['message' => 'Method not allowed'];
    }
    
    echo json_encode($result);
}

// Обработка запросов к пользователям
function handleUsersRequest($method, $uri, $user) {
    switch ($method) {
        case 'POST':
            if ($uri[1] === 'register') {
                // Регистрация
                $data = json_decode(file_get_contents('php://input'), true);
                $result = $user->register($data);
            } elseif ($uri[1] === 'login') {
                // Авторизация
                $data = json_decode(file_get_contents('php://input'), true);
                $result = $user->login($data);
            }
            break;
        default:
            http_response_code(405);
            $result = ['message' => 'Method not allowed'];
    }
    
    echo json_encode($result);
}

// Обработка запросов к корзине
function handleCartRequest($method, $uri, $cart) {
    $userId = isset($_GET['user_id']) ? (int)$_GET['user_id'] : null;
    
    if (!$userId) {
        http_response_code(400);
        echo json_encode(['message' => 'User ID is required']);
        return;
    }
    
    switch ($method) {
        case 'GET':
            $result = $cart->getCart($userId);
            break;
        case 'POST':
            $data = json_decode(file_get_contents('php://input'), true);
            $result = $cart->addToCart($userId, $data['product_id'], $data['quantity']);
            break;
        case 'DELETE':
            $productId = isset($uri[1]) ? (int)$uri[1] : null;
            if ($productId) {
                $result = $cart->removeFromCart($userId, $productId);
            } else {
                $result = $cart->clearCart($userId);
            }
            break;
        default:
            http_response_code(405);
            $result = ['message' => 'Method not allowed'];
    }
    
    echo json_encode($result);
}
?>
Файл config.php (настройки)
php
<?php
// Настройки базы данных
define('DB_HOST', 'localhost');
define('DB_NAME', 'ecommerce_store');
define('DB_USER', 'root');
define('DB_PASS', '');

// Настройки JWT (для аутентификации)
define('JWT_SECRET', 'your_secret_key');
define('JWT_EXPIRE', 3600); // 1 час
?>
Файл Database.php (класс для работы с БД)
php
<?php
class Database {
    private $connection;

    public function __construct($host, $dbname, $user, $password) {
        try {
            $this->connection = new PDO(
                "mysql:host=$host;dbname=$dbname;charset=utf8",
                $user,
                $password
            );
            $this->connection->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        } catch (PDOException $e) {
            die("Connection failed: " . $e->getMessage());
        }
    }

    public function getConnection() {
        return $this->connection;
    }
}
?>
Файл Product.php (класс для работы с товарами)
php
<?php
class Product {
    private $db;

    public function __construct($db) {
        $this->db = $db->getConnection();
    }

    public function getProducts() {
        $stmt = $this->db->query("SELECT * FROM products WHERE is_active = 1");
        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    public function getProduct($id) {
        $stmt = $this->db->prepare("SELECT * FROM products WHERE product_id = ? AND is_active = 1");
        $stmt->execute([$id]);
        return $stmt->fetch(PDO::FETCH_ASSOC);
    }

    public function createProduct($data) {
        $stmt = $this->db->prepare("
            INSERT INTO products (name, description, price, category_id, sku, stock_quantity)
            VALUES (:name, :description, :price, :category_id, :sku, :stock_quantity)
        ");
        $stmt->execute($data);
        return ['message' => 'Product created', 'id' => $this->db->lastInsertId()];
    }

    public function updateProduct($id, $data) {
        $data['product_id'] = $id;
        $stmt = $this->db->prepare("
            UPDATE products SET 
            name = :name, 
            description = :description, 
            price = :price, 
            category_id = :category_id, 
            sku = :sku, 
            stock_quantity = :stock_quantity
            WHERE product_id = :product_id
        ");
        $stmt->execute($data);
        return ['message' => 'Product updated'];
    }

    public function deleteProduct($id) {
        $stmt = $this->db->prepare("UPDATE products SET is_active = 0 WHERE product_id = ?");
        $stmt->execute([$id]);
        return ['message' => 'Product deleted'];
    }
}
?>
Файл User.php (класс для работы с пользователями)
php
<?php
class User {
    private $db;

    public function __construct($db) {
        $this->db = $db->getConnection();
    }

    public function register($data) {
        // Проверка, что email не занят
        $stmt = $this->db->prepare("SELECT user_id FROM users WHERE email = ?");
        $stmt->execute([$data['email']]);
        if ($stmt->fetch()) {
            return ['error' => 'Email already exists'];
        }

        // Хеширование пароля
        $data['password_hash'] = password_hash($data['password'], PASSWORD_BCRYPT);
        unset($data['password']);

        $stmt = $this->db->prepare("
            INSERT INTO users (username, email, password_hash, first_name, last_name, phone)
            VALUES (:username, :email, :password_hash, :first_name, :last_name, :phone)
        ");
        $stmt->execute($data);
        return ['message' => 'User registered', 'id' => $this->db->lastInsertId()];
    }

    public function login($data) {
        $stmt = $this->db->prepare("SELECT * FROM users WHERE email = ?");
        $stmt->execute([$data['email']]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$user || !password_verify($data['password'], $user['password_hash'])) {
            return ['error' => 'Invalid credentials'];
        }

        // Генерация JWT токена (упрощенный вариант)
        $token = $this->generateJWT($user['user_id']);
        return ['token' => $token, 'user_id' => $user['user_id']];
    }

    private function generateJWT($userId) {
        $header = json_encode(['typ' => 'JWT', 'alg' => 'HS256']);
        $payload = json_encode([
            'user_id' => $userId,
            'exp' => time() + JWT_EXPIRE
        ]);
        
        $base64Header = base64_encode($header);
        $base64Payload = base64_encode($payload);
        
        $signature = hash_hmac('sha256', "$base64Header.$base64Payload", JWT_SECRET, true);
        $base64Signature = base64_encode($signature);
        
        return "$base64Header.$base64Payload.$base64Signature";
    }
}
?>
Файл Cart.php (класс для работы с корзиной)
php
<?php
class Cart {
    private $db;

    public function __construct($db) {
        $this->db = $db->getConnection();
    }

    public function getCart($userId) {
        // Получаем корзину пользователя
        $stmt = $this->db->prepare("
            SELECT c.cart_id, ci.cart_item_id, p.product_id, p.name, p.price, ci.quantity, 
                   (p.price * ci.quantity) as subtotal
            FROM carts c
            JOIN cart_items ci ON c.cart_id = ci.cart_id
            JOIN products p ON ci.product_id = p.product_id
            WHERE c.user_id = ? AND p.is_active = 1
        ");
        $stmt->execute([$userId]);
        
        $items = $stmt->fetchAll(PDO::FETCH_ASSOC);
        
        // Рассчитываем общую сумму
        $total = array_reduce($items, function($sum, $item) {
            return $sum + $item['subtotal'];
        }, 0);
        
        return ['items' => $items, 'total' => $total];
    }

    public function addToCart($userId, $productId, $quantity = 1) {
        // Проверяем существование продукта
        $stmt = $this->db->prepare("SELECT product_id FROM products WHERE product_id = ? AND is_active = 1");
        $stmt->execute([$productId]);
        if (!$stmt->fetch()) {
            return ['error' => 'Product not found'];
        }
        
        // Получаем корзину пользователя (или создаем новую)
        $stmt = $this->db->prepare("SELECT cart_id FROM carts WHERE user_id = ?");
        $stmt->execute([$userId]);
        $cart = $stmt->fetch();
        
        if (!$cart) {
            $stmt = $this->db->prepare("INSERT INTO carts (user_id) VALUES (?)");
            $stmt->execute([$userId]);
            $cartId = $this->db->lastInsertId();
        } else {
            $cartId = $cart['cart_id'];
        }
        
        // Проверяем, есть ли уже такой товар в корзине
        $stmt = $this->db->prepare("
            SELECT cart_item_id, quantity FROM cart_items 
            WHERE cart_id = ? AND product_id = ?
        ");
        $stmt->execute([$cartId, $productId]);
        $existingItem = $stmt->fetch();
        
        if ($existingItem) {
            // Обновляем количество
            $newQuantity = $existingItem['quantity'] + $quantity;
            $stmt = $this->db->prepare("
                UPDATE cart_items SET quantity = ? 
                WHERE cart_item_id = ?
            ");
            $stmt->execute([$newQuantity, $existingItem['cart_item_id']]);
        } else {
            // Добавляем новый товар
            $stmt = $this->db->prepare("
                INSERT INTO cart_items (cart_id, product_id, quantity)
                VALUES (?, ?, ?)
            ");
            $stmt->execute([$cartId, $productId, $quantity]);
        }
        
        return ['message' => 'Product added to cart'];
    }

    public function removeFromCart($userId, $productId) {
        // Находим корзину пользователя
        $stmt = $this->db->prepare("SELECT cart_id FROM carts WHERE user_id = ?");
        $stmt->execute([$userId]);
        $cart = $stmt->fetch();
        
        if (!$cart) {
            return ['error' => 'Cart not found'];
        }
        
        // Удаляем товар из корзины
        $stmt = $this->db->prepare("
            DELETE FROM cart_items 
            WHERE cart_id = ? AND product_id = ?
        ");
        $stmt->execute([$cart['cart_id'], $productId]);
        
        return ['message' => 'Product removed from cart'];
    }

    public function clearCart($userId) {
        // Находим корзину пользователя
        $stmt = $this->db->prepare("SELECT cart_id FROM carts WHERE user_id = ?");
        $stmt->execute([$userId]);
        $cart = $stmt->fetch();
        
        if (!$cart) {
            return ['error' => 'Cart not found'];
        }
        
        // Очищаем корзину
        $stmt = $this->db->prepare("DELETE FROM cart_items WHERE cart_id = ?");
        $stmt->execute([$cart['cart_id']]);
        
        return ['message' => 'Cart cleared'];
    }
}
?>
~~~~

~~~~bash
npx create-react-app ecommerce-frontend
cd ecommerce-frontend
npm install axios react-router-dom @mui/material @mui/icons-material @emotion/react @emotion/styled
~~~~

Структура проекта
src/
├── components/
│   ├── Header.js
│   ├── ProductCard.js
│   ├── CartItem.js
│   └── AuthModal.js
├── pages/
│   ├── Home.js
│   ├── Product.js
│   ├── Cart.js
│   └── Login.js
├── context/
│   ├── AuthContext.js
│   └── CartContext.js
├── services/
│   ├── api.js
│   └── auth.js
├── App.js
├── App.css
└── index.js

~~~~js
src/services/api.js - Настройка API
jsx
import axios from 'axios';

const API_BASE_URL = 'http://localhost'; // Замените на ваш URL бэкенда

const api = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Добавляем интерцептор для JWT токена
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
src/context/AuthContext.js - Контекст аутентификации
jsx
import { createContext, useState, useEffect } from 'react';

export const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  useEffect(() => {
    const token = localStorage.getItem('token');
    const userId = localStorage.getItem('user_id');
    
    if (token && userId) {
      setUser({ id: userId });
      setIsAuthenticated(true);
    }
  }, []);

  const login = (token, userId) => {
    localStorage.setItem('token', token);
    localStorage.setItem('user_id', userId);
    setUser({ id: userId });
    setIsAuthenticated(true);
  };

  const logout = () => {
    localStorage.removeItem('token');
    localStorage.removeItem('user_id');
    setUser(null);
    setIsAuthenticated(false);
  };

  return (
    <AuthContext.Provider value={{ user, isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};
src/context/CartContext.js - Контекст корзины
jsx
import { createContext, useState, useEffect, useContext } from 'react';
import api from '../services/api';
import { AuthContext } from './AuthContext';

export const CartContext = createContext();

export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState([]);
  const [total, setTotal] = useState(0);
  const { isAuthenticated, user } = useContext(AuthContext);

  const fetchCart = async () => {
    if (!isAuthenticated || !user) return;
    
    try {
      const response = await api.get(`/cart?user_id=${user.id}`);
      setCart(response.data.items);
      setTotal(response.data.total);
    } catch (error) {
      console.error('Error fetching cart:', error);
    }
  };

  useEffect(() => {
    fetchCart();
  }, [isAuthenticated, user]);

  const addToCart = async (productId, quantity = 1) => {
    if (!isAuthenticated || !user) {
      alert('Please login to add items to cart');
      return;
    }
    
    try {
      await api.post('/cart', {
        user_id: user.id,
        product_id: productId,
        quantity
      });
      await fetchCart();
    } catch (error) {
      console.error('Error adding to cart:', error);
    }
  };

  const removeFromCart = async (productId) => {
    try {
      await api.delete(`/cart/${productId}?user_id=${user.id}`);
      await fetchCart();
    } catch (error) {
      console.error('Error removing from cart:', error);
    }
  };

  const clearCart = async () => {
    try {
      await api.delete(`/cart?user_id=${user.id}`);
      setCart([]);
      setTotal(0);
    } catch (error) {
      console.error('Error clearing cart:', error);
    }
  };

  return (
    <CartContext.Provider value={{ cart, total, addToCart, removeFromCart, clearCart, fetchCart }}>
      {children}
    </CartContext.Provider>
  );
};
src/components/Header.js - Шапка сайта
jsx
import { AppBar, Toolbar, Typography, Button, Badge, IconButton } from '@mui/material';
import ShoppingCartIcon from '@mui/icons-material/ShoppingCart';
import { Link, useNavigate } from 'react-router-dom';
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';
import { CartContext } from '../context/CartContext';

const Header = () => {
  const { isAuthenticated, logout } = useContext(AuthContext);
  const { cart } = useContext(CartContext);
  const navigate = useNavigate();

  const handleLogout = () => {
    logout();
    navigate('/');
  };

  return (
    <AppBar position="static">
      <Toolbar>
        <Typography variant="h6" component="div" sx={{ flexGrow: 1 }}>
          <Link to="/" style={{ textDecoration: 'none', color: 'inherit' }}>
            My Shop
          </Link>
        </Typography>
        
        {isAuthenticated ? (
          <>
            <Button color="inherit" onClick={handleLogout}>Logout</Button>
            <IconButton color="inherit" component={Link} to="/cart">
              <Badge badgeContent={cart.length} color="error">
                <ShoppingCartIcon />
              </Badge>
            </IconButton>
          </>
        ) : (
          <Button color="inherit" component={Link} to="/login">Login</Button>
        )}
      </Toolbar>
    </AppBar>
  );
};

export default Header;
src/components/ProductCard.js - Карточка товара
jsx
import { Card, CardMedia, CardContent, Typography, Button, CardActions } from '@mui/material';
import { useContext } from 'react';
import { CartContext } from '../context/CartContext';

const ProductCard = ({ product }) => {
  const { addToCart } = useContext(CartContext);

  return (
    <Card sx={{ maxWidth: 345, height: '100%', display: 'flex', flexDirection: 'column' }}>
      <CardMedia
        component="img"
        height="140"
        image={`https://via.placeholder.com/300?text=${product.name}`}
        alt={product.name}
      />
      <CardContent sx={{ flexGrow: 1 }}>
        <Typography gutterBottom variant="h5" component="div">
          {product.name}
        </Typography>
        <Typography variant="body2" color="text.secondary">
          {product.description}
        </Typography>
        <Typography variant="h6" mt={2}>
          ${product.price}
        </Typography>
      </CardContent>
      <CardActions>
        <Button 
          size="small" 
          onClick={() => addToCart(product.product_id)}
        >
          Add to Cart
        </Button>
      </CardActions>
    </Card>
  );
};

export default ProductCard;
src/pages/Home.js - Главная страница
jsx
import { useEffect, useState } from 'react';
import { Grid, Container, TextField } from '@mui/material';
import ProductCard from '../components/ProductCard';
import api from '../services/api';

const Home = () => {
  const [products, setProducts] = useState([]);
  const [searchTerm, setSearchTerm] = useState('');

  useEffect(() => {
    const fetchProducts = async () => {
      try {
        const response = await api.get('/products');
        setProducts(response.data);
      } catch (error) {
        console.error('Error fetching products:', error);
      }
    };

    fetchProducts();
  }, []);

  const filteredProducts = products.filter(product =>
    product.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
    product.description.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <Container sx={{ py: 4 }}>
      <TextField
        fullWidth
        label="Search products"
        variant="outlined"
        margin="normal"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        sx={{ mb: 4 }}
      />
      
      <Grid container spacing={4}>
        {filteredProducts.map((product) => (
          <Grid item key={product.product_id} xs={12} sm={6} md={4}>
            <ProductCard product={product} />
          </Grid>
        ))}
      </Grid>
    </Container>
  );
};

export default Home;
src/pages/Product.js - Страница товара
jsx
import { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import { Container, Grid, Typography, Button, Box, Paper } from '@mui/material';
import api from '../services/api';
import { useContext } from 'react';
import { CartContext } from '../context/CartContext';

const Product = () => {
  const { id } = useParams();
  const [product, setProduct] = useState(null);
  const { addToCart } = useContext(CartContext);

  useEffect(() => {
    const fetchProduct = async () => {
      try {
        const response = await api.get(`/products/${id}`);
        setProduct(response.data);
      } catch (error) {
        console.error('Error fetching product:', error);
      }
    };

    fetchProduct();
  }, [id]);

  if (!product) return <div>Loading...</div>;

  return (
    <Container sx={{ py: 4 }}>
      <Grid container spacing={4}>
        <Grid item xs={12} md={6}>
          <Paper elevation={3} sx={{ p: 2 }}>
            <img
              src={`https://via.placeholder.com/600?text=${product.name}`}
              alt={product.name}
              style={{ width: '100%', height: 'auto' }}
            />
          </Paper>
        </Grid>
        <Grid item xs={12} md={6}>
          <Typography variant="h3" gutterBottom>
            {product.name}
          </Typography>
          <Typography variant="h5" color="text.secondary" gutterBottom>
            ${product.price}
          </Typography>
          <Typography variant="body1" paragraph>
            {product.description}
          </Typography>
          <Box mt={4}>
            <Button
              variant="contained"
              size="large"
              onClick={() => addToCart(product.product_id)}
            >
              Add to Cart
            </Button>
          </Box>
        </Grid>
      </Grid>
    </Container>
  );
};

export default Product;
src/pages/Cart.js - Страница корзины
jsx
import { Container, Typography, Button, Grid, Paper, Box, Divider } from '@mui/material';
import { useContext } from 'react';
import { CartContext } from '../context/CartContext';
import CartItem from '../components/CartItem';

const Cart = () => {
  const { cart, total, removeFromCart, clearCart } = useContext(CartContext);

  if (cart.length === 0) {
    return (
      <Container sx={{ py: 4 }}>
        <Typography variant="h4" gutterBottom>
          Your cart is empty
        </Typography>
      </Container>
    );
  }

  return (
    <Container sx={{ py: 4 }}>
      <Typography variant="h4" gutterBottom>
        Shopping Cart
      </Typography>
      
      <Grid container spacing={4}>
        <Grid item xs={12} md={8}>
          {cart.map((item) => (
            <CartItem 
              key={item.cart_item_id} 
              item={item} 
              onRemove={removeFromCart} 
            />
          ))}
        </Grid>
        <Grid item xs={12} md={4}>
          <Paper elevation={3} sx={{ p: 3 }}>
            <Typography variant="h6" gutterBottom>
              Order Summary
            </Typography>
            <Divider sx={{ my: 2 }} />
            <Box display="flex" justifyContent="space-between" mb={2}>
              <Typography>Subtotal:</Typography>
              <Typography>${total.toFixed(2)}</Typography>
            </Box>
            <Box display="flex" justifyContent="space-between" mb={2}>
              <Typography>Shipping:</Typography>
              <Typography>Free</Typography>
            </Box>
            <Divider sx={{ my: 2 }} />
            <Box display="flex" justifyContent="space-between" mb={3}>
              <Typography variant="h6">Total:</Typography>
              <Typography variant="h6">${total.toFixed(2)}</Typography>
            </Box>
            <Button 
              variant="contained" 
              fullWidth 
              size="large"
              onClick={clearCart}
            >
              Checkout
            </Button>
          </Paper>
        </Grid>
      </Grid>
    </Container>
  );
};

export default Cart;
src/pages/Login.js - Страница авторизации
jsx
import { useState } from 'react';
import { Container, TextField, Button, Typography, Box, Link } from '@mui/material';
import { useNavigate } from 'react-router-dom';
import api from '../services/api';
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

const Login = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [isRegister, setIsRegister] = useState(false);
  const [name, setName] = useState('');
  const [error, setError] = useState('');
  const navigate = useNavigate();
  const { login } = useContext(AuthContext);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    try {
      if (isRegister) {
        const response = await api.post('/users/register', {
          username: email,
          email,
          password,
          first_name: name
        });
        // После регистрации автоматически входим
        await handleLogin(email, password);
      } else {
        await handleLogin(email, password);
      }
    } catch (err) {
      setError(err.response?.data?.error || 'An error occurred');
    }
  };

  const handleLogin = async (email, password) => {
    const response = await api.post('/users/login', { email, password });
    login(response.data.token, response.data.user_id);
    navigate('/');
  };

  return (
    <Container maxWidth="xs" sx={{ py: 4 }}>
      <Paper elevation={3} sx={{ p: 4 }}>
        <Typography variant="h4" align="center" gutterBottom>
          {isRegister ? 'Register' : 'Login'}
        </Typography>
        
        {error && (
          <Typography color="error" align="center" sx={{ mb: 2 }}>
            {error}
          </Typography>
        )}
        
        <form onSubmit={handleSubmit}>
          {isRegister && (
            <TextField
              label="Name"
              variant="outlined"
              fullWidth
              margin="normal"
              value={name}
              onChange={(e) => setName(e.target.value)}
              required
            />
          )}
          
          <TextField
            label="Email"
            type="email"
            variant="outlined"
            fullWidth
            margin="normal"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
          
          <TextField
            label="Password"
            type="password"
            variant="outlined"
            fullWidth
            margin="normal"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />
          
          <Button
            type="submit"
            variant="contained"
            fullWidth
            size="large"
            sx={{ mt: 3 }}
          >
            {isRegister ? 'Register' : 'Login'}
          </Button>
        </form>
        
        <Box mt={2} textAlign="center">
          <Typography variant="body2">
            {isRegister ? 'Already have an account? ' : "Don't have an account? "}
            <Link 
              component="button" 
              variant="body2"
              onClick={() => setIsRegister(!isRegister)}
            >
              {isRegister ? 'Login' : 'Register'}
            </Link>
          </Typography>
        </Box>
      </Paper>
    </Container>
  );
};

export default Login;
src/App.js - Основной компонент приложения
jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import { CartProvider } from './context/CartContext';
import Header from './components/Header';
import Home from './pages/Home';
import Product from './pages/Product';
import Cart from './pages/Cart';
import Login from './pages/Login';

function App() {
  return (
    <Router>
      <AuthProvider>
        <CartProvider>
          <Header />
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/product/:id" element={<Product />} />
            <Route path="/cart" element={<Cart />} />
            <Route path="/login" element={<Login />} />
          </Routes>
        </CartProvider>
      </AuthProvider>
    </Router>
  );
}

export default App;
~~~~


Товары:
GET /products - получить список товаров
GET /products/1 - получить товар с ID=1
POST /products - создать новый товар (требует авторизации)
PUT /products/1 - обновить товар с ID=1
DELETE /products/1 - удалить товар с ID=1

Пользователи:
POST /users/register - регистрация нового пользователя
POST /users/login - авторизация пользователя

Корзина:
GET /cart?user_id=1 - получить содержимое корзины пользователя с ID=1
POST /cart - добавить товар в корзину (в теле запроса: {"user_id":1, "product_id":2, "quantity":1})
DELETE /cart/2?user_id=1 - удалить товар с ID=2 из корзины пользователя с ID=1
DELETE /cart?user_id=1 - очистить корзину пользователя с ID=1


В каком редакторе кода писать php код? Как запустить сервер?
#### **1. Visual Studio Code (VS Code) – бесплатный и популярный**  
#### **2. PHPStorm (платный, но лучший для PHP)**  
🔹 **Плюсы**:  
- Полноценная IDE с автодополнением, рефакторингом, отладчиком  
- Встроенный HTTP-сервер, поддержка Docker, базы данных  

Pапустить PHP-сервер 

#### **1. Встроенный сервер PHP (для разработки)**  
PHP имеет встроенный веб-сервер (не для продакшена!).  

🔹 **Запуск (из папки с проектом в терминале)**:  
```sh
php -S localhost:8000
```
- Сервер запустится на `http://localhost:8000`  
- `index.php` будет открываться автоматически  

🔹 **Если нужно указать конкретный файл**:  
```sh
php -S localhost:8000 -t public/ index.php
```
(где `public/` – папка с точкой входа)  

#### **2. XAMPP **  
Если нужны MySQL:  

🔹 **XAMPP**
  1. Запустите `XAMPP Control Panel`  
  2. Включите **Apache** и **MySQL**  
  3. Код кладётся в `C:\xampp\htdocs\` 
  4. Открывайте `http://localhost/ваш_проект`  

### **Как проверить, что PHP работает?**  
1. Создайте файл `test.php`:  
   ```php
   <?php
   phpinfo();
   ?>
   ```
2. Запустите сервер (`php -S localhost:8000`)  
3. Откройте `http://localhost:8000/test.php`  
4. Если видите информацию о PHP – всё работает!  
