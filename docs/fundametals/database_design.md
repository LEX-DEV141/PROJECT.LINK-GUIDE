# Database Design & Normalisasi

## 🎯 Tujuan Pembelajaran
- Memahami prinsip-prinsip desain database yang efektif dan scalable
- Menguasai teknik normalisasi untuk menghindari redundansi data
- Mampu merancang ERD (Entity Relationship Diagram) yang optimal
- Memahami indexing strategy untuk performa query yang optimal
- Menerapkan best practices dalam penamaan dan struktur database

## 📚 Konsep Dasar

### Database Design Principles
Database design yang baik harus memenuhi kriteria ACID (Atomicity, Consistency, Isolation, Durability) dan mengikuti prinsip-prinsip berikut:

1. **Data Integrity**: Menjamin keakuratan dan konsistensi data
2. **Normalization**: Menghilangkan redundansi dan anomali data
3. **Performance**: Optimasi untuk operasi read/write yang efisien
4. **Scalability**: Mampu menangani pertumbuhan data dan user
5. **Security**: Perlindungan data sensitif dan akses control

### Normalisasi Database

#### First Normal Form (1NF)
- Setiap kolom harus berisi nilai atomic (tidak dapat dibagi lagi)
- Tidak ada repeating groups dalam satu row
- Setiap row harus unik

**Contoh Pelanggaran 1NF:**
```sql
-- SALAH: Multiple values dalam satu kolom
CREATE TABLE users_bad (
    id INT,
    name VARCHAR(100),
    hobbies VARCHAR(255) -- 'reading, gaming, cooking'
);

-- BENAR: Atomic values
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_hobbies (
    user_id INT,
    hobby VARCHAR(100),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### Second Normal Form (2NF)
- Harus sudah dalam 1NF
- Semua non-key attributes harus fully functional dependent pada primary key
- Tidak ada partial dependency

**Contoh Normalisasi ke 2NF:**
```sql
-- SALAH: Partial dependency
CREATE TABLE order_items_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100), -- Dependent hanya pada product_id
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)
);

-- BENAR: Separated tables
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

#### Third Normal Form (3NF)
- Harus sudah dalam 2NF
- Tidak ada transitive dependency
- Non-key attributes tidak boleh dependent pada non-key attributes lain

**Contoh Normalisasi ke 3NF:**
```sql
-- SALAH: Transitive dependency
CREATE TABLE employees_bad (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    department_name VARCHAR(100), -- Dependent pada department_id
    department_budget DECIMAL(15,2) -- Dependent pada department_id
);

-- BENAR: Eliminated transitive dependency
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    budget DECIMAL(15,2)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);
```

## 🔧 Implementasi Praktis

### Contoh Desain Database E-Commerce

```sql
-- Users Table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    email_verified_at TIMESTAMP NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_email (email),
    INDEX idx_created_at (created_at)
);

-- Categories Table
CREATE TABLE categories (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    parent_id INT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    sort_order INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL,
    INDEX idx_parent_id (parent_id),
    INDEX idx_slug (slug),
    INDEX idx_active_sort (is_active, sort_order)
);

-- Products Table
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    short_description VARCHAR(500),
    price DECIMAL(12,2) NOT NULL,
    sale_price DECIMAL(12,2) NULL,
    stock_quantity INT NOT NULL DEFAULT 0,
    manage_stock BOOLEAN DEFAULT TRUE,
    weight DECIMAL(8,2) NULL,
    dimensions VARCHAR(100) NULL,
    status ENUM('active', 'inactive', 'draft') DEFAULT 'draft',
    featured BOOLEAN DEFAULT FALSE,
    category_id INT,
    brand_id INT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (category_id) REFERENCES categories(id),
    INDEX idx_sku (sku),
    INDEX idx_slug (slug),
    INDEX idx_status (status),
    INDEX idx_category (category_id),
    INDEX idx_price (price),
    INDEX idx_featured (featured),
    FULLTEXT idx_search (name, description)
);

-- Orders Table
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    user_id BIGINT NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(12,2) NOT NULL,
    tax_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
    shipping_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
    discount_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'IDR',
    payment_status ENUM('pending', 'paid', 'failed', 'refunded') DEFAULT 'pending',
    payment_method VARCHAR(50),
    shipping_address TEXT NOT NULL,
    billing_address TEXT NOT NULL,
    notes TEXT,
    ordered_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    INDEX idx_payment_status (payment_status),
    INDEX idx_ordered_at (ordered_at),
    INDEX idx_order_number (order_number)
);

-- Order Items Table
CREATE TABLE order_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(12,2) NOT NULL,
    total_price DECIMAL(12,2) NOT NULL,
    product_snapshot JSON, -- Snapshot data produk saat order
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(id),
    INDEX idx_order_id (order_id),
    INDEX idx_product_id (product_id)
);
```

