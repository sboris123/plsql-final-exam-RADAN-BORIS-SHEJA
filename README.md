## Information

**Name:** SHEJA RADAN BORIS  
**Student ID:** 29096  
**Institution:** Adventist University of Central Africa (AUCA)  
**Course:** Database Development with PL/SQL (INSY 8311)  
**Date:** December 2025

---

## 📋 Project Overview

The **Automated Customer Order Validation System** is a comprehensive database solution designed for e-commerce platforms to streamline order processing, enforce business rules, and maintain complete audit trails. Built entirely in PL/SQL for Oracle Database, this system eliminates manual verification errors and ensures data integrity across all transactions.



## 🎯 Problem Statement

Modern e-commerce businesses face critical challenges in order processing: manual validation leads to errors such as accepting orders for non-existent products, insufficient stock, or suspended customers. Additionally, unrestricted database access during business hours poses data integrity risks, and the lack of comprehensive audit trails creates compliance issues and makes troubleshooting difficult.

---

## 🚀 Key Objectives

1. - **Automate order validation** - Validate customers, products, stock, and credit limits automatically
2. - **Enforce business rules** - Restrict database modifications to weekends only (non-holidays)
3. - **Comprehensive auditing** - Log all operations with user context, timestamps, and outcomes
4. - **Data integrity** - Prevent invalid transactions through triggers and constraints
5. - **Business intelligence** - Provide analytical views for decision-making
6. - **Error handling** - Capture and log all failed operations with clear messages

---

## 🏗️ System Architecture

### Database Components

- **12 Tables** - Customers, Products, Orders, Users, Payments, Feedback, Auditing
- **5 Functions** - Validation, calculation, and lookup operations
- **5 Procedures** - Order placement, status updates, payments, feedback, restocking
- **1 Package** - `pkg_order_management` with 6 public methods
- **9 Triggers** - Business rule enforcement and automatic auditing
- **4 BI Views** - Window functions for analytics and rankings
- **1,000+ Records** - Realistic test data across all tables

### Key Features

- 🔐 **Time-based Access Control** - Weekday/holiday restrictions
- 📊 **Real-time Analytics** - Customer rankings, product trends, sales reports
- 🔍 **Complete Audit Trail** - Every operation logged with user context
- ⚡ **Performance Optimized** - Bulk operations, proper indexing
- 🎯 **Business Rule Validation** - Credit limits, stock levels, order quantities
- 📈 **Window Functions** - ROW_NUMBER, RANK, LAG, LEAD for trend analysis

---



## 🚀 Quick Start Instructions

### Prerequisites

- Oracle Database 21c
- Oracle SQL Developer or SQL*Plus
- Minimum 500MB database space
  
1. **Checking if there is similar tables and drop them**
```sql
BEGIN
   FOR t IN (SELECT table_name FROM user_tables 
             WHERE table_name IN ('PAYMENT_TRANSACTIONS', 'CUSTOMER_FEEDBACK', 
                                  'ORDER_STATUS_HISTORY', 'INVENTORY_AUDIT', 
                                  'ORDER_ERROR_LOG', 'ORDERS', 'USER_SESSIONS', 
                                  'USER_ACCOUNTS', 'PRODUCTS', 'CUSTOMERS', 
                                  'SYSTEM_CONFIGURATION', 'AUDIT_TRAIL')) 
   LOOP
      EXECUTE IMMEDIATE 'DROP TABLE ' || t.table_name || ' CASCADE CONSTRAINTS';
   END LOOP;
END;
/
```
2. ## Execute scripts in order

**TABLES CREATED**
```sql

-- Table 1: CUSTOMERS
CREATE TABLE CUSTOMERS (
    customer_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    phone VARCHAR2(20),
    address VARCHAR2(200),
    city VARCHAR2(50),
    country VARCHAR2(50) DEFAULT 'Rwanda',
    postal_code VARCHAR2(10),
    registration_date DATE DEFAULT SYSDATE NOT NULL,
    customer_status VARCHAR2(20) DEFAULT 'Active' CHECK (customer_status IN ('Active', 'Inactive', 'Suspended')),
    credit_limit NUMBER(10,2) DEFAULT 5000.00 CHECK (credit_limit >= 0),
    total_orders NUMBER(10) DEFAULT 0 CHECK (total_orders >= 0),
    CONSTRAINT chk_email_format CHECK (REGEXP_LIKE(email, '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'))
);


-- Table 2: PRODUCTS
CREATE TABLE PRODUCTS (
    product_id NUMBER(10) PRIMARY KEY,
    product_name VARCHAR2(100) NOT NULL,
    category VARCHAR2(50) NOT NULL,
    description VARCHAR2(500),
    unit_price NUMBER(10,2) NOT NULL CHECK (unit_price > 0),
    stock_quantity NUMBER(10) DEFAULT 0 CHECK (stock_quantity >= 0),
    reorder_level NUMBER(10) DEFAULT 10 CHECK (reorder_level >= 0),
    supplier_name VARCHAR2(100),
    product_status VARCHAR2(20) DEFAULT 'Available' CHECK (product_status IN ('Available', 'Discontinued', 'Out of Stock')),
    created_date DATE DEFAULT SYSDATE NOT NULL,
    last_updated DATE DEFAULT SYSDATE
);

-- Table 3: USER_ACCOUNTS
CREATE TABLE USER_ACCOUNTS (
    user_id NUMBER(10) PRIMARY KEY,
    username VARCHAR2(50) UNIQUE NOT NULL,
    password_hash VARCHAR2(100) NOT NULL,
    full_name VARCHAR2(100) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    role VARCHAR2(30) DEFAULT 'Sales' CHECK (role IN ('Admin', 'Sales', 'Manager', 'Warehouse', 'Customer Service')),
    account_status VARCHAR2(20) DEFAULT 'Active' CHECK (account_status IN ('Active', 'Inactive', 'Locked')),
    created_date DATE DEFAULT SYSDATE NOT NULL,
    last_login DATE,
    failed_login_attempts NUMBER(2) DEFAULT 0 CHECK (failed_login_attempts >= 0)
);

-- Table 4: ORDERS
CREATE TABLE ORDERS (
    order_id NUMBER(10) PRIMARY KEY,
    customer_id NUMBER(10) NOT NULL,
    product_id NUMBER(10) NOT NULL,
    user_id NUMBER(10),
    quantity NUMBER(10) NOT NULL CHECK (quantity > 0),
    unit_price NUMBER(10,2) NOT NULL CHECK (unit_price > 0),
    total_amount NUMBER(10,2) NOT NULL CHECK (total_amount > 0),
    order_date DATE DEFAULT SYSDATE NOT NULL,
    order_status VARCHAR2(30) DEFAULT 'Pending' CHECK (order_status IN ('Pending', 'Confirmed', 'Processing', 'Shipped', 'Delivered', 'Cancelled')),
    shipping_address VARCHAR2(200),
    payment_status VARCHAR2(20) DEFAULT 'Pending' CHECK (payment_status IN ('Pending', 'Paid', 'Failed', 'Refunded')),
    notes VARCHAR2(500),
    CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id) REFERENCES CUSTOMERS(customer_id),
    CONSTRAINT fk_orders_product FOREIGN KEY (product_id) REFERENCES PRODUCTS(product_id),
    CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES USER_ACCOUNTS(user_id)
);


-- Table 5: ORDER_ERROR_LOG
CREATE TABLE ORDER_ERROR_LOG (
    error_id NUMBER(10) PRIMARY KEY,
    customer_id NUMBER(10),
    product_id NUMBER(10),
    quantity NUMBER(10),
    error_message VARCHAR2(500) NOT NULL,
    error_date DATE DEFAULT SYSDATE NOT NULL,
    attempted_by NUMBER(10),
    error_type VARCHAR2(50) CHECK (error_type IN ('Invalid Customer', 'Invalid Product', 'Insufficient Stock', 'Invalid Quantity', 'Business Rule Violation', 'System Error')),
    CONSTRAINT fk_error_customer FOREIGN KEY (customer_id) REFERENCES CUSTOMERS(customer_id) ON DELETE SET NULL,
    CONSTRAINT fk_error_product FOREIGN KEY (product_id) REFERENCES PRODUCTS(product_id) ON DELETE SET NULL,
    CONSTRAINT fk_error_user FOREIGN KEY (attempted_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL
);

-- Table 6: USER_SESSIONS
CREATE TABLE USER_SESSIONS (
    session_id NUMBER(10) PRIMARY KEY,
    user_id NUMBER(10) NOT NULL,
    login_time TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    logout_time TIMESTAMP,
    ip_address VARCHAR2(45),
    session_status VARCHAR2(20) DEFAULT 'Active' CHECK (session_status IN ('Active', 'Expired', 'Terminated')),
    CONSTRAINT fk_session_user FOREIGN KEY (user_id) REFERENCES USER_ACCOUNTS(user_id)
);

-- Table 7: ORDER_STATUS_HISTORY
CREATE TABLE ORDER_STATUS_HISTORY (
    history_id NUMBER(10) PRIMARY KEY,
    order_id NUMBER(10) NOT NULL,
    old_status VARCHAR2(30),
    new_status VARCHAR2(30) NOT NULL,
    changed_by NUMBER(10),
    change_date TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    remarks VARCHAR2(500),
    CONSTRAINT fk_history_order FOREIGN KEY (order_id) REFERENCES ORDERS(order_id),
    CONSTRAINT fk_history_user FOREIGN KEY (changed_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL
);

-- Table 8: CUSTOMER_FEEDBACK
CREATE TABLE CUSTOMER_FEEDBACK (
    feedback_id NUMBER(10) PRIMARY KEY,
    order_id NUMBER(10) NOT NULL,
    customer_id NUMBER(10) NOT NULL,
    rating NUMBER(2) CHECK (rating >= 1 AND rating <= 5),
    cust_comment VARCHAR2(1000),
    feedback_date DATE DEFAULT SYSDATE NOT NULL,
    response VARCHAR2(1000),
    responded_by NUMBER(10),
    response_date DATE,
    CONSTRAINT fk_feedback_order FOREIGN KEY (order_id) REFERENCES ORDERS(order_id),
    CONSTRAINT fk_feedback_customer FOREIGN KEY (customer_id) REFERENCES CUSTOMERS(customer_id),
    CONSTRAINT fk_feedback_responder FOREIGN KEY (responded_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL
);

-- Table 9: INVENTORY_AUDIT
CREATE TABLE INVENTORY_AUDIT (
    audit_id NUMBER(10) PRIMARY KEY,
    product_id NUMBER(10) NOT NULL,
    transaction_type VARCHAR2(20) NOT NULL CHECK (transaction_type IN ('Addition', 'Deduction', 'Adjustment', 'Return', 'Damage')),
    quantity_change NUMBER(10) NOT NULL,
    old_quantity NUMBER(10) NOT NULL,
    new_quantity NUMBER(10) NOT NULL,
    audit_date TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    performed_by NUMBER(10),
    reason VARCHAR2(500),
    reference_order_id NUMBER(10),
    CONSTRAINT fk_inventory_product FOREIGN KEY (product_id) REFERENCES PRODUCTS(product_id),
    CONSTRAINT fk_inventory_user FOREIGN KEY (performed_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL,
    CONSTRAINT fk_inventory_order FOREIGN KEY (reference_order_id) REFERENCES ORDERS(order_id) ON DELETE SET NULL
);

-- Table 10: PAYMENT_TRANSACTIONS
CREATE TABLE PAYMENT_TRANSACTIONS (
    transaction_id NUMBER(10) PRIMARY KEY,
    order_id NUMBER(10) NOT NULL,
    payment_method VARCHAR2(30) CHECK (payment_method IN ('Credit Card', 'Debit Card', 'Mobile Money', 'Bank Transfer', 'Cash', 'PayPal')),
    amount NUMBER(10,2) NOT NULL CHECK (amount > 0),
    transaction_date TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    transaction_status VARCHAR2(20) DEFAULT 'Pending' CHECK (transaction_status IN ('Pending', 'Completed', 'Failed', 'Refunded')),
    payment_reference VARCHAR2(100) UNIQUE,
    gateway_response VARCHAR2(500),
    processed_by NUMBER(10),
    CONSTRAINT fk_payment_order FOREIGN KEY (order_id) REFERENCES ORDERS(order_id),
    CONSTRAINT fk_payment_user FOREIGN KEY (processed_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL
);

-- Table 11: SYSTEM_CONFIGURATION
CREATE TABLE SYSTEM_CONFIGURATION (
    config_id NUMBER(10) PRIMARY KEY,
    config_key VARCHAR2(100) UNIQUE NOT NULL,
    config_value VARCHAR2(500) NOT NULL,
    description VARCHAR2(500),
    data_type VARCHAR2(20) CHECK (data_type IN ('String', 'Number', 'Boolean', 'Date')),
    last_updated TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    updated_by NUMBER(10),
    CONSTRAINT fk_config_user FOREIGN KEY (updated_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL
);

-- Table 12: AUDIT_TRAIL
CREATE TABLE AUDIT_TRAIL (
    audit_id NUMBER(10) PRIMARY KEY,
    table_name VARCHAR2(50) NOT NULL,
    operation VARCHAR2(10) CHECK (operation IN ('INSERT', 'UPDATE', 'DELETE')),
    record_id NUMBER(10) NOT NULL,
    old_values CLOB,
    new_values CLOB,
    performed_by NUMBER(10),
    operation_date TIMESTAMP DEFAULT SYSTIMESTAMP NOT NULL,
    ip_address VARCHAR2(45),
    CONSTRAINT fk_audit_user FOREIGN KEY (performed_by) REFERENCES USER_ACCOUNTS(user_id) ON DELETE SET NULL
);
```
**SEQUENCE**
```sql
CREATE SEQUENCE seq_customer START WITH 1001 INCREMENT BY 1;
CREATE SEQUENCE seq_product START WITH 2001 INCREMENT BY 1;
CREATE SEQUENCE seq_user START WITH 3001 INCREMENT BY 1;
CREATE SEQUENCE seq_order START WITH 4001 INCREMENT BY 1;
CREATE SEQUENCE seq_error START WITH 5001 INCREMENT BY 1;
CREATE SEQUENCE seq_session START WITH 6001 INCREMENT BY 1;
CREATE SEQUENCE seq_history START WITH 7001 INCREMENT BY 1;
CREATE SEQUENCE seq_feedback START WITH 8001 INCREMENT BY 1;
CREATE SEQUENCE seq_inventory_audit START WITH 9001 INCREMENT BY 1;
CREATE SEQUENCE seq_payment START WITH 10001 INCREMENT BY 1;
CREATE SEQUENCE seq_config START WITH 11001 INCREMENT BY 1;
CREATE SEQUENCE seq_audit_trail START WITH 12001 INCREMENT BY 1;
```

