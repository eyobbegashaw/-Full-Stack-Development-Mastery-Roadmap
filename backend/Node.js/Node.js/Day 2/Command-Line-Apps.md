# **Command Line Apps & Environment in Node.js - Detailed Notes & Code**

## **Environment & Configuration**

### **Environment Variables**
Environment variables are **key-value pairs stored in the operating system** that programs can access. In Node.js, use `process.env` to read them. They're essential for configuration, secrets (API keys, passwords), and environment-specific settings (development vs production). Never hardcode sensitive data; use env vars for portability and security.

```javascript
const port = process.env.PORT || 3000;
const apiKey = process.env.API_KEY;

if (!apiKey) {
  console.error('API_KEY environment variable is required');
  process.exit(1);
}
```

### **process.env**
`process.env` is a **global object containing environment variables**. It's mutable but changes only affect the current process. Common variables: `NODE_ENV`, `PORT`, `PATH`, user-defined variables. Use for configuration, feature flags, and environment detection. Variables set before starting Node.js are available here.

```javascript
// Set environment variable before running:
// NODE_ENV=production node app.js

console.log(process.env.NODE_ENV); // 'production'

// Set variable during runtime (only affects current process)
process.env.MY_VAR = 'value';
```

### **dotenv package**
`dotenv` loads environment variables from a `.env` file into `process.env`. Creates a `.env` file (never commit to version control) with key-value pairs. Use in development; in production, set vars directly. Supports different files (`.env.production`, `.env.test`) via `dotenv.config({ path: '.env.prod' })`.

```bash
# .env file
PORT=3000
API_KEY=secret123
DB_HOST=localhost
```

```javascript
require('dotenv').config(); // Loads .env file

console.log(process.env.PORT); // 3000
console.log(process.env.API_KEY); // secret123

// With options
require('dotenv').config({ 
  path: '.env.production',
  override: true // Override existing variables
});
```

## **Command Line Arguments**

### **Command line args**
Command-line arguments are values passed when executing a script: `node script.js arg1 arg2`. Accessed via `process.argv`. Useful for scripts accepting parameters, configuration, or flags. For complex CLIs, use libraries like `commander` or `yargs`.

```bash
node app.js --name John --age 30
```

### **process.argv**
`process.argv` is an **array containing command-line arguments**. Index 0: Node.js executable path. Index 1: JavaScript file path. Index 2+: User-provided arguments. Parse manually for simple cases, but use libraries for complex CLIs with flags, options, and help.

```javascript
// node app.js hello world
console.log(process.argv);
// [
//   '/usr/bin/node',
//   '/path/to/app.js', 
//   'hello',
//   'world'
// ]

const args = process.argv.slice(2);
console.log(args[0]); // 'hello'
```

### **commander**
`commander` is a **complete CLI framework** for building command-line applications. Supports commands, options, arguments, help text, and validation. Used by many popular CLI tools. Provides structure for complex CLIs with subcommands and auto-generated help.

```javascript
const { program } = require('commander');

program
  .name('my-cli')
  .description('A sample CLI tool')
  .version('1.0.0');

program
  .command('greet <name>')
  .description('Greet someone')
  .option('-c, --capitalize', 'Capitalize the name')
  .action((name, options) => {
    const greeting = options.capitalize 
      ? `Hello, ${name.toUpperCase()}!` 
      : `Hello, ${name}!`;
    console.log(greeting);
  });

program.parse();
```

## **Input/Output Handling**

### **Taking Input**
Command-line input can be received via command arguments, interactive prompts, or stdin. For interactive prompts, use `readline` module or packages like `inquirer`. Always validate and sanitize user input to prevent security issues.

```javascript
const readline = require('readline').createInterface({
  input: process.stdin,
  output: process.stdout
});

readline.question('What is your name? ', (name) => {
  console.log(`Hello, ${name}!`);
  readline.close();
});
```

### **process.stdin**
`process.stdin` is a **readable stream for standard input**. Used for interactive input, piping data, or reading from keyboard. Can be set to raw mode for character-by-character reading. Usually wrapped with `readline` for easier use.

```javascript
process.stdin.setEncoding('utf8');
process.stdin.on('data', (data) => {
  console.log(`You typed: ${data.trim()}`);
  if (data.trim() === 'exit') process.exit();
});

console.log('Type something (type "exit" to quit):');
```

