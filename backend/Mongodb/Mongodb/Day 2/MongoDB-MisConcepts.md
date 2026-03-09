You're absolutely right! I missed those topics. Let me provide comprehensive coverage for **Indexing**, **Query Optimization**, and **Encryption** as standalone sections:

---

## **Indexing - Comprehensive Guide**

### **Index Types and Strategies**

**Notes:** Indexing is crucial for MongoDB performance. Different index types serve different query patterns. Understanding when and how to use each type is key to optimal performance.

**Code:**
```javascript
// 1. Single Field Index (Most common)
db.users.createIndex({ email: 1 }); // Ascending
db.users.createIndex({ createdAt: -1 }); // Descending

// 2. Compound Index (Multiple fields)
// Follow ESR rule: Equality -> Sort -> Range
db.orders.createIndex({ customerId: 1, orderDate: -1, status: 1 });

// 3. Multikey Index (For arrays)
db.products.createIndex({ tags: 1 }); // Indexes each element in array

// 4. Text Index (Full-text search)
db.articles.createIndex(
  { title: "text", content: "text" },
  { 
    weights: { title: 10, content: 5 },
    default_language: "english",
    name: "article_text_idx"
  }
);

// 5. Geospatial Index
db.places.createIndex({ location: "2dsphere" });
db.places.createIndex({ coordinates: "2d" }); // Legacy 2d

// 6. Hashed Index (For sharding/hashing)
db.users.createIndex({ _id: "hashed" });
db.logs.createIndex({ userId: "hashed" });

// 7. Wildcard Index (Dynamic fields)
db.collection.createIndex({ "metadata.$**": 1 });
db.collection.createIndex({ "$**": "text" }); // Wildcard text index

// 8. Compound Geospatial Index
db.places.createIndex({ category: 1, location: "2dsphere" });

// 9. Partial Index (Conditional indexing)
db.users.createIndex(
  { email: 1 },
  { 
    partialFilterExpression: { 
      email: { $exists: true },
      status: "active"
    }
  }
);

// 10. Sparse Index (Skip null/missing)
db.products.createIndex({ sku: 1 }, { sparse: true });

// 11. TTL Index (Automatic expiration)
db.sessions.createIndex({ expiresAt: 1 }, { expireAfterSeconds: 0 });
db.logs.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 }); // 24h

// 12. Unique Index
db.users.createIndex({ email: 1 }, { unique: true });
db.products.createIndex({ sku: 1 }, { unique: true, sparse: true });

// 13. Case Insensitive Index
db.users.createIndex(
  { email: 1 },
  { 
    collation: { 
      locale: "en",
      strength: 2 // Case insensitive
    }
  }
);

// 14. Clustered Index (MongoDB 5.3+)
db.createCollection("events", {
  clusteredIndex: {
    key: { _id: 1 },
    unique: true,
    name: "clustered_idx"
  }
});
```

### **Index Management Operations**

**Code:**
```javascript
// List all indexes
db.collection.getIndexes();

// Create multiple indexes at once
db.collection.createIndexes([
  { key: { userId: 1 } },
  { key: { createdAt: -1 } },
  { key: { status: 1, priority: -1 } }
]);

// Drop index
db.collection.dropIndex("email_1");
db.collection.dropIndex({ createdAt: -1 });

// Hide/Unhide index (for testing)
db.collection.hideIndex("expensive_index");
db.collection.unhideIndex("expensive_index");

// Rebuild index
db.collection.reIndex();

// Modify index options
// Note: Cannot modify directly, must drop and recreate
db.collection.dropIndex("old_index");
db.collection.createIndex(
  { field: 1 },
  { 
    name: "new_index",
    partialFilterExpression: { status: "active" }
  }
);

// Monitor index usage
const indexStats = db.collection.aggregate([{ $indexStats: {} }]).toArray();
indexStats.forEach(idx => {
  console.log(`${idx.name}: ${idx.accesses.ops} operations`);
});

// Identify unused indexes
const unused = indexStats.filter(
  idx => idx.name !== "_id_" && idx.accesses.ops === 0
);

// Force index hint
db.collection.find({ status: "active" })
  .hint({ status: 1, createdAt: -1 })
  .explain();

// Index storage stats
const stats = db.collection.stats();
console.log({
  totalIndexSize: stats.totalIndexSize,
  indexSizes: stats.indexSizes,
  avgObjSize: stats.avgObjSize
});
```

### **Index Analysis and Optimization**

