I'll show you a complete, real-world **CRUD API** for a task management application, including authentication, validation, error handling, and best practices.

## 🏗️ Complete Backend Structure

```
task-manager-api/
├── config/
│   ├── database.js      # DB connection
│   └── cloudinary.js    # File upload config
├── models/
│   ├── User.js          # User model
│   ├── Task.js          # Task model
│   └── Comment.js       # Comment model (for relations)
├── middleware/
│   ├── auth.js          # Authentication
│   ├── upload.js        # File upload
│   └── validation.js    # Request validation
├── controllers/
│   ├── authController.js
│   ├── taskController.js
│   └── userController.js
├── routes/
│   ├── authRoutes.js
│   ├── taskRoutes.js
│   └── userRoutes.js
├── utils/
│   ├── AppError.js
│   ├── catchAsync.js
│   └── emailService.js
├── server.js
└── .env
```

## 📦 1. Database Models

**User Model** (`models/User.js`):
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Please provide a name'],
    trim: true,
    maxlength: [50, 'Name cannot exceed 50 characters']
  },
  email: {
    type: String,
    required: [true, 'Please provide an email'],
    unique: true,
    lowercase: true,
    match: [
      /^\w+([\.-]?\w+)*@\w+([\.-]?\w+)*(\.\w{2,3})+$/,
      'Please provide a valid email'
    ]
  },
  password: {
    type: String,
    required: [true, 'Please provide a password'],
    minlength: [6, 'Password must be at least 6 characters'],
    select: false // Don't return password in queries
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  avatar: {
    public_id: String,
    url: String
  },
  resetPasswordToken: String,
  resetPasswordExpire: Date,
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// Hash password before saving
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 10);
});

// Compare password method
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// Generate JWT token
userSchema.methods.generateAuthToken = function() {
  return jwt.sign(
    { id: this._id, role: this.role },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRE }
  );
};

// Generate password reset token
userSchema.methods.generateResetPasswordToken = function() {
  const resetToken = crypto.randomBytes(20).toString('hex');
  this.resetPasswordToken = crypto
    .createHash('sha256')
    .update(resetToken)
    .digest('hex');
  this.resetPasswordExpire = Date.now() + 10 * 60 * 1000; // 10 minutes
  return resetToken;
};

module.exports = mongoose.model('User', userSchema);
```

**Task Model** (`models/Task.js`):
```javascript
const mongoose = require('mongoose');

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, 'Please provide a task title'],
    trim: true,
    maxlength: [200, 'Title cannot exceed 200 characters']
  },
  description: {
    type: String,
    maxlength: [1000, 'Description cannot exceed 1000 characters']
  },
  status: {
    type: String,
    enum: ['todo', 'in-progress', 'review', 'completed'],
    default: 'todo'
  },
  priority: {
    type: String,
    enum: ['low', 'medium', 'high', 'critical'],
    default: 'medium'
  },
  dueDate: {
    type: Date,
    validate: {
      validator: function(date) {
        return date > Date.now();
      },
      message: 'Due date must be in the future'
    }
  },
  assignedTo: [{
    type: mongoose.Schema.ObjectId,
    ref: 'User'
  }],
  createdBy: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: true
  },
  attachments: [{
    public_id: String,
    url: String,
    filename: String,
    size: Number
  }],
  tags: [String],
  comments: [{
    type: mongoose.Schema.ObjectId,
    ref: 'Comment'
  }],
  isDeleted: {
    type: Boolean,
    default: false,
    select: false
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
}, {
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Update timestamp before saving
taskSchema.pre('save', function(next) {
  this.updatedAt = Date.now();
  next();
});

// Filter out deleted tasks by default
taskSchema.pre(/^find/, function(next) {
  this.find({ isDeleted: { $ne: true } });
  next();
});

// Virtual property for overdue tasks
taskSchema.virtual('isOverdue').get(function() {
  if (!this.dueDate) return false;
  return this.dueDate < Date.now() && this.status !== 'completed';
});

// Index for better query performance
taskSchema.index({ status: 1, priority: 1 });
taskSchema.index({ assignedTo: 1, dueDate: 1 });

module.exports = mongoose.model('Task', taskSchema);
```

## 🔐 2. Authentication Middleware

**Auth Middleware** (`middleware/auth.js`):
```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const AppError = require('../utils/AppError');

// Protect routes - user must be logged in
exports.protect = async (req, res, next) => {
  try {
    let token;
    
    // 1. Get token from headers
    if (req.headers.authorization?.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    } else if (req.cookies?.token) {
      token = req.cookies.token;
    }
    
    if (!token) {
      return next(new AppError('You are not logged in. Please log in to get access.', 401));
    }
    
    // 2. Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 3. Check if user still exists
    const user = await User.findById(decoded.id).select('+passwordChangedAt');
    if (!user) {
      return next(new AppError('The user belonging to this token no longer exists.', 401));
    }
    
    // 4. Check if user changed password after token was issued
    if (user.passwordChangedAt) {
      const changedTimestamp = parseInt(user.passwordChangedAt.getTime() / 1000, 10);
      if (decoded.iat < changedTimestamp) {
        return next(new AppError('User recently changed password. Please log in again.', 401));
      }
    }
    
    // 5. Grant access
    req.user = user;
    next();
  } catch (error) {
    return next(new AppError('Authentication failed', 401));
  }
};

// Restrict to specific roles
exports.restrictTo = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return next(new AppError('You do not have permission to perform this action', 403));
    }
    next();
  };
};

