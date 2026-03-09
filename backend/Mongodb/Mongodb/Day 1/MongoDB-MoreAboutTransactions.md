# **MongoDB Transactions - Deep Dive**

## **Overview & Architecture**

**Core Concept:** MongoDB transactions allow multiple read/write operations across different documents, collections, and databases to execute as a single atomic unit. They ensure **ACID** properties (Atomicity, Consistency, Isolation, Durability) in distributed environments.

**Evolution:**
- **MongoDB 4.0** (2018): Transactions for replica sets
- **MongoDB 4.2** (2019): Distributed transactions across sharded clusters
- **MongoDB 4.4+**: Performance improvements and snapshot isolation

**Key Components:**
1. **Logical Session** → Tracks transaction state
2. **Transaction Number** → Unique ID per transaction
3. **Oplog Entries** → Records for replication
4. **Snapshot** → Point-in-time view of data

---

## **Transaction Lifecycle**

### **1. Starting a Transaction**
```javascript
const session = client.startSession();
session.startTransaction({
  readConcern: { level: "snapshot" },
  writeConcern: { w: "majority", wtimeout: 5000 },
  readPreference: "primary"
});
```

### **2. Executing Operations**
```javascript
// All operations must use the same session
await collection1.insertOne({ doc: 1 }, { session });
await collection2.updateOne({ filter }, { update }, { session });
await collection3.deleteOne({ filter }, { session });
```

### **3. Committing or Aborting**
```javascript
try {
  // Successful completion
  await session.commitTransaction();
  console.log("Transaction committed successfully");
} catch (error) {
  // On any error, abort
  await session.abortTransaction();
  console.error("Transaction aborted:", error);
  throw error;
} finally {
  // Always end session
  session.endSession();
}
```

---

## **Transaction Options & Configuration**

### **Read Concern Levels:**
```javascript
// Available read concern levels for transactions:
{
  readConcern: {
    level: "local"          // Default, reads latest data (may be rolled back)
    level: "majority"       // Reads majority-committed data
    level: "snapshot"       // Recommended for transactions (since MongoDB 4.0)
    level: "linearizable"   // Strongest, for single-document reads
  }
}
```

### **Write Concern Options:**
```javascript
{
  writeConcern: {
    w: 1,                    // Acknowledge by primary only
    w: "majority",           // Acknowledge by majority of replica set
    w: 3,                    // Acknowledge by specific number of nodes
    j: true,                 // Journal durability (default: false)
    wtimeout: 5000           // Timeout in milliseconds
  }
}
```

### **Read Preference:**
```javascript
{
  readPreference: "primary",          // Default for transactions
  // Other options (not recommended for transactions):
  // readPreference: "primaryPreferred"
  // readPreference: "secondary"
  // readPreference: "secondaryPreferred"
  // readPreference: "nearest"
}
```

---

## **Isolation Levels & Behavior**

### **Snapshot Isolation (Default)**
- Transactions see a consistent snapshot of data from start time
- No dirty reads, non-repeatable reads, or phantom reads
- Write conflicts cause abort with `WriteConflict` error

### **Write Conflicts Example:**
```javascript
// Transaction 1
session1.startTransaction();
await db.accounts.updateOne(
  { _id: "acc1" },
  { $inc: { balance: 100 } },
  { session: session1 }
);

// Transaction 2 (concurrent)
session2.startTransaction();
await db.accounts.updateOne(
  { _id: "acc1" },
  { $inc: { balance: 50 } },
  { session: session2 }
);

// One transaction will abort with WriteConflict error
// MongoDB automatically retries aborted transactions (up to limit)
```

### **Retry Logic:**
```javascript
const runTransactionWithRetry = async (txnFunc, session) => {
  let retryCount = 0;
  const maxRetries = 3;
  
  while (retryCount < maxRetries) {
    try {
      await txnFunc(session);
      break; // Success
    } catch (error) {
      // Check if retryable error
      if (error.hasErrorLabel("TransientTransactionError")) {
        retryCount++;
        console.log(`Retrying transaction (attempt ${retryCount})`);
        await new Promise(resolve => setTimeout(resolve, 100 * retryCount)); // Backoff
        continue;
      } else if (error.hasErrorLabel("UnknownTransactionCommitResult")) {
        // Retry commit
        retryCount++;
        continue;
      } else {
        throw error; // Non-retryable error
      }
    }
  }
  
  if (retryCount >= maxRetries) {
    throw new Error("Max transaction retries exceeded");
  }
};
```