**Code:**
```javascript
// Analyze query with explain()
const explain = db.orders.find({
  customerId: "12345",
  status: "completed",
  total: { $gt: 100 }
})
.sort({ orderDate: -1 })
.limit(10)
.explain("executionStats");

console.log("Query Analysis:", {
  executionTime: explain.executionStats.executionTimeMillis,
  documentsExamined: explain.executionStats.totalDocsExamined,
  keysExamined: explain.executionStats.totalKeysExamined,
  indexUsed: explain.executionStats.executionStages.inputStage?.indexName,
  stage: explain.executionStats.executionStages.stage
});

// Covered Query Analysis
const coveredQuery = db.orders.find(
  { customerId: "12345", status: "completed" },
  { _id: 0, customerId: 1, status: 1, orderDate: 1 }
).explain();

if (coveredQuery.executionStats.executionStages.stage === "PROJECTION_COVERED") {
  console.log("Query is covered by index!");
}

// Index Advisor (MongoDB Atlas)
// Use Performance Advisor in Atlas UI
// Or use $indexStats to analyze usage patterns

// Create optimal compound index
function createOptimalIndex(collection, queryPatterns) {
  // Analyze common query patterns
  const equalityFields = new Set();
  const sortFields = new Set();
  const rangeFields = new Set();
  
  queryPatterns.forEach(pattern => {
    // Extract fields from query, sort, projection
    pattern.filters.forEach(field => equalityFields.add(field));
    pattern.sorts.forEach(field => sortFields.add(field));
    pattern.ranges.forEach(field => rangeFields.add(field));
  });
  
  // Apply ESR rule: Equality -> Sort -> Range
  const indexFields = {};
  [...equalityFields].forEach(field => indexFields[field] = 1);
  [...sortFields].forEach(field => indexFields[field] = 1);
  [...rangeFields].forEach(field => indexFields[field] = 1);
  
  return collection.createIndex(indexFields, {
    name: `auto_${Date.now()}_compound`,
    background: true
  });
}

// Index size optimization
async function optimizeIndexSizes() {
  const collections = await db.getCollectionNames();
  
  for (const collName of collections) {
    const collection = db[collName];
    const indexes = await collection.getIndexes();
    
    for (const idx of indexes) {
      // Consider dropping large, rarely used indexes
      const stats = await collection.aggregate([
        { $indexStats: {} },
        { $match: { name: idx.name } }
      ]).next();
      
      if (stats && stats.accesses.ops < 100) {
        console.log(`Candidate for removal: ${idx.name} (${stats.accesses.ops} ops)`);
      }
    }
  }
}
```

---

## **Query Optimization - Deep Dive**

### **Query Performance Patterns**

**Code:**
```javascript
// 1. Efficient Filtering
// GOOD: Uses index
db.users.find({ 
  status: "active",
  lastLogin: { $gte: new Date("2024-01-01") }
});

// BAD: Full collection scan
db.users.find({ 
  $where: "this.status === 'active' && this.lastLogin >= new Date('2024-01-01')" 
});

// 2. Selective Projection
// GOOD: Only needed fields
db.users.find(
  { status: "active" },
  { name: 1, email: 1, _id: 0 } // Minimal fields
);

// BAD: All fields (including large arrays)
db.users.find({ status: "active" });

// 3. Efficient Sorting
// GOOD: Index supports sort
db.orders.find({ customerId: "123" }).sort({ orderDate: -1 });

// BAD: Sort in memory (large dataset)
db.orders.find({}).sort({ total: 1 });

// 4. Pagination Strategies
// Keyset pagination (Efficient)
const lastId = ObjectId("...");
const page = await db.orders.find({
  _id: { $gt: lastId },
  status: "completed"
})
.sort({ _id: 1 })
.limit(50)
.toArray();

// Offset pagination (Inefficient for large offsets)
const page = await db.orders.find({ status: "completed" })
.skip(10000) // Expensive!
.limit(50)
.toArray();

// 5. Aggregation Optimization
db.orders.aggregate([
  { $match: { status: "completed", date: { $gte: startDate } } }, // Early filter
  { $project: { customerId: 1, total: 1, date: 1 } }, // Reduce data early
  { $group: { _id: "$customerId", totalSpent: { $sum: "$total" } } },
  { $sort: { totalSpent: -1 } },
  { $limit: 100 }
]);

// 6. Batch Operations
// Instead of multiple single updates:
async function updateManyInefficient(docs) {
  for (const doc of docs) {
    await db.collection.updateOne(
      { _id: doc._id },
      { $set: doc }
    );
  }
}

// Use bulkWrite:
async function updateManyEfficient(docs) {
  const operations = docs.map(doc => ({
    updateOne: {
      filter: { _id: doc._id },
      update: { $set: doc }
    }
  }));
  
  await db.collection.bulkWrite(operations, { ordered: false });
}

// 7. Avoid Large IN Clauses
// BAD: Large $in array
db.users.find({ _id: { $in: hugeArrayOfIds } });

// GOOD: Use multiple queries or aggregation
const batches = chunkArray(hugeArrayOfIds, 1000);
for (const batch of batches) {
  const users = await db.users.find({ _id: { $in: batch } }).toArray();
  // Process batch
}

// 8. Regex Optimization
// BAD: Leading wildcard (can't use index)
db.users.find({ name: { $regex: /.*smith/i } });

// GOOD: Anchored regex (can use index)
db.users.find({ name: { $regex: /^smith/i } });

// 9. Count Optimization
// Use estimated count for approximate
const approxCount = await db.collection.estimatedDocumentCount();

// Use countDocuments for exact with filter
const exactCount = await db.collection.countDocuments({ status: "active" });

// 10. Distinct Optimization
// BAD: Distinct on unindexed field
db.collection.distinct("unindexedField");

// GOOD: Use aggregation with index
db.collection.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$indexedField" } }
]);
```

