# **MongoDB Indexing, Performance & Ecosystem - Comprehensive Guide**

## **Indexing Fundamentals**

### **1. Creating Indexes**

**Notes:** Indexes in MongoDB are data structures that improve query performance by allowing efficient data retrieval without scanning every document. They're similar to book indexes, enabling quick lookup of specific information. Indexes consume disk space and memory, and affect write performance (each insert/update must update indexes). MongoDB supports various index types optimized for different query patterns. Indexes are created using `createIndex()` or `createIndexes()` methods and can be built in foreground (blocks operations) or background (non-blocking but slower).

**Code:**
```javascript
// Basic index creation
db.collection.createIndex({ field: 1 }); // Ascending order
db.collection.createIndex({ field: -1 }); // Descending order

// With options
db.users.createIndex(
  { email: 1 },
  {
    unique: true,           // Enforce uniqueness
    name: "email_index",    // Custom index name
    background: true,       // Build in background
    sparse: true            // Only index documents with field
  }
);

// Compound index
db.orders.createIndex(
  { customerId: 1, orderDate: -1 },
  { name: "customer_orders_desc" }
);
```

---

### **2. Single Field Indexes**

**Notes:** Single field indexes index one field in ascending (1) or descending (-1) order. They optimize queries filtering, sorting, or grouping by that field. Direction matters mostly for sort operations. MongoDB can traverse indexes in both directions efficiently, but compound indexes have directional significance for prefix patterns. Single field indexes support equality matches, range queries, and sorts on the indexed field.

**Code:**
```javascript
// Single field index examples
db.products.createIndex({ price: 1 });
db.users.createIndex({ createdAt: -1 });
db.logs.createIndex({ level: 1 });

// Queries using single field indexes
db.products.find({ price: { $gte: 50, $lte: 100 } }); // Range query
db.users.find().sort({ createdAt: -1 }).limit(10);    // Sort optimization
db.logs.find({ level: "ERROR" });                     // Equality match

// Check index usage with explain()
db.products.find({ price: 75 }).explain("executionStats");
```

---

### **3. Compound Indexes**

**Notes:** Compound indexes index multiple fields together, following field order (prefix pattern). They support queries on prefix subsets (first N fields) but not on non-prefix subsets. Field order is crucial: equality fields first, then range/sort fields (ESR rule: Equality, Sort, Range). Compound indexes can support both query predicates and sort operations. They're more efficient than multiple single-field indexes for complex queries.

**Code:**
```javascript
// Compound index following ESR rule
db.sales.createIndex(
  { region: 1, status: 1, date: -1, amount: 1 }, // Equality, Sort, Range
  { name: "sales_composite" }
);

// Supported queries (using prefix):
db.sales.find({ region: "west", status: "completed" }); // Uses index (prefix)
db.sales.find({ region: "west" }).sort({ date: -1 });   // Uses index (prefix)
db.sales.find({ region: "west", date: { $gte: ISODate("2024-01-01") } }); // Partial

// NOT supported (non-prefix):
db.sales.find({ status: "completed" }); // Can't use index (non-prefix)
db.sales.find({ date: { $gte: ISODate("2024-01-01") } }); // Can't use index

// Index intersection alternative
db.sales.createIndex({ status: 1 });
db.sales.createIndex({ date: -1 });
// MongoDB can intersect multiple indexes
```

---

### **4. Text Indexes**

**Notes:** Text indexes support text search queries on string content. They tokenize and stem text, removing stop words (like "the", "and"). Each collection can have only one text index (but can include multiple fields). Text indexes support language-specific stemming and weights for field importance. Queries use `$text` operator with `$search`. Text search is case-insensitive and diacritic-insensitive by default.

**Code:**
```javascript
// Create text index
db.articles.createIndex(
  { 
    title: "text", 
    content: "text",
    abstract: "text"
  },
  {
    name: "article_text_search",
    weights: {           // Field importance
      title: 10,
      abstract: 5,
      content: 1
    },
    default_language: "english",  // Language for stemming
    language_override: "language" // Field containing doc language
  }
);

// Text search queries
db.articles.find({
  $text: {
    $search: "mongodb database performance",  // Searches all terms
    $caseSensitive: false,
    $diacriticSensitive: false
  }
});

// Phrase search and exclusion
db.articles.find({
  $text: {
    $search: '"mongodb tutorial" -beginner', // Phrase and exclude
    $language: "english"
  }
});

// Text score for relevance sorting
db.articles.find(
  { $text: { $search: "database indexing" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });
```

---

### **5. Expiring (TTL) Indexes**

