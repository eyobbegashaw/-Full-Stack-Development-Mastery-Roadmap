## **MongoDB Query Operators - Notes & Explanations**

---

### **1. Query Operators Overview**
**Notes:** Query operators in MongoDB are special keywords prefixed with `$` that enable complex filtering and data manipulation within queries. They are categorized into comparison, logical, element, array, and projection operators. These operators allow for sophisticated query patterns beyond simple equality checks, including range queries, pattern matching, array manipulation, and conditional logic. Understanding these operators is essential for writing efficient queries that leverage MongoDB's indexing capabilities while maintaining readability and performance.

**Code:**
```javascript
// Basic structure of query operators
db.collection.find({
  field: { $operator: value }
});
// Example combining multiple operators
db.users.find({
  age: { $gte: 18, $lte: 65 },
  status: { $in: ["active", "pending"] }
});
```

---

### **2. Array Operators**
**Notes:** Array operators query and manipulate array fields. `$in` and `$nin` match values in/not in an array. `$all` requires all specified values in the array regardless of order. `$elemMatch` matches documents where at least one array element satisfies all given conditions, useful for querying arrays of subdocuments. `$size` matches arrays with specific lengths. These operators enable complex array queries while supporting multikey indexes for performance optimization.

**Code:**
```javascript
// Array operator examples
db.products.find({
  tags: { $all: ["electronics", "sale"] } // Must have both tags
});

db.students.find({
  scores: { $elemMatch: { $gte: 80, $lt: 90 } } // Scores between 80-90
});

db.orders.find({
  items: { $size: 3 } // Orders with exactly 3 items
});
```

---

### **3. Projection Operators**
**Notes:** Projection operators control which fields are returned in query results. `$project` (in aggregation) reshapes documents. In find operations, include fields with `1` or `true`, exclude with `0` or `false`. `$slice` limits array field elements returned (positive for first n, negative for last n, or [skip, limit]). Field exclusion is default for `_id` only; all other fields must be explicitly included. Projection reduces network overhead and memory usage.

**Code:**
```javascript
// Projection examples
db.users.find(
  { status: "active" },
  { 
    name: 1,           // Include name
    email: 1,          // Include email
    _id: 0,            // Exclude _id
    orders: { $slice: 5 } // Only first 5 orders
  }
);

// Aggregation $project
db.users.aggregate([
  { $project: { 
    fullName: { $concat: ["$firstName", " ", "$lastName"] },
    yearJoined: { $year: "$joinDate" }
  }}
]);
```

---

### **4. Element Operators**
**Notes:** Element operators query documents based on field existence or data type. `$exists` checks if a field exists (true) or doesn't exist (false). `$type` matches fields of specific BSON types (string, number, date, etc.). `$regex` provides pattern matching within strings using Perl-compatible regular expressions with optional flags (`i` for case-insensitive, `m` for multiline). These operators are essential for schema validation, data cleaning, and flexible querying of heterogeneous documents.

**Code:**
```javascript
// Element operator examples
db.customers.find({
  phone: { $exists: true },       // Has phone field
  email: { $regex: /@company\.com$/i } // Email ends with @company.com
});

db.data.find({
  value: { $type: "decimal" }     // Field is Decimal128 type
});

// Case-insensitive search for names starting with "john"
db.users.find({
  name: { $regex: /^john/i }      // 'i' flag = case-insensitive
});
```

---

### **5. Comparison Operators**
**Notes:** Comparison operators evaluate field values relative to specified values. `$eq` (equals, often implicit), `$ne` (not equals), `$gt` (greater than), `$gte` (greater than or equal), `$lt` (less than), `$lte` (less than or equal). These operators work with all BSON types following type comparison order. They support compound conditions on the same field and are efficiently indexed. Comparison operators are fundamental for range queries, inequality checks, and data filtering.

