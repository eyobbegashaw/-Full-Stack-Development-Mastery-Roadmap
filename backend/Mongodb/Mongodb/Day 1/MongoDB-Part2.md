## **MongoDB Collections & Methods - Notes & Explanations**

---

### **1. Counting Documents**
**Notes:** Counting documents in MongoDB can be done using `countDocuments()` for accurate counts or `estimatedDocumentCount()` for faster approximate counts. The `countDocuments()` method accepts a filter and returns the exact number of matching documents, while `estimatedDocumentCount()` returns an approximation based on collection metadata, which is faster for large collections. Both methods support read concern and can be used in transactions. The legacy `count()` method is deprecated due to inaccurate sharded cluster counts and collation inconsistencies.

**Code:**
```javascript
// Counting documents
const exactCount = await db.collection.countDocuments({ status: "active" });
const approxCount = await db.collection.estimatedDocumentCount();
console.log(`Active users: ${exactCount}, Total approx: ${approxCount}`);
```

---

### **2. insert() and relevant**
**Notes:** MongoDB provides several insert methods: `insertOne()` for single documents, `insertMany()` for multiple documents, and the legacy `insert()` method (deprecated in favor of the newer methods). These methods support write concerns for durability guarantees and bypass document validation when needed. `insertMany()` can be ordered (stops on first error) or unordered (continues despite errors). All insert methods automatically generate an `_id` if not provided and return acknowledgment with inserted IDs.

**Code:**
```javascript
// Insert operations
const resultOne = await db.users.insertOne({ name: "Alice", age: 25 });
const resultMany = await db.users.insertMany([
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 35 }
], { ordered: false }); // Continues on error
```

---

### **3. find() and relevant**
**Notes:** The `find()` method queries documents with optional filters, projections, and options. It returns a cursor that can be iterated, converted to an array with `toArray()`, or limited with cursor methods like `limit()`, `skip()`, and `sort()`. Related methods include `findOne()` for single document retrieval and `findOneAndUpdate()` for atomic find-and-modify operations. `find()` supports complex queries with operators (`$eq`, `$gt`, `$in`), regex patterns, and geospatial queries. Projections control which fields are returned.

**Code:**
```javascript
// Find operations with filtering and projection
const cursor = db.users.find(
  { age: { $gte: 18 } },           // Filter: adults only
  { projection: { name: 1, _id: 0 } } // Return only name field
).sort({ name: 1 }).limit(10);     // Sort and limit

const users = await cursor.toArray();
```

---

### **4. Collections & Methods**
**Notes:** Collections are groupings of MongoDB documents, analogous to tables in relational databases. Each collection has methods for CRUD operations, aggregation, indexing, and management. Key methods include `createIndex()`, `drop()`, `renameCollection()`, `stats()`, and `aggregate()`. Collections can be created implicitly on first insert or explicitly via `createCollection()` with options like capped size, validation rules, or storage engine settings. Collection methods support read/write concerns for consistency guarantees.

**Code:**
```javascript
// Collection management
await db.createCollection("logs", { 
  capped: true, 
  size: 1000000, // 1MB capped collection
  max: 1000      // Max 1000 documents
});

const stats = await db.logs.stats();
console.log(`Collection size: ${stats.size} bytes`);
```

---

### **5. update() and relevant**
**Notes:** Update operations modify existing documents using `updateOne()`, `updateMany()`, or `replaceOne()`. Update operators like `$set`, `$inc`, `$push`, and `$pull` allow granular field modifications. The legacy `update()` method is deprecated. Updates can include options like `upsert` (create if not exists) and array filters for updating nested array elements. `findOneAndUpdate()` returns the document before or after modification. All update methods respect write concerns and can use aggregation pipelines for complex updates in MongoDB 4.2+.

**Code:**
```javascript
// Update operations
await db.users.updateOne(
  { name: "Alice" },
  { $set: { age: 26, lastUpdated: new Date() } },
  { upsert: true } // Create if doesn't exist
);

await db.users.updateMany(
  { status: "pending" },
  { $set: { status: "approved" } }
);
```

---

### **6. delete() and relevant**
**Notes:** Delete operations remove documents using `deleteOne()` and `deleteMany()`, with the legacy `delete()` method deprecated. These methods accept filters to match documents for deletion and return deletion counts. `findOneAndDelete()` atomically finds and removes a document, returning the deleted document. Deletions respect write concerns and can be used in transactions. For removing all documents while keeping the collection, use `deleteMany({})`; to remove the entire collection, use `drop()`. Deleted documents are not recoverable through MongoDB (requires backups).

**Code:**
```javascript
// Delete operations
const deleteResult = await db.users.deleteOne({ name: "Bob" });
console.log(`Deleted ${deleteResult.deletedCount} document(s)`);

// Delete all inactive users older than 1 year
await db.users.deleteMany({
  status: "inactive",
  lastLogin: { $lt: new Date(Date.now() - 365*24*60*60*1000) }
});
```

---

### **7. bulkWrite() and relevant**
**Notes:** `bulkWrite()` executes multiple write operations in a single command for better performance. It supports insert, update, replace, and delete operations in ordered (sequential, stops on error) or unordered (parallel, continues despite errors) batches. Each operation in the array specifies its type and document/filter. Bulk writes reduce network round-trips and are atomic at the document level (not transactionally atomic across documents unless in a transaction). Useful for data migrations, batch updates, and ETL processes.

