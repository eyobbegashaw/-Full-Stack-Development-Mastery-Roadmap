# **Working with Databases in Node.js - Detailed Notes & Code**

## **Relational Databases**

### **Relational Databases**
Relational databases store data in structured tables with rows and columns, using SQL (Structured Query Language). They enforce relationships through foreign keys, support ACID transactions, and ensure data integrity. Popular options: PostgreSQL, MySQL, SQLite, MariaDB. Use when data structure is predictable and relationships are important.

```javascript
// SQL query example
const sql = 'SELECT * FROM users WHERE age > ? AND status = ?';
const results = await db.query(sql, [18, 'active']);
```

### **Native Drivers**
Native drivers provide low-level connectivity to databases using their specific protocols. Examples: `pg` (PostgreSQL), `mysql2`, `sqlite3`. They offer direct SQL query execution, connection pooling, and minimal abstraction. Use for maximum control, performance, or when ORMs don't support needed features.

```javascript
// PostgreSQL with pg driver
const { Pool } = require('pg');
const pool = new Pool({
  user: 'dbuser',
  host: 'localhost',
  database: 'mydb',
  password: 'password',
  port: 5432,
});

const result = await pool.query('SELECT * FROM users WHERE id = $1', [1]);
```

### **Sequelize**
Sequelize is a **promise-based Node.js ORM** for PostgreSQL, MySQL, MariaDB, SQLite, and SQL Server. Supports models, migrations, validations, associations, transactions, and raw queries. Feature-rich but can be heavy for simple projects. Good for complex SQL databases.

```javascript
const { Sequelize, DataTypes } = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres'
});

const User = sequelize.define('User', {
  username: { type: DataTypes.STRING, allowNull: false },
  email: { type: DataTypes.STRING, unique: true }
});

await User.sync();
const user = await User.create({ username: 'john', email: 'john@example.com' });
```

### **TypeORM**
TypeORM is an **ORM that supports both Active Record and Data Mapper patterns**, with TypeScript/JavaScript. Supports MySQL, PostgreSQL, SQLite, MongoDB, and more. Uses decorators for entity definition. Good for TypeScript projects and complex applications.

```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn } from 'typeorm';

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column({ unique: true })
  email: string;

  @CreateDateColumn()
  createdAt: Date;
}

// Usage
const userRepository = connection.getRepository(User);
const newUser = userRepository.create({ username: 'john', email: 'john@example.com' });
await userRepository.save(newUser);
```

### **Knex**
Knex is a **SQL query builder** for Node.js, not a full ORM. Supports PostgreSQL, MySQL, SQLite3, and Oracle. Provides chainable query building, migrations, and seeding. Lighter than ORMs, good for teams preferring SQL control with JavaScript syntax.

```javascript
const knex = require('knex')({
  client: 'pg',
  connection: {
    host: 'localhost',
    user: 'dbuser',
    password: 'password',
    database: 'mydb'
  }
});

// Query building
const users = await knex('users')
  .select('id', 'username', 'email')
  .where('age', '>', 18)
  .orderBy('created_at', 'desc')
  .limit(10);

// Raw SQL when needed
await knex.raw('SELECT * FROM users WHERE status = ?', ['active']);
```

### **Prisma (Relational)**
Prisma is a **next-generation ORM** with type-safe database access, auto-generated queries, and a schema file. Includes Prisma Client for queries, Prisma Migrate for schema changes, and Prisma Studio GUI. Excellent TypeScript support and developer experience.

```prisma
// schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  username  String   @unique
  email     String   @unique
  posts     Post[]
  createdAt DateTime @default(now())
}
```

```typescript
// Using Prisma Client
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

const user = await prisma.user.create({
  data: {
    username: 'john',
    email: 'john@example.com'
  },
  include: { posts: true }
});
```

### **Drizzle**
Drizzle is a **lightweight, type-safe ORM** focusing on TypeScript and SQL-like syntax. Minimal runtime overhead, close to SQL. Supports PostgreSQL, MySQL, SQLite. Schema is defined in TypeScript, not a separate DSL. Good for performance-critical applications.

```typescript
import { pgTable, serial, varchar, timestamp } from 'drizzle-orm/pg-core';
import { drizzle } from 'drizzle-orm/node-postgres';

const users = pgTable('users', {
  id: serial('id').primaryKey(),
  username: varchar('username', { length: 50 }).notNull(),
  email: varchar('email', { length: 100 }).unique().notNull(),
  createdAt: timestamp('created_at').defaultNow()
});

const db = drizzle(pool);
const result = await db.select().from(users).where(eq(users.id, 1));
```