### **Query Analysis Tools**

**Code:**
```javascript
// 1. Explain Plans
async function analyzeQuery(query) {
  const explanation = await query.explain("allPlansExecution");
  
  const analysis = {
    winningPlan: explanation.queryPlanner.winningPlan,
    rejectedPlans: explanation.queryPlanner.rejectedPlans,
    executionStats: explanation.executionStats,
    allPlansExecution: explanation.allPlansExecution
  };
  
  // Check for COLLSCAN (full collection scan)
  function hasCollectionScan(plan) {
    if (plan.stage === "COLLSCAN") return true;
    if (plan.inputStage) return hasCollectionScan(plan.inputStage);
    return false;
  }
  
  if (hasCollectionScan(analysis.winningPlan)) {
    console.warn("Query performing collection scan - consider adding index");
  }
  
  return analysis;
}

// 2. MongoDB Profiler
// Enable profiling
db.setProfilingLevel(1, 100); // Log queries > 100ms
db.setProfilingLevel(2); // Log all queries

// Check slow queries
const slowQueries = db.system.profile
  .find({ millis: { $gt: 100 } })
  .sort({ ts: -1 })
  .limit(10)
  .toArray();

// Analyze profile data
slowQueries.forEach(query => {
  console.log({
    op: query.op,
    ns: query.ns,
    command: query.command,
    millis: query.millis,
    docsExamined: query.docsExamined,
    keysExamined: query.keysExamined
  });
});

// 3. Database Commands for Monitoring
// Current operations
const currentOps = db.currentOp({
  "active": true,
  "secs_running": { "$gt": 5 }
});

// Server status
const serverStatus = db.adminCommand({ serverStatus: 1 });

// Collection stats
const collStats = db.collection.stats();
console.log({
  sizeMB: collStats.size / 1024 / 1024,
  count: collStats.count,
  avgObjSize: collStats.avgObjSize,
  storageSize: collStats.storageSize,
  totalIndexSize: collStats.totalIndexSize
});

// 4. Index Usage Analysis
async function analyzeIndexUsage(collectionName) {
  const collection = db[collectionName];
  const indexStats = await collection.aggregate([{ $indexStats: {} }]).toArray();
  
  const queryStats = await db.system.profile.aggregate([
    { $match: { ns: `${db.getName()}.${collectionName}` } },
    { $group: {
      _id: null,
      totalQueries: { $sum: 1 },
      avgMillis: { $avg: "$millis" },
      maxMillis: { $max: "$millis" }
    }}
  ]).toArray();
  
  return {
    indexes: indexStats,
    queryPerformance: queryStats[0] || {}
  };
}
```

### **Advanced Optimization Techniques**

