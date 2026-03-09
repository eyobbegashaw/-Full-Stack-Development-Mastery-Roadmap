# **Working with Files & Paths in Node.js - Detailed Notes & Code**

## **Path Utilities**

### **__dirname**
`__dirname` returns the **absolute path of the directory containing the current JavaScript file**. It's a global variable available in CommonJS modules (not in ES modules by default). Useful for constructing file paths relative to the current module's location. For ES modules, use `import.meta.url` or `path.dirname(url.fileURLToPath(import.meta.url))`.

```javascript
console.log(__dirname); // /Users/project/src
const filePath = path.join(__dirname, 'data.json');
```

### **__filename**
`__filename` returns the **absolute path of the current JavaScript file**. Includes the filename with extension. Like `__dirname`, it's a CommonJS global. Helps when you need to reference the current file's location for logging, configuration, or relative file operations.

```javascript
console.log(__filename); // /Users/project/src/app.js
const config = require(path.join(__filename, '../config.json'));
```

### **process.cwd()**
`process.cwd()` returns the **current working directory** where the Node.js process was launched. This can differ from `__dirname` when scripts are executed from different locations. It's dynamic and can be changed with `process.chdir()`. Use for paths relative to where the command was run.

```javascript
console.log(process.cwd()); // /Users/project (where node was started)
const dataFile = path.join(process.cwd(), 'data.json');
```

## **Core Modules**

### **path module**
The `path` module provides utilities for **working with file and directory paths**. It handles cross-platform differences (Windows vs Unix), normalizes paths, and provides methods for parsing, joining, resolving, and extracting path components. Essential for building reliable file paths.

```javascript
const path = require('path');
const fullPath = path.join(__dirname, 'data', 'file.txt');
const basename = path.basename(fullPath); // 'file.txt'
const dir = path.dirname(fullPath); // '/.../data'
const ext = path.extname(fullPath); // '.txt'
```

### **fs module**
The `fs` (File System) module provides **APIs for interacting with the file system**. Offers both callback-based and Promise-based (fs.promises) methods. Functions for reading/writing files, directories, permissions, stats, and watching for changes. Use `fs.promises` for modern async/await patterns.

```javascript
const fs = require('fs').promises;

async function readFile() {
  try {
    const data = await fs.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('Error reading:', err);
  }
}

// Sync version (blocking)
const fsSync = require('fs');
const data = fsSync.readFileSync('file.txt', 'utf8');
```

## **Working with Files**

### **File Operations Examples**
```javascript
const fs = require('fs').promises;
const path = require('path');

// Read file
async function readData() {
  return await fs.readFile('data.json', 'utf8');
}

// Write file
async function saveData(content) {
  await fs.writeFile('output.json', JSON.stringify(content, null, 2));
}

// Check if file exists
async function fileExists(filePath) {
  try {
    await fs.access(filePath);
    return true;
  } catch {
    return false;
  }
}

// List directory contents
async function listDir(dirPath) {
  return await fs.readdir(dirPath);
}

// Get file stats
async function getFileInfo(filePath) {
  const stats = await fs.stat(filePath);
  return {
    size: stats.size,
    modified: stats.mtime,
    isDirectory: stats.isDirectory()
  };
}
```

## **Open Source Packages**

### **glob**
`glob` is a **pattern matching library** for finding files using shell-like patterns (`*.js`, `**/*.txt`). It supports complex patterns, negation, and expansion. Useful for batch file operations, build scripts, and finding files matching patterns. Returns matching file paths.

```javascript
const glob = require('glob');

// Find all JavaScript files
glob('**/*.js', (err, files) => {
  if (err) throw err;
  console.log('Found JS files:', files);
});

// Sync version
const files = glob.sync('src/**/*.{js,ts}');
```

### **globby**
`globby` is a **user-friendly glob matching library** with Promise support, negation, and extended patterns. Built on `fast-glob`, it's faster than original `glob` and supports multiple patterns. Popular in build tools and scripts for modern Node.js projects.

```javascript
const globby = require('globby');

(async () => {
  const files = await globby(['**/*.js', '!node_modules/**']);
  console.log('Matched files:', files);
})();

// With options
const images = await globby('**/*.{png,jpg,gif}', {
  ignore: ['**/node_modules/**'],
  absolute: true
});
```

### **fs-extra**
`fs-extra` **extends the native fs module** with additional methods and Promise support. Adds useful utilities like `copy`, `move`, `ensureDir`, `remove`, `readJson`, `writeJson`, and `outputFile`. Reduces boilerplate code for common file operations.

```javascript
const fs = require('fs-extra');

// Ensure directory exists (creates if not)
await fs.ensureDir('./data/logs');

// Copy entire directory
await fs.copy('./src', './dist');

// Read JSON file (auto-parses)
const config = await fs.readJson('./config.json');

// Write JSON (auto-stringifies)
await fs.writeJson('./output.json', { data: 'test' }, { spaces: 2 });

// Remove directory (like rm -rf)
await fs.remove('./temp');
```

