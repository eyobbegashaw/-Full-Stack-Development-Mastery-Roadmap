# **MongoDB Scalability, Security & Advanced Topics - Comprehensive Guide**

## **High Availability & Scalability**

### **1. Replica Sets**

**Notes:** Replica sets are MongoDB's solution for high availability and data redundancy. A replica set consists of 3+ mongod instances (nodes) that maintain the same data: one primary (handles all writes) and secondaries (read replicas). Automatic failover occurs if primary fails. Supports read scaling through secondary reads (with eventual consistency). Configurable with arbitration nodes (voting only), hidden nodes (for reporting/backup), and delayed nodes (for recovery). Oplog (operations log) replicates writes from primary to secondaries.

**Code:**
```javascript
// Initialize a replica set
// 1. Start mongod instances with replSet option
// mongod --replSet "rs0" --port 27017 --dbpath /data/db1
// mongod --replSet "rs0" --port 27018 --dbpath /data/db2
// mongod --replSet "rs0" --port 27019 --dbpath /data/db3

// 2. Connect to primary and initiate
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "localhost:27017" },
    { _id: 1, host: "localhost:27018" },
    { _id: 2, host: "localhost:27019", arbiterOnly: true }
  ]
});

// Replica set status
rs.status();

// Add/remove members
rs.add("localhost:27020");
rs.remove("localhost:27017");

// Configure member priorities (0-1000)
rs.reconfig({
  _id: "rs0",
  version: 2,
  members: [
    { _id: 0, host: "db1.example.com:27017", priority: 2 },
    { _id: 1, host: "db2.example.com:27017", priority: 1 },
    { _id: 2, host: "db3.example.com:27017", priority: 0.5, hidden: true }
  ]
});

// Force reconfiguration (when majority unavailable)
rs.reconfig(config, { force: true });

// Read from secondaries (eventually consistent)
const client = new MongoClient("mongodb://localhost:27017,localhost:27018/?replicaSet=rs0", {
  readPreference: 'secondary',
  maxStalenessSeconds: 120 // Max data staleness tolerance
});
```

---

### **2. Sharded Clusters**

**Notes:** Sharding horizontally partitions data across multiple servers (shards) to support massive datasets and high throughput. Components: **shards** (store data), **config servers** (store cluster metadata), **mongos routers** (route queries). Shard key determines data distribution. Sharding strategies: range-based (contiguous ranges), hash-based (even distribution), zone-based (custom ranges). Requires careful shard key selection to avoid hotspots and ensure scalability.

**Code:**
```javascript
// Sharded cluster setup (simplified)
// 1. Start config servers (3 for production)
// mongod --configsvr --replSet configRS --port 27019 --dbpath /data/configdb

// 2. Start mongos router
// mongos --configdb configRS/localhost:27019 --port 27017

// 3. Add shards
sh.addShard("shard1/localhost:27020");
sh.addShard("shard2/localhost:27021");

// Enable sharding for database
sh.enableSharding("mydb");

// Shard a collection with shard key
sh.shardCollection("mydb.orders", { customerId: 1, orderDate: 1 });

// Hash-based sharding for even distribution
sh.shardCollection("mydb.logs", { _id: "hashed" });

// Zone sharding for geographic distribution
sh.addShardToZone("shard1", "US-East");
sh.addShardToZone("shard2", "EU-West");
sh.updateZoneKeyRange(
  "mydb.customers",
  { region: "US", _id: MinKey },
  { region: "US", _id: MaxKey },
  "US-East"
);

// Monitor sharding status
sh.status();
db.orders.getShardDistribution();

// Move chunks between shards (rebalancing)
// Automatic by default, can be manual
db.adminCommand({ moveChunk: "mydb.orders", find: { customerId: 1000 }, to: "shard2" });

// Split chunks manually
db.adminCommand({ split: "mydb.orders", middle: { customerId: 5000 } });
```

---