**Code:**
```javascript
// 1. Query Planner Hints
// Force specific index
db.collection.find({})
  .hint({ field1: 1, field2: -1 })
  .explain();

// Disable index intersection
db.collection.find({ field1: "A", field2: "B" })
  .hint({ $natural: 1 }) // Force natural order
  .explain();

// 2. Parallel Index Scanning
// For sharded collections, queries can scan indexes in parallel
const cursor = db.shardedCollection.find({ region: "US" });
// MongoDB automatically parallelizes across shards

// 3. Read Concern for Performance
// Local (fastest)
const fastRead = db.collection.find({}).readConcern("local");

// Majority (more consistent, slower)
const consistentRead = db.collection.find({}).readConcern("majority");

// 4. Write Concern for Performance
// Unacknowledged (fastest)
await db.collection.insertOne(doc, { writeConcern: { w: 0 } });

// Acknowledged (default)
await db.collection.insertOne(doc, { writeConcern: { w: 1 } });

// Journaled (most durable, slowest)
await db.collection.insertOne(doc, { 
  writeConcern: { w: 1, j: true, wtimeout: 5000 }
});

// 5. Cursor Management
// Stream results (memory efficient)
const cursor = db.largeCollection.find({}).batchSize(100);
while (await cursor.hasNext()) {
  const doc = await cursor.next();
  processDocument(doc);
}
cursor.close();

// Avoid cursor timeout
const cursor = db.collection.find({})
  .noCursorTimeout(true)
  .maxTimeMS(300000); // 5 minute timeout

// 6. Aggregation Pipeline Optimization
db.orders.aggregate([
  { $match: { status: "completed" } }, // Push match early
  { $sort: { orderDate: -1 } }, // Sort can use index
  { $limit: 1000 }, // Limit early
  { $lookup: { // Expensive operation last
    from: "customers",
    localField: "customerId",
    foreignField: "_id",
    as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: { 
    orderId: 1,
    total: 1,
    "customer.name": 1 
  }}
]);

// 7. Materialized Views Pattern
// Create aggregated collection
async function refreshMaterializedView() {
  const aggregated = await db.sourceCollection.aggregate([
    { $match: { date: { $gte: startDate } } },
    { $group: {
      _id: { 
        year: { $year: "$date" },
        month: { $month: "$date" },
        category: "$category"
      },
      total: { $sum: "$amount" },
      count: { $sum: 1 },
      avg: { $avg: "$amount" }
    }},
    { $out: "materialized_view" } // Write to new collection
  ]).toArray();
}

// Query materialized view
const fastResults = db.materialized_view.find({
  "_id.year": 2024,
  "_id.category": "electronics"
});

// 8. Connection Pool Optimization
const client = new MongoClient(uri, {
  maxPoolSize: 100, // Adjust based on workload
  minPoolSize: 10,
  maxIdleTimeMS: 30000,
  waitQueueTimeoutMS: 2000,
  retryWrites: true,
  retryReads: true,
  serverSelectionTimeoutMS: 30000,
  connectTimeoutMS: 10000
});

// 9. Compression for Network/Storage
// Enable compression
const client = new MongoClient(uri, {
  compressors: ['zstd', 'snappy', 'zlib'] // In order of preference
});

// 10. Monitoring and Alerting
async function setupQueryMonitoring() {
  // Create TTL index for old profile data
  db.system.profile.createIndex(
    { ts: 1 },
    { expireAfterSeconds: 7 * 24 * 60 * 60 } // 7 days
  );
  
  // Schedule regular analysis
  setInterval(async () => {
    const slowQueries = await analyzeSlowQueries();
    if (slowQueries.length > 0) {
      notifyTeam(slowQueries);
    }
  }, 300000); // Every 5 minutes
}
```

---

## **Encryption - Comprehensive Coverage**

### **1. Encryption at Rest**

**Code:**
```javascript
// WiredTiger Encryption Configuration
// mongod.conf
storage:
  wiredTiger:
    engineConfig:
      encryption:
        name: "kmip" # Options: kmip, local
        keyId: "<key identifier>" # For key rotation
        kmip:
          serverName: "kmip.example.com"
          port: 5696
          clientCertificateFile: "/path/to/client.pem"
          clientCertificatePassword: "password"
          serverCAFile: "/path/to/ca.pem"

// Local Key Encryption
security:
  enableEncryption: true
  encryptionCipherMode: "AES256-GCM" # AES256-CBC or AES256-GCM
  encryptionKeyFile: "/etc/mongodb/mongodb-keyfile"

// Generate encryption keyfile
// openssl rand -base64 32 > /etc/mongodb/mongodb-keyfile
// chmod 600 /etc/mongodb/mongodb-keyfile
// chown mongodb:mongodb /etc/mongodb/mongodb-keyfile

// Rotate encryption key
db.adminCommand({ rotateEncryptionKey: 1 });

// Check encryption status
const encryptionStatus = db.adminCommand({ 
  getParameter: 1, 
  enableEncryption: 1 
});
console.log("Encryption enabled:", encryptionStatus.enableEncryption);

// Encrypted backup
// mongodump with encryption
// mongodump --uri="mongodb://localhost:27017" --gzip --archive | \
//   openssl enc -aes-256-cbc -salt -out backup.enc -pass pass:"secret"

// Encrypted restore
// openssl enc -aes-256-cbc -d -in backup.enc -pass pass:"secret" | \
//   mongorestore --archive --gzip
```