// Check ownership
exports.checkOwnership = (model) => {
  return async (req, res, next) => {
    const document = await model.findById(req.params.id);
    
    if (!document) {
      return next(new AppError('Document not found', 404));
    }
    
    // Admin can do anything
    if (req.user.role === 'admin') {
      req.document = document;
      return next();
    }
    
    // Check if user owns the document
    if (document.createdBy.toString() !== req.user.id.toString()) {
      return next(new AppError('You are not authorized to perform this action', 403));
    }
    
    req.document = document;
    next();
  };
};
```

## 📝 3. CRUD Controllers

**Task Controller** (`controllers/taskController.js`):
```javascript
const Task = require('../models/Task');
const Comment = require('../models/Comment');
const AppError = require('../utils/AppError');
const catchAsync = require('../utils/catchAsync');
const APIFeatures = require('../utils/apiFeatures');

// Utility: Advanced query features
class APIFeatures {
  constructor(query, queryString) {
    this.query = query;
    this.queryString = queryString;
  }
  
  filter() {
    const queryObj = { ...this.queryString };
    const excludedFields = ['page', 'sort', 'limit', 'fields', 'search'];
    excludedFields.forEach(el => delete queryObj[el]);
    
    // Advanced filtering
    let queryStr = JSON.stringify(queryObj);
    queryStr = queryStr.replace(/\b(gte|gt|lte|lt|in)\b/g, match => `$${match}`);
    
    this.query = this.query.find(JSON.parse(queryStr));
    return this;
  }
  
  search() {
    if (this.queryString.search) {
      const searchRegex = new RegExp(this.queryString.search, 'i');
      this.query = this.query.find({
        $or: [
          { title: searchRegex },
          { description: searchRegex },
          { tags: searchRegex }
        ]
      });
    }
    return this;
  }
  
  sort() {
    if (this.queryString.sort) {
      const sortBy = this.queryString.sort.split(',').join(' ');
      this.query = this.query.sort(sortBy);
    } else {
      this.query = this.query.sort('-createdAt');
    }
    return this;
  }
  
  limitFields() {
    if (this.queryString.fields) {
      const fields = this.queryString.fields.split(',').join(' ');
      this.query = this.query.select(fields);
    } else {
      this.query = this.query.select('-__v');
    }
    return this;
  }
  