**Code:**
```javascript
// Comparison operator examples
db.products.find({
  price: { $gte: 10, $lte: 100 },     // Price between 10 and 100
  category: { $ne: "discontinued" }    // Not discontinued
});

db.orders.find({
  total: { $gt: 1000 },               // Orders over $1000
  status: "completed"
});

// Implicit $eq (both are equivalent)
db.users.find({ age: 25 });
db.users.find({ age: { $eq: 25 } });
```

---

### **6. Logical Operators**
**Notes:** Logical operators combine multiple query conditions. `$and` implicitly joins comma-separated conditions; explicit `$and` is needed for multiple conditions on same field. `$or` matches documents satisfying at least one condition. `$nor` matches documents failing all conditions. `$not` inverts the effect of another operator. These operators enable complex boolean logic, with careful attention to operator precedence: `$not` > `$and` > `$or`/`$nor`. Efficient use of logical operators is key to performant queries.

**Code:**
```javascript
// Logical operator examples
db.users.find({
  $and: [
    { age: { $gte: 18 } },
    { age: { $lte: 65 } }
  ]
});

db.products.find({
  $or: [
    { category: "electronics" },
    { price: { $lt: 50 } }
  ]
});

db.accounts.find({
  $nor: [
    { status: "closed" },
    { balance: { $lt: 0 } }
  ]
});

// $not example
db.data.find({
  value: { $not: { $regex: /test/i } } // Values not containing "test"
});
```

---

## **Combined Example**
**Notes:** Real-world queries often combine multiple operator types. For instance, finding active users with specific tags in their array, within an age range, and returning only selected fields. Understanding operator interaction is crucial: array operators work with array fields, comparison operators with scalar values, logical operators combine conditions, and projection operators control output. Performance considerations include index usage (comparison operators benefit most), avoiding collection scans with `$regex` without anchors, and minimizing `$or` conditions that can't use compound indexes efficiently.

**Code:**
```javascript
// Complex combined query example
db.users.find({
  // Logical operators combine conditions
  $and: [
    // Comparison operators for ranges
    { age: { $gte: 21, $lte: 40 } },
    // Element operator for field existence
    { email: { $exists: true } },
    // Array operator for array matching
    { tags: { $all: ["verified", "premium"] } }
  ],
  // Regular expression for pattern matching
  name: { $regex: /^[A-J]/i }
},
// Projection operators control returned fields
{
  name: 1,
  email: 1,
  age: 1,
  tags: { $slice: 3 },  // Only first 3 tags
  _id: 0
}).sort({ age: 1 }).limit(50); // Additional cursor methods
```

---

## **Summary**
MongoDB's query operators provide a powerful, expressive language for data retrieval and manipulation. Comparison operators handle value relationships, logical operators combine conditions, array operators work with list data, element operators check field properties, and projection operators control output shape. Mastering these operators enables efficient, targeted queries that leverage MongoDB's indexing capabilities while maintaining flexibility for complex data structures and query requirements. Proper operator selection significantly impacts query performance and result relevance.

## **MongoDB Aggregation, Transactions & Common Operators - Notes & Explanations**

---

### **1. Aggregation**
**Notes:** Aggregation in MongoDB is a powerful framework for data processing and transformation through a pipeline of stages. It processes documents sequentially through these stages, transforming them into aggregated results. The aggregation pipeline is more efficient than multiple find operations for complex data processing as it operates at the database level, reducing network overhead. Aggregation supports operations like filtering, grouping, sorting, joining (via `$lookup`), and computing derived fields. It can process terabytes of data and supports indexing optimization for pipeline stages. The `aggregate()` method returns a cursor for large result sets.

**Code:**
```javascript
// Basic aggregation pipeline
db.orders.aggregate([
  { $match: { status: "completed" } },      // Stage 1: Filter documents
  { $group: {                               // Stage 2: Group and calculate
    _id: "$customerId",
    totalSpent: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }},
  { $sort: { totalSpent: -1 } },            // Stage 3: Sort results
  { $limit: 10 }                            // Stage 4: Limit output
]);
```

---