### **3. Scaling MongoDB**

**Notes:** MongoDB scales vertically (more resources per server) and horizontally (more servers). **Read scaling**: replica set secondaries, read preferences. **Write scaling**: sharding. **Storage scaling**: compression, archiving, tiered storage. **Memory scaling**: working set optimization, SSDs for storage. Consider costs, operational complexity, and application requirements when scaling.

**Code:**
```javascript
// Read scaling configuration
const client = new MongoClient(uri, {
  readPreference: 'secondaryPreferred', // Read from secondaries if available
  readPreferenceTags: [ { region: "east" } ], // Prefer specific secondaries
  maxStalenessSeconds: 90,
  hedgeOptions: { enabled: true } // Send reads to multiple secondaries, use fastest
});

// Write concern for durability vs performance tradeoff
await collection.insertOne(doc, {
  writeConcern: { 
    w: 1, // Fastest (primary only)
    // w: "majority", // More durable
    // w: 3, // Specific number of nodes
    j: true // Journal durability
  }
});

// Connection pooling for performance
const client = new MongoClient(uri, {
  maxPoolSize: 100,     // Maximum connections
  minPoolSize: 10,      // Minimum connections
  maxIdleTimeMS: 30000, // Close idle connections after 30s
  waitQueueTimeoutMS: 5000 // Connection wait timeout
});

// Monitor scaling metrics
const serverStatus = await db.adminCommand({ serverStatus: 1 });
console.log({
  connections: serverStatus.connections,
  network: serverStatus.network,
  opcounters: serverStatus.opcounters,
  wiredTiger: serverStatus.wiredTiger
});
```

---

### **4. Tuning Configuration**

**Notes:** MongoDB configuration tuning optimizes performance for specific workloads. Key areas: **storage engine** (WiredTiger defaults), **cache size** (50% of RAM minus 1GB), **journaling**, **query optimizer**, **oplog size**. Use MongoDB Configuration File (YAML format) for production settings. Monitor and adjust based on workload patterns.

**Code:**
```yaml
# mongod.conf - Production configuration
systemLog:
  destination: file
  path: "/var/log/mongodb/mongod.log"
  logAppend: true
  verbosity: 0 # 0-5, higher = more verbose

storage:
  dbPath: "/var/lib/mongodb"
  journal:
    enabled: true
    commitIntervalMs: 100
  engine: "wiredTiger"
  wiredTiger:
    engineConfig:
      cacheSizeGB: 8 # Adjust based on RAM: (RAM - 1GB) * 0.5
      journalCompressor: "snappy"
      directoryForIndexes: true # Separate index files
    collectionConfig:
      blockCompressor: "zstd" # zstd, snappy, zlib, none
    indexConfig:
      prefixCompression: true

operationProfiling:
  mode: "slowOp" # off, slowOp, all
  slowOpThresholdMs: 100
  rateLimit: 100 # ops per second to sample

replication:
  oplogSizeMB: 10240 # 10GB oplog (adjust based on write volume)
  replSetName: "rs0"

net:
  port: 27017
  bindIp: "127.0.0.1,192.168.1.100" # Specific IPs
  maxIncomingConnections: 1000

security:
  authorization: "enabled"
  keyFile: "/etc/mongodb/keyfile"
```

```javascript
// Runtime configuration adjustments
// Set cache size
db.adminCommand({
  setParameter: 1,
  wiredTigerEngineRuntimeConfig: "cache_size=8G"
});

// Adjust oplog size (requires restart)
db.adminCommand({ replSetResizeOplog: 1, size: 10240 });

// Configure query optimizer
db.adminCommand({
  setParameter: 1,
  internalQueryExecMaxBlockingSortBytes: 100*1024*1024, // 100MB
  internalQueryMaxBlockingSortMemoryUsageBytes: 200*1024*1024
});

// TTL monitor interval
db.adminCommand({
  setParameter: 1,
  ttlMonitorSleepSecs: 60
});
```