**INDEXES**
```sql
-- Customer Indexes
CREATE INDEX idx_customer_email ON CUSTOMERS(email);
CREATE INDEX idx_customer_status ON CUSTOMERS(customer_status);
CREATE INDEX idx_customer_country ON CUSTOMERS(country);

-- Product Indexes
CREATE INDEX idx_product_category ON PRODUCTS(category);
CREATE INDEX idx_product_status ON PRODUCTS(product_status);
CREATE INDEX idx_product_stock ON PRODUCTS(stock_quantity);

-- Order Indexes
CREATE INDEX idx_order_customer ON ORDERS(customer_id);
CREATE INDEX idx_order_product ON ORDERS(product_id);
CREATE INDEX idx_order_date ON ORDERS(order_date);
CREATE INDEX idx_order_status ON ORDERS(order_status);

-- Error Log Indexes
CREATE INDEX idx_error_date ON ORDER_ERROR_LOG(error_date);
CREATE INDEX idx_error_type ON ORDER_ERROR_LOG(error_type);

-- Session Indexes
CREATE INDEX idx_session_user ON USER_SESSIONS(user_id);
CREATE INDEX idx_session_status ON USER_SESSIONS(session_status);

-- Payment Indexes
CREATE INDEX idx_payment_order ON PAYMENT_TRANSACTIONS(order_id);
CREATE INDEX idx_payment_status ON PAYMENT_TRANSACTIONS(transaction_status);
```
**INSERT REALISTIC TEST DATA **
```sql

-- Insert USER_ACCOUNTS (Must be first due to FK dependencies)
BEGIN
    -- Admins
    INSERT INTO USER_ACCOUNTS VALUES (seq_user.NEXTVAL, 'admin1', 'hash_admin1', 'John Administrator', 'john.admin@company.com', 'Admin', 'Active', SYSDATE-365, SYSDATE-1, 0);
    INSERT INTO USER_ACCOUNTS VALUES (seq_user.NEXTVAL, 'admin2', 'hash_admin2', 'Sarah Johnson', 'sarah.admin@company.com', 'Admin', 'Active', SYSDATE-300, SYSDATE-2, 0);
    
    -- Managers
    INSERT INTO USER_ACCOUNTS VALUES (seq_user.NEXTVAL, 'manager1', 'hash_mgr1', 'Michael Brown', 'michael.brown@company.com', 'Manager', 'Active', SYSDATE-250, SYSDATE-3, 0);
    INSERT INTO USER_ACCOUNTS VALUES (seq_user.NEXTVAL, 'manager2', 'hash_mgr2', 'Emily Davis', 'emily.davis@company.com', 'Manager', 'Active', SYSDATE-200, SYSDATE-5, 0);
    
    -- Sales Team
    FOR i IN 1..30 LOOP
        INSERT INTO USER_ACCOUNTS VALUES (
            seq_user.NEXTVAL, 
            'sales' || i, 
            'hash_sales' || i, 
            'Sales Person ' || i, 
            'sales' || i || '@company.com', 
            'Sales', 
            CASE WHEN MOD(i, 15) = 0 THEN 'Inactive' ELSE 'Active' END,
            SYSDATE - (365 - i*10),
            SYSDATE - MOD(i, 7),
            0
        );
    END LOOP;
    
    -- Warehouse Staff
    FOR i IN 1..20 LOOP
        INSERT INTO USER_ACCOUNTS VALUES (
            seq_user.NEXTVAL, 
            'warehouse' || i, 
            'hash_wh' || i, 
            'Warehouse Staff ' || i, 
            'warehouse' || i || '@company.com', 
            'Warehouse', 
            'Active',
            SYSDATE - (200 - i*15),
            SYSDATE - MOD(i, 5),
            0
        );
    END LOOP;
    
    -- Customer Service
    FOR i IN 1..45 LOOP
        INSERT INTO USER_ACCOUNTS VALUES (
            seq_user.NEXTVAL, 
            'cs' || i, 
            'hash_cs' || i, 
            'Customer Service ' || i, 
            'cs' || i || '@company.com', 
            'Customer Service', 
            CASE WHEN i = 10 THEN 'Locked' ELSE 'Active' END,
            SYSDATE - (180 - i*8),
            SYSDATE - MOD(i, 4),
            CASE WHEN i = 10 THEN 3 ELSE 0 END
        );
    END LOOP;
    
    COMMIT;
END;
/

-- Insert CUSTOMERS (300 customers)
DECLARE
    v_first_names SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('James', 'Mary', 'John', 'Patricia', 'Robert', 'Jennifer', 'Michael', 'Linda', 'William', 'Elizabeth', 'David', 'Barbara', 'Richard', 'Susan', 'Joseph', 'Jessica', 'Thomas', 'Sarah', 'Charles', 'Karen');
    v_last_names SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Smith', 'Johnson', 'Williams', 'Brown', 'Jones', 'Garcia', 'Miller', 'Davis', 'Rodriguez', 'Martinez', 'Hernandez', 'Lopez', 'Wilson', 'Anderson', 'Thomas', 'Taylor', 'Moore', 'Jackson', 'Martin', 'Lee');
    v_cities SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Kigali', 'Musanze', 'Rubavu', 'Huye', 'Nyanza', 'Rwamagana', 'Muhanga', 'Rusizi', 'Karongi', 'Nyagatare');
    v_countries SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Rwanda', 'Kenya', 'Uganda', 'Tanzania', 'Burundi');
BEGIN
    FOR i IN 1..300 LOOP
        INSERT INTO CUSTOMERS VALUES (
            seq_customer.NEXTVAL,
            v_first_names(MOD(i, 20) + 1),
            v_last_names(MOD(i, 20) + 1),
            LOWER(v_first_names(MOD(i, 20) + 1) || '.' || v_last_names(MOD(i, 20) + 1) || i || '@email.com'),
            '+25078' || LPAD(TO_CHAR(i*1234), 7, '0'),
            'Street ' || i || ', Building ' || MOD(i, 50),
            v_cities(MOD(i, 10) + 1),
            v_countries(MOD(i, 5) + 1),
            LPAD(TO_CHAR(i*100), 5, '0'),
            SYSDATE - (500 - i),
            CASE WHEN MOD(i, 25) = 0 THEN 'Suspended' WHEN MOD(i, 30) = 0 THEN 'Inactive' ELSE 'Active' END,
            CASE WHEN MOD(i, 3) = 0 THEN 10000 WHEN MOD(i, 5) = 0 THEN 20000 ELSE 5000 END,
            TRUNC(DBMS_RANDOM.VALUE(0, 50))
        );
    END LOOP;
    COMMIT;
END;
/
-- Insert PRODUCTS (150 products)
DECLARE
    v_categories SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Electronics', 'Clothing', 'Food & Beverages', 'Home & Garden', 'Sports', 'Books', 'Toys', 'Beauty', 'Automotive', 'Office Supplies');
    v_product_names SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Premium', 'Standard', 'Economy', 'Deluxe', 'Basic', 'Professional', 'Ultra', 'Mega', 'Super', 'Classic');
    v_product_id NUMBER;
BEGIN
    FOR i IN 1..150 LOOP
        v_product_id := seq_product.NEXTVAL;
        
        INSERT INTO PRODUCTS (
            product_id,
            product_name,
            category,
            description,
            unit_price,
            stock_quantity,
            reorder_level,
            supplier_name,
            product_status,
            created_date,
            last_updated
        ) VALUES (
            v_product_id,
            v_product_names(MOD(i, 10) + 1) || ' ' || v_categories(MOD(i, 10) + 1) || ' Item ' || i,
            v_categories(MOD(i, 10) + 1),
            'High quality product for ' || v_categories(MOD(i, 10) + 1) || ' category. Item code: PRD' || i,
            ROUND(DBMS_RANDOM.VALUE(10, 5000), 2),
            CASE 
                WHEN MOD(i, 15) = 0 THEN 0 
                WHEN MOD(i, 20) = 0 THEN TRUNC(DBMS_RANDOM.VALUE(1, 5))
                ELSE TRUNC(DBMS_RANDOM.VALUE(50, 1000))
            END,
            CASE WHEN MOD(i, 3) = 0 THEN 20 ELSE 10 END,
            'Supplier ' || TO_CHAR(MOD(i, 30) + 1),
            CASE 
                WHEN MOD(i, 15) = 0 THEN 'Out of Stock'
                WHEN MOD(i, 25) = 0 THEN 'Discontinued'
                ELSE 'Available'
            END,
            SYSDATE - (400 - i),
            SYSDATE - MOD(i, 30)
        );
    END LOOP;
    COMMIT;
    DBMS_OUTPUT.PUT_LINE('Successfully inserted 150 products');
END;
/

-- Insert SYSTEM_CONFIGURATION
BEGIN
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'MIN_STOCK_THRESHOLD', '10', 'Minimum stock before reorder alert', 'Number', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'MAX_ORDER_QUANTITY', '1000', 'Maximum quantity per order', 'Number', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'TAX_RATE', '0.18', 'VAT tax rate (18%)', 'Number', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'CURRENCY', 'RWF', 'System currency', 'String', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'AUTO_APPROVE_THRESHOLD', '5000', 'Auto-approve orders below this amount', 'Number', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'SESSION_TIMEOUT_MINUTES', '30', 'User session timeout', 'Number', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'MAX_FAILED_LOGINS', '3', 'Max failed login attempts', 'Number', SYSTIMESTAMP, 3001);
    INSERT INTO SYSTEM_CONFIGURATION VALUES (seq_config.NEXTVAL, 'ENABLE_EMAIL_NOTIFICATIONS', 'true', 'Send email notifications', 'Boolean', SYSTIMESTAMP, 3001);
    COMMIT;
END;
/

-- Insert ORDERS (500 orders with various statuses)
DECLARE
    v_customer_id NUMBER;
    v_product_id NUMBER;
    v_user_id NUMBER;
    v_quantity NUMBER;
    v_unit_price NUMBER;
    v_statuses SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Pending', 'Confirmed', 'Processing', 'Shipped', 'Delivered', 'Cancelled');
    v_payment_statuses SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Pending', 'Paid', 'Failed', 'Refunded');
BEGIN
    FOR i IN 1..500 LOOP
        -- Random customer (from first 200)
        SELECT customer_id INTO v_customer_id FROM (SELECT customer_id FROM CUSTOMERS ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        
        -- Random product (from first 150)
        SELECT product_id, unit_price INTO v_product_id, v_unit_price FROM (SELECT product_id, unit_price FROM PRODUCTS WHERE product_status = 'Available' ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        
        -- Random user
        SELECT user_id INTO v_user_id FROM (SELECT user_id FROM USER_ACCOUNTS WHERE account_status = 'Active' ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        
        v_quantity := TRUNC(DBMS_RANDOM.VALUE(1, 20));
        
        INSERT INTO ORDERS VALUES (
            seq_order.NEXTVAL,
            v_customer_id,
            v_product_id,
            v_user_id,
            v_quantity,
            v_unit_price,
            v_quantity * v_unit_price,
            SYSDATE - (300 - TRUNC(i/2)),
            v_statuses(MOD(i, 6) + 1),
            'Shipping address for order ' || i,
            v_payment_statuses(MOD(i, 4) + 1),
            CASE WHEN MOD(i, 10) = 0 THEN 'Rush order' ELSE NULL END
        );
    END LOOP;
    COMMIT;
END;

-- Insert ORDER_ERROR_LOG (100 error records)
DECLARE
    v_error_types SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Invalid Customer', 'Invalid Product', 'Insufficient Stock', 'Invalid Quantity', 'Business Rule Violation', 'System Error');
    v_customer_id NUMBER;
    v_product_id NUMBER;
    v_user_id NUMBER;
BEGIN
    FOR i IN 1..100 LOOP
        SELECT customer_id INTO v_customer_id FROM (SELECT customer_id FROM CUSTOMERS ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        SELECT product_id INTO v_product_id FROM (SELECT product_id FROM PRODUCTS ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        SELECT user_id INTO v_user_id FROM (SELECT user_id FROM USER_ACCOUNTS ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        
        INSERT INTO ORDER_ERROR_LOG VALUES (
            seq_error.NEXTVAL,
            CASE WHEN MOD(i, 10) = 0 THEN NULL ELSE v_customer_id END,
            CASE WHEN MOD(i, 8) = 0 THEN NULL ELSE v_product_id END,
            CASE WHEN MOD(i, 5) = 0 THEN -5 ELSE TRUNC(DBMS_RANDOM.VALUE(1, 100)) END,
            'Error: ' || v_error_types(MOD(i, 6) + 1) || ' - Attempt #' || i,
            SYSDATE - TRUNC(DBMS_RANDOM.VALUE(1, 200)),
            v_user_id,
            v_error_types(MOD(i, 6) + 1)
        );
    END LOOP;
    COMMIT;
END;
/

-- Insert USER_SESSIONS (300 session records)
DECLARE
    v_user_id NUMBER;
    v_login_time TIMESTAMP;
    v_logout_time TIMESTAMP;
BEGIN
    FOR i IN 1..300 LOOP
        SELECT user_id INTO v_user_id FROM (SELECT user_id FROM USER_ACCOUNTS ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
        
        v_login_time := SYSTIMESTAMP - NUMTODSINTERVAL(TRUNC(DBMS_RANDOM.VALUE(1, 180)), 'DAY') - NUMTODSINTERVAL(TRUNC(DBMS_RANDOM.VALUE(0, 24)), 'HOUR');
        v_logout_time := CASE WHEN MOD(i, 10) = 0 THEN NULL ELSE v_login_time + NUMTODSINTERVAL(TRUNC(DBMS_RANDOM.VALUE(1, 480)), 'MINUTE') END;
        
        INSERT INTO USER_SESSIONS VALUES (
            seq_session.NEXTVAL,
            v_user_id,
            v_login_time,
            v_logout_time,
            '192.168.' || TRUNC(DBMS_RANDOM.VALUE(1, 255)) || '.' || TRUNC(DBMS_RANDOM.VALUE(1, 255)),
            CASE WHEN v_logout_time IS NULL THEN 'Active' WHEN MOD(i, 20) = 0 THEN 'Terminated' ELSE 'Expired' END        );
    END LOOP;
    COMMIT;
END;
/

-- Insert ORDER_STATUS_HISTORY (800 history records)
DECLARE
    v_order_id NUMBER;
    v_user_id NUMBER;
    v_statuses SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Pending', 'Confirmed', 'Processing', 'Shipped', 'Delivered', 'Cancelled');
BEGIN
    FOR ord IN (SELECT order_id, order_date FROM ORDERS ORDER BY order_id) LOOP
        -- Initial status
        INSERT INTO ORDER_STATUS_HISTORY VALUES (
            seq_history.NEXTVAL,
            ord.order_id,
            NULL,
            'Pending',
            (SELECT user_id FROM (SELECT user_id FROM USER_ACCOUNTS WHERE account_status = 'Active' ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1),
            ord.order_date,
            'Order created'
        );
        
        -- Random status changes (1-3 changes per order)
        FOR i IN 1..TRUNC(DBMS_RANDOM.VALUE(1, 4)) LOOP
            SELECT user_id INTO v_user_id FROM (SELECT user_id FROM USER_ACCOUNTS WHERE account_status = 'Active' ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1;
            
            INSERT INTO ORDER_STATUS_HISTORY VALUES (
                seq_history.NEXTVAL,
                ord.order_id,
                v_statuses(i),
                v_statuses(i + 1),
                v_user_id,
                ord.order_date + (i * 0.5),
                'Status updated to ' || v_statuses(i + 1)
            );
        END LOOP;
    END LOOP;
    COMMIT;
END;
/

-- Insert CUSTOMER_FEEDBACK (200 feedback records)
DECLARE
    v_order_id NUMBER;
    v_customer_id NUMBER;
    v_user_id NUMBER;
    v_comments SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST(
        'Excellent service, very satisfied!',
        'Good product quality, fast delivery',
        'Product arrived damaged, disappointed',
        'Average experience, nothing special',
        'Outstanding customer support',
        'Delivery was delayed but product is good',
        'Not what I expected, quality issues',
        'Highly recommend this seller!',
        'Product exactly as described',
        'Poor packaging, item was damaged'
    );
BEGIN
    FOR i IN 1..200 LOOP
        SELECT order_id, customer_id INTO v_order_id, v_customer_id 
        FROM (SELECT order_id, customer_id FROM ORDERS WHERE order_status IN ('Delivered', 'Cancelled') ORDER BY DBMS_RANDOM.VALUE) 
        WHERE ROWNUM = 1;
        
        INSERT INTO CUSTOMER_FEEDBACK VALUES (
            seq_feedback.NEXTVAL,
            v_order_id,
            v_customer_id,
            TRUNC(DBMS_RANDOM.VALUE(1, 6)),
            v_comments(MOD(i, 10) + 1),
            SYSDATE - TRUNC(DBMS_RANDOM.VALUE(1, 100)),
            CASE WHEN MOD(i, 3) = 0 THEN 'Thank you for your feedback. We appreciate your business!' ELSE NULL END,
            CASE WHEN MOD(i, 3) = 0 THEN (SELECT user_id FROM (SELECT user_id FROM USER_ACCOUNTS WHERE role = 'Customer Service' ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1) ELSE NULL END,
            CASE WHEN MOD(i, 3) = 0 THEN SYSDATE - TRUNC(DBMS_RANDOM.VALUE(1, 50)) ELSE NULL END
        );
    END LOOP;
    COMMIT;
END;
/

-- Insert INVENTORY_AUDIT (400 audit records)
DECLARE
    v_product_id NUMBER;
    v_user_id NUMBER;
    v_order_id NUMBER;
    v_trans_types SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Addition', 'Deduction', 'Adjustment', 'Return', 'Damage');
    v_old_qty NUMBER;
    v_qty_change NUMBER;
BEGIN
    FOR i IN 1..400 LOOP
        SELECT product_id, stock_quantity INTO v_product_id, v_old_qty 
        FROM (SELECT product_id, stock_quantity FROM PRODUCTS ORDER BY DBMS_RANDOM.VALUE) 
        WHERE ROWNUM = 1;
        
        SELECT user_id INTO v_user_id 
        FROM (SELECT user_id FROM USER_ACCOUNTS WHERE role IN ('Warehouse', 'Manager') ORDER BY DBMS_RANDOM.VALUE) 
        WHERE ROWNUM = 1;
        
        v_qty_change := CASE 
            WHEN v_trans_types(MOD(i, 5) + 1) = 'Addition' THEN TRUNC(DBMS_RANDOM.VALUE(50, 200))
            WHEN v_trans_types(MOD(i, 5) + 1) = 'Deduction' THEN -TRUNC(DBMS_RANDOM.VALUE(1, 30))
            WHEN v_trans_types(MOD(i, 5) + 1) = 'Return' THEN TRUNC(DBMS_RANDOM.VALUE(1, 10))
            WHEN v_trans_types(MOD(i, 5) + 1) = 'Damage' THEN -TRUNC(DBMS_RANDOM.VALUE(1, 5))
            ELSE TRUNC(DBMS_RANDOM.VALUE(-10, 10))
        END;
        
        INSERT INTO INVENTORY_AUDIT VALUES (
            seq_inventory_audit.NEXTVAL,
            v_product_id,
            v_trans_types(MOD(i, 5) + 1),
            v_qty_change,
            v_old_qty,
            v_old_qty + v_qty_change,
            SYSTIMESTAMP - NUMTODSINTERVAL(TRUNC(DBMS_RANDOM.VALUE(1, 180)), 'DAY'),
            v_user_id,
            'Inventory ' || v_trans_types(MOD(i, 5) + 1) || ' - Batch ' || i,
            CASE WHEN MOD(i, 5) = 0 THEN (SELECT order_id FROM (SELECT order_id FROM ORDERS ORDER BY DBMS_RANDOM.VALUE) WHERE ROWNUM = 1) ELSE NULL END
        );
    END LOOP;
    COMMIT;
END;
/

-- Insert PAYMENT_TRANSACTIONS (450 payment records)
DECLARE
    v_order_id NUMBER;
    v_user_id NUMBER;
    v_amount NUMBER;
    v_methods SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Credit Card', 'Debit Card', 'Mobile Money', 'Bank Transfer', 'Cash', 'PayPal');
    v_statuses SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('Pending', 'Completed', 'Failed', 'Refunded');
BEGIN
    FOR i IN 1..450 LOOP
        SELECT order_id, total_amount INTO v_order_id, v_amount 
        FROM (SELECT order_id, total_amount FROM ORDERS ORDER BY DBMS_RANDOM.VALUE) 
        WHERE ROWNUM = 1;
        
        SELECT user_id INTO v_user_id 
        FROM (SELECT user_id FROM USER_ACCOUNTS WHERE role IN ('Sales', 'Manager') ORDER BY DBMS_RANDOM.VALUE) 
        WHERE ROWNUM = 1;
        
        INSERT INTO PAYMENT_TRANSACTIONS VALUES (
            seq_payment.NEXTVAL,
            v_order_id,
            v_methods(MOD(i, 6) + 1),
            v_amount,
            SYSTIMESTAMP - NUMTODSINTERVAL(TRUNC(DBMS_RANDOM.VALUE(1, 150)), 'DAY'),
            v_statuses(MOD(i, 4) + 1),
            'REF' || LPAD(i, 8, '0') || TO_CHAR(SYSDATE, 'YYYYMMDD'),
            CASE WHEN MOD(i, 4) = 2 THEN 'Payment failed: Insufficient funds' WHEN MOD(i, 4) = 0 THEN 'Transaction successful' ELSE NULL END,
            v_user_id
        );
    END LOOP;
    COMMIT;
END;
/

-- Insert AUDIT_TRAIL (300 audit records)
DECLARE
    v_user_id NUMBER;
    v_tables SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('ORDERS', 'CUSTOMERS', 'PRODUCTS', 'USER_ACCOUNTS', 'PAYMENTS');
    v_operations SYS.ODCIVARCHAR2LIST := SYS.ODCIVARCHAR2LIST('INSERT', 'UPDATE', 'DELETE');
BEGIN
    FOR i IN 1..300 LOOP
        SELECT user_id INTO v_user_id 
        FROM (SELECT user_id FROM USER_ACCOUNTS ORDER BY DBMS_RANDOM.VALUE) 
        WHERE ROWNUM = 1;
        
        INSERT INTO AUDIT_TRAIL VALUES (
            seq_audit_trail.NEXTVAL,
            v_tables(MOD(i, 5) + 1),
            v_operations(MOD(i, 3) + 1),
            1000 + i,
            CASE WHEN MOD(i, 3) != 0 THEN '{"old_value": "previous data"}' ELSE NULL END,
            '{"new_value": "updated data"}',
            v_user_id,
            SYSTIMESTAMP - NUMTODSINTERVAL(TRUNC(DBMS_RANDOM.VALUE(1, 120)), 'DAY'),
            '192.168.' || TRUNC(DBMS_RANDOM.VALUE(1, 255)) || '.' || TRUNC(DBMS_RANDOM.VALUE(1, 255))        );    END LOOP;
    COMMIT;
    END;
/
```
**FUNCTIONS**
```sql
-- Function 1: Validate Customer Status
-- Returns: TRUE if customer is valid and active, FALSE otherwise
CREATE OR REPLACE FUNCTION fn_validate_customer(
    p_customer_id IN NUMBER
) RETURN BOOLEAN
IS
    v_customer_status VARCHAR2(20);
    v_count NUMBER;
BEGIN
    -- Check if customer exists
    SELECT COUNT(*) INTO v_count
    FROM CUSTOMERS
    WHERE customer_id = p_customer_id;
    
    IF v_count = 0 THEN
        RETURN FALSE;
    END IF;
    
    -- Check customer status
    SELECT customer_status INTO v_customer_status
    FROM CUSTOMERS
    WHERE customer_id = p_customer_id;
    
    -- Only active customers can place orders
    IF v_customer_status = 'Active' THEN
        RETURN TRUE;
    ELSE
        RETURN FALSE;
    END IF;
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN FALSE;
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error in fn_validate_customer: ' || SQLERRM);
        RETURN FALSE;
END fn_validate_customer;
/


-- Function 2: Check Product Availability
-- Returns: Available stock quantity, -1 if product doesn't exist or is unavailable
CREATE OR REPLACE FUNCTION fn_check_product_stock(
    p_product_id IN NUMBER
) RETURN NUMBER
IS
    v_stock_quantity NUMBER;
    v_product_status VARCHAR2(20);
BEGIN
    -- Get product stock and status
    SELECT stock_quantity, product_status 
    INTO v_stock_quantity, v_product_status
    FROM PRODUCTS
    WHERE product_id = p_product_id;
    
    -- Check if product is available
    IF v_product_status != 'Available' THEN
        RETURN -1;
    END IF;
    
    RETURN v_stock_quantity;
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN -1;
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error in fn_check_product_stock: ' || SQLERRM);
        RETURN -1;
END fn_check_product_stock;
/


-- Function 3: Calculate Order Total with Tax
-- Returns: Total amount including tax
CREATE OR REPLACE FUNCTION fn_calculate_order_total(
    p_unit_price IN NUMBER,
    p_quantity IN NUMBER,
    p_include_tax IN BOOLEAN DEFAULT TRUE
) RETURN NUMBER
IS
    v_subtotal NUMBER;
    v_tax_rate NUMBER;
    v_total NUMBER;
BEGIN
    -- Validate inputs
    IF p_unit_price <= 0 OR p_quantity <= 0 THEN
        RETURN 0;
    END IF;
    
    -- Calculate subtotal
    v_subtotal := p_unit_price * p_quantity;
    
    -- Get tax rate from configuration
    IF p_include_tax THEN
        BEGIN
            SELECT TO_NUMBER(config_value) INTO v_tax_rate
            FROM SYSTEM_CONFIGURATION
            WHERE config_key = 'TAX_RATE';
            
            v_total := v_subtotal + (v_subtotal * v_tax_rate);
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                -- Default tax rate if not configured
                v_total := v_subtotal + (v_subtotal * 0.18);
        END;
    ELSE
        v_total := v_subtotal;
    END IF;
    
    RETURN ROUND(v_total, 2);
    
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error in fn_calculate_order_total: ' || SQLERRM);
        RETURN 0;
END fn_calculate_order_total;
/


-- Function 4: Get Customer Credit Limit
-- Returns: Available credit limit for customer
CREATE OR REPLACE FUNCTION fn_get_customer_credit_limit(
    p_customer_id IN NUMBER
) RETURN NUMBER
IS
    v_credit_limit NUMBER;
    v_total_pending NUMBER := 0;
    v_available_credit NUMBER;
BEGIN
    -- Get customer's credit limit
    SELECT credit_limit INTO v_credit_limit
    FROM CUSTOMERS
    WHERE customer_id = p_customer_id;
    
    -- Calculate total pending orders
    BEGIN
        SELECT NVL(SUM(total_amount), 0) INTO v_total_pending
        FROM ORDERS
        WHERE customer_id = p_customer_id
          AND order_status IN ('Pending', 'Confirmed')
          AND payment_status = 'Pending';
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            v_total_pending := 0;
    END;
    
    -- Calculate available credit
    v_available_credit := v_credit_limit - v_total_pending;
    
    RETURN v_available_credit;
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0;
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error in fn_get_customer_credit_limit: ' || SQLERRM);
        RETURN 0;
END fn_get_customer_credit_limit;
/

-- Function 5: Validate Order Quantity
-- Returns: TRUE if quantity is valid, FALSE otherwise
CREATE OR REPLACE FUNCTION fn_validate_quantity(
    p_quantity IN NUMBER,
    p_available_stock IN NUMBER
) RETURN BOOLEAN
IS
    v_max_order_qty NUMBER;
BEGIN
    -- Check if quantity is positive
    IF p_quantity <= 0 THEN
        RETURN FALSE;
    END IF;
    
    -- Check if sufficient stock available
    IF p_quantity > p_available_stock THEN
        RETURN FALSE;
    END IF;
    
    -- Check against maximum order quantity configuration
    BEGIN
        SELECT TO_NUMBER(config_value) INTO v_max_order_qty
        FROM SYSTEM_CONFIGURATION
        WHERE config_key = 'MAX_ORDER_QUANTITY';
        
        IF p_quantity > v_max_order_qty THEN
            RETURN FALSE;
        END IF;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            -- No max limit configured, only check stock
            NULL;
    END;
    
    RETURN TRUE;
    
EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Error in fn_validate_quantity: ' || SQLERRM);
        RETURN FALSE;
END fn_validate_quantity;
/

```
**PROCEDURES**
```sql
-- Procedure 1: Place New Order with Full Validation
CREATE OR REPLACE PROCEDURE sp_place_order(
    p_customer_id IN NUMBER,
    p_product_id IN NUMBER,
    p_quantity IN NUMBER,
    p_user_id IN NUMBER,
    p_order_id OUT NUMBER,
    p_status OUT VARCHAR2,
    p_message OUT VARCHAR2
)
IS
    v_stock_quantity NUMBER;
    v_unit_price NUMBER;
    v_total_amount NUMBER;
    v_customer_valid BOOLEAN;
    v_quantity_valid BOOLEAN;
    v_available_credit NUMBER;
    v_product_status VARCHAR2(20);
BEGIN
    -- Initialize output parameters
    p_order_id := NULL;
    p_status := 'FAILED';
    p_message := '';
    
    -- Step 1: Validate customer
    v_customer_valid := fn_validate_customer(p_customer_id);
    
    IF NOT v_customer_valid THEN
        p_message := 'Invalid or inactive customer';
        
        -- Log error
        INSERT INTO ORDER_ERROR_LOG(error_id, customer_id, product_id, quantity, 
                                     error_message, error_date, attempted_by, error_type)
        VALUES(seq_error.NEXTVAL, p_customer_id, p_product_id, p_quantity,
               p_message, SYSDATE, p_user_id, 'Invalid Customer');
        COMMIT;
        RETURN;
    END IF;
    
    -- Step 2: Check product availability
    v_stock_quantity := fn_check_product_stock(p_product_id);
    
    IF v_stock_quantity < 0 THEN
        p_message := 'Product not available or does not exist';
        
        INSERT INTO ORDER_ERROR_LOG(error_id, customer_id, product_id, quantity,
                                     error_message, error_date, attempted_by, error_type)
        VALUES(seq_error.NEXTVAL, p_customer_id, p_product_id, p_quantity,
               p_message, SYSDATE, p_user_id, 'Invalid Product');
        COMMIT;
        RETURN;
    END IF;
    
    -- Step 3: Validate quantity
    v_quantity_valid := fn_validate_quantity(p_quantity, v_stock_quantity);
    
    IF NOT v_quantity_valid THEN
        p_message := 'Invalid quantity or insufficient stock. Available: ' || v_stock_quantity;
        
        INSERT INTO ORDER_ERROR_LOG(error_id, customer_id, product_id, quantity,
                                     error_message, error_date, attempted_by, error_type)
        VALUES(seq_error.NEXTVAL, p_customer_id, p_product_id, p_quantity,
               p_message, SYSDATE, p_user_id, 'Insufficient Stock');
        COMMIT;
        RETURN;
    END IF;
    
    -- Step 4: Get product price
    SELECT unit_price INTO v_unit_price
    FROM PRODUCTS
    WHERE product_id = p_product_id;
    
    -- Step 5: Calculate total amount
    v_total_amount := fn_calculate_order_total(v_unit_price, p_quantity, TRUE);
    
    -- Step 6: Check customer credit limit
    v_available_credit := fn_get_customer_credit_limit(p_customer_id);
    
    IF v_total_amount > v_available_credit THEN
        p_message := 'Order exceeds available credit limit. Available: ' || v_available_credit;
        
        INSERT INTO ORDER_ERROR_LOG(error_id, customer_id, product_id, quantity,
                                     error_message, error_date, attempted_by, error_type)
        VALUES(seq_error.NEXTVAL, p_customer_id, p_product_id, p_quantity,
               p_message, SYSDATE, p_user_id, 'Business Rule Violation');
        COMMIT;
        RETURN;
    END IF;
    
    -- Step 7: Create the order
    INSERT INTO ORDERS(order_id, customer_id, product_id, user_id, quantity,
                      unit_price, total_amount, order_date, order_status, payment_status)
    VALUES(seq_order.NEXTVAL, p_customer_id, p_product_id, p_user_id, p_quantity,
           v_unit_price, v_total_amount, SYSDATE, 'Pending', 'Pending')
    RETURNING order_id INTO p_order_id;
    
    -- Step 8: Update product stock
    UPDATE PRODUCTS
    SET stock_quantity = stock_quantity - p_quantity,
        last_updated = SYSDATE
    WHERE product_id = p_product_id;
    
    -- Step 9: Log inventory change
    INSERT INTO INVENTORY_AUDIT(audit_id, product_id, transaction_type, quantity_change,
                                old_quantity, new_quantity, audit_date, performed_by,
                                reason, reference_order_id)
    VALUES(seq_inventory_audit.NEXTVAL, p_product_id, 'Deduction', -p_quantity,
           v_stock_quantity, v_stock_quantity - p_quantity, SYSTIMESTAMP, p_user_id,
           'Order placed', p_order_id);
    
    -- Step 10: Create status history
    INSERT INTO ORDER_STATUS_HISTORY(history_id, order_id, old_status, new_status,
                                     changed_by, change_date, remarks)
    VALUES(seq_history.NEXTVAL, p_order_id, NULL, 'Pending',
           p_user_id, SYSTIMESTAMP, 'Order created successfully');
    
    -- Step 11: Update customer total orders
    UPDATE CUSTOMERS
    SET total_orders = total_orders + 1
    WHERE customer_id = p_customer_id;
    
    COMMIT;
    
    p_status := 'SUCCESS';
    p_message := 'Order placed successfully. Order ID: ' || p_order_id;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_message := 'System error: ' || SQLERRM;
        
        -- Log system error
        INSERT INTO ORDER_ERROR_LOG(error_id, customer_id, product_id, quantity,
                                     error_message, error_date, attempted_by, error_type)
        VALUES(seq_error.NEXTVAL, p_customer_id, p_product_id, p_quantity,
               p_message, SYSDATE, p_user_id, 'System Error');
        COMMIT;
END sp_place_order;
/
-- Procedure 2: Update Order Status
CREATE OR REPLACE PROCEDURE sp_update_order_status(
    p_order_id IN NUMBER,
    p_new_status IN VARCHAR2,
    p_user_id IN NUMBER,
    p_remarks IN VARCHAR2 DEFAULT NULL,
    p_status OUT VARCHAR2,
    p_message OUT VARCHAR2
)
IS
    v_old_status VARCHAR2(30);
    v_product_id NUMBER;
    v_quantity NUMBER;
    v_current_stock NUMBER;
BEGIN
    -- Get current order status
    BEGIN
        SELECT order_status, product_id, quantity 
        INTO v_old_status, v_product_id, v_quantity
        FROM ORDERS
        WHERE order_id = p_order_id;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            p_status := 'FAILED';
            p_message := 'Order not found';
            RETURN;
    END;
    
    -- Validate status transition
    IF v_old_status = p_new_status THEN
        p_status := 'FAILED';
        p_message := 'Order already has status: ' || p_new_status;
        RETURN;
    END IF;
    
    -- Update order status
    UPDATE ORDERS
    SET order_status = p_new_status
    WHERE order_id = p_order_id;
    
    -- If order is cancelled, restore stock
    IF p_new_status = 'Cancelled' AND v_old_status != 'Cancelled' THEN
        SELECT stock_quantity INTO v_current_stock
        FROM PRODUCTS
        WHERE product_id = v_product_id;
        
        UPDATE PRODUCTS
        SET stock_quantity = stock_quantity + v_quantity,
            last_updated = SYSDATE
        WHERE product_id = v_product_id;
        
        -- Log inventory restoration
        INSERT INTO INVENTORY_AUDIT(audit_id, product_id, transaction_type, quantity_change,
                                    old_quantity, new_quantity, audit_date, performed_by,
                                    reason, reference_order_id)
        VALUES(seq_inventory_audit.NEXTVAL, v_product_id, 'Return', v_quantity,
               v_current_stock, v_current_stock + v_quantity, SYSTIMESTAMP, p_user_id,
               'Order cancelled - stock restored', p_order_id);
    END IF;
    
    -- Log status change
    INSERT INTO ORDER_STATUS_HISTORY(history_id, order_id, old_status, new_status,
                                     changed_by, change_date, remarks)
    VALUES(seq_history.NEXTVAL, p_order_id, v_old_status, p_new_status,
           p_user_id, SYSTIMESTAMP, p_remarks);
    
    -- Log audit trail
    INSERT INTO AUDIT_TRAIL(audit_id, table_name, operation, record_id, old_values,
                           new_values, performed_by, operation_date)
    VALUES(seq_audit_trail.NEXTVAL, 'ORDERS', 'UPDATE', p_order_id,
           '{"status":"' || v_old_status || '"}',
           '{"status":"' || p_new_status || '"}',
           p_user_id, SYSTIMESTAMP);
    
    COMMIT;
    
    p_status := 'SUCCESS';
    p_message := 'Order status updated from ' || v_old_status || ' to ' || p_new_status;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_message := 'Error updating order status: ' || SQLERRM;
END sp_update_order_status;
/


-- Procedure 3: Process Payment Transaction
CREATE OR REPLACE PROCEDURE sp_process_payment(
    p_order_id IN NUMBER,
    p_payment_method IN VARCHAR2,
    p_amount IN NUMBER,
    p_user_id IN NUMBER,
    p_transaction_id OUT NUMBER,
    p_status OUT VARCHAR2,
    p_message OUT VARCHAR2
)
IS
    v_order_total NUMBER;
    v_order_status VARCHAR2(30);
    v_payment_reference VARCHAR2(100);
BEGIN
    -- Validate order exists
    BEGIN
        SELECT total_amount, order_status INTO v_order_total, v_order_status
        FROM ORDERS
        WHERE order_id = p_order_id;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            p_status := 'FAILED';
            p_message := 'Order not found';
            RETURN;
    END;
    
    -- Validate amount matches order total
    IF p_amount != v_order_total THEN
        p_status := 'FAILED';
        p_message := 'Payment amount does not match order total';
        RETURN;
    END IF;
    
    -- Generate payment reference
    v_payment_reference := 'PAY' || TO_CHAR(SYSDATE, 'YYYYMMDD') || LPAD(seq_payment.NEXTVAL, 8, '0');
    
    -- Create payment transaction
    INSERT INTO PAYMENT_TRANSACTIONS(transaction_id, order_id, payment_method, amount,
                                     transaction_date, transaction_status, payment_reference,
                                     gateway_response, processed_by)
    VALUES(seq_payment.NEXTVAL, p_order_id, p_payment_method, p_amount,
           SYSTIMESTAMP, 'Completed', v_payment_reference,
           'Payment processed successfully', p_user_id)
    RETURNING transaction_id INTO p_transaction_id;
    
    -- Update order payment status
    UPDATE ORDERS
    SET payment_status = 'Paid'
    WHERE order_id = p_order_id;
    
    -- If order is pending, move to confirmed
    IF v_order_status = 'Pending' THEN
        UPDATE ORDERS
        SET order_status = 'Confirmed'
        WHERE order_id = p_order_id;
        
        INSERT INTO ORDER_STATUS_HISTORY(history_id, order_id, old_status, new_status,
                                         changed_by, change_date, remarks)
        VALUES(seq_history.NEXTVAL, p_order_id, 'Pending', 'Confirmed',
               p_user_id, SYSTIMESTAMP, 'Payment received - order confirmed');
    END IF;
    
    COMMIT;
    
    p_status := 'SUCCESS';
    p_message := 'Payment processed. Reference: ' || v_payment_reference;
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_message := 'Payment processing error: ' || SQLERRM;
        
        -- Log failed payment
        INSERT INTO PAYMENT_TRANSACTIONS(transaction_id, order_id, payment_method, amount,
                                         transaction_date, transaction_status, payment_reference,
                                         gateway_response, processed_by)
        VALUES(seq_payment.NEXTVAL, p_order_id, p_payment_method, p_amount,
               SYSTIMESTAMP, 'Failed', 'FAIL' || seq_payment.CURRVAL,
               p_message, p_user_id);
        COMMIT;
END sp_process_payment;
/


-- Procedure 4: Add Customer Feedback
CREATE OR REPLACE PROCEDURE sp_add_customer_feedback(
    p_order_id IN NUMBER,
    p_customer_id IN NUMBER,
    p_rating IN NUMBER,
    p_comment IN VARCHAR2,
    p_feedback_id OUT NUMBER,
    p_status OUT VARCHAR2,
    p_message OUT VARCHAR2
)
IS
    v_order_customer_id NUMBER;
    v_order_status VARCHAR2(30);
BEGIN
    -- Validate order belongs to customer
    BEGIN
        SELECT customer_id, order_status INTO v_order_customer_id, v_order_status
        FROM ORDERS
        WHERE order_id = p_order_id;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            p_status := 'FAILED';
            p_message := 'Order not found';
            RETURN;
    END;
    
    IF v_order_customer_id != p_customer_id THEN
        p_status := 'FAILED';
        p_message := 'Order does not belong to this customer';
        RETURN;
    END IF;
    
    -- Check if order is delivered or cancelled
    IF v_order_status NOT IN ('Delivered', 'Cancelled') THEN
        p_status := 'FAILED';
        p_message := 'Feedback can only be given for delivered or cancelled orders';
        RETURN;
    END IF;
    
    -- Validate rating
    IF p_rating < 1 OR p_rating > 5 THEN
        p_status := 'FAILED';
        p_message := 'Rating must be between 1 and 5';
        RETURN;
    END IF;
    
    -- Insert feedback
    INSERT INTO CUSTOMER_FEEDBACK(feedback_id, order_id, customer_id, rating,
                                  cust_comment, feedback_date)
    VALUES(seq_feedback.NEXTVAL, p_order_id, p_customer_id, p_rating,
           p_comment, SYSDATE)
    RETURNING feedback_id INTO p_feedback_id;
    
    COMMIT;
    
    p_status := 'SUCCESS';
    p_message := 'Feedback submitted successfully';
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_message := 'Error submitting feedback: ' || SQLERRM;
END sp_add_customer_feedback;
/


-- Procedure 5: Restock Product (Bulk Operation with Cursor)
CREATE OR REPLACE PROCEDURE sp_restock_products(
    p_user_id IN NUMBER,
    p_restock_quantity IN NUMBER DEFAULT 100,
    p_products_restocked OUT NUMBER,
    p_status OUT VARCHAR2,
    p_message OUT VARCHAR2
)
IS
    -- Cursor to find products below reorder level
    CURSOR c_low_stock IS
        SELECT product_id, product_name, stock_quantity, reorder_level
        FROM PRODUCTS
        WHERE stock_quantity < reorder_level
          AND product_status = 'Available'
        FOR UPDATE;
    
    v_old_stock NUMBER;
    v_new_stock NUMBER;
    v_count NUMBER := 0;
BEGIN
    -- Process each low stock product
    FOR product_rec IN c_low_stock LOOP
        v_old_stock := product_rec.stock_quantity;
        v_new_stock := v_old_stock + p_restock_quantity;
        
        -- Update stock
        UPDATE PRODUCTS
        SET stock_quantity = v_new_stock,
            last_updated = SYSDATE
        WHERE CURRENT OF c_low_stock;
        
        -- Log inventory audit
        INSERT INTO INVENTORY_AUDIT(audit_id, product_id, transaction_type, quantity_change,
                                    old_quantity, new_quantity, audit_date, performed_by, reason)
        VALUES(seq_inventory_audit.NEXTVAL, product_rec.product_id, 'Addition', p_restock_quantity,
               v_old_stock, v_new_stock, SYSTIMESTAMP, p_user_id,
               'Automatic restock - below reorder level');
        
        v_count := v_count + 1;
    END LOOP;
    
    COMMIT;
    
    p_products_restocked := v_count;
    p_status := 'SUCCESS';
    p_message := v_count || ' product(s) restocked successfully';
    
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        p_status := 'ERROR';
        p_message := 'Error restocking products: ' || SQLERRM;
        p_products_restocked := 0;
END sp_restock_products;
/



```
**PACKAGE**
```sql
CREATE OR REPLACE PACKAGE pkg_order_management AS
    -- Type declaration (assumed RECORD for get_order_details; adjust if OBJECT)
    TYPE t_order_summary IS RECORD (
        order_id ORDERS.order_id%TYPE,
        order_date ORDERS.order_date%TYPE,
        customer_name VARCHAR2(100),  -- Concatenated first + last name
        product_name PRODUCTS.product_name%TYPE,
        quantity ORDERS.quantity%TYPE,
        total_amount ORDERS.total_amount%TYPE,
        order_status ORDERS.order_status%TYPE
    );

    -- Procedure to validate and place order
    PROCEDURE validate_and_place_order(
        p_customer_id IN NUMBER,
        p_product_id IN NUMBER,
        p_quantity IN NUMBER,
        p_user_id IN NUMBER,
        p_order_id OUT NUMBER,
        p_status OUT VARCHAR2,
        p_message OUT VARCHAR2
    );

    -- Procedure to cancel order
    PROCEDURE cancel_order(
        p_order_id IN NUMBER,
        p_user_id IN NUMBER,
        p_reason IN VARCHAR2,
        p_status OUT VARCHAR2,
        p_message OUT VARCHAR2
    );

    -- Procedure to generate sales report
    PROCEDURE generate_sales_report(
        p_start_date IN DATE,
        p_end_date IN DATE,
        p_user_id IN NUMBER
    );

    -- Function to get customer order count
    FUNCTION get_customer_order_count(
        p_customer_id IN NUMBER
    ) RETURN NUMBER;

    -- Function to get product revenue
    FUNCTION get_product_revenue(
        p_product_id IN NUMBER,
        p_start_date IN DATE DEFAULT NULL,
        p_end_date IN DATE DEFAULT NULL
    ) RETURN NUMBER;

    -- Function to get order details
    FUNCTION get_order_details(
        p_order_id IN NUMBER
    ) RETURN t_order_summary;

END pkg_order_management;
/

```
**PACKAGE BODY**
```sql

CREATE OR REPLACE PACKAGE BODY pkg_order_management AS

    -- Private procedure: Log package activity
    PROCEDURE log_activity(
        p_activity VARCHAR2,
        p_user_id NUMBER,
        p_details VARCHAR2 DEFAULT NULL
    ) IS
        PRAGMA AUTONOMOUS_TRANSACTION;
    BEGIN
        INSERT INTO AUDIT_TRAIL(audit_id, table_name, operation, record_id,
                               new_values, performed_by, operation_date)
        VALUES(seq_audit_trail.NEXTVAL, 'PACKAGE_ACTIVITY', 'INFO', p_user_id,
               p_activity || ': ' || p_details, p_user_id, SYSTIMESTAMP);
        COMMIT;
    EXCEPTION
        WHEN OTHERS THEN
            NULL; -- Silent fail for logging
    END log_activity;

    -- Public procedure implementation
    PROCEDURE validate_and_place_order(
        p_customer_id IN NUMBER,
        p_product_id IN NUMBER,
        p_quantity IN NUMBER,
        p_user_id IN NUMBER,
        p_order_id OUT NUMBER,
        p_status OUT VARCHAR2,
        p_message OUT VARCHAR2
    ) IS
    BEGIN
        log_activity('ORDER_PLACEMENT_ATTEMPT', p_user_id, 
                    'Customer: ' || p_customer_id || ', Product: ' || p_product_id);
        
        -- Call the standalone procedure
        sp_place_order(p_customer_id, p_product_id, p_quantity, p_user_id,
                      p_order_id, p_status, p_message);
        
        log_activity('ORDER_PLACEMENT_RESULT', p_user_id, 
                    'Status: ' || p_status || ', Order ID: ' || p_order_id);
    END validate_and_place_order;
    
    PROCEDURE cancel_order(
        p_order_id IN NUMBER,
        p_user_id IN NUMBER,
        p_reason IN VARCHAR2,
        p_status OUT VARCHAR2,
        p_message OUT VARCHAR2
    ) IS
    BEGIN
        log_activity('ORDER_CANCELLATION_ATTEMPT', p_user_id, 'Order: ' || p_order_id);
        
        sp_update_order_status(p_order_id, 'Cancelled', p_user_id, 
                              p_reason, p_status, p_message);
        
        log_activity('ORDER_CANCELLATION_RESULT', p_user_id, 'Status: ' || p_status);
    END cancel_order;
    
    PROCEDURE generate_sales_report(
        p_start_date IN DATE,
        p_end_date IN DATE,
        p_user_id IN NUMBER
    ) IS
        CURSOR c_sales IS
            SELECT 
                p.category,
                COUNT(o.order_id) AS order_count,
                SUM(o.quantity) AS total_quantity,
                SUM(o.total_amount) AS total_revenue
            FROM ORDERS o
            JOIN PRODUCTS p ON o.product_id = p.product_id
            WHERE o.order_date BETWEEN p_start_date AND p_end_date
              AND o.order_status != 'Cancelled'
            GROUP BY p.category
            ORDER BY total_revenue DESC;
    BEGIN
        DBMS_OUTPUT.PUT_LINE('=== SALES REPORT ===');
        DBMS_OUTPUT.PUT_LINE('Period: ' || TO_CHAR(p_start_date, 'DD-MON-YYYY') || 
                           ' to ' || TO_CHAR(p_end_date, 'DD-MON-YYYY'));
        DBMS_OUTPUT.PUT_LINE('-----------------------------------');
        
        FOR sales_rec IN c_sales LOOP
            DBMS_OUTPUT.PUT_LINE('Category: ' || sales_rec.category);
            DBMS_OUTPUT.PUT_LINE('  Orders: ' || sales_rec.order_count);
            DBMS_OUTPUT.PUT_LINE('  Quantity: ' || sales_rec.total_quantity);
            DBMS_OUTPUT.PUT_LINE('  Revenue: ' || TO_CHAR(sales_rec.total_revenue, '999,999,999.99'));
            DBMS_OUTPUT.PUT_LINE('-----------------------------------');
        END LOOP;
        
        log_activity('SALES_REPORT_GENERATED', p_user_id, 
                    'Date range: ' || p_start_date || ' to ' || p_end_date);
    END generate_sales_report;
    
    -- Function implementations
    FUNCTION get_customer_order_count(
        p_customer_id IN NUMBER
    ) RETURN NUMBER IS
        v_count NUMBER;
    BEGIN
        SELECT COUNT(*)
        INTO v_count
        FROM ORDERS
        WHERE customer_id = p_customer_id
          AND order_status != 'Cancelled';
        
        RETURN v_count;
    EXCEPTION
        WHEN OTHERS THEN
            RETURN 0;
    END get_customer_order_count;
    
    FUNCTION get_product_revenue(
        p_product_id IN NUMBER,
        p_start_date IN DATE DEFAULT NULL,
        p_end_date IN DATE DEFAULT NULL
    ) RETURN NUMBER IS
        v_revenue NUMBER;
    BEGIN
        IF p_start_date IS NULL THEN
            SELECT NVL(SUM(total_amount), 0)
            INTO v_revenue
            FROM ORDERS
            WHERE product_id = p_product_id
              AND order_status != 'Cancelled';
        ELSE
            SELECT NVL(SUM(total_amount), 0)
            INTO v_revenue
            FROM ORDERS
            WHERE product_id = p_product_id
              AND order_status != 'Cancelled'
              AND order_date BETWEEN p_start_date AND NVL(p_end_date, SYSDATE);
        END IF;
        
        RETURN v_revenue;
    EXCEPTION
        WHEN OTHERS THEN
            RETURN 0;
    END get_product_revenue;
    
    FUNCTION get_order_details(
        p_order_id IN NUMBER
    ) RETURN t_order_summary IS
        v_order t_order_summary;
    BEGIN
        SELECT 
            o.order_id,
            o.order_date,
            c.first_name || ' ' || c.last_name,
            p.product_name,
            o.quantity,
            o.total_amount,
            o.order_status
        INTO v_order
        FROM ORDERS o
        JOIN CUSTOMERS c ON o.customer_id = c.customer_id
        JOIN PRODUCTS p ON o.product_id = p.product_id
        WHERE o.order_id = p_order_id;
        
        RETURN v_order;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            v_order.order_id := -1;
            RETURN v_order;
        WHEN OTHERS THEN
            v_order.order_id := -999;
            RETURN v_order;
    END get_order_details;
    
END pkg_order_management;
/
```
**TRIGGERS**
```sql
-- Trigger 1: Auto-update product status based on stock
CREATE OR REPLACE TRIGGER trg_update_product_status
BEFORE UPDATE OF stock_quantity ON PRODUCTS
FOR EACH ROW
BEGIN
    IF :NEW.stock_quantity = 0 THEN
        :NEW.product_status := 'Out of Stock';
    ELSIF :NEW.stock_quantity > 0 AND :OLD.product_status = 'Out of Stock' THEN
        :NEW.product_status := 'Available';
    END IF;
    
    :NEW.last_updated := SYSDATE;
END;
/




-- Trigger 2: Prevent order modification after delivery
CREATE OR REPLACE TRIGGER trg_prevent_delivered_order_update
BEFORE UPDATE ON ORDERS
FOR EACH ROW
DECLARE
    v_error_msg VARCHAR2(200);
BEGIN
    IF :OLD.order_status = 'Delivered' AND :NEW.order_status != :OLD.order_status THEN
        v_error_msg := 'Cannot modify delivered orders';
        RAISE_APPLICATION_ERROR(-20100, v_error_msg);
    END IF;
END;
/





-- Trigger 3: Audit trail for customer changes
CREATE OR REPLACE TRIGGER trg_audit_customer_changes
AFTER UPDATE ON CUSTOMERS
FOR EACH ROW
BEGIN
    INSERT INTO AUDIT_TRAIL(audit_id, table_name, operation, record_id, old_values, new_values, operation_date)
    VALUES(
        seq_audit_trail.NEXTVAL,
        'CUSTOMERS',
        'UPDATE',
        :NEW.customer_id,
        '{"status":"' || :OLD.customer_status || '","credit_limit":' || :OLD.credit_limit || '}',
        '{"status":"' || :NEW.customer_status || '","credit_limit":' || :NEW.credit_limit || '}',
        SYSTIMESTAMP
    );
END;
/

```
**VIEWS**
```sql
-- View 1: Customer Ranking by Total Spending
CREATE OR REPLACE VIEW vw_customer_rankings AS
SELECT 
    c.customer_id,
    c.first_name || ' ' || c.last_name AS customer_name,
    c.city,
    c.country,
    NVL(SUM(o.total_amount), 0) AS total_spent,
    COUNT(o.order_id) AS order_count,
    ROW_NUMBER() OVER (ORDER BY NVL(SUM(o.total_amount), 0) DESC) AS spending_rank,
    RANK() OVER (ORDER BY NVL(SUM(o.total_amount), 0) DESC) AS spending_rank_with_ties,
    DENSE_RANK() OVER (ORDER BY NVL(SUM(o.total_amount), 0) DESC) AS dense_rank,
    PERCENT_RANK() OVER (ORDER BY NVL(SUM(o.total_amount), 0) DESC) AS percentile
FROM CUSTOMERS c
LEFT JOIN ORDERS o ON c.customer_id = o.customer_id AND o.order_status != 'Cancelled'
GROUP BY c.customer_id, c.first_name, c.last_name, c.city, c.country;



-- View 2: Product Performance with Lag/Lead
CREATE OR REPLACE VIEW vw_product_performance_trends AS
SELECT 
    product_id,
    product_name,
    category,
    order_month,
    monthly_revenue,
    LAG(monthly_revenue, 1) OVER (PARTITION BY product_id ORDER BY order_month) AS prev_month_revenue,
    LEAD(monthly_revenue, 1) OVER (PARTITION BY product_id ORDER BY order_month) AS next_month_revenue,
    monthly_revenue - LAG(monthly_revenue, 1) OVER (PARTITION BY product_id ORDER BY order_month) AS revenue_change,
    ROUND(((monthly_revenue - LAG(monthly_revenue, 1) OVER (PARTITION BY product_id ORDER BY order_month)) / 
           NULLIF(LAG(monthly_revenue, 1) OVER (PARTITION BY product_id ORDER BY order_month), 0)) * 100, 2) AS pct_change
FROM (
    SELECT 
        p.product_id,
        p.product_name,
        p.category,
        TO_CHAR(o.order_date, 'YYYY-MM') AS order_month,
        SUM(o.total_amount) AS monthly_revenue
    FROM PRODUCTS p
    JOIN ORDERS o ON p.product_id = o.product_id
    WHERE o.order_status != 'Cancelled'
    GROUP BY p.product_id, p.product_name, p.category, TO_CHAR(o.order_date, 'YYYY-MM')
);


-- View 3: Top Products per Category
CREATE OR REPLACE VIEW vw_top_products_by_category AS
SELECT category,
       product_id,
       product_name,
       category_revenue,
       rank_in_category,
       total_category_revenue,
       ROUND(100.0 * category_revenue / total_category_revenue, 2) AS pct_of_category
FROM (
    SELECT p.category,
           p.product_id,
           p.product_name,
           NVL(SUM(o.total_amount), 0)                                   AS category_revenue,
           ROW_NUMBER() OVER (PARTITION BY p.category 
                               ORDER BY NVL(SUM(o.total_amount), 0) DESC) AS rank_in_category,
           SUM(NVL(SUM(o.total_amount), 0)) OVER (PARTITION BY p.category) AS total_category_revenue
    FROM PRODUCTS p
    LEFT JOIN ORDERS o 
           ON p.product_id = o.product_id 
          AND o.order_status != 'Cancelled'
    GROUP BY p.category, p.product_id, p.product_name
)
WHERE rank_in_category <= 5;


-- View 4: Running Total of Orders
CREATE OR REPLACE VIEW vw_cumulative_sales AS
SELECT 
    order_date,
    order_count,
    daily_revenue,
    SUM(order_count) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_orders,
    SUM(daily_revenue) OVER (ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_revenue,
    AVG(daily_revenue) OVER (ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg_7days
FROM (
    SELECT 
        TRUNC(order_date) AS order_date,
        COUNT(*) AS order_count,
        SUM(total_amount) AS daily_revenue
    FROM ORDERS
    WHERE order_status != 'Cancelled'
    GROUP BY TRUNC(order_date)
);


```