---

## **Performance Considerations & Best Practices**

### **1. Keep Transactions Short**
```javascript
// GOOD: Short, focused transaction
async function transferFunds(fromId, toId, amount) {
  const session = client.startSession();
  try {
    session.startTransaction();
    
    // Quick operations
    await withdraw(fromId, amount, session);
    await deposit(toId, amount, session);
    await recordTransaction(fromId, toId, amount, session);
    
    await session.commitTransaction();
  } catch (error) {
    await session.abortTransaction();
    throw error;
  } finally {
    session.endSession();
  }
}

// BAD: Long-running transaction
async function processBatch(batchSize) {
  const session = client.startSession();
  session.startTransaction();
  
  for (let i = 0; i < batchSize; i++) {
    await processItem(i, session); // Each iteration adds latency
    await externalAPICall();       // External calls increase timeout risk
  }
  
  // High risk of timeout or contention
}
```

### **2. Limit Transaction Size**
- **Document Count**: Keep under 1000 operations per transaction
- **Data Size**: Keep under 16MB total (BSON document limit)
- **Duration**: Keep under 60 seconds (default timeout)

### **3. Indexing Strategy**
```javascript
// Ensure indexes on fields used in transaction queries
db.accounts.createIndex({ _id: 1, balance: 1 });
db.transactions.createIndex({ fromAccount: 1, timestamp: -1 });

// Compound indexes for multi-document queries
db.orders.createIndex({ customerId: 1, status: 1, date: -1 });
```

### **4. Sharding Considerations**
```javascript
// For sharded clusters, all shard keys in transaction must be known
// Example: Transaction across sharded collections

// GOOD: Single shard key value
session.startTransaction();
await shardedCollection.updateOne(
  { shardKey: "value1" },  // Affects only one shard
  { $set: { status: "active" } },
  { session }
);

// BAD: Multiple shard key values (cross-shard transaction - slower)
session.startTransaction();
await shardedCollection.updateMany(
  { shardKey: { $in: ["value1", "value2", "value3"] } }, // Affects multiple shards
  { $set: { status: "active" } },
  { session }
);
```

---

## **Error Handling Patterns**

### **1. Comprehensive Error Handling**
```javascript
async function executeTransaction() {
  const session = client.startSession();
  
  try {
    session.startTransaction({
      readConcern: { level: "snapshot" },
      writeConcern: { w: "majority", j: true },
      maxTimeMS: 30000 // 30 second timeout
    });
    
    // Transaction operations
    await operation1(session);
    await operation2(session);
    
    // Attempt commit
    await session.commitTransaction();
    
  } catch (error) {
    // Classify error type
    if (error.errorLabels && error.errorLabels.includes("TransientTransactionError")) {
      console.log("Transient error, can retry:", error.message);
      await session.abortTransaction();
      throw new RetryableError(error.message);
      
    } else if (error.errorLabels && error.errorLabels.includes("UnknownTransactionCommitResult")) {
      console.log("Commit result unknown, check status");
      // Check if transaction actually committed
      const committed = await checkTransactionStatus(session);
      if (!committed) {
        await session.abortTransaction();
      }
      throw error;
      
    } else if (error.code === 251) { // NoSuchTransaction
      console.log("Transaction no longer exists");
      // Clean up without abort
      
    } else if (error.code === 50) { // ExceededTimeLimit
      console.log("Transaction timeout");
      await session.abortTransaction();
      throw new TimeoutError("Transaction exceeded time limit");
      
    } else {
      // Non-retryable error
      console.error("Fatal transaction error:", error);
      await session.abortTransaction();
      throw error;
    }
    
  } finally {
    // Always clean up session
    if (session.inTransaction()) {
      try {
        await session.abortTransaction();
      } catch (abortError) {
        console.warn("Error aborting transaction:", abortError);
      }
    }
    session.endSession();
  }
}
```