---

## **Security**

### **1. Role-based Access Control (RBAC)**

**Notes:** MongoDB RBAC provides fine-grained access control through roles (collections of privileges) assigned to users. Built-in roles: read, readWrite, dbAdmin, userAdmin, clusterAdmin, backup, restore. Custom roles for specific needs. Principle of least privilege: grant minimum necessary permissions. Roles are database-specific except cluster roles.

**Code:**
```javascript
// Create user with built-in role
db.createUser({
  user: "appUser",
  pwd: "securePassword123",
  roles: [
    { role: "readWrite", db: "appdb" },
    { role: "read", db: "reporting" }
  ]
});

// Create custom role
db.createRole({
  role: "dataAnalyst",
  privileges: [
    {
      resource: { db: "analytics", collection: "events" },
      actions: ["find", "insert", "update", "remove"]
    },
    {
      resource: { db: "analytics", collection: "" },
      actions: ["listCollections", "listIndexes"]
    }
  ],
  roles: [] // Can inherit other roles
});

// Assign custom role to user
db.grantRolesToUser("analyst1", ["dataAnalyst"]);

// Role management
db.updateRole("dataAnalyst", {
  privileges: [
    {
      resource: { db: "analytics", collection: "metrics" },
      actions: ["find"]
    }
  ]
});

db.revokeRolesFromUser("analyst1", ["dataAnalyst"]);

// View roles and privileges
db.getRole("dataAnalyst", { showPrivileges: true });
db.getUser("appUser", { showPrivileges: true });

// Enable authentication (in mongod.conf)
security:
  authorization: "enabled"
```

---

### **2. X.509 Certificate Authentication**

**Notes:** X.509 uses SSL/TLS certificates for authentication instead of passwords. Requires TLS/SSL configuration. Certificates must be signed by trusted CA. Users are identified by certificate subject or subjectAlternativeName. More secure than password-based auth, suitable for server-to-server communication.

**Code:**
```javascript
// MongoDB configuration for X.509
security:
  authorization: "enabled"
  clusterAuthMode: "x509"

net:
  ssl:
    mode: "requireSSL"
    PEMKeyFile: "/etc/ssl/mongodb.pem"
    CAFile: "/etc/ssl/ca.pem"
    allowConnectionsWithoutCertificates: false
    allowInvalidHostnames: true # For testing only

// Create user mapped to certificate
db.getSiblingDB("$external").runCommand({
  createUser: "CN=client1,OU=Dev,O=Company,L=City,ST=State,C=US",
  roles: [
    { role: "readWrite", db: "appdb" }
  ]
});

// Client connection with certificate
const client = new MongoClient(
  "mongodb://server.example.com:27017/?ssl=true&authMechanism=MONGODB-X509",
  {
    tlsCertificateKeyFile: "/path/to/client.pem",
    tlsCAFile: "/path/to/ca.pem"
  }
);

// For replica sets
const client = new MongoClient(
  "mongodb://server1:27017,server2:27017/?replicaSet=rs0&ssl=true&authMechanism=MONGODB-X509"
);
```

---

### **3. Kerberos Authentication**

**Notes:** Kerberos provides enterprise-grade single sign-on authentication using tickets. Integrates with Active Directory, MIT Kerberos. Requires Kerberos infrastructure. MongoDB acts as service principal. Users authenticate with Kerberos tickets instead of passwords.

**Code:**
```yaml
# mongod.conf for Kerberos
security:
  authorization: "enabled"
  sasl:
    hostName: "mongo.server.com"
    serviceName: "mongodb"

# Create keytab file
# ktutil: add_entry -password -p mongodb/mongo.server.com@REALM.COM -k 1 -e aes256-cts-hmac-sha1-96

# Start mongod with keytab
# mongod --config /etc/mongod.conf --setParameter saslHostName=mongo.server.com \
#   --setParameter authenticationMechanisms=GSSAPI
```