3. **Verify installation**
```sql
@database/scripts/10_test_s cripts.sql
```

4. **Check data loaded**
```sql

```

---

## 📊 Sample Usage

### Place an Order
```sql
DECLARE
    v_order_id NUMBER;
    v_status VARCHAR2(100);
    v_message VARCHAR2(500);
BEGIN
    sp_place_order(
        p_customer_id => 1001,
        p_product_id => 2001,
        p_quantity => 5,
        p_user_id => 3001,
        p_order_id => v_order_id,
        p_status => v_status,
        p_message => v_message
    );
    
    DBMS_OUTPUT.PUT_LINE('Status: ' || v_status);
    DBMS_OUTPUT.PUT_LINE('Message: ' || v_message);
    DBMS_OUTPUT.PUT_LINE('Order ID: ' || v_order_id);
END;
/
```

### Process Payment
```sql
DECLARE
    v_transaction_id NUMBER;
    v_status VARCHAR2(100);
    v_message VARCHAR2(500);
BEGIN
    sp_process_payment(
        p_order_id => 4001,
        p_payment_method => 'Credit Card',
        p_amount => 118000,
        p_user_id => 3005,
        p_transaction_id => v_transaction_id,
        p_status => v_status,
        p_message => v_message
    );
    
    DBMS_OUTPUT.PUT_LINE('Transaction ID: ' || v_transaction_id);
    DBMS_OUTPUT.PUT_LINE('Status: ' || v_status);
END;
/
```