### **chokidar**
`chokidar` is a **file watching library** that efficiently watches files/directories for changes. More reliable than `fs.watch`, works across platforms, and provides a clean API. Used in development servers, build tools, and live-reload functionality.

```javascript
const chokidar = require('chokidar');

// Initialize watcher
const watcher = chokidar.watch('./src', {
  ignored: /(^|[\/\\])\../, // ignore dotfiles
  persistent: true
});

// Add event listeners
watcher
  .on('add', path => console.log(`File ${path} added`))
  .on('change', path => console.log(`File ${path} changed`))
  .on('unlink', path => console.log(`File ${path} removed`));

// Close watcher
// watcher.close();
```

## **Practical Examples**

### **Project Structure Scanner**
```javascript
const fs = require('fs-extra');
const path = require('path');
const globby = require('globby');

async function scanProject(rootDir) {
  const results = {
    files: [],
    totalSize: 0,
    byExtension: {}
  };

  const files = await globby('**/*', {
    cwd: rootDir,
    ignore: ['**/node_modules/**', '**/.git/**'],
    absolute: true
  });

  for (const file of files) {
    const stats = await fs.stat(file);
    if (stats.isFile()) {
      const ext = path.extname(file).toLowerCase();
      results.files.push({
        path: path.relative(rootDir, file),
        size: stats.size,
        extension: ext || '(none)'
      });
      results.totalSize += stats.size;
      results.byExtension[ext] = (results.byExtension[ext] || 0) + 1;
    }
  }

  return results;
}
```

### **File Backup Utility**
```javascript
const fs = require('fs-extra');
const path = require('path');

class BackupManager {
  constructor(sourceDir, backupDir) {
    this.sourceDir = sourceDir;
    this.backupDir = backupDir;
  }

  async createBackup() {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
    const backupPath = path.join(this.backupDir, `backup-${timestamp}`);
    
    await fs.ensureDir(this.backupDir);
    await fs.copy(this.sourceDir, backupPath);
    
    console.log(`Backup created at: ${backupPath}`);
    return backupPath;
  }

  async cleanupOldBackups(maxBackups = 5) {
    const backups = await fs.readdir(this.backupDir);
    const backupDirs = backups
      .filter(name => name.startsWith('backup-'))
      .sort()
      .reverse();
    
    if (backupDirs.length > maxBackups) {
      const toRemove = backupDirs.slice(maxBackups);
      for (const dir of toRemove) {
        await fs.remove(path.join(this.backupDir, dir));
        console.log(`Removed old backup: ${dir}`);
      }
    }
  }
}
```

### **Live Reload Server**
```javascript
const chokidar = require('chokidar');
const WebSocket = require('ws');
const http = require('http');

class LiveReloadServer {
  constructor(port = 35729) {
    this.port = port;
    this.wss = null;
    this.watcher = null;
  }

  start(watchPath = '.') {
    const server = http.createServer();
    this.wss = new WebSocket.Server({ server });
    
    this.watcher = chokidar.watch(watchPath, {
      ignored: /(^|[\/\\])\../,
      ignoreInitial: true
    });

    this.watcher.on('change', (path) => {
      console.log(`File changed: ${path}`);
      this.broadcast({ type: 'reload', path });
    });

    this.wss.on('connection', (ws) => {
      console.log('Client connected');
      ws.send(JSON.stringify({ type: 'hello' }));
    });

    server.listen(this.port, () => {
      console.log(`Live reload server on ws://localhost:${this.port}`);
    });
  }

  broadcast(message) {
    if (this.wss) {
      this.wss.clients.forEach(client => {
        if (client.readyState === WebSocket.OPEN) {
          client.send(JSON.stringify(message));
        }
      });
    }
  }
}
```

## **Best Practices**

1. **Use `path.join()`** instead of string concatenation for cross-platform compatibility
2. **Always handle errors** in file operations
3. **Use async/await with `fs.promises`** for modern code
4. **Validate user-provided paths** to prevent directory traversal attacks
5. **Clean up temporary files** to avoid disk space issues
6. **Use streams** for large files to avoid memory issues
7. **Set appropriate file permissions** when creating files
8. **Watch for `ENOENT` errors** (file not found) and handle gracefully

```javascript
// Safe file operations
async function safeFileOperation(filePath) {
  try {
    // Normalize and resolve path
    const resolvedPath = path.resolve(filePath);
    
    // Prevent directory traversal
    if (!resolvedPath.startsWith(process.cwd())) {
      throw new Error('Access denied');
    }
    
    // Check if file exists
    const exists = await fs.pathExists(resolvedPath);
    if (!exists) {
      throw new Error('File not found');
    }
    
    // Perform operation
    return await fs.readFile(resolvedPath, 'utf8');
  } catch (error) {
    console.error('File operation failed:', error.message);
    return null;
  }
}
```

These tools and patterns form the foundation of file system operations in Node.js, enabling everything from simple file reads to complex build systems and file watchers.