  paginate() {
    const page = this.queryString.page * 1 || 1;
    const limit = this.queryString.limit * 1 || 10;
    const skip = (page - 1) * limit;
    
    this.query = this.query.skip(skip).limit(limit);
    return this;
  }
}

// CREATE - Create new task
exports.createTask = catchAsync(async (req, res, next) => {
  // Add creator to request body
  req.body.createdBy = req.user.id;
  
  // Handle file uploads if any
  if (req.files?.attachments) {
    req.body.attachments = req.files.attachments.map(file => ({
      public_id: file.public_id,
      url: file.url,
      filename: file.originalname,
      size: file.size
    }));
  }
  
  const task = await Task.create(req.body);
  
  // Populate creator info
  await task.populate('createdBy', 'name email avatar');
  
  res.status(201).json({
    status: 'success',
    data: { task }
  });
});

// READ - Get all tasks (with filtering, pagination, etc.)
exports.getAllTasks = catchAsync(async (req, res, next) => {
  // For non-admin users, only show their assigned tasks or created tasks
  let filter = {};
  if (req.user.role !== 'admin') {
    filter = { $or: [
      { createdBy: req.user.id },
      { assignedTo: req.user.id }
    ]};
  }
  
  // Execute query with all features
  const features = new APIFeatures(Task.find(filter), req.query)
    .filter()
    .search()
    .sort()
    .limitFields()
    .paginate();
  
  const tasks = await features.query.populate([
    { path: 'createdBy', select: 'name email avatar' },
    { path: 'assignedTo', select: 'name email avatar' },
    { path: 'comments', select: 'content createdAt', populate: { path: 'user', select: 'name avatar' } }
  ]);
  
  // Get total count for pagination
  const totalFeatures = new APIFeatures(Task.find(filter), req.query)
    .filter()
    .search();
  
  const total = await Task.countDocuments(totalFeatures.query);
  
  res.status(200).json({
    status: 'success',
    results: tasks.length,
    total,
    currentPage: req.query.page * 1 || 1,
    totalPages: Math.ceil(total / (req.query.limit * 1 || 10)),
    data: { tasks }
  });
});

// READ - Get single task
exports.getTask = catchAsync(async (req, res, next) => {
  const task = await Task.findById(req.params.id)
    .populate('createdBy', 'name email avatar')
    .populate('assignedTo', 'name email avatar')
    .populate({
      path: 'comments',
      populate: { path: 'user', select: 'name avatar' }
    });
  
  if (!task) {
    return next(new AppError('No task found with that ID', 404));
  }
  
  res.status(200).json({
    status: 'success',
    data: { task }
  });
});

// UPDATE - Update task
exports.updateTask = catchAsync(async (req, res, next) => {
  // Don't allow changing createdBy
  if (req.body.createdBy) {
    delete req.body.createdBy;
  }
  
  // Handle file uploads for new attachments
  if (req.files?.attachments) {
    const newAttachments = req.files.attachments.map(file => ({
      public_id: file.public_id,
      url: file.url,
      filename: file.originalname,
      size: file.size
    }));
    
    if (req.body.attachments && Array.isArray(req.body.attachments)) {
      req.body.attachments = [...req.body.attachments, ...newAttachments];
    } else {
      req.body.attachments = newAttachments;
    }
  }
  
  const task = await Task.findByIdAndUpdate(
    req.params.id,
    req.body,
    {
      new: true, // Return updated document
      runValidators: true // Run model validators
    }
  ).populate('createdBy', 'name email avatar');
  
  if (!task) {
    return next(new AppError('No task found with that ID', 404));
  }
  
  res.status(200).json({
    status: 'success',
    data: { task }
  });
});