**Notes:** TTL indexes automatically remove documents after a specified time period. They're useful for session data, logs, temporary data. Indexed field must be a date or array of dates. MongoDB background thread removes expired documents every 60 seconds. TTL indexes are single-field indexes with `expireAfterSeconds` option. Can be combined with partial indexes for conditional expiration.

**Code:**
```javascript
// Basic TTL index - expires 24 hours after createdAt
db.sessions.createIndex(
  { createdAt: 1 },
  { 
    expireAfterSeconds: 24 * 60 * 60, // 24 hours
    name: "session_ttl"
  }
);

// Conditional TTL with partial index
db.notifications.createIndex(
  { expiresAt: 1 },
  {
    expireAfterSeconds: 0, // Uses expiresAt field value directly
    partialFilterExpression: { status: "temporary" },
    name: "temp_notifications_ttl"
  }
);

// For data that expires at specific time (not duration)
db.events.createIndex(
  { expireAt: 1 },
  { expireAfterSeconds: 0 } // Document expires at expireAt value
);

// Insert document with specific expiration time
db.events.insertOne({
  event: "promo",
  data: { discount: 20 },
  expireAt: new Date("2024-12-31T23:59:59Z") // Expires at year end
});
```

---

### **6. Geospatial Indexes**

**Notes:** Geospatial indexes support location-based queries. Two types: **2dsphere** for spherical geometry (Earth-like), **2d** for flat geometry. 2dsphere supports GeoJSON points, lines, polygons. Queries include `$near`, `$geoWithin`, `$geoIntersects`. Indexes enable efficient distance calculations and spatial relationships. Can be combined with other indexes for compound queries.

**Code:**
```javascript
// 2dsphere index for Earth-like geometry
db.places.createIndex(
  { location: "2dsphere" },
  { name: "location_geo" }
);

// Insert GeoJSON point
db.places.insertOne({
  name: "Central Park",
  location: {
    type: "Point",
    coordinates: [-73.9654, 40.7829] // [longitude, latitude]
  },
  category: "park"
});

// Geo queries
// Find places within 1000 meters of a point
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.98, 40.75]
      },
      $maxDistance: 1000 // meters
    }
  }
});

// Find places within polygon (neighborhood boundary)
db.places.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [-74.0, 40.7], [-73.9, 40.7],
          [-73.9, 40.8], [-74.0, 40.8],
          [-74.0, 40.7]
        ]]
      }
    }
  }
});

// Compound geo index
db.places.createIndex(
  { category: 1, location: "2dsphere" },
  { name: "category_location" }
);
```

---

## **Performance Optimization**

### **1. Query Optimization**

**Notes:** Query optimization involves analyzing and improving query performance using `explain()`, indexes, and query patterns. Key strategies: use covered queries (index-only), avoid `$where` and JavaScript, limit returned fields, use pagination, avoid large `$in` arrays, and use aggregation pipelines for complex operations. Monitor slow queries with profiling and database commands.

**Code:**
```javascript
// Analyze query performance
const explain = db.orders.find({
  customerId: "cust123",
  status: "completed",
  total: { $gt: 100 }
}).explain("executionStats");

console.log("Execution stats:", {
  executionTimeMillis: explain.executionStats.executionTimeMillis,
  totalDocsExamined: explain.executionStats.totalDocsExamined,
  totalKeysExamined: explain.executionStats.totalKeysExamined,
  stage: explain.executionStats.executionStages.stage
});

// Covered query (index only)
db.orders.createIndex({ customerId: 1, status: 1, total: 1 });
db.orders.find(
  { customerId: "cust123", status: "completed" },
  { _id: 0, customerId: 1, status: 1, total: 1 } // Project only indexed fields
);

// Efficient pagination (avoid large skips)
const lastId = ObjectId("...");
db.orders.find({
  _id: { $gt: lastId },
  status: "completed"
}).sort({ _id: 1 }).limit(20);

// Use aggregation for complex operations
db.orders.aggregate([
  { $match: { status: "completed", date: { $gte: startDate } } },
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 100 }
]);
```

### **2. Index Optimization Strategies**

**Code:**
```javascript
// Partial indexes for selective data
db.users.createIndex(
  { email: 1 },
  { 
    partialFilterExpression: { email: { $exists: true } },
    name: "active_email_index"
  }
);

// Sparse indexes for missing fields
db.products.createIndex(
  { sku: 1 },
  { sparse: true } // Only index documents with sku field
);

// Hidden indexes for testing
db.collection.createIndex({ field: 1 }, { name: "test_index", hidden: true });
// Query planner ignores hidden indexes
// Unhide: db.collection.unhideIndex("test_index");

// Collation for language-specific sorting
db.users.createIndex(
  { name: 1 },
  { 
    collation: { 
      locale: "en", 
      strength: 2 // Case and diacritic insensitive
    }
  }
);

// Use index hinting when needed
db.orders.find({ status: "pending" })
  .hint("status_date_index") // Force specific index
  .explain();
```