### **2. Idempotent Operations Pattern**
```javascript
// Design operations to be safely retryable
async function createOrderWithIdempotency(orderData, idempotencyKey) {
  const session = client.startSession();
  
  try {
    session.startTransaction();
    
    // Check if already processed
    const existing = await db.idempotencyKeys.findOne(
      { key: idempotencyKey },
      { session }
    );
    
    if (existing) {
      // Return existing result (idempotent)
      return existing.result;
    }
    
    // Process order
    const orderResult = await processOrder(orderData, session);
    
    // Record idempotency key
    await db.idempotencyKeys.insertOne({
      key: idempotencyKey,
      result: orderResult,
      createdAt: new Date()
    }, { session });
    
    await session.commitTransaction();
    return orderResult;
    
  } catch (error) {
    await session.abortTransaction();
    throw error;
  } finally {
    session.endSession();
  }
}
```

---

## **Monitoring & Diagnostics**

### **1. Transaction Metrics**
```javascript
// Check transaction statistics
const serverStatus = await db.adminCommand({ serverStatus: 1 });
console.log("Transaction metrics:", serverStatus.transactions);

// Output includes:
// {
//   "currentActive": 5,           // Active transactions
//   "currentInactive": 10,        // Inactive but open transactions
//   "currentOpen": 15,            // Total open transactions
//   "totalStarted": 1000,         // Lifetime started
//   "totalAborted": 50,           // Lifetime aborted
//   "totalCommitted": 950,        // Lifetime committed
//   "lastCommittedOpTime": {...}  // Last commit timestamp
// }
```

### **2. Current Operations View**
```javascript
// Monitor active transactions
const currentOps = await db.adminCommand({ currentOp: 1, $all: true });
const transactions = currentOps.inprog.filter(op => 
  op.planningTimeMillis && op.lsid // Transaction operations
);

transactions.forEach(txn => {
  console.log(`Transaction ID: ${txn.lsid.id}`);
  console.log(`Running for: ${txn.microsecs_running}µs`);
  console.log(`Operations: ${txn.op}`);
  console.log(`State: ${txn.transaction.state}`);
});
```

### **3. Oplog Monitoring**
```javascript
// Transaction entries in oplog
const oplog = db.getSiblingDB("local").oplog.rs;
const txnEntries = await oplog.find({
  "lsid": { $exists: true },          // Session ID present
  "txnNumber": { $exists: true }      // Transaction number present
}).sort({ ts: -1 }).limit(10).toArray();
```

---

## **Common Pitfalls & Solutions**

### **1. Stale Snapshot Reads**
```javascript
// Problem: Reading data that changes during transaction
session.startTransaction();
const user = await db.users.findOne({ _id: userId }, { session });
// ... long delay ...
const account = await db.accounts.findOne({ userId }, { session });
// Account may have changed since user was read

// Solution: Read all needed data early
session.startTransaction();
const [user, account] = await Promise.all([
  db.users.findOne({ _id: userId }, { session }),
  db.accounts.findOne({ userId }, { session })
]);
```

### **2. Deadlocks**
```javascript
// Problem: Transaction 1 locks A then B, Transaction 2 locks B then A
// Solution: Always acquire locks in consistent order
async function updateResources(resourceIds, session) {
  // Sort IDs to ensure consistent lock order
  const sortedIds = resourceIds.sort();
  
  for (const id of sortedIds) {
    await db.resources.updateOne(
      { _id: id },
      { $inc: { version: 1 } },
      { session }
    );
  }
}
```

### **3. Large Transactions**
```javascript
// Problem: Transaction exceeds 16MB or 1000 operations
// Solution: Batch processing pattern
async function processLargeBatch(data, batchSize = 100) {
  for (let i = 0; i < data.length; i += batchSize) {
    const batch = data.slice(i, i + batchSize);
    
    const session = client.startSession();
    try {
      session.startTransaction();
      
      for (const item of batch) {
        await processItem(item, session);
      }
      
      await session.commitTransaction();
    } catch (error) {
      await session.abortTransaction();
      // Log batch failure, continue with next batch
      console.error(`Batch ${i/batchSize} failed:`, error);
    } finally {
      session.endSession();
    }
  }
}
```

