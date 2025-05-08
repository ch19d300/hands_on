# Sample Datasets for Database Workshops

This document provides information about sample datasets you can use for the hands-on sessions on PostgreSQL (RDBMS), MongoDB (Document DB), and Neo4j (Graph DB).

## 1. PostgreSQL Sample Datasets

### 1.1 DVD Rental Database

The DVD Rental database is a standard sample database for PostgreSQL that includes 15 tables with data about a DVD rental store.

**Download Link**: [PostgreSQL Sample Database](https://www.postgresqltutorial.com/wp-content/uploads/2019/05/dvdrental.zip)

**Database Size**: ~16MB

**Installation**:
```bash
# Extract the zip file to get dvdrental.tar
unzip dvdrental.zip

# Copy the tar file to your Docker container
docker cp dvdrental.tar workshop-postgres:/tmp/

# Restore the database
docker exec -it workshop-postgres pg_restore -U workshop -d workshop /tmp/dvdrental.tar
```

**Database Schema**:
- actor: information about actors
- film: information about films
- film_actor: relationships between films and actors
- category: film categories
- film_category: relationships between films and categories
- store: store information
- inventory: inventory information
- rental: rental information
- payment: payment information
- staff: staff information
- customer: customer information
- address: address information for staff and customers
- city: city information
- country: country information
- language: language information for films

### 1.2 Northwind Database

The Northwind database is a sample database that was originally created by Microsoft for MS Access and later ported to other database systems.

**Download Link**: [Northwind PostgreSQL Script](https://github.com/pthom/northwind_psql/blob/master/northwind.sql)

**Database Size**: ~1MB

**Installation**:
```bash
# Download the SQL script
curl -o northwind.sql https://raw.githubusercontent.com/pthom/northwind_psql/master/northwind.sql

# Copy the SQL script to your Docker container
docker cp northwind.sql workshop-postgres:/tmp/

# Run the SQL script
docker exec -it workshop-postgres psql -U workshop -d workshop -f /tmp/northwind.sql
```

**Database Schema**:
- categories: product categories
- customers: customer information
- employees: employee information
- order_details: details about orders
- orders: order information
- products: product information
- regions: region information
- shippers: shipping company information
- suppliers: supplier information
- territories: territory information

### 1.3 E-commerce Database (Custom)

For the workshop, we've also created a custom e-commerce database schema that can be loaded using the following script:

```sql
-- Create tables for an e-commerce database
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE product_categories (
    category_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT
);

CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
    inventory_count INTEGER NOT NULL DEFAULT 0,
    category_id INTEGER REFERENCES product_categories(category_id)
);

CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(customer_id),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) CHECK (status IN ('pending', 'processing', 'shipped', 'delivered', 'cancelled')),
    total_amount DECIMAL(10, 2)
);

CREATE TABLE order_items (
    order_item_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id),
    product_id INTEGER REFERENCES products(product_id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price DECIMAL(10, 2) NOT NULL
);

-- Sample data (save as separate file - sample_data.sql)
INSERT INTO product_categories (name, description) VALUES 
('Electronics', 'Electronic devices and accessories'),
('Clothing', 'Apparel and fashion items'),
('Books', 'Books, e-books and publications'),
('Home & Kitchen', 'Home appliances and kitchen items'),
('Sports & Outdoors', 'Sporting goods and outdoor equipment');

INSERT INTO products (name, description, price, inventory_count, category_id) VALUES 
('Smartphone XL', '6.5 inch screen, 128GB storage', 699.99, 50, 1),
('Laptop Pro', '15 inch, 16GB RAM, 512GB SSD', 1299.99, 20, 1),
('Bluetooth Headphones', 'Wireless noise-cancelling headphones', 149.99, 75, 1),
('Smart Watch', 'Fitness tracking and notifications', 249.99, 30, 1),
('Cotton T-Shirt', 'Comfortable cotton t-shirt, available in multiple colors', 19.99, 100, 2),
('Jeans', 'Classic denim jeans', 39.99, 80, 2),
('Winter Jacket', 'Waterproof insulated jacket', 89.99, 40, 2),
('SQL Database Guide', 'Comprehensive guide to SQL databases', 39.99, 30, 3),
('Fiction Bestseller', 'Latest bestselling novel', 24.99, 60, 3),
('Cookbook', 'Recipes from around the world', 34.99, 25, 3),
('Coffee Maker', 'Programmable coffee maker, 12-cup capacity', 79.99, 15, 4),
('Blender', 'High-speed blender for smoothies and more', 69.99, 20, 4),
('Toaster Oven', 'Compact countertop oven and toaster', 59.99, 15, 4),
('Basketball', 'Official size and weight basketball', 29.99, 35, 5),
('Yoga Mat', 'Non-slip exercise mat', 24.99, 45, 5),
('Dumbbells Set', '5-25lb adjustable dumbbell set', 149.99, 10, 5);

INSERT INTO customers (first_name, last_name, email, phone) VALUES 
('John', 'Doe', 'john.doe@example.com', '555-123-4567'),
('Jane', 'Smith', 'jane.smith@example.com', '555-987-6543'),
('Michael', 'Johnson', 'michael.j@example.com', '555-222-3333'),
('Emily', 'Brown', 'emily.b@example.com', '555-444-5555'),
('David', 'Wilson', 'david.w@example.com', '555-666-7777'),
('Sarah', 'Taylor', 'sarah.t@example.com', '555-888-9999'),
('James', 'Anderson', 'james.a@example.com', '555-111-2222'),
('Lisa', 'Thomas', 'lisa.t@example.com', '555-333-4444'),
('Robert', 'Jackson', 'robert.j@example.com', '555-555-6666'),
('Jennifer', 'White', 'jennifer.w@example.com', '555-777-8888');

INSERT INTO orders (customer_id, status, total_amount) VALUES 
(1, 'delivered', 699.99),
(2, 'delivered', 1349.98),
(3, 'shipped', 174.98),
(4, 'processing', 129.97),
(5, 'pending', 79.99),
(1, 'delivered', 249.99),
(2, 'cancelled', 59.99),
(6, 'delivered', 224.97),
(7, 'shipped', 39.99),
(8, 'delivered', 94.98);

INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES 
(1, 1, 1, 699.99),
(2, 2, 1, 1299.99),
(2, 5, 2, 19.99),
(3, 8, 1, 39.99),
(3, 9, 1, 24.99),
(3, 10, 1, 34.99),
(4, 5, 3, 19.99),
(4, 9, 1, 24.99),
(4, 15, 1, 24.99),
(5, 11, 1, 79.99),
(6, 4, 1, 249.99),
(7, 13, 1, 59.99),
(8, 3, 1, 149.99),
(8, 15, 2, 24.99),
(8, 9, 1, 24.99),
(9, 6, 1, 39.99),
(10, 10, 1, 34.99),
(10, 15, 1, 24.99),
(10, 5, 1, 19.99),
(10, 9, 1, 24.99);
```

**Installation**:
```bash
# Save the SQL scripts to files
echo "CREATE TABLE customers..." > ecommerce_schema.sql
echo "INSERT INTO product_categories..." > ecommerce_data.sql

# Copy the SQL scripts to your Docker container
docker cp ecommerce_schema.sql workshop-postgres:/tmp/
docker cp ecommerce_data.sql workshop-postgres:/tmp/

# Run the SQL scripts
docker exec -it workshop-postgres psql -U workshop -d workshop -f /tmp/ecommerce_schema.sql
docker exec -it workshop-postgres psql -U workshop -d workshop -f /tmp/ecommerce_data.sql
```

## 2. MongoDB Sample Datasets

### 2.1 Airbnb Sample Dataset

The Airbnb sample dataset contains listings data from Airbnb, including information about properties, hosts, reviews, and pricing.

**Download Link**: [MongoDB Sample Datasets](https://github.com/mongodb/docs-assets/raw/geospatial/dataset/sample_airbnb.zip)

**Database Size**: ~15MB

**Installation**:
```bash
# Download and extract the dataset
curl -o sample_airbnb.zip https://github.com/mongodb/docs-assets/raw/geospatial/dataset/sample_airbnb.zip
unzip sample_airbnb.zip

# Copy JSON files to Docker container
docker cp sample_airbnb workshop-mongodb:/tmp/

# Import into MongoDB
docker exec -it workshop-mongodb mongoimport --db sample_airbnb --collection listings --file /tmp/sample_airbnb/sample_airbnb/listingsAndReviews.json --username workshop --password workshop
```

**Data Schema**:
- Listings with embedded reviews
- Property details including location, amenities, pricing
- Host information
- Review details and ratings

### 2.2 E-commerce Sample Dataset

A custom e-commerce dataset for document database modeling.

**Create the following JSON file (ecommerce.json):**
```json
[
  {
    "name": "Smartphone XL",
    "description": "6.5 inch screen, 128GB storage",
    "price": 699.99,
    "category": "Electronics",
    "inventory_count": 50,
    "tags": ["smartphone", "tech", "gadget"],
    "specifications": {
      "display": "6.5 inch OLED",
      "processor": "Octa-core",
      "storage": "128GB",
      "battery": "4500mAh"
    },
    "reviews": [
      {
        "user_id": "user123",
        "username": "TechFan",
        "rating": 4.5,
        "comment": "Great phone, excellent camera quality",
        "date": "2023-03-15"
      },
      {
        "user_id": "user456",
        "username": "GadgetGuru",
        "rating": 5,
        "comment": "Best smartphone I've ever owned",
        "date": "2023-04-22"
      }
    ]
  },
  {
    "name": "Laptop Pro",
    "description": "15 inch, 16GB RAM, 512GB SSD",
    "price": 1299.99,
    "category": "Electronics",
    "inventory_count": 20,
    "tags": ["laptop", "computer", "tech"],
    "specifications": {
      "display": "15 inch Retina",
      "processor": "Intel i7",
      "memory": "16GB",
      "storage": "512GB SSD"
    },
    "reviews": [
      {
        "user_id": "user789",
        "username": "DigitalNomad",
        "rating": 4,
        "comment": "Fast and reliable for work",
        "date": "2023-02-08"
      }
    ]
  },
  {
    "name": "Cotton T-Shirt",
    "description": "Comfortable cotton t-shirt, available in multiple colors",
    "price": 19.99,
    "category": "Clothing",
    "inventory_count": 100,
    "tags": ["shirt", "apparel", "casual"],
    "available_sizes": ["S", "M", "L", "XL"],
    "available_colors": ["Red", "Blue", "Black", "White"],
    "reviews": []
  }
]
```

**Installation**:
```bash
# Copy the JSON file to Docker container
docker cp ecommerce.json workshop-mongodb:/tmp/

# Import into MongoDB
docker exec -it workshop-mongodb mongoimport --db ecommerce --collection products --file /tmp/ecommerce.json --username workshop --password workshop
```

### 2.3 MongoDB Atlas Sample Datasets

MongoDB Atlas provides several ready-to-use sample datasets that you can load directly if you're using Atlas.

**Available collections include**:
- sample_airbnb
- sample_analytics
- sample_geospatial
- sample_mflix (movies data)
- sample_restaurants
- sample_supplies
- sample_training
- sample_weatherdata

If you're using Atlas, you can load these datasets directly from the Atlas UI.

## 3. Neo4j Sample Datasets

### 3.1 Movies Dataset

The Movies dataset is a small, entertainment-focused graph dataset that models movies, actors, directors, and their relationships.

**Installation**:
Neo4j Browser includes this dataset. Connect to your Neo4j instance and run:
```cypher
:play movies
```

Then click the "Create" button to install the dataset.

**Data Model**:
- Person nodes (actors, directors) with properties: name, born
- Movie nodes with properties: title, released, tagline
- Relationships: ACTED_IN, DIRECTED, PRODUCED, REVIEWED, FOLLOWS

### 3.2 Northwind Graph Dataset

A graph version of the classic Northwind database.

**Installation**:
```cypher
// Create constraints
CREATE CONSTRAINT FOR (p:Product) REQUIRE p.productID IS UNIQUE;
CREATE CONSTRAINT FOR (c:Category) REQUIRE c.categoryID IS UNIQUE;
CREATE CONSTRAINT FOR (s:Supplier) REQUIRE s.supplierID IS UNIQUE;
CREATE CONSTRAINT FOR (e:Employee) REQUIRE e.employeeID IS UNIQUE;
CREATE CONSTRAINT FOR (c:Customer) REQUIRE c.customerID IS UNIQUE;
CREATE CONSTRAINT FOR (o:Order) REQUIRE o.orderID IS UNIQUE;
CREATE CONSTRAINT FOR (s:Shipper) REQUIRE s.shipperID IS UNIQUE;
CREATE CONSTRAINT FOR (r:Region) REQUIRE r.regionID IS UNIQUE;
CREATE CONSTRAINT FOR (t:Territory) REQUIRE t.territoryID IS UNIQUE;

// Load CSV files
LOAD CSV WITH HEADERS FROM "https://raw.githubusercontent.com/neo4j-contrib/northwind-neo4j/master/data/products.csv" AS row
MERGE (n:Product {productID: toInteger(row.productID)})
SET n = row,
    n.unitPrice = toFloat(row.unitPrice),
    n.unitsInStock = toInteger(row.unitsInStock),
    n.unitsOnOrder = toInteger(row.unitsOnOrder),
    n.reorderLevel = toInteger(row.reorderLevel),
    n.discontinued = (row.discontinued = "1");

// Additional LOAD CSV statements for other entities...
// For the full script, visit: https://github.com/neo4j-contrib/northwind-neo4j
```

The full import script is quite lengthy. You can find it at the [Northwind Neo4j GitHub repository](https://github.com/neo4j-contrib/northwind-neo4j).

### 3.3 Social Network Dataset (Custom)

Create a custom social network dataset for the workshop:

```cypher
// Create Person nodes
CREATE (alice:Person {name: "Alice", age: 32})
CREATE (bob:Person {name: "Bob", age: 45})
CREATE (charlie:Person {name: "Charlie", age: 28})
CREATE (diana:Person {name: "Diana", age: 35})
CREATE (eve:Person {name: "Eve", age: 27})
CREATE (frank:Person {name: "Frank", age: 40})
CREATE (grace:Person {name: "Grace", age: 33})
CREATE (henry:Person {name: "Henry", age: 42})

// Create City nodes
CREATE (nyc:City {name: "New York"})
CREATE (sf:City {name: "San Francisco"})
CREATE (la:City {name: "Los Angeles"})
CREATE (chicago:City {name: "Chicago"})
CREATE (boston:City {name: "Boston"})

// Create Interest nodes
CREATE (tech:Interest {name: "Technology"})
CREATE (music:Interest {name: "Music"})
CREATE (sports:Interest {name: "Sports"})
CREATE (travel:Interest {name: "Travel"})
CREATE (cooking:Interest {name: "Cooking"})

// Create LIVES_IN relationships
CREATE (alice)-[:LIVES_IN {since: 2015}]->(nyc)
CREATE (bob)-[:LIVES_IN {since: 2010}]->(nyc)
CREATE (charlie)-[:LIVES_IN {since: 2018}]->(sf)
CREATE (diana)-[:LIVES_IN {since: 2019}]->(sf)
CREATE (eve)-[:LIVES_IN {since: 2017}]->(la)
CREATE (frank)-[:LIVES_IN {since: 2012}]->(chicago)
CREATE (grace)-[:LIVES_IN {since: 2016}]->(boston)
CREATE (henry)-[:LIVES_IN {since: 2014}]->(boston)

// Create FRIENDS_WITH relationships
CREATE (alice)-[:FRIENDS_WITH {since: 2018, strength: 8}]->(bob)
CREATE (alice)-[:FRIENDS_WITH {since: 2020, strength: 6}]->(charlie)
CREATE (alice)-[:FRIENDS_WITH {since: 2019, strength: 7}]->(diana)
CREATE (bob)-[:FRIENDS_WITH {since: 2015, strength: 9}]->(frank)
CREATE (charlie)-[:FRIENDS_WITH {since: 2019, strength: 8}]->(diana)
CREATE (charlie)-[:FRIENDS_WITH {since: 2018, strength: 7}]->(eve)
CREATE (diana)-[:FRIENDS_WITH {since: 2017, strength: 6}]->(eve)
CREATE (eve)-[:FRIENDS_WITH {since: 2016, strength: 8}]->(frank)
CREATE (frank)-[:FRIENDS_WITH {since: 2020, strength: 5}]->(grace)
CREATE (grace)-[:FRIENDS_WITH {since: 2015, strength: 9}]->(henry)
CREATE (bob)-[:FRIENDS_WITH {since: 2017, strength: 7}]->(diana)

// Create INTERESTED_IN relationships
CREATE (alice)-[:INTERESTED_IN {level: "high"}]->(tech)
CREATE (alice)-[:INTERESTED_IN {level: "medium"}]->(music)
CREATE (bob)-[:INTERESTED_IN {level: "medium"}]->(sports)
CREATE (bob)-[:INTERESTED_IN {level: "high"}]->(travel)
CREATE (charlie)-[:INTERESTED_IN {level: "high"}]->(tech)
CREATE (charlie)-[:INTERESTED_IN {level: "high"}]->(music)
CREATE (diana)-[:INTERESTED_IN {level: "medium"}]->(sports)
CREATE (diana)-[:INTERESTED_IN {level: "high"}]->(cooking)
CREATE (eve)-[:INTERESTED_IN {level: "high"}]->(music)
CREATE (eve)-[:INTERESTED_IN {level: "medium"}]->(cooking)
CREATE (frank)-[:INTERESTED_IN {level: "high"}]->(sports)
CREATE (frank)-[:INTERESTED_IN {level: "medium"}]->(travel)
CREATE (grace)-[:INTERESTED_IN {level: "high"}]->(cooking)
CREATE (grace)-[:INTERESTED_IN {level: "medium"}]->(travel)
CREATE (henry)-[:INTERESTED_IN {level: "high"}]->(sports)
CREATE (henry)-[:INTERESTED_IN {level: "medium"}]->(tech)
```

## 4. Cross-Database Comparison Dataset

To effectively compare different database types during the workshop, you can use a consistent domain model across all three databases. Here's a simplified e-commerce dataset adapted for each database type:

### 4.1 E-commerce for PostgreSQL (RDBMS)
Already covered in section 1.3 above.

### 4.2 E-commerce for MongoDB (Document DB)

```json
// products.json
[
  {
    "product_id": 1,
    "name": "Smartphone XL",
    "price": 699.99,
    "category": "Electronics",
    "description": "6.5 inch screen, 128GB storage",
    "specs": {
      "dimensions": "160 x 75 x 8 mm",
      "weight": "190g",
      "battery": "4500mAh"
    },
    "inventory_count": 50
  },
  // Add more products...
]

// customers.json
[
  {
    "customer_id": 1,
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@example.com",
    "addresses": [
      {
        "type": "shipping",
        "street": "123 Main St",
        "city": "Anytown",
        "state": "CA",
        "zip": "12345"
      },
      {
        "type": "billing",
        "street": "123 Main St",
        "city": "Anytown",
        "state": "CA",
        "zip": "12345"
      }
    ]
  },
  // Add more customers...
]

// orders.json
[
  {
    "order_id": 1,
    "customer_id": 1,
    "order_date": "2023-05-15T10:30:00Z",
    "status": "delivered",
    "shipping_address": {
      "street": "123 Main St",
      "city": "Anytown",
      "state": "CA",
      "zip": "12345"
    },
    "items": [
      {
        "product_id": 1,
        "name": "Smartphone XL",
        "quantity": 1,
        "unit_price": 699.99
      },
      {
        "product_id": 3,
        "name": "Bluetooth Headphones",
        "quantity": 1,
        "unit_price": 149.99
      }
    ],
    "total_amount": 849.98
  },
  // Add more orders...
]
```

### 4.3 E-commerce for Neo4j (Graph DB)

```cypher
// Create product categories
CREATE (electronics:Category {name: "Electronics", category_id: 1})
CREATE (clothing:Category {name: "Clothing", category_id: 2})
CREATE (books:Category {name: "Books", category_id: 3})

// Create products
CREATE (smartphone:Product {product_id: 1, name: "Smartphone XL", price: 699.99, description: "6.5 inch screen, 128GB storage", inventory_count: 50})
CREATE (laptop:Product {product_id: 2, name: "Laptop Pro", price: 1299.99, description: "15 inch, 16GB RAM, 512GB SSD", inventory_count: 20})
CREATE (headphones:Product {product_id: 3, name: "Bluetooth Headphones", price: 149.99, description: "Wireless noise-cancelling headphones", inventory_count: 75})
CREATE (tshirt:Product {product_id: 5, name: "Cotton T-Shirt", price: 19.99, description: "Comfortable cotton t-shirt", inventory_count: 100})
CREATE (book:Product {product_id: 8, name: "SQL Database Guide", price: 39.99, description: "Comprehensive guide to SQL databases", inventory_count: 30})

// Create category relationships
CREATE (smartphone)-[:BELONGS_TO]->(electronics)
CREATE (laptop)-[:BELONGS_TO]->(electronics)
CREATE (headphones)-[:BELONGS_TO]->(electronics)
CREATE (tshirt)-[:BELONGS_TO]->(clothing)
CREATE (book)-[:BELONGS_TO]->(books)

// Create customers
CREATE (john:Customer {customer_id: 1, first_name: "John", last_name: "Doe", email: "john.doe@example.com"})
CREATE (jane:Customer {customer_id: 2, first_name: "Jane", last_name: "Smith", email: "jane.smith@example.com"})

// Create addresses
CREATE (johnAddr:Address {address_id: 1, street: "123 Main St", city: "Anytown", state: "CA", zip: "12345"})
CREATE (janeAddr:Address {address_id: 2, street: "456 Oak Ave", city: "Othertown", state: "NY", zip: "67890"})

// Link customers to addresses
CREATE (john)-[:HAS_ADDRESS]->(johnAddr)
CREATE (jane)-[:HAS_ADDRESS]->(janeAddr)

// Create orders
CREATE (order1:Order {order_id: 1, order_date: "2023-05-15", status: "delivered", total_amount: 849.98})
CREATE (order2:Order {order_id: 2, order_date: "2023-06-10", status: "shipped", total_amount: 1319.98})

// Create relationships between orders, customers, and addresses
CREATE (john)-[:PLACED]->(order1)
CREATE (jane)-[:PLACED]->(order2)
CREATE (order1)-[:SHIPPED_TO]->(johnAddr)
CREATE (order2)-[:SHIPPED_TO]->(janeAddr)

// Create order items
CREATE (order1)-[:CONTAINS {quantity: 1, unit_price: 699.99}]->(smartphone)
CREATE (order1)-[:CONTAINS {quantity: 1, unit_price: 149.99}]->(headphones)
CREATE (order2)-[:CONTAINS {quantity: 1, unit_price: 1299.99}]->(laptop)
CREATE (order2)-[:CONTAINS {quantity: 1, unit_price: 19.99}]->(tshirt)
```

## 5. Data Generation Scripts

For generating larger datasets, consider using these tools:

1. **Mockaroo**: Web-based tool for generating realistic test data
   - [https://www.mockaroo.com/](https://www.mockaroo.com/)
   - Can export to CSV, JSON, SQL, etc.

2. **Faker.js**: JavaScript library for generating fake data
   - [https://github.com/faker-js/faker](https://github.com/faker-js/faker)
   - Good for programmatically creating large datasets

3. **Python Faker**: Python library similar to Faker.js
   - [https://github.com/joke2k/faker](https://github.com/joke2k/faker)
   - Useful with pandas for creating structured datasets

Example Python script for generating e-commerce data:

```python
import pandas as pd
from faker import Faker
import random
import datetime

fake = Faker()

# Generate customers
def generate_customers(num=100):
    customers = []
    for i in range(1, num+1):
        customers.append({
            'customer_id': i,
            'first_name': fake.first_name(),
            'last_name': fake.last_name(),
            'email': fake.email(),
            'phone': fake.phone_number(),
            'created_at': fake.date_time_between(start_date='-3y', end_date='now')
        })
    return pd.DataFrame(customers)

# Generate products
def generate_products(num=50):
    categories = ['Electronics', 'Clothing', 'Books', 'Home & Kitchen', 'Sports & Outdoors']
    products = []
    for i in range(1, num+1):
        category = random.choice(categories)
        products.append({
            'product_id': i,
            'name': fake.word().capitalize() + ' ' + fake.word().capitalize(),
            'description': fake.text(max_nb_chars=100),
            'price': round(random.uniform(9.99, 999.99), 2),
            'category': category,
            'inventory_count': random.randint(0, 100)
        })
    return pd.DataFrame(products)

# Generate orders and order items
def generate_orders(num_orders=200, num_customers=100, num_products=50):
    orders = []
    order_items = []
    statuses = ['pending', 'processing', 'shipped', 'delivered', 'cancelled']
    
    for i in range(1, num_orders+1):
        customer_id = random.randint(1, num_customers)
        order_date = fake.date_time_between(start_date='-1y', end_date='now')
        status = random.choice(statuses)
        
        orders.append({
            'order_id': i,
            'customer_id': customer_id,
            'order_date': order_date,
            'status': status
        })
        
        # Generate 1-5 items per order
        num_items = random.randint(1, 5)
        total = 0
        
        products_in_order = random.sample(range(1, num_products+1), num_items)
        for product_id in products_in_order:
            quantity = random.randint(1, 3)
            unit_price = round(random.uniform(9.99, 999.99), 2)
            total += quantity * unit_price
            
            order_items.append({
                'order_id': i,
                'product_id': product_id,
                'quantity': quantity,
                'unit_price': unit_price
            })
        
        # Update order with total
        orders[-1]['total_amount'] = round(total, 2)
            
    return pd.DataFrame(orders), pd.DataFrame(order_items)

# Generate data
customers_df = generate_customers(100)
products_df = generate_products(50)
orders_df, order_items_df = generate_orders(200, 100, 50)

# Export to CSV
customers_df.to_csv('customers.csv', index=False)
products_df.to_csv('products.csv', index=False)
orders_df.to_csv('orders.csv', index=False)
order_items_df.to_csv('order_items.csv', index=False)
```

This script can be modified to generate data in different formats (SQL, JSON, etc.) for the different database workshops.