### **2. Transactions**
**Notes:** Transactions in MongoDB provide ACID (Atomicity, Consistency, Isolation, Durability) guarantees across multiple documents and collections. Introduced in MongoDB 4.0 for replica sets and 4.2 for sharded clusters, transactions allow grouping operations that either all succeed or all fail. They maintain data consistency during complex operations like financial transfers or inventory management. Transactions use snapshot isolation and can be configured with read/write concerns. While powerful, they have performance overhead and should be used judiciously. Session-based transactions require explicit start, commit, or abort calls.

**Code:**
```javascript
// Multi-document transaction example
const session = client.startSession();
try {
  session.startTransaction({
    readConcern: { level: "snapshot" },
    writeConcern: { w: "majority" }
  });
  
  // Transfer funds between accounts
  await db.accounts.updateOne(
    { _id: "acc1", balance: { $gte: 100 } },
    { $inc: { balance: -100 } },
    { session }
  );
  
  await db.accounts.updateOne(
    { _id: "acc2" },
    { $inc: { balance: 100 } },
    { session }
  );
  
  // Record the transaction
  await db.transactions.insertOne({
    from: "acc1",
    to: "acc2",
    amount: 100,
    timestamp: new Date()
  }, { session });
  
  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```

---

### **3. Common Operators**

#### **$match**
**Notes:** The `$match` operator filters documents to pass only those that match specified conditions to the next pipeline stage. It uses the same query syntax as `find()` and should be placed early in the pipeline to reduce the number of documents processed. `$match` can leverage indexes when placed before `$project`, `$unwind`, or `$group` stages. It's essential for performance optimization in aggregation pipelines and can be used multiple times in a single pipeline.

**Code:**
```javascript
// $match example
db.sales.aggregate([
  { $match: { 
    date: { $gte: new Date("2024-01-01"), $lt: new Date("2024-02-01") },
    region: { $in: ["North", "South"] }
  }}
]);
```

#### **$group**
**Notes:** The `$group` operator groups documents by a specified `_id` expression and applies accumulator expressions to each group. The `_id` can be a single field, multiple fields, or a computed expression. Accumulators like `$sum`, `$avg`, `$min`, `$max`, `$push`, and `$addToSet` calculate values per group. `$group` is memory-intensive and benefits from being preceded by `$match` and `$sort` stages to reduce working set size.

**Code:**
```javascript
// $group with multiple accumulators
db.orders.aggregate([
  { $group: {
    _id: { 
      year: { $year: "$orderDate" },
      month: { $month: "$orderDate" }
    },
    totalRevenue: { $sum: "$amount" },
    averageOrder: { $avg: "$amount" },
    orderIds: { $push: "$orderId" },
    uniqueCustomers: { $addToSet: "$customerId" },
    count: { $sum: 1 }
  }}
]);
```

#### **$sort**
**Notes:** The `$sort` operator orders all documents in the pipeline by specified field(s) and sort order (1 for ascending, -1 for descending). It can sort by multiple fields for tie-breaking. `$sort` is memory-intensive for large datasets and benefits from indexes when possible. Placing `$sort` before `$group` can improve group performance, while `$sort` after `$group` orders aggregated results.

**Code:**
```javascript
// $sort examples
db.products.aggregate([
  { $sort: { category: 1, price: -1 } }, // Sort by category asc, then price desc
  { $limit: 100 }
]);

// Efficient: sort before group when grouping needs sorted data
db.logs.aggregate([
  { $match: { level: "ERROR" } },
  { $sort: { timestamp: -1 } },
  { $group: {
    _id: { $dateToString: { format: "%Y-%m-%d", date: "$timestamp" } },
    errors: { $push: "$$ROOT" }
  }}
]);
```

#### **$project**
**Notes:** The `$project` operator reshapes documents by including, excluding, or adding new fields. It can compute new fields using expressions, rename fields, and create subdocuments. `$project` can significantly change document structure and should be used strategically to reduce data volume in later stages. Multiple `$project` stages can be used, but consolidating them improves performance. Field exclusion is generally more efficient than inclusion when removing many fields.