### **4. Missing Session Propagation**
```javascript
// Problem: Forgetting to pass session to all operations
async function updateOrder(orderId, updates, session) {
  // BAD: Missing session parameter
  await db.orders.updateOne({ _id: orderId }, updates);
  await db.orderHistory.insertOne({ orderId, updates }); // Not in transaction!
  
  // GOOD: All operations use session
  await db.orders.updateOne({ _id: orderId }, updates, { session });
  await db.orderHistory.insertOne({ orderId, updates }, { session });
}
```

---

## **Advanced Patterns**

### **1. Saga Pattern for Long-Running Processes**
```javascript
// Compensating transactions for multi-step processes
class OrderSaga {
  async createOrder(orderData) {
    const steps = [
      { execute: this.reserveInventory, compensate: this.releaseInventory },
      { execute: this.chargeCustomer, compensate: this.refundCustomer },
      { execute: this.shipOrder, compensate: this.cancelShipment }
    ];
    
    const executed = [];
    
    try {
      for (const step of steps) {
        await step.execute(orderData);
        executed.push(step);
      }
    } catch (error) {
      // Compensate in reverse order
      for (const step of executed.reverse()) {
        await step.compensate(orderData);
      }
      throw error;
    }
  }
}
```

### **2. Two-Phase Commit (Legacy Pattern)**
```javascript
// For MongoDB versions before 4.0
async function twoPhaseCommit(participants, coordinator) {
  // Phase 1: Prepare
  const prepared = await Promise.all(
    participants.map(p => p.prepare(coordinator))
  );
  
  if (prepared.every(p => p === "yes")) {
    // Phase 2: Commit
    await Promise.all(participants.map(p => p.commit(coordinator)));
  } else {
    // Phase 2: Abort
    await Promise.all(participants.map(p => p.abort(coordinator)));
  }
}
```

### **3. Change Streams with Transactions**
```javascript
// Watch for changes and process in transaction
const changeStream = db.collection.watch([
  { $match: { operationType: "insert" } }
]);

for await (const change of changeStream) {
  const session = client.startSession();
  try {
    session.startTransaction();
    
    // Process the change within transaction
    await processChange(change, session);
    
    // Record processing
    await db.processedChanges.insertOne({
      changeId: change._id,
      processedAt: new Date()
    }, { session });
    
    await session.commitTransaction();
  } catch (error) {
    await session.abortTransaction();
    console.error("Failed to process change:", error);
  } finally {
    session.endSession();
  }
}
```

---

## **Production Checklist**

### **Before Deployment:**
- [ ] Set appropriate `maxTransactionLockRequestTimeoutMillis` (default: 5ms)
- [ ] Configure `transactionLifetimeLimitSeconds` (default: 60)
- [ ] Enable sufficient oplog size for transaction volume
- [ ] Implement comprehensive monitoring and alerting
- [ ] Create indexes on all fields used in transaction queries
- [ ] Test with production-like data volumes

### **In Application Code:**
- [ ] Use connection pooling with session support
- [ ] Implement retry logic for transient errors
- [ ] Set reasonable timeouts per transaction
- [ ] Log transaction IDs for debugging
- [ ] Implement circuit breakers for transaction failures
- [ ] Use idempotency keys where appropriate

### **Monitoring:**
- [ ] Track transaction success/failure rates
- [ ] Monitor average transaction duration
- [ ] Alert on transaction timeouts
- [ ] Monitor lock contention metrics
- [ ] Track oplog growth rate

---

## **Summary**

MongoDB transactions provide strong consistency guarantees for complex operations across distributed data. Key takeaways:

1. **Use for critical operations** requiring atomicity across multiple documents
2. **Keep transactions short** to minimize contention and timeout risks
3. **Implement robust error handling** with retry logic for transient failures
4. **Monitor performance metrics** to identify bottlenecks
5. **Design idempotent operations** for reliable retries
6. **Test thoroughly** with production data patterns
7. **Consider alternatives** like embedded documents or application-level consistency for non-critical paths

Transactions are powerful but come with performance costs—use them judiciously where ACID properties are truly required.