## **NoSQL Databases**

### **NoSQL DBs**
NoSQL databases store unstructured or semi-structured data, often as JSON documents. Types: document (MongoDB), key-value (Redis), wide-column (Cassandra), graph (Neo4j). Scalable, flexible schema, good for unstructured data. Use when requirements change frequently or data is document-oriented.

```javascript
// MongoDB document example
{
  "_id": "507f1f77bcf86cd799439011",
  "username": "john",
  "email": "john@example.com",
  "preferences": {
    "theme": "dark",
    "notifications": true
  },
  "tags": ["user", "premium"]
}
```

### **Native Drivers (NoSQL)**
Native drivers connect directly to NoSQL databases. Examples: `mongodb` driver for MongoDB, `redis` for Redis. Provide low-level API, full database feature access, and maximum performance. Use for simple projects or when ORMs lack needed features.

```javascript
// MongoDB native driver
const { MongoClient } = require('mongodb');
const client = new MongoClient('mongodb://localhost:27017');

await client.connect();
const db = client.db('mydb');
const collection = db.collection('users');

const result = await collection.insertOne({
  username: 'john',
  email: 'john@example.com'
});
```

### **Mongoose**
Mongoose is an **ODM (Object Document Mapper)** for MongoDB and Node.js. Provides schema validation, middleware, population (joins), and query building. Enforces structure on MongoDB's flexible documents. Most popular MongoDB ODM.

```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/mydb');

const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 18 },
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);

const user = new User({
  username: 'john',
  email: 'john@example.com',
  age: 25
});
await user.save();

// Query with population
const userWithPosts = await User.findById(userId).populate('posts');
```

### **Prisma (NoSQL)**
Prisma supports **MongoDB as a NoSQL database** alongside relational databases. Uses same Prisma Client API. MongoDB support includes embedded documents, relations, and migrations. Provides type safety for MongoDB operations.

```prisma
// schema.prisma with MongoDB
datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  username  String   @unique
  email     String   @unique
  profile   Profile?
  posts     Post[]
}

model Profile {
  id     String @id @default(auto()) @map("_id") @db.ObjectId
  bio    String?
  avatar String?
  user   User   @relation(fields: [userId], references: [id])
  userId String @unique @db.ObjectId
}
```

```typescript
// Using Prisma with MongoDB
const user = await prisma.user.create({
  data: {
    username: 'john',
    email: 'john@example.com',
    profile: {
      create: {
        bio: 'Software developer',
        avatar: 'avatar.jpg'
      }
    }
  },
  include: { profile: true }
});
```

## **Comparison & Decision Guide**

### **ORM/ODM Comparison**
```javascript
// Raw SQL with pg driver
const result = await pool.query('SELECT * FROM users WHERE id = $1', [1]);

// Knex query builder
const result = await knex('users').where('id', 1).first();

// Sequelize ORM
const result = await User.findByPk(1);

// TypeORM
const result = await userRepository.findOne({ where: { id: 1 } });

// Prisma
const result = await prisma.user.findUnique({ where: { id: 1 } });

// Mongoose ODM
const result = await User.findById(1);
```

### **When to Use What**
1. **Simple projects, raw SQL comfort**: Native drivers or Knex
2. **Complex relational data, migrations needed**: Sequelize or TypeORM
3. **TypeScript focus, developer experience**: Prisma or TypeORM
4. **MongoDB with schema enforcement**: Mongoose
5. **Performance critical, close to SQL**: Drizzle or Knex
6. **Multiple database support**: Prisma or TypeORM
7. **Simple MongoDB access**: Native MongoDB driver

## **Complete Database Examples**

### **PostgreSQL with Prisma**
```javascript
// package.json scripts
"scripts": {
  "migrate": "prisma migrate dev",
  "studio": "prisma studio",
  "generate": "prisma generate"
}

// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
}
```