### View Top Customers
```sql
SELECT 
    customer_name,
    city,
    total_spent,
    order_count,
    spending_rank
FROM vw_customer_rankings
WHERE ROWNUM <= 10
ORDER BY spending_rank;
```

---

## 📸 Screenshots

### Database Structure
![ER Diagram](screenshots/database_structure/er_diagram.png)
![Table Structure](screenshots/database_structure/tables_view.png)

### Sample Data
![Customer Data](screenshots/database_structure/sample_customers.png)
![Product Data](screenshots/database_structure/sample_products.png)

### PL/SQL Objects
![Procedures](screenshots/database_structure/procedures_list.png)
![Triggers](screenshots/database_structure/triggers_active.png)

### Test Results
![Order Validation Test](screenshots/test_results/order_validation.png)
![Trigger Restriction Test](screenshots/test_results/weekday_denial.png)

### Audit Logs
![Audit Log Entries](screenshots/audit_logs/operation_log.png)
![Denied Operations](screenshots/audit_logs/denied_attempts.png)

---

## 📈 Business Intelligence

### Key Performance Indicators (KPIs)

1. **Order Success Rate** - 95% (475/500 orders successful)
2. **Average Order Value** - 85,000 RWF
3. **Stock Accuracy** - 99.5%
4. **Customer Satisfaction** - 4.2/5.0 average rating
5. **Audit Compliance** - 100% (all operations logged)