### Indexing Strategy

```sql
-- Primary Index (sudah otomatis pada PRIMARY KEY)
-- Secondary Index untuk query yang sering digunakan

-- Composite Index untuk query multi-kolom
CREATE INDEX idx_products_category_status ON products(category_id, status);
CREATE INDEX idx_orders_user_status ON orders(user_id, status);

-- Covering Index untuk query SELECT tertentu
CREATE INDEX idx_products_list_cover ON products(status, featured) 
INCLUDE (id, name, price, sale_price);

-- Partial Index untuk kondisi tertentu (MySQL 8.0+)
CREATE INDEX idx_active_products ON products(name) WHERE status = 'active';

-- Index untuk sorting dan pagination
CREATE INDEX idx_products_created_desc ON products(created_at DESC);
CREATE INDEX idx_orders_date_amount ON orders(ordered_at DESC, total_amount DESC);
```

### Database Migrations Best Practices

```sql
-- Migration: Create products table
-- File: 2024_01_15_120000_create_products_table.sql

START TRANSACTION;

CREATE TABLE products (
    -- kolom definisi di sini
);

-- Tambahkan index setelah tabel dibuat untuk performa
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_products_status ON products(status);

COMMIT;

-- Rollback script (terpisah)
-- File: 2024_01_15_120000_drop_products_table.sql
DROP TABLE IF EXISTS products;
```

## ✅ Best Practices

### Do's ✅

1. **Penamaan Konsisten**
   - Gunakan snake_case untuk tabel dan kolom
   - Nama tabel dalam bentuk plural (users, products, orders)
   - Gunakan prefiks yang konsisten untuk foreign key (user_id, product_id)

2. **Data Types Optimization**
   - Gunakan data type yang tepat dan efisien
   - VARCHAR dengan ukuran yang sesuai, bukan selalu VARCHAR(255)
   - Gunakan ENUM untuk status dengan nilai terbatas
   - Gunakan DECIMAL untuk currency, bukan FLOAT

3. **Constraints dan Validations**
   - Selalu definisikan PRIMARY KEY
   - Gunakan FOREIGN KEY constraints
   - Tambahkan NOT NULL untuk kolom wajib
   - Gunakan CHECK constraints untuk validasi data

4. **Indexing Strategy**
   - Index kolom yang sering digunakan dalam WHERE clause
   - Buat composite index untuk query multi-kolom
   - Monitor dan optimize index secara berkala

5. **Security**
   - Jangan simpan password dalam plain text
   - Gunakan prepared statements untuk prevent SQL injection
   - Implement proper access control

### Don'ts ❌

1. **Jangan gunakan SELECT * dalam production code**
2. **Jangan buat tabel dengan nama yang ambigu (data, info, temp)**
3. **Jangan gunakan NULL untuk string kosong, gunakan empty string**
4. **Jangan buat index berlebihan tanpa monitoring penggunaan**
5. **Jangan simpan calculated values yang bisa dihitung on-the-fly**
6. **Jangan gunakan BLOB untuk file besar, simpan path/URL saja**

## 🧪 Latihan Praktis

### Exercise 1: Normalisasi Database
Diberikan tabel yang tidak normalized:

```sql
CREATE TABLE student_courses_bad (
    student_id INT,
    student_name VARCHAR(100),
    student_email VARCHAR(100),
    course_id INT,
    course_name VARCHAR(100),
    course_credits INT,
    instructor_name VARCHAR(100),
    instructor_email VARCHAR(100),
    grade CHAR(2)
);
```

**Tugas**: Normalize tabel di atas sampai 3NF dan buat ERD-nya.