```javascript
// Create user in $external database
use $external
db.createUser({
  user: "username@REALM.COM",
  roles: [
    { role: "readWrite", db: "appdb" }
  ]
});

// Client connection with Kerberos
const { MongoClient } = require('mongodb');

const client = new MongoClient(
  "mongodb://mongo.server.com:27017/?authMechanism=GSSAPI&authMechanismProperties=SERVICE_NAME:mongodb",
  {
    auth: {
      username: "username@REALM.COM"
    }
  }
);
```

---

### **4. LDAP Proxy Authentication**

**Notes:** LDAP authentication delegates authentication to LDAP directory services (Active Directory, OpenLDAP). MongoDB binds to LDAP server to validate credentials. Supports proxy and delegated modes. Requires careful configuration of user-to-DN mapping and authorization.

**Code:**
```yaml
# mongod.conf for LDAP
security:
  authorization: "enabled"
  ldap:
    servers: "ldap.example.com:636"
    transportSecurity: tls
    bind:
      method: "simple"
      queryUser: "CN=admin,DC=example,DC=com"
      queryPassword: "ldapPassword"
    userToDNMapping: |
      [
        {
          match: "(.+)",
          ldapQuery: "DC=example,DC=com??sub?(userPrincipalName={0})"
        }
      ]
    authz:
      queryTemplate: "DC=example,DC=com??sub?(&(objectClass=group)(member={PROVIDED_USER}))"
setParameter:
  authenticationMechanisms: "PLAIN"
```

```javascript
// Client connection with LDAP
const client = new MongoClient(
  "mongodb://username:password@server.example.com:27017/?authMechanism=PLAIN&authSource=$external"
);
```

---

### **5. Encryption at Rest**

**Notes:** Encryption at rest protects data on disk using storage engine encryption (WiredTiger) or filesystem encryption. WiredTiger native encryption supports AES256-CBC and AES256-GCM. Requires encryption key management via KMIP or local keyfile. Transparent to applications.

**Code:**
```yaml
# mongod.conf for encryption at rest
security:
  enableEncryption: true
  encryptionCipherMode: "AES256-GCM" # or "AES256-CBC"
  encryptionKeyFile: "/etc/mongodb/keyfile"
  # For KMIP:
  kmip:
    serverName: "kmip.server.com"
    port: 5696
    clientCertificateFile: "/etc/ssl/mongodb.pem"
    clientCertificatePassword: "password"
    serverCAFile: "/etc/ssl/ca.pem"

storage:
  wiredTiger:
    engineConfig:
      encryption:
        name: "kmip" # or "local"
        keyId: "keyIdentifier" # For key rotation
```

```bash
# Generate encryption keyfile
openssl rand -base64 32 > /etc/mongodb/keyfile
chmod 600 /etc/mongodb/keyfile
chown mongodb:mongodb /etc/mongodb/keyfile

# Rotate encryption key (without downtime)
# 1. Add new key to keyfile or KMIP
# 2. Rotate master key
db.adminCommand({ rotateEncryptionKey: 1 });
```

---

### **6. Queryable Encryption**

**Notes:** Queryable Encryption (MongoDB 7.0+) enables encrypted queries on sensitive fields while keeping data encrypted end-to-end. Supports equality queries on encrypted fields. Keys managed by client-side Key Management System (KMS). Protects data from database administrators and infrastructure providers.

