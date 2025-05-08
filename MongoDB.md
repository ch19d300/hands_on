# MongoDB Hands-On Workshop

## Introduction to Unstructured Databases with MongoDB

This workshop will guide you through practical exercises using MongoDB, a popular document-oriented NoSQL database that provides flexibility for handling unstructured and semi-structured data.

## Workshop Objectives
- Understand document-oriented database concepts
- Design schema-less document models
- Practice CRUD operations and complex queries
- Explore indexing and performance optimization
- Implement data aggregation pipelines

## Connecting to MongoDB

### Using Mongosh (Command Line)
```bash
# From Docker
docker exec -it workshop-mongodb mongosh --username workshop --password workshop

# Direct connection (if MongoDB client is installed)
mongosh "mongodb://workshop:workshop@localhost:27017/"
```

### Using MongoDB Compass (GUI)
1. Download and install MongoDB Compass from [https://www.mongodb.com/products/compass](https://www.mongodb.com/products/compass)
2. Create a new connection:
   - Connection string: `mongodb://workshop:workshop@localhost:27017/`

## Part 1: Document Model Design

### Document Database Concepts
Unlike relational databases, MongoDB stores data as flexible, JSON-like documents where:
- Fields can vary between documents
- Data structure can be changed over time
- Related data can be embedded in a single document
- No need for predefined schemas

### Basic Terminology
- **Database**: Container for collections
- **Collection**: Group of documents (similar to a table)
- **Document**: Set of key-value pairs (similar to a row)
- **Field**: Key-value pair in a document (similar to a column)
- **Embedded Document**: Document nested inside another document

### Creating a Database and Collections

```javascript
// Create and use a database
use ecommerce

// Collections are created implicitly when documents are inserted
// But we can create them explicitly with schema validation
db.createCollection("customers", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["first_name", "last_name", "email"],
      properties: {
        first_name: { bsonType: "string" },
        last_name: { bsonType: "string" },
        email: { bsonType: "string" },
        phone: { bsonType: "string" },
        created_at: { bsonType: "date" }
      }
    }
  }
})

// Create other collections
db.createCollection("products")
db.createCollection("orders")
```

### Exercise 1: Data Modeling
Design a document model for a blog platform with posts and comments. Consider how to structure the relationship between posts and comments:

1. Embedded model (comments inside posts)
2. Referenced model (separate collections with references)

Compare the advantages and disadvantages of each approach.

## Part 2: Basic CRUD Operations

### Create (Insert) Operations

```javascript
// Insert one document
db.customers.insertOne({
  first_name: "John",
  last_name: "Doe",
  email: "john.doe@example.com",
  phone: "555-123-4567",
  created_at: new Date()
})

// Insert multiple documents
db.products.insertMany([
  {
    name: "Smartphone XL",
    description: "6.5 inch screen, 128GB storage",
    price: 699.99,
    category: "Electronics",
    inventory_count: 50,
    tags: ["smartphone", "gadget", "tech"]
  },
  {
    name: "Laptop Pro",
    description: "15 inch, 16GB RAM, 512GB SSD",
    price: 1299.99,
    category: "Electronics",
    inventory_count: 20,
    specifications: {
      processor: "Intel i7",
      memory: "16GB",
      storage: "512GB SSD"
    }
  },
  {
    name: "Cotton T-Shirt",
    description: "Comfortable cotton t-shirt, available in multiple colors",
    price: 19.99,
    category: "Clothing",
    inventory_count: 100,
    available_sizes: ["S", "M", "L", "XL"],
    available_colors: ["Red", "Blue", "Black", "White"]
  }
])

// Insert an order with embedded items
db.orders.insertOne({
  customer_id: ObjectId("..."), // Replace with actual customer _id
  order_date: new Date(),
  status: "pending",
  shipping_address: {
    street: "123 Main St",
    city: "Anytown",
    state: "CA",
    zip: "12345",
    country: "USA"
  },
  items: [
    {
      product_id: ObjectId("..."), // Replace with actual product _id
      name: "Smartphone XL",
      quantity: 2,
      unit_price: 699.99
    },
    {
      product_id: ObjectId("..."), // Replace with actual product _id
      name: "Cotton T-Shirt",
      quantity: 3,
      unit_price: 19.99
    }
  ],
  total_amount: 1459.95
})
```

### Read (Query) Operations

```javascript
// Find all documents in a collection
db.products.find()

// Format the output
db.products.find().pretty()

// Query with filters
db.products.find({ category: "Electronics" })

// Query with operators
db.products.find({ price: { $gt: 500 } })

// Query with multiple conditions
db.products.find({
  category: "Electronics",
  price: { $lt: 1000 },
  inventory_count: { $gte: 10 }
})

// Query embedded documents
db.orders.find({ "shipping_address.city": "Anytown" })

// Query arrays
db.products.find({ tags: "smartphone" })
db.products.find({ available_sizes: "M" })

// Limit fields returned
db.products.find({ category: "Electronics" }, { name: 1, price: 1, _id: 0 })

// Sort results
db.products.find().sort({ price: -1 }) // Descending
db.products.find().sort({ category: 1, price: -1 }) // Ascending category, descending price

// Limit and skip (pagination)
db.products.find().limit(10)
db.products.find().skip(10).limit(10) // Second page
```

### Update Operations

```javascript
// Update one document
db.products.updateOne(
  { name: "Smartphone XL" },
  { $set: { price: 649.99 } }
)

// Update multiple documents
db.products.updateMany(
  { category: "Electronics" },
  { $inc: { inventory_count: -5 } }
)

// Update with multiple operations
db.products.updateOne(
  { name: "Cotton T-Shirt" },
  {
    $set: { "description": "Updated description" },
    $push: { "available_colors": "Green" },
    $inc: { "inventory_count": 50 }
  }
)

// Upsert (insert if not exists)
db.products.updateOne(
  { name: "New Product" },
  { $set: { price: 99.99, category: "Accessories" } },
  { upsert: true }
)
```

### Delete Operations

```javascript
// Delete one document
db.products.deleteOne({ name: "Discontinued Product" })

// Delete multiple documents
db.products.deleteMany({ inventory_count: 0 })

// Delete all documents in a collection
db.discontinued_products.deleteMany({})

// Drop a collection
db.discontinued_products.drop()
```

### Exercise 2: CRUD Operations
Complete the following tasks:

1. Create at least 5 more products with varying attributes
2. Find all products in a specific price range
3. Update inventory for multiple products
4. Add a new field to all products in a specific category
5. Delete products that meet certain criteria

## Part 3: Advanced Queries and Indexing

### Complex Queries

```javascript
// Logical operators
db.products.find({
  $or: [
    { category: "Electronics" },
    { price: { $gt: 500 } }
  ]
})

db.products.find({
  $and: [
    { category: { $in: ["Electronics", "Accessories"] } },
    { inventory_count: { $gt: 0 } },
    { $or: [
      { price: { $lt: 100 } },
      { tags: "premium" }
    ]}
  ]
})

// Element operators
db.products.find({ specifications: { $exists: true } })
db.products.find({ description: { $type: "string" } })

// Array operators
db.products.find({ tags: { $all: ["smartphone", "tech"] } })
db.products.find({ available_sizes: { $size: 4 } })

// Text search (requires text index)
db.products.createIndex({ description: "text" })
db.products.find({ $text: { $search: "comfortable cotton" } })
```

### Indexing

```javascript
// Create a single field index
db.products.createIndex({ name: 1 }) // 1 for ascending, -1 for descending

// Create a compound index
db.products.createIndex({ category: 1, price: -1 })

// Create a unique index
db.customers.createIndex({ email: 1 }, { unique: true })

// List all indexes
db.products.getIndexes()

// Drop an index
db.products.dropIndex("name_1")

// Create a TTL index (Time To Live)
db.sessions.createIndex({ created_at: 1 }, { expireAfterSeconds: 3600 })
```

### Exercise 3: Advanced Queries and Indexing
Complete the following tasks:

1. Create a compound index for a frequently used query
2. Write a complex query that uses logical operators
3. Use text search to find products with specific terms in their description
4. Analyze query performance with and without appropriate indexes

## Part 4: Aggregation Framework

### Basic Aggregation

```javascript
// Simple group and count
db.products.aggregate([
  { $group: { _id: "$category", count: { $sum: 1 } } }
])

// Multiple stages
db.products.aggregate([
  { $match: { price: { $gt: 100 } } },
  { $group: { _id: "$category", avg_price: { $avg: "$price" } } },
  { $sort: { avg_price: -1 } }
])

// Project to reshape documents
db.products.aggregate([
  { $project: { 
      name: 1, 
      category: 1, 
      price: 1,
      price_range: {
        $switch: {
          branches: [
            { case: { $lt: ["$price", 50] }, then: "Budget" },
            { case: { $lt: ["$price", 500] }, then: "Mid-range" }
          ],
          default: "Premium"
        }
      }
    }
  }
])
```

### Complex Aggregation

```javascript
// Multiple operations
db.orders.aggregate([
  { $match: { status: "delivered" } },
  { $unwind: "$items" },
  { $group: {
      _id: "$items.product_id",
      product_name: { $first: "$items.name" },
      total_quantity: { $sum: "$items.quantity" },
      total_revenue: { $sum: { $multiply: ["$items.quantity", "$items.unit_price"] } }
    }
  },
  { $sort: { total_revenue: -1 } },
  { $limit: 5 }
])

// Lookup (similar to JOIN)
db.orders.aggregate([
  { $match: { status: "delivered" } },
  { $lookup: {
      from: "customers",
      localField: "customer_id",
      foreignField: "_id",
      as: "customer_info"
    }
  },
  { $unwind: "$customer_info" },
  { $project: {
      order_id: "$_id",
      customer_name: { $concat: ["$customer_info.first_name", " ", "$customer_info.last_name"] },
      total_amount: 1,
      order_date: 1
    }
  }
])

// Facets (multiple aggregation pipelines)
db.products.aggregate([
  { $facet: {
      "categoryCounts": [
        { $group: { _id: "$category", count: { $sum: 1 } } }
      ],
      "priceRanges": [
        { $bucket: {
            groupBy: "$price",
            boundaries: [0, 50, 200, 500, 1000, 2000],
            default: "2000+",
            output: { count: { $sum: 1 } }
          }
        }
      ],
      "inventorySummary": [
        { $group: {
            _id: null,
            totalProducts: { $sum: 1 },
            totalInventory: { $sum: "$inventory_count" },
            avgPrice: { $avg: "$price" }
          }
        }
      ]
    }
  }
])
```

### Exercise 4: Aggregation
Complete the following tasks:

1. Create an aggregation pipeline to find the most popular product categories by sales
2. Generate a report of monthly sales over time
3. Create a customer spending analysis
4. Use facets to create a dashboard summary of product data

## Part 5: Data Modeling Patterns

### Embedding vs. Referencing

```javascript
// Embedding example (one-to-few)
db.orders.insertOne({
  customer_id: ObjectId("..."),
  order_items: [
    { product_id: ObjectId("..."), quantity: 2, price: 19.99 },
    { product_id: ObjectId("..."), quantity: 1, price: 29.99 }
  ]
})

// Referencing example (one-to-many)
db.posts.insertOne({
  title: "MongoDB Patterns",
  content: "...",
  author_id: ObjectId("...")
})

db.comments.insertMany([
  { post_id: ObjectId("..."), text: "Great post!", author_id: ObjectId("...") },
  { post_id: ObjectId("..."), text: "Thanks for sharing", author_id: ObjectId("...") }
])
```

### Common Patterns

```javascript
// Subset pattern (store most frequently accessed fields)
db.products.insertOne({
  name: "Popular Product",
  price: 129.99,
  summary: "Brief summary...", // Frequently accessed
  category: "Electronics",
  tags: ["popular", "featured"],
  // Detailed fields kept in a separate collection
  detailed_id: ObjectId("...")
})

// Computed pattern (pre-compute expensive operations)
db.products.insertOne({
  name: "Widget",
  price: 49.99,
  calculated_fields: {
    tax: 4.50,
    shipping_estimate: 5.99,
    total_with_tax: 54.49
  }
})

// Bucket pattern (group related documents)
db.temperature_readings.insertOne({
  sensor_id: "sensor1",
  date: ISODate("2023-01-01"),
  readings: [
    { time: ISODate("2023-01-01T00:00:00Z"), value: 21.5 },
    { time: ISODate("2023-01-01T01:00:00Z"), value: 21.3 },
    // ... more hourly readings
  ]
})
```

### Exercise 5: Data Modeling
Choose one of the following scenarios and design an appropriate data model:

1. A social media platform with users, posts, comments, and likes
2. An e-learning system with courses, lessons, students, and progress tracking
3. An inventory management system for multiple warehouses

Consider:
- Embedding vs. referencing decisions
- What indexes would be needed
- How to optimize for common queries

## Sample Dataset

For more extensive practice, we'll use the sample Airbnb dataset:

```javascript
// Import sample dataset (Airbnb listings)
// In your Docker environment, run:
docker exec -it workshop-mongodb mongosh --username workshop --password workshop --eval "use sample_airbnb" --shell

// Alternative: use mongoimport with provided JSON file
docker cp airbnb.json workshop-mongodb:/tmp/
docker exec -it workshop-mongodb mongoimport --db sample_airbnb --collection listings --file /tmp/airbnb.json --username workshop --password workshop

// Or you can use MongoDB Atlas sample datasets directly
// Just click on "Load Sample Dataset" in MongoDB Atlas

// Explore the sample data
db.listings.find().limit(1).pretty()
db.listings.countDocuments()

// Sample queries with the Airbnb dataset
// Find all listings in a specific location
db.listings.find({ "address.country": "United States", "address.city": "New York" }).limit(10)

// Find average price by room type
db.listings.aggregate([
  { $group: { _id: "$room_type", avg_price: { $avg: "$price" }, count: { $sum: 1 } } },
  { $sort: { avg_price: -1 } }
])

// Find top-rated listings with at least 50 reviews
db.listings.find({ 
  "review_scores.review_scores_rating": { $gte: 95 },
  "number_of_reviews": { $gte: 50 }
}).sort({ "number_of_reviews": -1 }).limit(10)

## Additional Exercises

1. **Data Analysis**: Find the neighborhoods with the highest average rental prices
2. **Geospatial Queries**: Find all listings within 2km of a specific location
3. **Data Enrichment**: Add weather data to listings based on their location
4. **Performance Tuning**: Create appropriate indexes and measure query performance improvements

## MongoDB vs. RDBMS Comparison Exercise

Create equivalent solutions using both MongoDB and PostgreSQL for the following scenario:

- Track order processing including customer details, order items, shipping information
- Implement product inventory management
- Generate sales reports by category, time period, and customer

Compare:
- Data modeling approaches
- Query complexity
- Performance characteristics
- Flexibility for schema changes
- Implementation effort

## Resources
- [MongoDB Documentation](https://docs.mongodb.com/)
- [MongoDB University (Free Courses)](https://university.mongodb.com/)
- [MongoDB Compass Documentation](https://docs.mongodb.com/compass/current/)
- [MongoDB Schema Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)