### Exercise 2: E-Commerce Database Design
**Scenario**: Merancang database untuk sistem e-commerce dengan requirements:
- Multi-vendor marketplace
- Product variants (size, color, dll)
- Shopping cart functionality
- Order tracking
- Review dan rating system
- Inventory management

**Tugas**: 
1. Buat ERD lengkap
2. Tulis DDL untuk semua tabel
3. Tentukan indexing strategy
4. Buat sample data dan query untuk testing

### Exercise 3: Performance Optimization
Diberikan query yang lambat:

```sql
SELECT p.name, p.price, c.name as category_name, 
       AVG(r.rating) as avg_rating
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
LEFT JOIN reviews r ON p.id = r.product_id
WHERE p.status = 'active' 
  AND p.price BETWEEN 100000 AND 500000
GROUP BY p.id
ORDER BY avg_rating DESC, p.created_at DESC
LIMIT 20;
```

**Tugas**: Analisis dan optimize query di atas dengan:
1. Index strategy yang tepat
2. Query rewrite jika perlu
3. Explain plan analysis

## 📖 Resources Tambahan

### Dokumentasi Resmi
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MongoDB Documentation](https://docs.mongodb.com/)

### Tools dan Software
- **Database Design**: MySQL Workbench, phpMyAdmin, Adminer
- **ERD Tools**: Draw.io, Lucidchart, dbdiagram.io
- **Migration Tools**: Flyway, Liquibase
- **Monitoring**: Percona Monitoring, New Relic

### Buku Rekomendasi
- "Database Design for Mere Mortals" - Michael Hernandez
- "SQL Performance Explained" - Markus Winand
- "High Performance MySQL" - Baron Schwartz

### Online Courses
- [Database Design Course - Coursera](https://www.coursera.org/specializations/database-design)
- [SQL Performance Tuning - Pluralsight](https://www.pluralsight.com/courses/sql-server-performance-tuning)

## ❓ FAQ

### Q: Kapan sebaiknya denormalisasi database?
**A**: Denormalisasi dipertimbangkan ketika:
- Query read performance menjadi bottleneck
- Data warehouse atau reporting system
- Caching layer dengan data yang jarang berubah
- Sistem dengan read-heavy workload

Namun, selalu ukur performance impact sebelum denormalisasi.

### Q: Bagaimana menangani soft delete?
**A**: Implementasi soft delete dengan kolom `deleted_at`:

```sql
ALTER TABLE products ADD COLUMN deleted_at TIMESTAMP NULL;

-- Untuk "delete"
UPDATE products SET deleted_at = NOW() WHERE id = 1;

-- Query dengan soft delete
SELECT * FROM products WHERE deleted_at IS NULL;

-- Index untuk performance
CREATE INDEX idx_products_active ON products(deleted_at) WHERE deleted_at IS NULL;
```

### Q: UUID vs Auto-increment ID?
**A**: 
- **Auto-increment**: Lebih efisien untuk storage dan performance, sequential
- **UUID**: Lebih aman untuk public API, distributed system friendly, tidak predictable

Gunakan UUID jika sistem distributed atau security concern, gunakan auto-increment untuk performa optimal.

### Q: Bagaimana handle JSON data di database?
**A**: Modern database mendukung JSON:

```sql
-- MySQL/PostgreSQL
CREATE TABLE user_preferences (
    user_id BIGINT,
    preferences JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Query JSON data
SELECT * FROM user_preferences 
WHERE JSON_EXTRACT(preferences, '$.theme') = 'dark';

-- Index JSON field (MySQL 8.0+)
CREATE INDEX idx_theme ON user_preferences((JSON_EXTRACT(preferences, '$.theme')));
```

### Q: Bagaimana strategi backup dan restore?
**A**: Implement 3-2-1 backup strategy:
- 3 copies of data
- 2 different storage types
- 1 offsite backup

```bash
# Daily backup
mysqldump --single-transaction --routines --triggers database_name > backup_$(date +%Y%m%d).sql

# Point-in-time recovery setup
# Enable binary logging di MySQL config
log-bin = mysql-bin
expire_logs_days = 7
```

---

**Catatan**: Panduan ini adalah living document yang akan terus diupdate sesuai perkembangan teknologi dan kebutuhan tim LEX DEV.