**Code:**
```javascript
// Client-side field level encryption with queryable encryption
const { MongoClient, AutoEncryption } = require('mongodb');

const kmsProviders = {
  aws: {
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_KEY
  }
};

const schemaMap = {
  "medical.records": {
    "bsonType": "object",
    "encryptMetadata": {
      "keyId": "/key-id"
    },
    "properties": {
      "patientId": {
        "encrypt": {
          "bsonType": "string",
          "algorithm": "AEAD_AES_256_CBC_HMAC_SHA_512-Random",
          "keyId": [UUID("...")]
        }
      },
      "ssn": {
        "encrypt": {
          "bsonType": "string",
          "algorithm": "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic" // Queryable
        }
      },
      "medicalHistory": {
        "encrypt": {
          "bsonType": "string",
          "algorithm": "AEAD_AES_256_CBC_HMAC_SHA_512-Random"
        }
      }
    }
  }
};

const client = new MongoClient(uri, {
  autoEncryption: {
    keyVaultNamespace: "encryption.__keyVault",
    kmsProviders,
    schemaMap
  }
});

// Query on encrypted field (equality only)
const result = await db.medical.records.find({
  ssn: "123-45-6789" // Encrypted query
});
```

---

### **7. Client-Side Field Level Encryption**

**Notes:** Client-side field level encryption (CSFLE) encrypts specific fields in application before sending to MongoDB. Different from queryable encryption (QE) - QE supports queries, CSFLE doesn't. Uses deterministic or randomized encryption. Keys managed in Key Vault collection. Protects sensitive data like PII, PHI, financial info.

**Code:**
```javascript
// Client-side field level encryption setup
const { ClientEncryption } = require('mongodb-client-encryption');

// Create data encryption keys
const encryption = new ClientEncryption(client, {
  keyVaultNamespace: "encryption.__keyVault",
  kmsProviders: {
    local: { key: Buffer.from(process.env.LOCAL_KEY, 'base64') }
  }
});

// Create DEK (Data Encryption Key)
const dekId = await encryption.createDataKey("local");

// Manual encryption/decryption
const encryptedSSN = await encryption.encrypt("123-45-6789", {
  keyId: dekId,
  algorithm: "AEAD_AES_256_CBC_HMAC_SHA_512-Deterministic"
});

const decryptedSSN = await encryption.decrypt(encryptedSSN);

// Automatic encryption with MongoClient
const client = new MongoClient(uri, {
  autoEncryption: {
    keyVaultNamespace: "encryption.__keyVault",
    kmsProviders,
    bypassAutoEncryption: false
  }
});
```

---

### **8. TLS/SSL Encryption**

**Notes:** TLS/SSL encrypts network traffic between clients and MongoDB, and between cluster members. Required for production. MongoDB supports TLS 1.2+. Configuration includes certificates, cipher suites, and protocol versions. Mutual TLS (mTLS) authenticates both client and server.

**Code:**
```yaml
# mongod.conf TLS/SSL configuration
net:
  ssl:
    mode: "requireSSL" # requireSSL, allowSSL, disabled
    PEMKeyFile: "/etc/ssl/mongodb.pem"
    PEMKeyPassword: "password" # If key is encrypted
    CAFile: "/etc/ssl/ca.pem"
    CRLFile: "/etc/ssl/crl.pem" # Certificate Revocation List
    allowConnectionsWithoutCertificates: false
    allowInvalidHostnames: false # Validate hostnames
    disabledProtocols: "TLS1_0,TLS1_1" # Disable weak protocols
    cipherSuite: "HIGH:!EXPORT:!aNULL@STRENGTH" # Strong ciphers only

# mongos and config servers need similar config
```

```javascript
// Client connection with TLS
const fs = require('fs');

const client = new MongoClient(uri, {
  tls: true,
  tlsCertificateKeyFile: "/path/to/client.pem",
  tlsCAFile: "/path/to/ca.pem",
  tlsAllowInvalidCertificates: false, // Production: false
  tlsAllowInvalidHostnames: false,    // Production: false
  tlsInsecure: false                  // NEVER true in production
});

// For replica set with TLS
const client = new MongoClient(
  "mongodb://host1:27017,host2:27017,host3:27017/?replicaSet=rs0&tls=true"
);
```

---

### **9. MongoDB Audit**