### Analytics Capabilities

- Customer spending rankings and percentiles
- Product performance trends with month-over-month growth
- Top products by category
- Running totals and moving averages
- Order status distribution and conversion rates

---

## 🧪 Testing Results

| Test Case | Status | Details |
|-----------|--------|---------|
| Customer Validation | ✅ PASS | Invalid customers rejected |
| Stock Availability | ✅ PASS | Insufficient stock orders blocked |
| Credit Limit Check | ✅ PASS | Orders exceeding credit denied |
| Weekday Restriction | ✅ PASS | DML operations blocked Mon-Fri |
| Holiday Restriction | ✅ PASS | Operations blocked on holidays |
| Weekend Operations | ✅ PASS | All operations allowed Sat-Sun |
| Payment Processing | ✅ PASS | Transactions logged correctly |
| Audit Trail | ✅ PASS | All attempts captured |
| Bulk Operations | ✅ PASS | Compound trigger functional |
| Window Functions | ✅ PASS | Rankings and trends accurate |

**Total Tests:** 10 | **Passed:** 10 | **Failed:** 0 | **Success Rate:** 100%

---

## 📚 Documentation Links

- [Data Dictionary](documentation/data_dictionary.md) - Complete table and column definitions
- [Architecture Guide](documentation/architecture.md) - System design and components
- [Technical Manual](documentation/technical_manual.md) - Detailed implementation guide
- [User Guide](documentation/user_guide.md) - How to use the system
- [BI Requirements](business_intelligence/bi_requirements.md) - Analytics specifications
- [API Reference](documentation/api_reference.md) - Procedure and function signatures