### **2. Field Level Encryption**

**Code:**
```javascript
// Client-Side Field Level Encryption (CSFLE)
const { MongoClient, ClientEncryption } = require('mongodb');

// 1. Setup Key Management
const kmsProviders = {
  // Local Key Provider
  local: {
    key: Buffer.from(process.env.LOCAL_KEY, 'base64')
  },
  // AWS KMS
  aws: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    sessionToken: process.env.AWS_SESSION_TOKEN // Optional
  },
  // Azure Key Vault
  azure: {
    tenantId: process.env.AZURE_TENANT_ID,
    clientId: process.env.AZURE_CLIENT_ID,
    clientSecret: process.env.AZURE_CLIENT_SECRET
  },
  // GCP KMS
  gcp: {
    email: process.env.GCP_EMAIL,
    privateKey: process.env.GCP_PRIVATE_KEY,
    endpoint: process.env.GCP_ENDPOINT // Optional
  }
};

// 2. Create Data Encryption Keys (DEK)
async function setupEncryption() {
  const client = new MongoClient(uri);
  await client.connect();
  
  const encryption = new ClientEncryption(client, {
    keyVaultNamespace: "encryption.__keyVault",
    kmsProviders
  });
  
  // Create DEK for each sensitivity level
  const dekPatient = await encryption.createDataKey("local", {
    keyAltNames: ["patient-data-key"]
  });
  
  const dekFinancial = await encryption.createDataKey("aws", {
    keyAltNames: ["financial-data-key"],
    masterKey: {
      key: "arn:aws:kms:us-east-1:123456789012:key/abcd1234",
      region: "us-east-1"
    }
  });
  
  return { dekPatient, dekFinancial };
}

// 3. JSON Schema for Automatic Encryption
const schemaMap = {
  "medical.patients": {
    bsonType: "object",
    encryptMetadata: {
      keyId: [UUID("...")] // Reference to DEK
    },
    properties: {
      _id: { bsonType: "objectId" },
      ssn: {
        encrypt: {
          bsonType: "string",
          algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic", // Queryable
          keyId: [UUID("...")]
        }
      },
      medicalHistory: {
        encrypt: {
          bsonType: "string",
          algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random" // Not queryable
        }
      },
      prescriptions: {
        bsonType: "array",
        items: {
          bsonType: "object",
          properties: {
            drugName: { bsonType: "string" },
            dosage: {
              encrypt: {
                bsonType: "string",
                algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
              }
            }
          }
        }
      }
    }
  }
};

// 4. Client with Automatic Encryption
const encryptedClient = new MongoClient(uri, {
  autoEncryption: {
    keyVaultNamespace: "encryption.__keyVault",
    kmsProviders,
    schemaMap,
    bypassAutoEncryption: false, // Enable automatic encryption
    extraOptions: {
      cryptSharedLibPath: "/path/to/mongo_crypt_v1.so" // For Queryable Encryption
    }
  }
});

// 5. Manual Encryption/Decryption
async function manualEncryptionExample() {
  const client = new MongoClient(uri);
  await client.connect();
  
  const encryption = new ClientEncryption(client, {
    keyVaultNamespace: "encryption.__keyVault",
    kmsProviders
  });
  
  // Encrypt a value
  const encryptedSSN = await encryption.encrypt("123-45-6789", {
    keyId: dekPatient,
    algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic"
  });
  
  // Decrypt a value
  const decryptedSSN = await encryption.decrypt(encryptedSSN);
  
  // Insert encrypted document
  await db.patients.insertOne({
    name: "John Doe",
    ssn: encryptedSSN,
    otherData: "unencrypted data"
  });
}

// 6. Queryable Encryption (MongoDB 7.0+)
const queryableSchema = {
  "bank.accounts": {
    bsonType: "object",
    properties: {
      accountNumber: {
        encrypt: {
          bsonType: "string",
          algorithm: "Equality", // Special queryable algorithm
          keyId: [UUID("...")],
          contentionFactor: 4 // For security vs performance tradeoff
        }
      },
      balance: {
        encrypt: {
          bsonType: "double",
          algorithm: "RangePreview", // Range queries (preview)
          keyId: [UUID("...")],
          min: 0,
          max: 1000000,
          precision: 2
        }
      }
    }
  }
};

// Query encrypted fields
const result = await db.accounts.find({
  accountNumber: "1234567890" // Encrypted equality query
});

// 7. Key Rotation
async function rotateDataKey(oldKeyId, newKeyId) {
  const client = new MongoClient(uri, {
    autoEncryption: {
      keyVaultNamespace: "encryption.__keyVault",
      kmsProviders,
      schemaMap,
      extraOptions: {
        cryptSharedLibPath: "/path/to/mongo_crypt_v1.so"
      }
    }
  });
  
  // MongoDB handles re-encryption automatically on read/write
  // Or use explicit re-encryption
  const cursor = db.sensitiveCollection.find({});
  while (await cursor.hasNext()) {
    const doc = await cursor.next();
    // Re-encrypt with new key
    const reEncrypted = await reencryptDocument(doc, oldKeyId, newKeyId);
    await db.sensitiveCollection.replaceOne({ _id: doc._id }, reEncrypted);
  }
}

// 8. Encryption Audit Logging
const auditLog = {
  "encryption.events": {
    bsonType: "object",
    properties: {
      timestamp: { bsonType: "date" },
      operation: { bsonType: "string" },
      keyId: { 
        encrypt: {
          bsonType: "binData",
          algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
        }
      },
      userId: { bsonType: "string" },
      ipAddress: { bsonType: "string" }
    }
  }
};
```