**Code:**
```javascript
// $project with field transformation
db.users.aggregate([
  { $project: {
    fullName: { $concat: ["$firstName", " ", "$lastName"] },
    email: 1,
    age: {
      $floor: {
        $divide: [
          { $subtract: [new Date(), "$birthDate"] },
          365 * 24 * 60 * 60 * 1000
        ]
      }
    },
    metadata: {
      accountAgeDays: { $dateDiff: { startDate: "$createdAt", endDate: new Date(), unit: "day" } },
      hasProfile: { $cond: [{ $ifNull: ["$profileImage", false] }, true, false] }
    },
    _id: 0
  }}
]);
```

#### **$skip & $limit**
**Notes:** `$skip` bypasses a specified number of documents, while `$limit` restricts the number of documents passed to the next stage. Together they implement pagination but can be inefficient for large skips as MongoDB must still process skipped documents. For efficient pagination, combine with `$sort` and use range queries or keyset pagination instead. `$limit` without `$skip` is useful for sampling or top-N queries.

**Code:**
```javascript
// Pagination with $skip and $limit (inefficient for large skips)
db.products.aggregate([
  { $sort: { createdAt: -1 } },
  { $skip: 100 },   // Skip first 100 documents
  { $limit: 20 }    // Return next 20 documents
]);

// Better: Keyset pagination for large datasets
const lastSeenId = ObjectId("...");
db.products.aggregate([
  { $match: { _id: { $gt: lastSeenId } } },
  { $sort: { _id: 1 } },
  { $limit: 20 }
]);
```

#### **$unwind**
**Notes:** `$unwind` deconstructs an array field, outputting one document per array element while preserving other fields. It's essential for working with array data in aggregations but can create many documents (cartesian explosion). Options include `preserveNullAndEmptyArrays` to output documents even when the array is null/empty. Use `$unwind` strategically after filtering with `$match` to reduce document multiplication.

**Code:**
```javascript
// $unwind examples
db.orders.aggregate([
  { $unwind: "$items" }, // Creates one doc per order item
  { $group: {
    _id: "$items.productId",
    totalQuantity: { $sum: "$items.quantity" },
    totalRevenue: { $sum: { $multiply: ["$items.price", "$items.quantity"] } }
  }}
]);

// With preserveNullAndEmptyArrays
db.users.aggregate([
  { $unwind: { 
    path: "$tags",
    preserveNullAndEmptyArrays: true // Keep users without tags
  }}
]);
```

#### **$lookup**
**Notes:** The `$lookup` operator performs a left outer join between two collections, adding an array of matching documents from the "joined" collection. It supports simple equality joins, uncorrelated subqueries, and pipeline-based joins for complex conditions. `$lookup` can be performance-intensive; ensure joined fields are indexed. Multiple `$lookup` stages can create complex data relationships, similar to SQL joins.

**Code:**
```javascript
// $lookup examples
// Simple equality join
db.orders.aggregate([
  { $lookup: {
    from: "customers",
    localField: "customerId",
    foreignField: "_id",
    as: "customerInfo"
  }},
  { $unwind: "$customerInfo" } // Convert array to object
]);

// Pipeline-based join with conditions
db.products.aggregate([
  { $lookup: {
    from: "reviews",
    let: { productId: "$_id" },
    pipeline: [
      { $match: { 
        $expr: { $eq: ["$productId", "$$productId"] },
        rating: { $gte: 4 }
      }},
      { $limit: 5 }
    ],
    as: "topReviews"
  }}
]);
```

#### **$sum**
**Notes:** `$sum` is an accumulator operator used primarily in `$group` stages to calculate totals. It can sum numeric values from documents (`$sum: "$field"`) or count documents (`$sum: 1`). When used with `$project`, it can sum array elements via `$reduce` or conditional sums with `$cond`. `$sum` ignores non-numeric values (treats as 0) and null/missing fields.