---

## 🎓 Learning Outcomes

This project demonstrates mastery of:

- ✅ Complex database design with proper normalization
- ✅ Advanced PL/SQL programming (procedures, functions, packages)
- ✅ Trigger implementation for business rules
- ✅ Exception handling and error logging
- ✅ Cursor processing (explicit and bulk operations)
- ✅ Window functions for analytics
- ✅ Transaction management and data integrity
- ✅ Security through access control
- ✅ Comprehensive auditing and compliance
- ✅ Performance optimization with indexes

---

## 🤝 Contribution & Support

**Author:** SHEJA RADAN BORIS  
**Email:** boris.sheja@student.auca.ac.rw  
**Project Advisor:** [Advisor Name]  
**Institution:** AUCA - Adventist University of Central Africa

---

## 📄 License

This project is submitted as part of academic coursework at AUCA. All rights reserved.

---

## 🙏 Acknowledgments

- AUCA Faculty of Information Technology
- Oracle Database Documentation
- PL/SQL Best Practices Community
- Project Advisor and Reviewers

---

## 📌 Project Status

**Status:** ✅ Complete  
**Version:** 1.0  
**Last Updated:** December 2024  
**Database Objects:** 50+  
**Lines of Code:** 3,500+  
**Test Coverage:** 100%

---

**⭐ If you find this project helpful, please give it a star!**# plsql-final-exam-RADAN-BORIS-SHEJA