### **3. TLS/SSL Encryption in Detail**

**Code:**
```yaml
# Complete TLS Configuration (mongod.conf)
net:
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/ssl/mongodb.pem
    certificateKeyFilePassword: "password" # If PEM is encrypted
    CAFile: /etc/ssl/ca.pem
    CRLFile: /etc/ssl/crl.pem # Certificate Revocation List
    clusterFile: /etc/ssl/mongodb-cluster.pem # For internal auth
    clusterPassword: "clusterPassword"
    allowConnectionsWithoutCertificates: false
    allowInvalidHostnames: false # Strict hostname validation
    disabledProtocols: TLS1_0,TLS1_1 # Disable insecure protocols
    cipherSuites: "TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256"
    # FIPS mode (for compliance)
    # FIPSMode: true

# PEM file should contain both certificate and private key
# -----BEGIN CERTIFICATE-----
# ... certificate ...
# -----END CERTIFICATE-----
# -----BEGIN PRIVATE KEY-----
# ... private key ...
# -----END PRIVATE KEY-----

# For certificate management with Let's Encrypt
# Use certbot to obtain certificates
# certbot certonly --standalone -d mongodb.example.com
```

```javascript
// Node.js Client with Advanced TLS
const fs = require('fs');
const { MongoClient } = require('mongodb');

const client = new MongoClient(uri, {
  tls: true,
  tlsCertificateKeyFile: '/path/to/client.pem',
  tlsCAFile: '/path/to/ca.pem',
  tlsCertificateKeyFilePassword: 'clientPassword',
  tlsAllowInvalidCertificates: false, // Always false in production
  tlsAllowInvalidHostnames: false,    // Validate hostnames
  tlsInsecure: false,                 // NEVER true in production
  // TLS specific options
  checkServerIdentity: (hostname, cert) => {
    // Custom server identity validation
    if (hostname !== cert.subject.CN) {
      throw new Error('Certificate hostname mismatch');
    }
  },
  secureContext: { // Advanced TLS context
    ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384',
    honorCipherOrder: true,
    minVersion: 'TLSv1.2'
  },
  // SNI (Server Name Indication)
  servername: 'mongodb.example.com'
});

// Testing TLS connection
async function testTLSConnection() {
  try {
    await client.connect();
    const result = await client.db('admin').command({ ismaster: 1 });
    console.log('TLS Connection successful:', {
      host: result.host,
      tlsSupported: true,
      serverVersion: result.version
    });
    
    // Verify certificate
    const tlsInfo = await client.db('admin').command({ ismaster: 1 });
    console.log('TLS Connection details:', {
      cipher: tlsInfo.connectionInfo?.cipher,
      protocol: tlsInfo.connectionInfo?.protocol
    });
    
  } catch (error) {
    console.error('TLS Connection failed:', error.message);
  }
}

// Certificate rotation without downtime
async function rotateTLSCertificate() {
  // 1. Generate new certificate
  // openssl req -new -x509 -days 365 -nodes -out new-cert.pem -keyout new-key.pem
  
  // 2. Combine certificate and key
  // cat new-cert.pem new-key.pem > mongodb-new.pem
  
  // 3. Update mongod.conf with new file
  // 4. Reload mongod configuration
  db.adminCommand({ setParameter: 1, tlsCertificateSelector: "subject=CN=new.mongodb.example.com" });
  
  // 5. Wait for connections to migrate
  await new Promise(resolve => setTimeout(resolve, 30000));
  
  // 6. Remove old certificate
}

// Monitoring TLS connections
async function monitorTLS() {
  const serverStatus = await db.adminCommand({ serverStatus: 1 });
  console.log('TLS Statistics:', {
    currentTLSConnections: serverStatus.network?.ssl?.current,
    totalTLSConnections: serverStatus.network?.ssl?.total,
    tlsHandshakeErrors: serverStatus.network?.ssl?.handshakeErrors
  });
  
  // Current connections with TLS info
  const currentOps = await db.adminCommand({ 
    currentOp: 1, 
    $all: true,
    desc: "tls connections" 
  });
  
  const tlsConnections = currentOps.inprog.filter(op => 
    op.clientMetadata?.driver?.tls === true
  );
}
```