```typescript
// app.ts
import { PrismaClient } from '@prisma/client';
const prisma = new PrismaClient();

async function main() {
  // Create user with posts
  const user = await prisma.user.create({
    data: {
      email: 'alice@example.com',
      name: 'Alice',
      posts: {
        create: [
          { title: 'Hello World', content: 'My first post' },
          { title: 'Second Post', content: 'Another post' }
        ]
      }
    },
    include: { posts: true }
  });

  // Complex query
  const posts = await prisma.post.findMany({
    where: {
      OR: [
        { title: { contains: 'hello', mode: 'insensitive' } },
        { content: { contains: 'world', mode: 'insensitive' } }
      ],
      published: true
    },
    include: { author: true },
    orderBy: { createdAt: 'desc' },
    take: 10
  });
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### **MongoDB with Mongoose**
```javascript
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: [true, 'Username is required'],
    unique: true,
    trim: true,
    minlength: 3
  },
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email']
  },
  password: {
    type: String,
    required: [true, 'Password is required'],
    minlength: 6,
    select: false // Don't return in queries by default
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  profile: {
    bio: String,
    avatar: String,
    location: String
  },
  settings: {
    theme: { type: String, default: 'light' },
    notifications: { type: Boolean, default: true }
  },
  lastLogin: Date,
  isActive: { type: Boolean, default: true }
}, {
  timestamps: true // Adds createdAt and updatedAt
});

// Pre-save middleware for password hashing
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Instance method for password verification
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// Static method for finding active users
userSchema.statics.findActive = function() {
  return this.find({ isActive: true });
};

module.exports = mongoose.model('User', userSchema);
```

```javascript
// database connection and usage
const mongoose = require('mongoose');
const User = require('./models/User');

// Connection with options
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  maxPoolSize: 10, // Connection pool size
  serverSelectionTimeoutMS: 5000
});

const db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));
db.once('open', () => console.log('Connected to MongoDB'));

// Usage examples
async function createUser() {
  const user = await User.create({
    username: 'john_doe',
    email: 'john@example.com',
    password: 'secure123',
    profile: {
      bio: 'Software developer',
      location: 'San Francisco'
    }
  });
  return user;
}

async function authenticateUser(username, password) {
  const user = await User.findOne({ username }).select('+password');
  if (!user || !await user.comparePassword(password)) {
    throw new Error('Invalid credentials');
  }
  user.lastLogin = new Date();
  await user.save();
  return user;
}

async function getUsersWithPagination(page = 1, limit = 10) {
  const skip = (page - 1) * limit;
  return await User.find({ isActive: true })
    .sort({ createdAt: -1 })
    .skip(skip)
    .limit(limit)
    .select('-password'); // Exclude password field
}
```

### **PostgreSQL with Knex and Objection.js**
```javascript
// database/knex.js
const knex = require('knex');
const { Model } = require('objection');

const knexConfig = {
  client: 'pg',
  connection: {
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    port: process.env.DB_PORT
  },
  pool: {
    min: 2,
    max: 10
  },
  migrations: {
    tableName: 'knex_migrations',
    directory: './database/migrations'
  },
  seeds: {
    directory: './database/seeds'
  }
};

const db = knex(knexConfig);
Model.knex(db); // Bind Objection models to knex instance

module.exports = { db, Model };
```

```javascript
// models/User.js with Objection.js
const { Model } = require('objection');

class User extends Model {
  static get tableName() {
    return 'users';
  }

  static get jsonSchema() {
    return {
      type: 'object',
      required: ['username', 'email'],
      properties: {
        id: { type: 'integer' },
        username: { type: 'string', minLength: 1, maxLength: 255 },
        email: { type: 'string', format: 'email', maxLength: 255 },
        age: { type: 'integer', minimum: 0 },
        metadata: { type: 'object' }
      }
    };
  }

  static get relationMappings() {
    const Post = require('./Post');
    return {
      posts: {
        relation: Model.HasManyRelation,
        modelClass: Post,
        join: {
          from: 'users.id',
          to: 'posts.user_id'
        }
      }
    };
  }
}

module.exports = User;
```

## **Best Practices**

1. **Use connection pooling** for database connections
2. **Always handle database errors** gracefully
3. **Use environment variables** for database credentials
4. **Implement database migrations** for schema changes
5. **Use transactions** for atomic operations
6. **Optimize queries** with indexes and proper joins
7. **Sanitize user inputs** to prevent SQL injection (ORMs help)
8. **Monitor database performance** with query logging
9. **Implement retry logic** for transient failures
10. **Use proper data types** and constraints in schemas

```javascript
// Transaction example
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({ data: userData });
  await tx.account.create({ 
    data: { userId: user.id, balance: 0 } 
  });
  await tx.auditLog.create({
    data: { action: 'USER_CREATED', userId: user.id }
  });
});

// Connection pooling with pg
const pool = new Pool({
  max: 20, // Maximum connections
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

These tools provide various levels of abstraction for working with databases in Node.js, from raw SQL control to high-level ORM convenience. Choose based on project requirements, team expertise, and performance needs.