**Code:**
```javascript
// Bulk write operations
const bulkResult = await db.orders.bulkWrite([
  { insertOne: { document: { orderId: 101, total: 50 } } },
  { updateOne: { 
    filter: { orderId: 102 },
    update: { $set: { status: "shipped" } }
  }},
  { deleteOne: { filter: { orderId: 103 } } }
], { ordered: true });

console.log(`Processed ${bulkResult.insertedCount + bulkResult.modifiedCount} operations`);
```

---

### **8. validate()**
**Notes:** The `validate()` method checks a collection's storage structure for corruption and returns validation statistics. It scans data and indexes for inconsistencies, reporting errors in the output. This is an administrative operation that can be resource-intensive and require a database lock in older MongoDB versions (less intrusive in 4.4+). Options include `full` (more thorough) and `repair` (fixes issues where possible). Typically used for maintenance, troubleshooting, or after unexpected crashes, but not for routine data validation (use schema validation instead).

**Code:**
```javascript
// Collection validation
const validation = await db.runCommand({
  validate: "users",
  full: true,      // Thorough validation
  repair: false    // Don't automatically repair
});

console.log(`Valid: ${validation.valid}`);
if (!validation.valid) {
  console.log(`Errors: ${validation.errors}`);
}
```

---

### **9. Read / Write Concerns**
**Notes:** Read concern specifies the consistency guarantee for reads: `local` (default, fastest), `available` (for sharded clusters), `majority` (reads confirmed by majority replicas), `linearizable` (strongest, for single document). Write concern specifies acknowledgment requirements: `w` (number of replicas), `j` (journal durability), `wtimeout` (timeout period). Together with read preference (where to read from), they control MongoDB's consistency, availability, and performance tradeoffs. These are crucial for distributed systems to ensure data durability and consistency.

**Code:**
```javascript
// Using read and write concerns
await db.users.insertOne(
  { name: "Alice", balance: 100 },
  { 
    writeConcern: { w: "majority", j: true } // Majority replicas, journaled
  }
);

const user = await db.users.findOne(
  { name: "Alice" },
  { 
    readConcern: { level: "majority" } // Read majority-committed data
  }
);
```

---

### **10. Useful concepts**
**Notes:** Key MongoDB concepts beyond basic CRUD include: Transactions (multi-document ACID operations), Change Streams (real-time data change notifications), Aggregation Pipeline (powerful data processing), Schema Validation (enforcing document structure), Text Search (full-text indexing), Geospatial Queries (location-based queries), TTL Indexes (automatic document expiration), Partial Indexes (index subset of documents), Collation (language-specific string comparison), and Views (aggregated collection perspectives). These enable sophisticated data management patterns.

**Code:**
```javascript
// Example of useful concepts
// TTL index for automatic expiration
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Schema validation
db.createCollection("products", {
  validator: {
    $jsonSchema: {
      required: ["name", "price"],
      properties: {
        name: { bsonType: "string" },
        price: { bsonType: "decimal", minimum: 0 }
      }
    }
  }
});
```

---

### **11. Cursors**
**Notes:** Cursors are pointers to result sets returned by `find()` operations, enabling efficient batch processing of large datasets. Instead of loading all documents into memory, cursors fetch documents in batches (default 101 documents). Cursor methods include `next()` (get next document), `forEach()` (iterate), `toArray()` (convert to array), `limit()`, `skip()`, `sort()`, `project()`, and `count()`. Cursors timeout after 10 minutes of inactivity (configurable with `noCursorTimeout`). They can be transformed into streams for reactive programming patterns.

**Code:**
```javascript
// Working with cursors
const cursor = db.largeCollection.find({ type: "event" })
  .batchSize(50)          // Fetch 50 documents at a time
  .noCursorTimeout(true); // Prevent automatic timeout

while (await cursor.hasNext()) {
  const doc = await cursor.next();
  processDocument(doc); // Process one at a time
}
await cursor.close(); // Always close cursor to free resources
```

---

### **12. Retryable Reads / Writes**
**Notes:** Retryable reads and writes automatically retry operations on transient network errors or replica set failovers. Enabled by default in MongoDB drivers 3.6+ for supported operations. Retryable writes ensure at-most-once semantics using transaction numbers. Not all operations are retryable (e.g., multi-document transactions require manual retry logic). Retry attempts follow exponential backoff. This feature improves application resilience in distributed environments without requiring complex error handling code. Can be configured with `retryReads` and `retryWrites` connection options.

**Code:**
```javascript
// Configuring retryable operations
const client = new MongoClient(uri, {
  retryWrites: true,  // Auto-retry write operations
  retryReads: true,   // Auto-retry read operations
  maxPoolSize: 10,
  minPoolSize: 2
});

// The driver will automatically retry this if it fails temporarily
await db.orders.insertOne({
  orderId: 123,
  items: ["item1", "item2"],
  total: 99.99
});
```

---

## **Summary**
MongoDB's collection methods provide comprehensive data manipulation capabilities from basic CRUD to advanced bulk operations. Understanding cursors enables efficient large dataset processing, while read/write concerns control consistency in distributed environments. Retryable operations improve resilience, and validation helps maintain data integrity. These concepts form the foundation for building robust, scalable applications with MongoDB.