### **4. Complete Encryption Implementation**

**Code:**
```javascript
// Complete security implementation with all encryption types
class MongoDBSecurityManager {
  constructor(config) {
    this.config = config;
    this.clients = new Map();
  }
  
  async initialize() {
    // 1. Setup TLS connections
    await this.setupTLS();
    
    // 2. Setup Field Level Encryption
    await this.setupFieldEncryption();
    
    // 3. Setup Key Management
    await this.setupKeyManagement();
    
    // 4. Setup Audit Logging
    await this.setupAudit();
    
    console.log('MongoDB Security Manager initialized');
  }
  
  async setupTLS() {
    // TLS configuration for production
    const tlsOptions = {
      tls: true,
      tlsCertificateKeyFile: this.config.tls.certPath,
      tlsCAFile: this.config.tls.caPath,
      tlsAllowInvalidCertificates: false,
      tlsAllowInvalidHostnames: false,
      minHeartbeatFrequencyMS: 1000,
      serverSelectionTimeoutMS: 30000,
      connectTimeoutMS: 10000
    };
    
    this.tlsClient = new MongoClient(this.config.uri, tlsOptions);
    await this.tlsClient.connect();
    
    // Verify TLS
    const tlsStatus = await this.tlsClient.db('admin').command({ 
      getParameter: 1, 
      sslMode: 1 
    });
    console.log(`TLS Mode: ${tlsStatus.sslMode}`);
  }
  
  async setupFieldEncryption() {
    // KMS Providers
    const kmsProviders = {
      aws: {
        accessKeyId: this.config.aws.accessKeyId,
        secretAccessKey: this.config.aws.secretAccessKey
      }
    };
    
    // Encryption schema for sensitive collections
    const schemaMap = {
      [`${this.config.database}.patients`]: this.createPatientSchema(),
      [`${this.config.database}.transactions`]: this.createTransactionSchema()
    };
    
    // Create encrypted client
    this.encryptedClient = new MongoClient(this.config.uri, {
      autoEncryption: {
        keyVaultNamespace: `${this.config.database}.__keyVault`,
        kmsProviders,
        schemaMap,
        extraOptions: {
          cryptSharedLibPath: this.config.cryptLibPath
        }
      },
      ...this.getTLSConfig()
    });
    
    await this.encryptedClient.connect();
  }
  
  createPatientSchema() {
    return {
      bsonType: "object",
      encryptMetadata: {
        keyId: [this.config.keys.patientKeyId]
      },
      properties: {
        ssn: {
          encrypt: {
            bsonType: "string",
            algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic"
          }
        },
        medicalRecord: {
          encrypt: {
            bsonType: "string",
            algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
          }
        },
        prescriptions: {
          bsonType: "array",
          items: {
            bsonType: "object",
            properties: {
              drugName: { bsonType: "string" },
              dosage: {
                encrypt: {
                  bsonType: "string",
                  algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
                }
              }
            }
          }
        }
      }
    };
  }
  
  async setupKeyManagement() {
    // Initialize key vault
    const keyVault = this.encryptedClient
      .db(this.config.database)
      .collection('__keyVault');
    
    // Create indexes for key vault
    await keyVault.createIndex(
      { keyAltNames: 1 },
      { 
        unique: true, 
        partialFilterExpression: { keyAltNames: { $exists: true } }
      }
    );
    
    // Create initial keys if they don't exist
    const existingKeys = await keyVault.countDocuments();
    if (existingKeys === 0) {
      await this.createInitialKeys();
    }
  }
  
  async setupAudit() {
    // Enable audit logging
    await this.tlsClient.db('admin').command({
      setParameter: 1,
      auditAuthorizationSuccess: true,
      auditAuthorizationFailure: true
    });
    
    // Create audit collection with encryption
    const auditSchema = {
      bsonType: "object",
      properties: {
        timestamp: { bsonType: "date" },
        userId: {
          encrypt: {
            bsonType: "string",
            algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic"
          }
        },
        action: { bsonType: "string" },
        details: {
          encrypt: {
            bsonType: "string",
            algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
          }
        }
      }
    };
  }
  
  getTLSConfig() {
    return {
      tls: true,
      tlsCertificateKeyFile: this.config.tls.certPath,
      tlsCAFile: this.config.tls.caPath,
      tlsAllowInvalidCertificates: false,
      tlsAllowInvalidHostnames: false
    };
  }
  
  async auditLog(event) {
    const auditCollection = this.encryptedClient
      .db(this.config.database)
      .collection('audit_log');
    
    await auditCollection.insertOne({
      timestamp: new Date(),
      userId: event.userId,
      action: event.action,
      details: event.details,
      ipAddress: event.ipAddress,
      userAgent: event.userAgent
    });
  }
  
  async close() {
    await this.tlsClient.close();
    await this.encryptedClient.close();
    console.log('Security Manager closed');
  }
}

// Usage
const securityConfig = {
  uri: process.env.MONGODB_URI,
  database: 'secure_app',
  tls: {
    certPath: '/etc/ssl/mongodb.pem',
    caPath: '/etc/ssl/ca.pem'
  },
  aws: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
  },
  cryptLibPath: '/usr/lib/mongo_crypt_v1.so',
  keys: {
    patientKeyId: UUID('...'),
    transactionKeyId: UUID('...')
  }
};

const securityManager = new MongoDBSecurityManager(securityConfig);
await securityManager.initialize();

// Use encrypted client for sensitive operations
const encryptedDb = securityManager.encryptedClient.db('secure_app');

// Perform encrypted operations
await encryptedDb.collection('patients').insertOne({
  name: 'John Doe',
  ssn: '123-45-6789', // Automatically encrypted
  medicalRecord: 'Sensitive data...' // Automatically encrypted
});

// Query encrypted field
const patient = await encryptedDb.collection('patients').findOne({
  ssn: '123-45-6789' // Query works on encrypted field
});
```