### **Exitting / Exit Codes**
Exit codes indicate success (0) or failure (non-zero) when a process ends. Use `process.exit(code)` to exit immediately. Standard codes: 0=success, 1=general error, 2=misuse of shell builtins. Custom codes should be ≥ 1. Exit gracefully with cleanup if needed.

```javascript
// Success
process.exit(0);

// Failure with custom code
if (error) {
  console.error('Failed:', error.message);
  process.exit(1);
}

// With cleanup
function gracefulExit() {
  console.log('Cleaning up...');
  // Close DB connections, files, etc.
  process.exit(0);
}
process.on('SIGINT', gracefulExit); // Ctrl+C
```

### **Printing Output**
Output is printed to stdout (normal output) or stderr (error messages). Use `console.log`, `console.error`, or direct stream writes. For formatted output, use packages like `chalk`. Never block stdout with synchronous operations in servers.

```javascript
// Stdout (normal output)
process.stdout.write('Hello\n');
console.log('World');

// Stderr (error/output)
process.stderr.write('Error occurred\n');
console.error('Something went wrong');
```

### **stdout / stderr**
`process.stdout` and `process.stderr` are **writable streams for standard output and error**. `stdout` is for normal output, `stderr` for errors/logs. They're TTY streams in terminals, files when redirected. Check `isTTY` property to adapt output.

```javascript
// Check if running in terminal
if (process.stdout.isTTY) {
  console.log('Running in terminal');
} else {
  console.log('Output redirected to file');
}

// Write to stderr
process.stderr.write('Warning: Low memory\n');
```

## **Interactive CLI Packages**

### **Inquirer Package**
`inquirer` provides **interactive command-line prompts** with validation, filtering, and transformations. Supports various question types: input, confirm, list, checkbox, password. Used for configuration wizards, setup scripts, and interactive tools.

```javascript
const inquirer = require('inquirer');

inquirer.prompt([
  {
    type: 'input',
    name: 'username',
    message: 'Enter username:',
    validate: input => input.length > 0 || 'Username required'
  },
  {
    type: 'list',
    name: 'role',
    message: 'Select role:',
    choices: ['admin', 'user', 'guest']
  },
  {
    type: 'confirm',
    name: 'confirm',
    message: 'Are you sure?'
  }
]).then(answers => {
  console.log('Answers:', answers);
});
```

### **prompts package**
`prompts` is a **lightweight alternative to inquirer** with similar functionality but smaller footprint. Supports various prompt types, validation, and styling. Good for simple interactive CLIs where bundle size matters.

```javascript
const prompts = require('prompts');

(async () => {
  const response = await prompts({
    type: 'text',
    name: 'name',
    message: 'What is your name?'
  });
  
  console.log(`Hello, ${response.name}!`);
})();
```

## **Output Formatting Packages**

### **chalk package**
`chalk` adds **color and styling to terminal output**. Supports colors, background colors, text styles (bold, underline), and 256/Truecolor modes. Improves CLI readability and aesthetics. Chainable API: `chalk.red.bold('Error!')`.

```javascript
const chalk = require('chalk');

console.log(chalk.blue('Info message'));
console.log(chalk.red.bold('Error!'));
console.log(chalk.green.bgWhite('Success'));

// Template literal style
const name = 'John';
console.log(chalk`Hello {red ${name}}!`);

// Conditional coloring
const status = 'error';
console.log(
  status === 'error' ? chalk.red(status) : chalk.green(status)
);
```

### **figlet package**
`figlet` creates **ASCII art text banners** from regular text. Uses FIGfont files for different styles. Great for CLI tool headers, logos, or decorative output. Sync and async APIs available.

```javascript
const figlet = require('figlet');

// Synchronous
console.log(figlet.textSync('My CLI', {
  font: 'Standard',
  horizontalLayout: 'default',
  verticalLayout: 'default'
}));

// Asynchronous
figlet('Hello', (err, data) => {
  if (err) return;
  console.log(data);
});
```

### **cli-progress**
`cli-progress` creates **progress bars** in the terminal. Supports multiple bars, custom formats, eta calculation, and events. Useful for downloads, file processing, or any long-running operation.

