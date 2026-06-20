# 10 - Modules and Packages

Modules split code across files.

```javascript
// math.js
export function add(a, b) {
  return a + b;
}
```

```javascript
// index.js
import { add } from "./math.js";
console.log(add(2, 3));
```

Packages are installed with npm.

```bash
npm install lodash
```

Practice: export one function from a file and import it in another.