// DELETE - Soft delete task
exports.deleteTask = catchAsync(async (req, res, next) => {
  const task = await Task.findById(req.params.id);
  
  if (!task) {
    return next(new AppError('No task found with that ID', 404));
  }
  
  // Soft delete - mark as deleted
  task.isDeleted = true;
  await task.save();
  
  res.status(204).json({
    status: 'success',
    data: null
  });
});

// DELETE - Hard delete task (admin only)
exports.hardDeleteTask = catchAsync(async (req, res, next) => {
  const task = await Task.findByIdAndDelete(req.params.id);
  
  if (!task) {
    return next(new AppError('No task found with that ID', 404));
  }
  
  // Delete associated comments
  await Comment.deleteMany({ task: req.params.id });
  
  res.status(204).json({
    status: 'success',
    data: null
  });
});

// SPECIAL OPERATIONS

// Assign task to users
exports.assignTask = catchAsync(async (req, res, next) => {
  const { userIds } = req.body;
  
  if (!userIds || !Array.isArray(userIds)) {
    return next(new AppError('Please provide an array of user IDs', 400));
  }
  
  const task = await Task.findById(req.params.id);
  
  if (!task) {
    return next(new AppError('No task found with that ID', 404));
  }
  
  // Add users to assignedTo array (avoid duplicates)
  userIds.forEach(userId => {
    if (!task.assignedTo.includes(userId)) {
      task.assignedTo.push(userId);
    }
  });
  
  await task.save();
  
  // Populate assigned users info
  await task.populate('assignedTo', 'name email avatar');
  
  res.status(200).json({
    status: 'success',
    data: { task }
  });
});

// Add comment to task
exports.addComment = catchAsync(async (req, res, next) => {
  const { content } = req.body;
  
  const comment = await Comment.create({
    content,
    user: req.user.id,
    task: req.params.id
  });
  
  // Add comment to task
  await Task.findByIdAndUpdate(
    req.params.id,
    { $push: { comments: comment._id } }
  );
  
  await comment.populate('user', 'name avatar');
  
  res.status(201).json({
    status: 'success',
    data: { comment }
  });
});

// Upload attachment
exports.uploadAttachment = catchAsync(async (req, res, next) => {
  if (!req.file) {
    return next(new AppError('Please upload a file', 400));
  }
  
  const task = await Task.findById(req.params.id);
  
  if (!task) {
    return next(new AppError('No task found with that ID', 404));
  }
  
  task.attachments.push({
    public_id: req.file.public_id,
    url: req.file.url,
    filename: req.file.originalname,
    size: req.file.size
  });
  
  await task.save();
  
  res.status(200).json({
    status: 'success',
    data: { 
      attachment: task.attachments[task.attachments.length - 1] 
    }
  });
});

// Get task statistics
exports.getTaskStats = catchAsync(async (req, res, next) => {
  const stats = await Task.aggregate([
    {
      $match: {
        $or: [
          { createdBy: req.user._id },
          { assignedTo: req.user._id }
        ]
      }
    },
    {
      $group: {
        _id: '$status',
        count: { $sum: 1 },
        avgPriority: { $avg: { 
          $switch: {
            branches: [
              { case: { $eq: ['$priority', 'low'] }, then: 1 },
              { case: { $eq: ['$priority', 'medium'] }, then: 2 },
              { case: { $eq: ['$priority', 'high'] }, then: 3 },
              { case: { $eq: ['$priority', 'critical'] }, then: 4 }
            ],
            default: 2
          }
        }}
      }
    },
    {
      $addFields: { status: '$_id' }
    },
    {
      $project: { _id: 0 }
    },
    {
      $sort: { count: -1 }
    }
  ]);
  
  const overdue = await Task.countDocuments({
    $or: [
      { createdBy: req.user._id },
      { assignedTo: req.user._id }
    ],
    dueDate: { $lt: new Date() },
    status: { $ne: 'completed' }
  });
  
  res.status(200).json({
    status: 'success',
    data: {
      stats,
      overdue,
      total: stats.reduce((acc, curr) => acc + curr.count, 0)
    }
  });
});
```

## 🛣️ 4. Routes

**Task Routes** (`routes/taskRoutes.js`):
```javascript
const express = require('express');
const router = express.Router();
const taskController = require('../controllers/taskController');
const authMiddleware = require('../middleware/auth');
const uploadMiddleware = require('../middleware/upload');
const validationMiddleware = require('../middleware/validation');