```javascript
const cliProgress = require('cli-progress');

// Create progress bar
const bar = new cliProgress.SingleBar({
  format: 'Progress |{bar}| {percentage}% | {value}/{total}'
}, cliProgress.Presets.shades_classic);

// Start with total of 100
bar.start(100, 0);

// Update
for (let i = 0; i <= 100; i++) {
  bar.update(i);
  // Simulate work
  await new Promise(r => setTimeout(r, 50));
}

bar.stop();
```

## **Complete CLI Application Example**

```javascript
#!/usr/bin/env node

const { program } = require('commander');
const chalk = require('chalk');
const figlet = require('figlet');
const inquirer = require('inquirer');
const cliProgress = require('cli-progress');

// Load environment variables
require('dotenv').config();

// CLI setup
program
  .name('myapp')
  .description('A sample CLI application')
  .version('1.0.0');

// Banner command
program
  .command('banner')
  .description('Display banner')
  .action(() => {
    console.log(
      chalk.cyan(
        figlet.textSync('MyApp', { font: 'Standard' })
      )
    );
  });

// Interactive setup command  
program
  .command('setup')
  .description('Interactive setup')
  .action(async () => {
    const answers = await inquirer.prompt([
      {
        type: 'input',
        name: 'apiKey',
        message: 'Enter API key:',
        default: process.env.API_KEY
      },
      {
        type: 'list',
        name: 'environment',
        message: 'Select environment:',
        choices: ['development', 'staging', 'production']
      }
    ]);
    
    console.log(chalk.green('Setup complete!'));
    console.log('API Key:', chalk.yellow(answers.apiKey));
    console.log('Environment:', chalk.yellow(answers.environment));
  });

// Process files with progress bar
program
  .command('process <files...>')
  .description('Process files with progress bar')
  .option('-v, --verbose', 'Verbose output')
  .action((files, options) => {
    console.log(chalk.blue(`Processing ${files.length} files...`));
    
    const bar = new cliProgress.SingleBar({
      format: 'Progress |{bar}| {percentage}% | {value}/{total} files'
    }, cliProgress.Presets.shades_classic);
    
    bar.start(files.length, 0);
    
    // Simulate processing
    let processed = 0;
    const interval = setInterval(() => {
      processed++;
      bar.update(processed);
      
      if (options.verbose) {
        console.log(chalk.gray(`Processed: ${files[processed-1]}`));
      }
      
      if (processed >= files.length) {
        clearInterval(interval);
        bar.stop();
        console.log(chalk.green('✓ All files processed!'));
      }
    }, 500);
  });

// Environment info command
program
  .command('env')
  .description('Show environment information')
  .action(() => {
    console.log(chalk.underline('Environment Variables:'));
    console.log('NODE_ENV:', chalk.yellow(process.env.NODE_ENV || 'not set'));
    console.log('PORT:', chalk.yellow(process.env.PORT || '3000 (default)'));
    
    if (process.env.API_KEY) {
      console.log('API_KEY:', chalk.yellow('***' + process.env.API_KEY.slice(-4)));
    }
  });

// Error handling
program.configureOutput({
  writeErr: (str) => process.stderr.write(chalk.red(str))
});

// Parse arguments
program.parse();
```

## **Best Practices**

1. **Always validate user input** to prevent security issues
2. **Use exit codes properly** (0 for success, non-zero for errors)
3. **Color code output**: green for success, yellow for warnings, red for errors
4. **Provide clear help** with examples
5. **Handle Ctrl+C (SIGINT)** gracefully
6. **Use environment variables** for configuration
7. **Test CLI tools** with different inputs and edge cases
8. **Make output readable** with proper spacing and formatting

```javascript
// Error handling template
function handleError(error) {
  console.error(chalk.red('Error:'), error.message);
  if (process.env.NODE_ENV === 'development') {
    console.error(chalk.gray(error.stack));
  }
  process.exit(1);
}

// Graceful shutdown
process.on('SIGINT', () => {
  console.log(chalk.yellow('\nShutting down gracefully...'));
  // Cleanup code here
  process.exit(0);
});
```

These tools enable building professional, user-friendly command-line applications that handle input, output, configuration, and interaction effectively.