---

## **Summary of Key Concepts**

### **Indexing:**
1. **Choose the right index type** for your query patterns
2. **Follow ESR rule** for compound indexes: Equality → Sort → Range
3. **Monitor index usage** and remove unused indexes
4. **Use partial indexes** for conditional indexing
5. **Consider index size** and maintenance overhead

### **Query Optimization:**
1. **Use explain()** to analyze query performance
2. **Avoid collection scans** with proper indexing
3. **Implement efficient pagination** (keyset vs offset)
4. **Optimize aggregation pipelines** (push $match early)
5. **Monitor slow queries** with profiling
6. **Use covered queries** when possible
7. **Batch operations** for better performance

### **Encryption:**
1. **Enable TLS/SSL** for all network traffic
2. **Use encryption at rest** for data protection
3. **Implement field-level encryption** for sensitive data
4. **Choose appropriate encryption algorithms** based on needs
5. **Manage encryption keys** securely (KMS, key rotation)
6. **Use queryable encryption** for encrypted queries
7. **Implement audit logging** for compliance
8. **Test encryption configurations** thoroughly

### **Best Practices:**
1. **Security in depth**: Multiple layers of protection
2. **Regular audits**: Monitor and review security configurations
3. **Automated testing**: Test security configurations automatically
4. **Documentation**: Document all security measures
5. **Training**: Train team on security best practices
6. **Incident response**: Have a plan for security incidents
7. **Compliance**: Regularly check against compliance requirements
8. **Updates**: Keep MongoDB and security configurations updated