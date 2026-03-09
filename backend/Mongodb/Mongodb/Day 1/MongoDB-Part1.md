## **MongoDB Notes & Explanations with Code Examples**

---

### **1. MongoDB**
**Notes:** MongoDB is a leading NoSQL database that uses a document-oriented data model instead of traditional tables. It stores data in flexible, JSON-like documents called BSON, which can have varying structures. This flexibility allows developers to evolve the data model without costly migrations. MongoDB excels at horizontal scaling through sharding and provides high availability via replica sets. Its query language supports rich queries, geospatial indexing, and real-time aggregation, making it suitable for modern applications that handle diverse, rapidly changing data.

**Code:**
```javascript
// Connecting to MongoDB
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/myapp', { useNewUrlParser: true });
```

---

### **2. BSON vs JSON**
**Notes:** BSON (Binary JSON) is MongoDB's binary storage format that extends JSON with additional types like Date, Binary, and ObjectId. While JSON is human-readable and ideal for APIs, BSON is optimized for storage efficiency and fast traversal. BSON supports embedded documents and arrays natively, and includes length prefixes for quick scanning. The binary format makes BSON slightly larger than JSON on disk but significantly faster to parse. MongoDB drivers automatically convert between BSON and language-native types.

**Code:**
```javascript
// JSON sent to API vs BSON stored in MongoDB
const json = '{"name": "Alice", "age": 30}';
const bsonDoc = { name: "Alice", age: 30, _id: new ObjectId(), created: new Date() };
```

---

### **3. SQL vs NoSQL**
**Notes:** SQL databases use predefined schemas with tables, rows, and columns, enforcing ACID properties through rigid relationships. NoSQL databases like MongoDB offer schema flexibility with documents that can store nested data. SQL excels at complex joins and transactions; NoSQL excels at horizontal scaling and handling unstructured data. SQL requires migrations for schema changes; MongoDB allows dynamic field addition. The choice depends on data structure predictability and scalability needs.

**Code:**
```sql
-- SQL: Structured schema
CREATE TABLE users (id INT PRIMARY KEY, name VARCHAR(100));
```
```javascript
// NoSQL: Flexible schema
db.users.insertOne({ name: "Alice", preferences: { theme: "dark" } });
```

---

### **4. Embedded Objects & Arrays**
**Notes:** Embedded documents store related data within a parent document, reducing the need for joins and improving read performance. This approach works well for one-to-many relationships where the child data has a lifecycle tied to the parent. Arrays can store lists of embedded documents or values. The embedded model supports atomic operations on the entire document but can lead to large documents if not managed carefully. This contrasts with normalized references where related data lives in separate collections.

**Code:**
```javascript
// Embedded orders in a customer document
{
  _id: 1,
  name: "John",
  orders: [
    { orderId: 101, total: 50.00 },
    { orderId: 102, total: 75.50 }
  ]
}
```

---

### **5. What is MongoDB?**
**Notes:** MongoDB is an open-source, cross-platform document database first released in 2009. It provides high performance through indexing, in-memory computation, and native sharding. MongoDB supports rich querying with operators for comparison, logical operations, and geospatial queries. Its aggregation framework enables complex data processing pipelines. MongoDB Atlas offers managed cloud deployment, while MongoDB Compass provides a GUI for data exploration. The database is widely adopted in e-commerce, IoT, mobile apps, and real-time analytics.

**Code:**
```javascript
// Basic MongoDB operation
const db = client.db('test');
const collection = db.collection('inventory');
await collection.insertOne({ item: "book", qty: 25 });
```

---

### **6. Double**
**Notes:** The Double type in MongoDB represents IEEE 754 64-bit floating-point numbers, suitable for scientific calculations and most fractional values. However, due to binary representation, some decimal numbers (like 0.1) cannot be represented exactly, leading to precision issues in financial calculations. Doubles are the default numeric type in MongoDB when no specific type is declared. For applications requiring exact decimal representation (like currency), Decimal128 is preferred.