// Apply authentication to all routes
router.use(authMiddleware.protect);

// GET /api/tasks - Get all tasks (with filters)
router.get(
  '/',
  validationMiddleware.validateQueryParams,
  taskController.getAllTasks
);

// GET /api/tasks/stats - Get task statistics
router.get('/stats', taskController.getTaskStats);

// POST /api/tasks - Create new task
router.post(
  '/',
  uploadMiddleware.upload.array('attachments', 5), // Max 5 files
  uploadMiddleware.resizeAttachments,
  validationMiddleware.validateTask,
  taskController.createTask
);

// GET /api/tasks/:id - Get single task
router.get('/:id', taskController.getTask);

// PATCH /api/tasks/:id - Update task
router.patch(
  '/:id',
  authMiddleware.checkOwnership(require('../models/Task')),
  uploadMiddleware.upload.array('attachments', 5),
  uploadMiddleware.resizeAttachments,
  validationMiddleware.validateTaskUpdate,
  taskController.updateTask
);

// DELETE /api/tasks/:id - Soft delete
router.delete(
  '/:id',
  authMiddleware.checkOwnership(require('../models/Task')),
  taskController.deleteTask
);

// DELETE /api/tasks/:id/hard - Hard delete (admin only)
router.delete(
  '/:id/hard',
  authMiddleware.restrictTo('admin'),
  taskController.hardDeleteTask
);

// POST /api/tasks/:id/assign - Assign task to users
router.post(
  '/:id/assign',
  authMiddleware.checkOwnership(require('../models/Task')),
  validationMiddleware.validateAssignment,
  taskController.assignTask
);

// POST /api/tasks/:id/comments - Add comment
router.post(
  '/:id/comments',
  validationMiddleware.validateComment,
  taskController.addComment
);

// POST /api/tasks/:id/attachments - Upload attachment
router.post(
  '/:id/attachments',
  authMiddleware.checkOwnership(require('../models/Task')),
  uploadMiddleware.upload.single('file'),
  uploadMiddleware.resizeAttachment,
  taskController.uploadAttachment
);

module.exports = router;
```

## ⚙️ 5. Server Setup

**Main Server** (`server.js`):
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const mongoSanitize = require('express-mongo-sanitize');
const xss = require('xss-clean');
const hpp = require('hpp');
const cookieParser = require('cookie-parser');
const compression = require('compression');
const morgan = require('morgan');
require('dotenv').config();

// Import routes
const authRoutes = require('./routes/authRoutes');
const taskRoutes = require('./routes/taskRoutes');
const userRoutes = require('./routes/userRoutes');

// Import error handlers
const AppError = require('./utils/AppError');
const globalErrorHandler = require('./middleware/errorHandler');

const app = express();

// 1. GLOBAL MIDDLEWARE

// Security headers
app.use(helmet());

// Enable CORS
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:3000',
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  max: 100,
  windowMs: 15 * 60 * 1000, // 15 minutes
  message: 'Too many requests from this IP. Please try again later.'
});
app.use('/api', limiter);

// Body parser
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true, limit: '10kb' }));
app.use(cookieParser());

// Data sanitization
app.use(mongoSanitize()); // Against NoSQL query injection
app.use(xss()); // Against XSS attacks

// Prevent parameter pollution
app.use(hpp({
  whitelist: ['priority', 'status', 'dueDate']
}));

// Compression
app.use(compression());

// Logging
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// 2. DATABASE CONNECTION
mongoose.connect(process.env.MONGODB_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useCreateIndex: true,
  useFindAndModify: false
}).then(() => {
  console.log('✅ MongoDB connected successfully');
}).catch(err => {
  console.error('❌ MongoDB connection error:', err);
});

// 3. ROUTES
app.use('/api/v1/auth', authRoutes);
app.use('/api/v1/tasks', taskRoutes);
app.use('/api/v1/users', userRoutes);

// Health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'success',
    message: 'Server is running',
    timestamp: new Date().toISOString(),
    uptime: process.uptime()
  });
});

// 4. ERROR HANDLING
// Handle undefined routes
app.all('*', (req, res, next) => {
  next(new AppError(`Can't find ${req.originalUrl} on this server`, 404));
});

