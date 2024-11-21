
# **E-commerce Website Deployment with AWS**

This repository contains the complete setup and implementation for an E-commerce Order Management System. The system is deployed using AWS infrastructure with a Python-based backend and MariaDB for database management.

---

## **Table of Contents**
1. [Project Architecture](#project-architecture)
2. [Technologies Used](#technologies-used)
3. [Implementation Steps](#implementation-steps)
4. [Code Files](#code-files)
    - Backend: Python (Flask)
    - Frontend: HTML
5. [Database Schema](#database-schema)
6. [Testing and Validation](#testing-and-validation)

---

## **Project Architecture**
The project includes:
- A Virtual Private Cloud (VPC) with public and private subnets.
- EC2 instances for application and database hosting.
- An Auto Scaling Group for high availability.
- MariaDB primary and standby instances with replication.
- S3 for database backups and DNS setup.
- Amazon QuickSight for data visualization.

---

## **Technologies Used**
- **AWS Services**: EC2, VPC, Auto Scaling, S3, QuickSight
- **Backend**: Python with Flask
- **Database**: MariaDB
- **Frontend**: HTML, CSS, JavaScript

---

## **Implementation Steps**

### **2. Launch EC2 Instances**
- **Public Instances**:
  - Deploy the Python Flask application.
  
- **Private Instances**:
  - Deploy MariaDB (primary and standby).

### **3. Application Setup on EC2 (Public Instances)**
1. **Install Python and Flask**:
   ```bash
   sudo yum update -y
   sudo yum install python3 -y
   pip3 install flask pymysql
   ```

2. **Upload the following Python application script to `/home/ec2-user/app.py`**.

---

## **Code Files**

### **1. Backend: Python (Flask)**

**app.py**:
```python
from flask import Flask, request, jsonify, render_template
import pymysql
from datetime import datetime

app = Flask(__name__)

# Database connection settings
db_host = "10.0.134.170"  # Private IP of your MariaDB primary instance
db_user = "ecommerce_user"
db_password = "securepassword"
db_name = "ecommerce"

# Create connection to MariaDB
def create_connection():
    return pymysql.connect(host=db_host, user=db_user, password=db_password, database=db_name)

@app.route('/')
def home():
    return render_template("ecommerce_application.html")

@app.route('/add_order', methods=['POST'])
def add_order():
    customer_name = request.form['customer_name']
    product = request.form['product']
    quantity = request.form['quantity']
    order_date = request.form['order_date']
    order_date = datetime.strptime(order_date, '%Y-%m-%dT%H:%M')

    try:
        conn = create_connection()
        cursor = conn.cursor()
        query = "INSERT INTO orders (customer_name, product, quantity, order_date) VALUES (%s, %s, %s, %s)"
        cursor.execute(query, (customer_name, product, quantity, order_date))
        conn.commit()
        conn.close()
        return jsonify({"message": "Order added successfully!"}), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/get_orders', methods=['GET'])
def get_orders():
    try:
        conn = create_connection()
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM orders")
        result = cursor.fetchall()
        conn.close()
        orders = [{"id": row[0], "customer_name": row[1], "product": row[2], "quantity": row[3], "order_date": row[4].strftime('%Y-%m-%d %H:%M:%S')} for row in result]
        return jsonify(orders), 200
    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### **2. Frontend: HTML**

**ecommerce_application.html**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>E-commerce Platform</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f4f4; text-align: center; }
        .form-container { margin: 20px auto; }
    </style>
</head>
<body>
    <h1>E-commerce Order Management</h1>
    <div class="form-container">
        <form id="orderForm">
            <input type="text" id="customer_name" placeholder="Customer Name">
            <input type="text" id="product" placeholder="Product">
            <input type="number" id="quantity" placeholder="Quantity">
            <input type="datetime-local" id="order_date">
            <button type="submit">Add Order</button>
        </form>
    </div>
    <button onclick="fetchOrders()">Fetch Orders</button>
    <ul id="orders-list"></ul>
    <script>
        function fetchOrders() { /* JS Fetch Code */ }
    </script>
</body>
</html>
```

---

### **4. Database Schema**
**Table: `orders`**:
```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_name VARCHAR(255) NOT NULL,
    product VARCHAR(255) NOT NULL,
    quantity INT NOT NULL,
    order_date DATETIME NOT NULL
);
```

---

## **Testing and Validation**
1. Access the Flask app via the public instance IP.
2. Add and view orders through the frontend.

---

**Complete the remaining steps to integrate Auto Scaling, S3 backups, and QuickSight visualization!**