**Code:**
```javascript
// $sum examples
db.sales.aggregate([
  { $group: {
    _id: "$salesperson",
    totalSales: { $sum: "$amount" },
    transactionCount: { $sum: 1 },
    // Conditional sum
    highValueSales: { 
      $sum: { 
        $cond: [{ $gte: ["$amount", 1000] }, "$amount", 0] 
      } 
    }
  }}
]);

// In $project to sum array values
db.orders.aggregate([
  { $project: {
    orderId: 1,
    itemCount: { $size: "$items" },
    totalAmount: {
      $sum: {
        $map: {
          input: "$items",
          in: { $multiply: ["$$this.price", "$$this.quantity"] }
        }
      }
    }
  }}
]);
```

---

## **Complete Aggregation Example**
**Notes:** This comprehensive example demonstrates how multiple operators work together in a real-world analytics pipeline. It processes orders data to generate monthly sales reports by category, including customer demographics and product details. The pipeline filters completed orders, joins with customer and product data, groups by time and category, calculates metrics, and formats the output. Each stage builds upon the previous one, transforming raw data into business insights.

**Code:**
```javascript
// Complete sales analytics pipeline
db.orders.aggregate([
  // STAGE 1: Filter relevant documents
  { $match: { 
    status: { $in: ["completed", "shipped"] },
    orderDate: { $gte: new Date("2024-01-01") }
  }},
  
  // STAGE 2: Join with customers collection
  { $lookup: {
    from: "customers",
    localField: "customerId",
    foreignField: "_id",
    as: "customer"
  }},
  { $unwind: "$customer" },
  
  // STAGE 3: Unwind order items for per-product analysis
  { $unwind: "$items" },
  
  // STAGE 4: Join with products collection
  { $lookup: {
    from: "products",
    localField: "items.productId",
    foreignField: "_id",
    as: "product"
  }},
  { $unwind: "$product" },
  
  // STAGE 5: Group by time period and product category
  { $group: {
    _id: {
      year: { $year: "$orderDate" },
      month: { $month: "$orderDate" },
      category: "$product.category"
    },
    // Sales metrics
    totalRevenue: { $sum: { $multiply: ["$items.price", "$items.quantity"] } },
    totalUnits: { $sum: "$items.quantity" },
    orderCount: { $sum: 1 },
    // Customer metrics
    uniqueCustomers: { $addToSet: "$customerId" },
    avgCustomerAge: { $avg: "$customer.age" },
    // Product metrics
    productsSold: { $addToSet: "$product.name" },
    avgProductRating: { $avg: "$product.rating" }
  }},
  
  // STAGE 6: Calculate derived fields
  { $project: {
    period: {
      $concat: [
        { $toString: "$_id.year" },
        "-",
        { $toString: "$_id.month" }
      ]
    },
    category: "$_id.category",
    metrics: {
      revenue: "$totalRevenue",
      units: "$totalUnits",
      orders: "$orderCount",
      avgOrderValue: { $divide: ["$totalRevenue", "$orderCount"] },
      customerCount: { $size: "$uniqueCustomers" },
      avgCustomerAge: { $round: ["$avgCustomerAge", 1] },
      productCount: { $size: "$productsSold" },
      avgProductRating: { $round: ["$avgProductRating", 2] }
    }
  }},
  
  // STAGE 7: Sort results
  { $sort: { "period": -1, "metrics.revenue": -1 } },
  
  // STAGE 8: Pagination or limit
  { $limit: 100 }
]);
```

---

## **Summary**
MongoDB's aggregation framework provides a comprehensive toolkit for data transformation and analysis, with operators like `$match`, `$group`, `$project`, and `$lookup` enabling complex data processing pipelines. Transactions ensure data integrity across multiple operations, while common operators handle everything from filtering and sorting to joining and computing aggregates. Mastering these concepts allows developers to perform sophisticated analytics, generate reports, and maintain data consistency in MongoDB applications, moving beyond simple CRUD operations to full-featured data processing capabilities.