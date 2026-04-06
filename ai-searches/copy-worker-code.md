## Implementation to copy worker code to a file

```js
const fs = require("fs");
const path = require("path");

const SRC_DIR = path.resolve(process.cwd(), "src");
const OUTPUT_FILE = path.resolve(process.cwd(), "combined.txt");

// Optional: ignore patterns (build artifacts, etc.)
const IGNORE = ["node_modules", ".git", ".next", "dist"];

// Recursively get all files
function getAllFiles(dir, baseDir) {
  let results = [];

  const list = fs.readdirSync(dir);

  for (const file of list) {
    const fullPath = path.join(dir, file);

    // Skip ignored folders/files
    if (IGNORE.some(i => fullPath.includes(i))) continue;

    const stat = fs.statSync(fullPath);

    if (stat && stat.isDirectory()) {
      results = results.concat(getAllFiles(fullPath, baseDir));
    } else {
      results.push(path.relative(baseDir, fullPath));
    }
  }

  return results;
}

// Main execution
function combineFiles() {
  const files = getAllFiles(SRC_DIR, SRC_DIR);

  let output = "";

  for (const relativeFilePath of files) {
    const absolutePath = path.join(SRC_DIR, relativeFilePath);

    const content = fs.readFileSync(absolutePath, "utf-8");

    output += `src/${relativeFilePath}\n`;
    output += content.trim() + "\n";
    output += "------\n";
  }

  fs.writeFileSync(OUTPUT_FILE, output, "utf-8");

  console.log(`Combined ${files.length} files into ${OUTPUT_FILE}`);
}

combineFiles();
```

---

## Run

```bash
node scripts/combine-src.js
```