---

## **Atlas Search Indexes**

**Notes:** MongoDB Atlas Search provides full-text search capabilities built on Apache Lucene. Offers more features than text indexes: faceted search, autocomplete, fuzzy matching, synonyms, custom analyzers. Integrated with MongoDB Atlas, managed service. Search indexes are defined separately from database indexes using Atlas UI or API.

**Code:**
```javascript
// Example Atlas Search index definition (JSON)
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": {
        "type": "string",
        "analyzer": "lucene.english"
      },
      "description": {
        "type": "string",
        "analyzer": "lucene.english"
      },
      "category": {
        "type": "stringFacet" // For faceted search
      },
      "price": {
        "type": "number"
      },
      "tags": {
        "type": "string",
        "multi": true
      }
    }
  }
}

// Atlas Search queries using $search
db.products.aggregate([
  {
    $search: {
      "index": "default", // Search index name
      "compound": {
        "must": [
          {
            "text": {
              "query": "wireless headphones",
              "path": ["title", "description"],
              "fuzzy": { "maxEdits": 1 } // Allow 1 character difference
            }
          }
        ],
        "filter": [
          {
            "range": {
              "path": "price",
              "gte": 50,
              "lte": 200
            }
          }
        ],
        "should": [ // Boost relevance
          {
            "text": {
              "query": "noise cancelling",
              "path": "description",
              "score": { "boost": { "value": 2 } }
            }
          }
        ]
      }
    }
  },
  {
    $project: {
      title: 1,
      description: 1,
      price: 1,
      score: { $meta: "searchScore" }
    }
  }
]);
```

---

## **Language Drivers & Ecosystem**

### **1. Language Drivers**

**Notes:** MongoDB provides official drivers for all major programming languages: Node.js, Python, Java, C#, Go, Ruby, etc. Drivers handle connection pooling, BSON serialization, retry logic, and API consistency. Each driver follows similar patterns but with language-specific idioms. Community drivers also exist but lack official support.

**Code Examples:**

**Node.js:**
```javascript
const { MongoClient } = require('mongodb');

async function main() {
  const client = new MongoClient(uri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
    maxPoolSize: 10,
    minPoolSize: 2,
    retryWrites: true,
    retryReads: true
  });
  
  await client.connect();
  const db = client.db('mydb');
  const collection = db.collection('users');
  
  // Operations...
  await client.close();
}
```

**Python:**
```python
from pymongo import MongoClient
from pymongo.server_api import ServerApi

client = MongoClient(
    "mongodb+srv://username:password@cluster.mongodb.net/",
    server_api=ServerApi('1'),
    maxPoolSize=10,
    minPoolSize=2
)

db = client.mydb
collection = db.users
```

**Java:**
```java
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoDatabase;

MongoClient client = MongoClients.create(
    "mongodb+srv://username:password@cluster.mongodb.net/"
);

MongoDatabase database = client.getDatabase("mydb");
MongoCollection<Document> collection = database.getCollection("users");
```

---

### **2. MongoDB Connectors**

**Notes:** Connectors integrate MongoDB with other data systems: BI tools (Tableau, Power BI), ETL platforms, analytics systems. Include ODBC, JDBC, Kafka, Spark connectors. Allow SQL queries on MongoDB via BI tools. Connectors translate between MongoDB query language and target system's protocol.

**Code:**
```sql
-- Example SQL via MongoDB BI Connector
SELECT customer_id, SUM(amount) as total_spent
FROM orders
WHERE status = 'completed'
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 10;
```

---

### **3. Kafka Connector**

**Notes:** MongoDB Kafka Connector streams data between Kafka and MongoDB in real-time. Supports source (MongoDB → Kafka) and sink (Kafka → MongoDB) connectors. Configurable for change streams, bulk operations, error handling. Used for data pipelines, event sourcing, CDC (Change Data Capture).

**Code:**
```properties
# Kafka Connect configuration for MongoDB Sink
name=mongodb-sink
connector.class=com.mongodb.kafka.connect.MongoSinkConnector
topics=orders-topic
connection.uri=mongodb://localhost:27017
database=analytics
collection=orders
key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
```

---

### **4. Spark Connector**

**Notes:** MongoDB Spark Connector enables reading/writing MongoDB data in Spark applications. Supports DataFrames, Datasets, RDDs. Optimized for distributed processing with predicate pushdown, schema inference. Used for large-scale analytics, ML pipelines, data transformations.