**Code:**
```javascript
// Double precision issues
{ price: 0.1 + 0.2 } // Actually stores ≈0.30000000000000004
```

---

### **7. String**
**Notes:** Strings in MongoDB store UTF-8 encoded text with virtually no length limit (up to 16MB per document). They support indexing, including text indexes for full-text search and regular expression queries for pattern matching. String fields can be validated using schema validation rules. Collation settings allow language-specific sorting rules. Strings are the most common data type for storing names, addresses, descriptions, and other textual data.

**Code:**
```javascript
// String with text index for search
db.articles.createIndex({ content: "text" });
db.articles.insertOne({ title: "Introduction", content: "MongoDB is a NoSQL database." });
```

---

### **8. MongoDB Basics**
**Notes:** MongoDB fundamentals include databases (logical containers), collections (group of documents), and documents (individual records). Each document requires a unique `_id` field, auto-generated if not provided. Basic operations follow CRUD principles: Create (`insertOne/insertMany`), Read (`find`), Update (`updateOne/updateMany`), and Delete (`deleteOne/deleteMany`). Indexes improve query performance, and the aggregation framework enables complex data transformations. MongoDB uses JavaScript/JSON syntax for queries.

**Code:**
```javascript
// Basic CRUD operations
db.users.insertOne({ name: "Alice", age: 25 });
const user = db.users.findOne({ name: "Alice" });
db.users.updateOne({ name: "Alice" }, { $set: { age: 26 } });
db.users.deleteOne({ name: "Alice" });
```

---

### **9. When to use MongoDB?**
**Notes:** MongoDB is ideal for applications with evolving schemas, hierarchical data relationships, or requirements for high write throughput. Use cases include content management systems (flexible content structures), real-time analytics (fast aggregations), IoT (time-series data), mobile apps (offline synchronization), and catalogs (variable product attributes). Avoid MongoDB for applications requiring multi-document ACID transactions across many collections or complex relational queries with frequent joins.

**Code:**
```javascript
// Good for MongoDB: Flexible product catalog
{
  product: "Laptop",
  attributes: { 
    brand: "Dell", 
    specs: { ram: "16GB", storage: "512GB" },
    tags: ["electronics", "computers"]
  }
}
```

---

### **10. Array**
**Notes:** Arrays in MongoDB can store ordered lists of values, including mixed data types. They support powerful query operators like `$elemMatch` for matching array elements, `$size` for counting elements, and `$slice` for pagination. Arrays can be indexed (multikey indexes) for fast element queries. Update operators like `$push`, `$pull`, and `$addToSet` modify array contents. Arrays are ideal for tags, comments, order items, or any repeating data within a document.

**Code:**
```javascript
// Array operations
db.products.updateOne(
  { _id: 1 },
  { $push: { tags: "new" } } // Adds to array
);
db.products.find({ tags: { $in: ["sale", "featured"] } });
```

---

### **11. Object**
**Notes:** Objects (embedded documents) allow hierarchical data modeling within a single document. This reduces data fragmentation and improves read performance for related data. Objects can be nested multiple levels deep and support dot notation for querying (`"address.city"`). Schema validation can enforce structure on embedded objects. Objects are particularly useful for user profiles, configuration settings, or any data that naturally groups together and is accessed as a unit.

**Code:**
```javascript
// Nested object with dot notation query
{
  user: "Bob",
  address: {
    street: "123 Main",
    city: "Boston",
    coordinates: { lat: 42.36, lng: -71.06 }
  }
}
// Query: db.users.find({ "address.city": "Boston" })
```

---

### **12. What is MongoDB Atlas?**
**Notes:** MongoDB Atlas is a fully managed database-as-a-service (DBaaS) that automates deployment, scaling, backup, and monitoring of MongoDB clusters. It provides global distribution across cloud regions, automated tiered storage, and built-in security features like encryption and network isolation. Atlas offers serverless instances that scale automatically with workload, and supports real-time analytics through Atlas Data Lake. It simplifies operations while maintaining compatibility with MongoDB drivers and tools.