// Global error handler
app.use(globalErrorHandler);

// 5. START SERVER
const PORT = process.env.PORT || 5000;
const server = app.listen(PORT, () => {
  console.log(`🚀 Server running on port ${PORT} in ${process.env.NODE_ENV} mode`);
});

// 6. GRACEFUL SHUTDOWN
process.on('SIGTERM', () => {
  console.log('👋 SIGTERM received. Shutting down gracefully...');
  server.close(() => {
    console.log('💥 Process terminated');
  });
});

// Handle unhandled rejections
process.on('unhandledRejection', (err) => {
  console.error('❌ UNHANDLED REJECTION! Shutting down...');
  console.error(err.name, err.message);
  server.close(() => {
    process.exit(1);
  });
});

// Handle uncaught exceptions
process.on('uncaughtException', (err) => {
  console.error('❌ UNCAUGHT EXCEPTION! Shutting down...');
  console.error(err.name, err.message);
  process.exit(1);
});

module.exports = app;
```

## 🌐 6. Frontend Integration Example

**React Component for Task Management:**
```javascript
// components/TaskManager.jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import TaskForm from './TaskForm';
import TaskList from './TaskList';
import TaskStats from './TaskStats';

// Axios setup with auth token
const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL,
  withCredentials: true
});

// Add token to requests
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

const TaskManager = () => {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(true);
  const [filters, setFilters] = useState({
    status: '',
    priority: '',
    search: '',
    sort: '-createdAt',
    page: 1,
    limit: 10
  });

  // Fetch tasks with filters
  const fetchTasks = async () => {
    try {
      setLoading(true);
      const queryParams = new URLSearchParams(filters).toString();
      const response = await api.get(`/tasks?${queryParams}`);
      setTasks(response.data.data.tasks);
    } catch (error) {
      console.error('Error fetching tasks:', error);
      alert(error.response?.data?.message || 'Failed to fetch tasks');
    } finally {
      setLoading(false);
    }
  };

  // Create new task
  const createTask = async (taskData, attachments) => {
    try {
      const formData = new FormData();
      Object.keys(taskData).forEach(key => {
        formData.append(key, taskData[key]);
      });
      
      attachments.forEach(file => {
        formData.append('attachments', file);
      });

      const response = await api.post('/tasks', formData, {
        headers: { 'Content-Type': 'multipart/form-data' }
      });
      
      setTasks([response.data.data.task, ...tasks]);
      return { success: true, data: response.data };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data?.message || 'Failed to create task' 
      };
    }
  };

  // Update task
  const updateTask = async (id, updates) => {
    try {
      const response = await api.patch(`/tasks/${id}`, updates);
      setTasks(tasks.map(task => 
        task._id === id ? response.data.data.task : task
      ));
      return { success: true };
    } catch (error) {
      return { 
        success: false, 
        error: error.response?.data?.message || 'Failed to update task' 
      };
    }
  };

  // Delete task (soft delete)
  const deleteTask = async (id) => {
    if (!window.confirm('Are you sure you want to delete this task?')) return;
    
    try {
      await api.delete(`/tasks/${id}`);
      setTasks(tasks.filter(task => task._id !== id));
    } catch (error) {
      alert(error.response?.data?.message || 'Failed to delete task');
    }
  };

  // Assign task to users
  const assignTask = async (taskId, userIds) => {
    try {
      const response = await api.post(`/tasks/${taskId}/assign`, { userIds });
      setTasks(tasks.map(task => 
        task._id === taskId ? response.data.data.task : task
      ));
    } catch (error) {
      alert(error.response?.data?.message || 'Failed to assign task');
    }
  };

  // Add comment to task
  const addComment = async (taskId, content) => {
    try {
      const response = await api.post(`/tasks/${taskId}/comments`, { content });
      setTasks(tasks.map(task => {
        if (task._id === taskId) {
          return {
            ...task,
            comments: [...task.comments, response.data.data.comment]
          };
        }
        return task;
      }));
    } catch (error) {
      alert(error.response?.data?.message || 'Failed to add comment');
    }
  };

  useEffect(() => {
    fetchTasks();
  }, [filters]);

  return (
    <div className="task-manager">
      <TaskStats />
      <TaskForm onCreate={createTask} />
      <TaskList
        tasks={tasks}
        loading={loading}
        onUpdate={updateTask}
        onDelete={deleteTask}
        onAssign={assignTask}
        onAddComment={addComment}
        filters={filters}
        onFilterChange={setFilters}
      />
    </div>
  );
};