**Notes:** MongoDB audit log records authentication, authorization, and schema operations for compliance (HIPAA, PCI DSS, GDPR). Configurable to log specific events, users, or collections. Output to console, file, syslog, or Windows Event Log. Performance impact varies with verbosity.

**Code:**
```yaml
# mongod.conf audit configuration
auditLog:
  destination: "file" # file, syslog, console
  path: "/var/log/mongodb/audit.log"
  format: "JSON" # JSON or BSON
  filter: '{ "users": { $elemMatch: { "user": "admin", "db": "admin" } } }'

# OR filter specific operations
auditLog:
  destination: "syslog"
  filter:
    '{
      "atype": { "$in": ["authenticate", "createUser", "dropUser"] },
      "param.ns": { "$ne": "config.system.sessions" }
    }'
```

```javascript
// Enable audit at runtime
db.adminCommand({
  setParameter: 1,
  auditAuthorizationSuccess: true
});

// View audit configuration
db.adminCommand({ getParameter: 1, auditAuthorizationSuccess: 1 });

// Common audit filters
const filter = {
  atype: { 
    $in: [
      "authCheck",          // Authorization checks
      "createCollection",   // Schema operations
      "dropCollection",
      "createIndex",
      "dropIndex",
      "createUser",
      "dropUser",
      "authenticate",       // Authentication
      "shardCollection"     // Sharding operations
    ]
  },
  "param.db": { $ne: "admin" } // Exclude admin operations
};

// Rotate audit log
db.adminCommand({ logRotate: 1 });
```

---

## **Advanced Security Configuration Example**

**Code:**
```yaml
# Complete production security configuration
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
  verbosity: 0

processManagement:
  fork: true
  pidFilePath: /var/run/mongodb/mongod.pid

net:
  port: 27017
  bindIp: 10.0.1.100,10.0.1.101 # Specific IPs only
  maxIncomingConnections: 1000
  compression:
    compressors: snappy,zstd,zlib
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/mongodb/ssl/mongod.pem
    CAFile: /etc/mongodb/ssl/ca.pem
    disabledProtocols: TLS1_0,TLS1_1
    cipherSuites: "TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256"
    allowInvalidCertificates: false
    allowInvalidHostnames: false

security:
  authorization: enabled
  keyFile: /etc/mongodb/keyfile
  javascriptEnabled: false # Disable server-side JS
  sasl:
    hostName: mongodb.prod.example.com
    serviceName: mongodb
  ldap:
    servers: ldaps://ldap.prod.example.com:636
    transportSecurity: tls
    bind:
      method: simple
      queryUser: CN=mongo-auth,OU=ServiceAccounts,DC=prod,DC=example,DC=com
      queryPassword: ENCRYPTED_PASSWORD
  enableEncryption: true
  encryptionCipherMode: AES256-GCM
  kmip:
    serverName: kmip.prod.example.com
    port: 5696
    clientCertificateFile: /etc/mongodb/ssl/client.pem
    serverCAFile: /etc/mongodb/ssl/kmip-ca.pem

storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 32
      journalCompressor: zstd
      encryption:
        name: kmip
    collectionConfig:
      blockCompressor: zstd

auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.json
  filter: '{ "atype": { "$in": ["authenticate", "authCheck", "createUser", "dropUser"] } }'

replication:
  replSetName: rsProd
  oplogSizeMB: 20480

setParameter:
  ttlMonitorEnabled: true
  ttlMonitorSleepSecs: 60
  cursorTimeoutMillis: 600000
  notablescan: true # Reject queries without indexes
  enableLocalhostAuthBypass: false
```

---

## **Security Monitoring & Compliance Script**