**Code:**
```scala
// Scala Spark with MongoDB
import com.mongodb.spark._

val spark = SparkSession.builder()
  .master("local")
  .appName("MongoSparkConnector")
  .config("spark.mongodb.input.uri", "mongodb://127.0.0.1/test.coll")
  .config("spark.mongodb.output.uri", "mongodb://127.0.0.1/test.coll")
  .getOrCreate()

val df = spark.read.format("mongo").load()
df.filter($"age" > 25).show()

// Write back to MongoDB
val result = df.groupBy("department").avg("salary")
result.write.format("mongo").mode("overwrite").save()
```

---

### **5. Elastic Search Connector**

**Notes:** MongoDB Connector for Elasticsearch synchronizes data between MongoDB and Elasticsearch in real-time. Uses MongoDB change streams to capture changes. Elasticsearch provides advanced search capabilities while MongoDB handles transactional operations. Common pattern: write to MongoDB, sync to Elasticsearch for search.

**Code:**
```yaml
# Elasticsearch connector configuration
mongodb:
  connection:
    uri: "mongodb://localhost:27017"
    database: "mydb"
    collection: "products"
    
elasticsearch:
  connection:
    hosts: ["http://localhost:9200"]
    index: "products-index"
    
pipeline:
  - name: "products"
    collection: "products"
    index: "products-index"
    namespace_regex: "mydb.products"
    parse_queue: "products-queue"
```

---

## **Backup & Recovery**

### **1. mongodump & mongorestore**

**Notes:** `mongodump` creates binary exports of MongoDB data. `mongorestore` imports these backups. Support full database, collection-level, or query-based backups. Options for compression, parallelism, oplog inclusion. For large databases, use filesystem snapshots or MongoDB Atlas backups. Best practice: regular backups with retention policy.

**Code:**
```bash
# Basic backup of entire database
mongodump --uri="mongodb://localhost:27017" --out=/backup/2024-01-15

# Backup specific collection with query
mongodump \
  --uri="mongodb://localhost:27017/mydb" \
  --collection=orders \
  --query='{"status": "completed", "date": {"$gte": {"$date": "2024-01-01T00:00:00Z"}}}' \
  --out=/backup/orders_export

# Backup with compression
mongodump --uri="mongodb://localhost:27017" --gzip --archive=/backup/db.archive

# Parallel backup for speed
mongodump --uri="mongodb://localhost:27017" --numParallelCollections=4 --out=/backup/

# Restore database
mongorestore --uri="mongodb://localhost:27017" /backup/2024-01-15/

# Restore specific collection
mongorestore \
  --uri="mongodb://localhost:27017/mydb" \
  --collection=orders \
  /backup/2024-01-15/mydb/orders.bson

# Restore with drop (existing data)
mongorestore --uri="mongodb://localhost:27017" --drop /backup/2024-01-15/

# Point-in-time recovery with oplog
mongodump --uri="mongodb://localhost:27017" --oplog --out=/backup/
mongorestore --uri="mongodb://localhost:27017" --oplogReplay /backup/
```

### **2. Advanced Backup Strategies**

**Code:**
```bash
# Script for automated backup with rotation
#!/bin/bash
BACKUP_DIR="/backups/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Create backup
mongodump --uri="$MONGODB_URI" \
  --gzip \
  --archive="$BACKUP_DIR/backup_$DATE.archive"

# Clean old backups
find "$BACKUP_DIR" -name "backup_*.archive" -mtime +$RETENTION_DAYS -delete

# Verify backup integrity
mongorestore --uri="$MONGODB_URI" \
  --archive="$BACKUP_DIR/backup_$DATE.archive" \
  --dryRun \
  --objcheck

# Backup only specific databases (exclude system)
mongodump --uri="$MONGODB_URI" \
  --db=mydb \
  --db=analytics \
  --excludeCollection=logs \
  --out="$BACKUP_DIR/"

# Encrypted backups
mongodump --uri="$MONGODB_URI" \
  --gzip \
  --archive | openssl enc -aes-256-cbc -salt \
  -out "$BACKUP_DIR/encrypted_backup.enc" \
  -pass pass:"$ENCRYPTION_KEY"
```

---

## **Performance Monitoring & Maintenance**