export default TaskManager;
```

## 🔧 7. Environment Variables

**.env file:**
```env
# Server
NODE_ENV=development
PORT=5000
CLIENT_URL=http://localhost:3000

# Database
MONGODB_URI=mongodb://localhost:27017/task_manager

# JWT
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
JWT_EXPIRE=7d
JWT_COOKIE_EXPIRE=7

# File Upload (Cloudinary)
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
MAX_FILE_SIZE=5242880 # 5MB

# Email (SendGrid)
SENDGRID_API_KEY=your_sendgrid_key
EMAIL_FROM=noreply@yourdomain.com

# Rate Limiting
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX=100
```

## 📊 8. Common Real-World Operations

Here's a quick reference for common operations:

| **Operation** | **HTTP Method** | **Endpoint** | **Example Use Case** |
|--------------|----------------|--------------|---------------------|
| **Bulk Create** | `POST` | `/api/tasks/bulk` | Import tasks from CSV |
| **Bulk Update** | `PATCH` | `/api/tasks/bulk` | Update multiple task statuses |
| **Search** | `GET` | `/api/tasks?search=urgent` | Find tasks by keyword |
| **Filter** | `GET` | `/api/tasks?status=completed&priority=high` | Filter tasks by criteria |
| **Sort** | `GET` | `/api/tasks?sort=-dueDate,priority` | Sort by due date (desc) then priority |
| **Paginate** | `GET` | `/api/tasks?page=2&limit=20` | Get second page of results |
| **Export** | `GET` | `/api/tasks/export?format=csv` | Export tasks to CSV |
| **Statistics** | `GET` | `/api/tasks/stats` | Get completion statistics |
| **Activity Log** | `GET` | `/api/tasks/:id/activity` | Get task activity history |
| **Restore** | `PATCH` | `/api/tasks/:id/restore` | Restore soft-deleted task |
| **Duplicate** | `POST` | `/api/tasks/:id/duplicate` | Duplicate existing task |

## 🚀 Deployment Commands

**Package.json scripts:**
```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "jest --coverage",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "seed": "node scripts/seedDatabase.js",
    "migrate": "node scripts/migrateDatabase.js",
    "build": "npm run lint && npm test",
    "docker:build": "docker build -t task-manager-api .",
    "docker:run": "docker run -p 5000:5000 task-manager-api"
  }
}
```

This is a **production-ready** CRUD implementation with authentication, validation, file uploads, error handling, and best practices. Each component is modular and follows the Single Responsibility Principle.

Would you like me to dive deeper into any specific part, like the authentication flow, file upload implementation, or testing strategy?