**Code:**
```javascript
// Atlas connection string format
mongodb+srv://username:password@cluster-name.mongodb.net/database?retryWrites=true&w=majority
```

---

### **13. Binary Data**
**Notes:** The Binary Data (BinData) type stores arbitrary binary information like images, PDFs, or encrypted data. MongoDB supports several binary subtypes for different purposes (generic, function, UUID, MD5). Binary data is base64-encoded when transmitted via JSON but stored efficiently in BSON. While MongoDB can store binary data, for large files GridFS (which chunks files across multiple documents) is recommended to avoid exceeding the 16MB document size limit.

**Code:**
```javascript
// Storing binary data
const binaryData = Buffer.from("binary content");
db.files.insertOne({ 
  filename: "image.png", 
  data: new BinData(0, binaryData.toString('base64')) 
});
```

---

### **14. Undefined**
**Notes:** The Undefined type exists in BSON specification but is deprecated and rarely used in modern MongoDB applications. JavaScript's `undefined` values are converted to `null` when stored. In practice, missing fields or `null` values should be used instead. Some MongoDB drivers may not support `undefined` serialization. The type remains in the specification for backward compatibility but has no practical use in new applications.

**Code:**
```javascript
// Avoid undefined
db.users.insertOne({ name: "Alice", middleName: undefined }); // Converts to null
db.users.insertOne({ name: "Bob" }); // middleName field doesn't exist - preferred
```

---

### **15. Data Model & Data Types**
**Notes:** MongoDB's document data model allows flexible schema design where each document can have different fields. BSON supports numerous data types including primitive types (String, Number, Boolean), complex types (Array, Object), special types (ObjectId, Date, Binary), and numeric types with different precisions (Int32, Int64, Decimal128). This flexibility enables modeling real-world entities with all their complexity while maintaining query capability through MongoDB's rich query language.

**Code:**
```javascript
// Document showing multiple BSON types
{
  _id: ObjectId(),           // ObjectId
  name: "Product",          // String
  price: NumberDecimal("99.99"), // Decimal128
  inStock: true,            // Boolean
  tags: ["new", "sale"],    // Array
  details: { weight: 1.5 }, // Object
  createdAt: new Date(),    // Date
  version: NumberLong(42)   // Int64
}
```

---

### **16. MongoDB Terminology**
**Notes:** Key MongoDB concepts: Database (container for collections), Collection (group of documents similar to a table), Document (basic unit like a row), Field (key-value pair like a column), `_id` (unique primary key), Index (optimizes queries), Cursor (result set pointer), Aggregation Pipeline (data processing stages), Shard (horizontal partition), Replica Set (high-availability cluster), Mongos (query router for sharded clusters), Oplog (operations log for replication).

**Code:**
```javascript
// Terminology in practice
const database = client.db("company"); // Database
const collection = database.collection("employees"); // Collection
const document = { _id: 1, name: "Alice" }; // Document
const field = document.name; // Field value "Alice"
```

---

### **17. Object ID**
**Notes:** ObjectId is a 12-byte identifier consisting of: 4-byte timestamp (seconds since Unix epoch), 5-byte random value (machine+process identifier), and 3-byte counter (starting with random value). This structure ensures uniqueness across distributed systems without coordination. ObjectIds are sortable by creation time and can be extracted using `getTimestamp()`. While auto-generated, custom `_id` values can be used. ObjectId is the default `_id` type unless specified otherwise.

**Code:**
```javascript
// Working with ObjectId
const id = new ObjectId();
console.log(id.getTimestamp()); // Returns creation date
console.log(id.toString()); // Returns hex string "507f1f77bcf86cd799439011"
db.users.find({ _id: ObjectId("507f1f77bcf86cd799439011") });
```

---

### **18. Boolean**
**Notes:** Boolean stores `true` or `false` values for binary states. In queries, boolean fields can be used with `$ne`, `$exists`, and logical operators. MongoDB also treats `0`, `null`, and empty strings as falsy in some contexts (like `$cond`), but stored booleans maintain strict true/false values. Booleans are commonly used for flags like `active`, `verified`, or `completed` status indicators. They support indexing and can be part of compound indexes.