**Code:**
```javascript
// Database statistics
const stats = db.stats();
console.log("Database stats:", {
  sizeMB: (stats.dataSize / 1024 / 1024).toFixed(2),
  storageMB: (stats.storageSize / 1024 / 1024).toFixed(2),
  indexMB: (stats.indexSize / 1024 / 1024).toFixed(2),
  collections: stats.collections,
  objects: stats.objects
});

// Index statistics
db.collection.aggregate([
  { $indexStats: {} }
]);

// Identify unused indexes
const indexStats = db.collection.aggregate([{ $indexStats: {} }]).toArray();
const unusedIndexes = indexStats.filter(idx => idx.accesses.ops === 0);
unusedIndexes.forEach(idx => {
  console.log(`Unused index: ${idx.name}`);
  // Consider: db.collection.dropIndex(idx.name);
});

// Collection compaction (for MMAPv1)
db.runCommand({ compact: "collectionName" });

// Reindex collection
db.collection.reIndex();

// View current operations
db.currentOp({
  "active": true,
  "secs_running": { "$gt": 5 } // Operations running >5 seconds
});

// Kill long-running operation
db.killOp(opid);
```

---

## **Complete Performance Optimization Example**

**Code:**
```javascript
// Comprehensive performance optimization script
async function optimizeDatabase() {
  console.log("Starting database optimization...");
  
  // 1. Analyze current indexes
  const collections = await db.listCollections().toArray();
  
  for (const collInfo of collections) {
    const collName = collInfo.name;
    const collection = db[collName];
    
    console.log(`\nAnalyzing collection: ${collName}`);
    
    // Get index info
    const indexes = await collection.getIndexes();
    console.log(`Has ${indexes.length} indexes`);
    
    // Identify missing indexes for common queries
    const slowQueries = await analyzeSlowQueries(collName);
    
    // Create recommended indexes
    for (const query of slowQueries) {
      const recommendedIndex = buildOptimalIndex(query);
      await createIndexIfNeeded(collection, recommendedIndex);
    }
    
    // Identify and drop unused indexes (except _id)
    const indexStats = await collection.aggregate([{ $indexStats: {} }]).toArray();
    for (const idx of indexStats) {
      if (idx.name !== "_id_" && idx.accesses.ops === 0) {
        console.log(`  Considering dropping unused index: ${idx.name}`);
        // await collection.dropIndex(idx.name);
      }
    }
    
    // Update collection statistics
    await collection.stats();
  }
  
  // 2. Optimize storage
  console.log("\nOptimizing storage...");
  await db.adminCommand({ compact: "largeCollection" });
  
  // 3. Set up monitoring
  await db.setProfilingLevel(1, 100); // Log slow operations >100ms
  await db.adminCommand({
    setParameter: 1,
    logLevel: 1,
    slowMS: 100,
    "operationProfiling.mode": "slowOp"
  });
  
  console.log("Database optimization complete!");
}

// Helper functions
async function analyzeSlowQueries(collectionName) {
  // Query system.profile for slow operations
  const slowOps = await db.system.profile
    .find({ ns: `mydb.${collectionName}`, millis: { $gt: 100 } })
    .sort({ ts: -1 })
    .limit(50)
    .toArray();
  
  return slowOps.map(op => ({
    filter: op.query,
    sort: op.sort,
    projection: op.projection,
    millis: op.millis
  }));
}

function buildOptimalIndex(query) {
  // Implement ESR (Equality, Sort, Range) rule
  const equalityFields = [];
  const sortFields = [];
  const rangeFields = [];
  
  // Analyze query pattern (simplified)
  // In practice, use MongoDB's index advisor or query planner
  
  return {
    keys: { ...equalityFields, ...sortFields, ...rangeFields },
    options: { background: true, name: `auto_${Date.now()}` }
  };
}

async function createIndexIfNeeded(collection, indexSpec) {
  const existing = await collection.indexExists(indexSpec.name);
  if (!existing) {
    console.log(`  Creating index: ${indexSpec.name}`);
    await collection.createIndex(indexSpec.keys, indexSpec.options);
  }
}
```

---

## **Summary**

MongoDB's indexing and performance ecosystem provides comprehensive tools for optimizing data access:

1. **Index Types**: Choose appropriate index (single, compound, text, geospatial, TTL) based on query patterns
2. **Performance**: Use `explain()`, profiling, and monitoring to identify bottlenecks
3. **Atlas Search**: Advanced full-text search capabilities for complex search requirements
4. **Connectors**: Integrate with Kafka, Spark, Elasticsearch for data pipelines and analytics
5. **Backup**: Regular backups with `mongodump/mongorestore` and retention policies
6. **Drivers**: Use official language drivers for optimal performance and reliability
7. **Maintenance**: Regular index optimization, storage management, and monitoring setup

Effective MongoDB performance management requires ongoing monitoring, regular index maintenance, and strategic use of the broader ecosystem for specific use cases like real-time streaming or advanced search.