**Code:**
```javascript
// Security compliance check script
async function securityAudit() {
  const report = {
    timestamp: new Date(),
    findings: []
  };

  // 1. Check authentication
  const authStatus = await db.adminCommand({ getParameter: 1, auth: 1 });
  if (!authStatus.auth) {
    report.findings.push({ severity: "HIGH", issue: "Authentication disabled" });
  }

  // 2. Check users and roles
  const users = await db.system.users.find().toArray();
  const adminUsers = users.filter(u => u.db === "admin");
  
  adminUsers.forEach(user => {
    if (user.user === "root" || user.user === "admin") {
      report.findings.push({ 
        severity: "MEDIUM", 
        issue: "Default admin user found", 
        user: user.user 
      });
    }
  });

  // 3. Check network exposure
  const config = await db.adminCommand({ getCmdLineOpts: 1 });
  const bindIp = config.parsed.net?.bindIp;
  if (bindIp === "0.0.0.0") {
    report.findings.push({ 
      severity: "HIGH", 
      issue: "Binding to all interfaces (0.0.0.0)" 
    });
  }

  // 4. Check TLS/SSL
  const tlsStatus = await db.adminCommand({ getParameter: 1, sslMode: 1 });
  if (tlsStatus.sslMode !== "requireSSL") {
    report.findings.push({ 
      severity: "HIGH", 
      issue: "TLS not required", 
      currentMode: tlsStatus.sslMode 
    });
  }

  // 5. Check encryption at rest
  const encryptionStatus = await db.adminCommand({ 
    getParameter: 1, 
    enableEncryption: 1 
  });
  if (!encryptionStatus.enableEncryption) {
    report.findings.push({ severity: "HIGH", issue: "Encryption at rest disabled" });
  }

  // 6. Check audit logging
  const auditStatus = await db.adminCommand({ getParameter: 1, auditAuthorizationSuccess: 1 });
  if (!auditStatus.auditAuthorizationSuccess) {
    report.findings.push({ severity: "MEDIUM", issue: "Audit logging not enabled for auth success" });
  }

  // 7. Check for default ports
  const port = config.parsed.net?.port;
  if (port === 27017) {
    report.findings.push({ 
      severity: "LOW", 
      issue: "Using default MongoDB port", 
      suggestion: "Consider changing default port" 
    });
  }

  return report;
}

// Run compliance check
async function main() {
  try {
    const auditReport = await securityAudit();
    console.log(JSON.stringify(auditReport, null, 2));
    
    // Generate compliance score
    const score = calculateComplianceScore(auditReport.findings);
    console.log(`Security Compliance Score: ${score}%`);
    
    // Alert on critical findings
    const critical = auditReport.findings.filter(f => f.severity === "HIGH");
    if (critical.length > 0) {
      console.error(`CRITICAL: ${critical.length} high-severity findings`);
      // Send alert via email/Slack/etc.
    }
  } catch (error) {
    console.error("Audit failed:", error);
  }
}
```

---

## **Summary**

MongoDB provides comprehensive scalability and security features:

### **Scalability:**
- **Replica Sets**: High availability, automatic failover, read scaling
- **Sharded Clusters**: Horizontal scaling for massive datasets
- **Configuration Tuning**: Optimize for specific workloads
- **Connection Management**: Pooling, timeouts, read preferences

### **Security:**
- **RBAC**: Fine-grained access control with custom roles
- **Authentication**: Multiple mechanisms (X.509, Kerberos, LDAP)
- **Encryption**: TLS in transit, encryption at rest, field-level encryption
- **Auditing**: Compliance logging for regulatory requirements
- **Network Security**: IP binding, firewall rules, VPN integration

### **Best Practices:**
1. Always enable authentication and authorization
2. Use TLS/SSL for all network traffic
3. Implement principle of least privilege (RBAC)
4. Regular security audits and compliance checks
5. Monitor logs for suspicious activity
6. Keep MongoDB and drivers updated
7. Use encryption at rest for sensitive data
8. Implement backup and disaster recovery plans
9. Regular penetration testing and vulnerability assessments
10. Document security policies and procedures

MongoDB's security features, when properly configured, meet enterprise security requirements and regulatory compliance standards including HIPAA, PCI DSS, GDPR, and SOC 2.