**Code:**
```javascript
// Boolean usage
db.users.insertOne({ name: "Alice", active: true, admin: false });
db.users.find({ active: true, admin: { $ne: true } });
db.users.createIndex({ active: 1 }); // Index on boolean field
```

---

### **19. Date**
**Notes:** The Date type stores datetime as signed 64-bit integer representing milliseconds since Unix epoch (Jan 1, 1970). Dates support rich query operators like `$gt`, `$lt`, and date aggregation operators like `$year`, `$month`. Timezone information is not stored; applications must handle timezone conversion. ISODate() helper creates dates from ISO strings. Date objects in MongoDB are timezone-aware in application code but stored as UTC milliseconds.

**Code:**
```javascript
// Date operations
const today = new Date();
db.events.insertOne({ name: "Launch", date: today });
db.events.find({ date: { $gte: new Date("2024-01-01") } });
db.events.aggregate([{ $project: { year: { $year: "$date" } } }]);
```

---

### **20. Null**
**Notes:** Null represents the absence of a value in MongoDB. It's distinct from a missing field: `{x: null}` vs `{}`. Queries can distinguish using `{x: null}` (matches null) vs `{x: {$exists: true}}` (matches any value including null). Null values can be indexed and participate in sorts (typically treated as lowest values). Use null for explicitly marking fields as empty rather than omitting them, especially when field presence matters for application logic.

**Code:**
```javascript
// Null vs missing
db.items.insertMany([
  { _id: 1, value: null },    // Field exists with null
  { _id: 2 }                   // Field doesn't exist
]);
db.items.find({ value: null }); // Returns only doc 1
```

---

### **21. Regular Expression**
**Notes:** Regular expressions in MongoDB enable pattern matching in string queries. Regex queries can use flags like `i` (case-insensitive) and support PCRE syntax. While powerful, regex queries on large collections can be slow unless anchored (starting with `^`). MongoDB also supports text indexes for full-text search which are more efficient than regex for word-based searches. Regex can be stored as a BSON type but is typically used in query operators.

**Code:**
```javascript
// Regex queries
db.users.find({ name: /^Ali/i }); // Names starting with "Ali" case-insensitive
db.users.find({ name: { $regex: "son$", $options: "i" } }); // Ends with "son"
db.products.find({ sku: { $regex: /^ABC-\d{3}$/ } }); // Pattern matching
```

---

### **22. JavaScript**
**Notes:** MongoDB supports JavaScript code execution through the `$where` operator and stored functions. `$where` allows arbitrary JavaScript in queries but is inefficient as it cannot use indexes and executes for each document. Stored JavaScript can be saved in `system.js` collection for reuse. Due to security and performance concerns, JavaScript execution is often disabled in production. Modern MongoDB favors aggregation pipeline over JavaScript for data processing.

**Code:**
```javascript
// JavaScript in queries (avoid when possible)
db.users.find({ 
  $where: function() { 
    return this.age > 18 && this.name.startsWith("A");
  }
});
```

---

### **23. Symbol**
**Notes:** Symbol is a legacy BSON type from early MongoDB versions that stored unique symbols similar to JavaScript's Symbol type. It's deprecated and not supported by most modern drivers. Symbols were intended for language-specific symbol tables but offered little advantage over strings. Current MongoDB applications should use String type instead. If encountered in legacy data, drivers typically convert symbols to strings automatically.

**Code:**
```javascript
// Symbol usage (historical)
{ currency: Symbol("USD") } // Deprecated - use string instead
{ currency: "USD" } // Modern approach
```

---

### **24. Int64 / Long**
**Notes:** Int64 (Long) stores 64-bit signed integers ranging from -9.22e18 to 9.22e18. Necessary for counts, IDs, or timestamps exceeding 32-bit range (~2.14 billion). MongoDB drivers provide special Long types (like `NumberLong()` in shell, `Long` in Node.js). Arithmetic on Longs may require special methods to avoid precision loss. Int64 is used internally for MongoDB's timestamp type and large counters.

**Code:**
```javascript
// Int64 operations
db.counters.insertOne({ _id: "pageViews", count: NumberLong("10000000000") });
db.counters.updateOne(
  { _id: "pageViews" },
  { $inc: { count: NumberLong(1) } }
);
```

---

### **25. Int32/Int**
**Notes:** Int32 stores 32-bit signed integers ranging from -2,147,483,648 to 2,147,483,647. Default integer type in many programming languages and MongoDB shell (when no suffix). Use `NumberInt()` constructor to explicitly store as Int32 instead of Double. Int32 is more space-efficient than Double for whole numbers and ensures integer semantics. Most counters, ages, quantities, and IDs fit within Int32 range.

**Code:**
```javascript
// Explicit Int32
db.products.insertOne({ 
  sku: "ABC123", 
  stock: NumberInt(1000), // Stored as 32-bit integer
  price: 29.99 // Stored as Double by default
});
```

---

### **26. Timestamp**
**Notes:** MongoDB Timestamp is a special 64-bit type consisting of two 32-bit parts: seconds since epoch and an incrementing ordinal. Used internally for replication (oplog) and sharding. Not meant for general date storage (use Date type instead). Timestamps are unique and monotonically increasing within a mongod process. Applications rarely use Timestamp directly unless implementing custom replication or versioning systems.

**Code:**
```javascript
// Timestamp internal usage
// In oplog: { ts: Timestamp(1625097600, 1), op: "i", ... }
// Application code should generally use Date type
const regularDate = new Date(); // For application dates
```

---

### **27. Decimal128**
**Notes:** Decimal128 is a 128-bit decimal floating-point type that exactly represents decimal numbers with up to 34 digits of precision. Essential for financial applications where rounding errors from Double are unacceptable. Decimal128 follows IEEE 754-2008 standard and supports proper decimal rounding. Use `NumberDecimal()` constructor. Slower than Double for calculations but provides exact decimal representation for currencies, percentages, and scientific measurements.

**Code:**
```javascript
// Decimal128 for exact arithmetic
db.transactions.insertOne({
  amount: NumberDecimal("100.50"),
  tax: NumberDecimal("8.525"),
  total: NumberDecimal("109.025") // Exact, not 109.02499999999999
});
```

---

### **28. Min Key**
**Notes:** MinKey is a special BSON type that compares as less than all other BSON types in sort order. Used internally for query boundaries and sharding. In sharded clusters, MinKey represents the lower bound of a shard key range. Rarely used in application code except for advanced queries requiring comparison with all possible values. MinKey and MaxKey together define open-ended ranges in shard key distributions.

**Code:**
```javascript
// MinKey in sharding context
// Shard key range: { _id: { $minKey: 1 } } to { _id: { $maxKey: 1 } }
// Application query example (rare):
db.data.find({ value: { $gt: MinKey() } }); // All documents
```

---

### **29. Max Key**
**Notes:** MaxKey compares as greater than all other BSON types, serving as the upper bound complement to MinKey. Primarily used in sharding to define the upper bound of shard key ranges. Like MinKey, MaxKey appears in system-generated queries but rarely in application logic. Useful when implementing custom comparison logic or querying "all values up to some maximum." Both MinKey and MaxKey are singleton types with no additional data.

**Code:**
```javascript
// MaxKey usage parallels MinKey
// In shard configuration: 
// sh.splitAt("database.collection", { _id: MaxKey() })
// Query example:
db.logs.find({ timestamp: { $lte: MaxKey() } }); // All documents
```

---

**Summary:** MongoDB's flexible document model with rich BSON data types enables sophisticated data modeling while maintaining performance at scale. Each data type serves specific use cases, from exact decimals (Decimal128) for finance to binary data for file storage. Understanding these types and when to use embedded vs referenced data is key to effective MongoDB schema design.