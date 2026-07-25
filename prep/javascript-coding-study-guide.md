# LinkedIn JavaScript Coding Study Guide

An interview-focused reference organized by source and priority. All material from the previous guide is preserved; recruiter-highlighted questions have been added at the front.

> Priority order: recruiter prep-call samples, previously reported LinkedIn questions, predicted/high-probability questions, then supporting drills. Code favors clear, explainable interview solutions, with production limitations called out explicitly.

## Table of Contents

1. [Recruiter Prep-Call Sample Questions](#recruiter-prep)
   - [Arithmetic Expression Calculator](#arithmetic-calculator)
     - [Plus-only calculator](#calculator-plus-only)
     - [`+` and `*` with precedence](#calculator-add-multiply)
     - [`+`, `-`, `*`, `/`](#calculator-four-operators)
     - [Parentheses / recursive-descent parser](#calculator-parentheses)
   - [Levenshtein Distance / Edit Distance](#edit-distance)
     - [Why dynamic programming?](#edit-distance-why-dp)
     - [Main solution: 2D DP](#edit-distance-2d-dp)
     - [Space-optimized rolling rows](#edit-distance-space-optimized)
     - [Return an optimal edit sequence](#edit-distance-steps)
     - [Closest dictionary word / spell checker](#edit-distance-dictionary)
   - [Maximum Contiguous Subarray Sum](#maximum-contiguous-subarray)
     - [Sum only: Kadane’s algorithm](#maximum-subarray-sum-only)
     - [Return start and end indices](#maximum-subarray-indices)
     - [Return the actual values](#maximum-subarray-values)
     - [Circular maximum subarray](#maximum-subarray-circular)
2. [Previously Reported LinkedIn Questions](#reported-linkedin)
   - [Fibonacci](#fibonacci)
   - [Memoization](#memoization)
   - [Capturing, Bubbling, and Delegation](#capturing-bubbling-and-delegation)
   - [DOM Traversal](#dom-traversal)
   - [Maximum Subarray](#maximum-subarray)
   - [Reverse a Doubly Linked List](#reverse-a-doubly-linked-list)
   - [String Repeat](#string-repeat)
   - [Tuple Parser](#tuple-parser)
   - [Infinite Scroll with IntersectionObserver](#infinite-scroll-with-intersectionobserver)
   - [Infinite Scroll with a Throttled Scroll Handler](#infinite-scroll-with-a-throttled-scroll-handler)
3. [Predicted / High-Probability Questions](#predicted-high-probability)
   - [Promises and the Event Loop](#promises-and-the-event-loop)
   - [AbortController](#abortcontroller)
   - [Retry with Exponential Backoff](#retry-with-exponential-backoff)
   - [Debounce](#debounce)
   - [Throttle](#throttle)
   - [Timeout Manager](#timeout-manager)
   - [Event Emitter](#event-emitter)
   - [LRU Cache](#lru-cache)
4. [Possible Omissions / Supporting Drills](#supporting-drills)
   - [Once](#once)
   - [Reverse a String In Place](#reverse-a-string-in-place)
   - [Palindrome](#palindrome)
   - [Flatten a Nested Array](#flatten-a-nested-array)
   - [Group By](#group-by)
   - [Interview Communication Checklist](#interview-communication-checklist)

---

<a id="recruiter-prep"></a>
## 1. Recruiter Prep-Call Sample Questions

These questions were explicitly mentioned during recruiter preparation. Study them first.

<a id="arithmetic-calculator"></a>
### 1.1 Arithmetic Expression Calculator

#### Clarify before coding

> Can I assume the input contains only non-negative integers, spaces, and the supported operators? Is the expression valid? Should division preserve decimals or truncate toward zero? Do I need to support parentheses or unary signs?

<a id="calculator-plus-only"></a>
#### A. Plus-only calculator

```js
function calculatePlus(expr) {
  let sum = 0;        // Sum of completed numbers.
  let num = 0;        // Number currently being parsed.
  let hasNum = false; // Whether at least one digit has been read.

  for (let i = 0; i <= expr.length; i += 1) {
    // Use a virtual "+" to process the final number.
    const ch = i === expr.length ? "+" : expr[i];

    // Ignore spaces between tokens.
    if (ch === " ") {
      continue;
    }

    // Build a multi-digit number.
    if (ch >= "0" && ch <= "9") {
      num = num * 10 + Number(ch);
      hasNum = true;
      continue;
    }

    // Commit the completed number.
    if (ch === "+" && hasNum) {
      sum += num;
      num = 0;
      hasNum = false;
      continue;
    }

    // Reject empty input and malformed operators.
    throw new SyntaxError("Invalid expression");
  }

  return sum;
}
```

```js
calculatePlus("12 + 3 + 45"); // 60
calculatePlus("100");         // 100
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> I scan the expression once and build each multi-digit number character by character. When I reach a plus operator, I add the completed number to the running sum. A virtual plus at the end lets the same logic process the final number.

A stack would also work in `O(n)` time, but it would store all parsed numbers and use `O(n)` extra space. Since completed numbers never need to be revisited, a running sum is simpler and uses `O(1)` space.

<a id="calculator-add-multiply"></a>
#### B. `+` and `*` with precedence

```js
function calculateAddMultiply(expr) {
  let sum = 0;        // Sum of completed multiplication groups.
  let term = 0;       // Current multiplication group.
  let num = 0;        // Number currently being parsed.
  let op = "+";       // Operator before the current number.
  let hasNum = false; // Whether a number is ready to process.

  for (let i = 0; i <= expr.length; i += 1) {
    // Use "+" to flush the final number and term.
    const ch = i === expr.length ? "+" : expr[i];

    // Ignore spaces between tokens.
    if (ch === " ") {
      continue;
    }

    // Build a multi-digit number.
    if (ch >= "0" && ch <= "9") {
      num = num * 10 + Number(ch);
      hasNum = true;
      continue;
    }

    // Process the number using the previous operator.
    if ((ch === "+" || ch === "*") && hasNum) {
      if (op === "+") {
        sum += term; // Finish the previous group.
        term = num;  // Start a new group.
      } else {
        term *= num; // Continue the current group.
      }

      op = ch;       // Save the next operator.
      num = 0;       // Reset the current number.
      hasNum = false;
      continue;
    }

    throw new SyntaxError("Invalid expression");
  }

  return sum + term; // Add the final group.
}
```

```js
calculateAddMultiply("1 + 2 * 3 + 4 * 5 * 6"); // 127
calculateAddMultiply("2 * 3 + 4");              // 10
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> `sum` stores completed multiplication groups, while `term` stores the group that may still be extended by multiplication. This preserves multiplication precedence in one pass without a stack.

<a id="calculator-four-operators"></a>
#### C. Follow-up: `+`, `-`, `*`, `/`

```js
function calculateAllOperators(expr) {
  const stack = [];   // Stores signed terms.
  let num = 0;        // Number currently being parsed.
  let op = "+";       // Operator before the current number.
  let hasNum = false; // Whether a number is ready to process.

  for (let i = 0; i <= expr.length; i += 1) {
    // Use "+" to process the final number.
    const ch = i === expr.length ? "+" : expr[i];

    // Ignore spaces between tokens.
    if (ch === " ") {
      continue;
    }

    // Build a multi-digit number.
    if (ch >= "0" && ch <= "9") {
      num = num * 10 + Number(ch);
      hasNum = true;
      continue;
    }

    // Process the number using the previous operator.
    if ("+-*/".includes(ch) && hasNum) {
      if (op === "+") {
        stack.push(num); // Store a positive term.
      } else if (op === "-") {
        stack.push(-num); // Store a negative term.
      } else if (op === "*") {
        // Apply high-precedence multiplication immediately.
        stack.push(stack.pop() * num);
      } else {
        if (num === 0) {
          throw new RangeError("Division by zero");
        }

        // Use Math.trunc only when integer division is required.
        stack.push(Math.trunc(stack.pop() / num));
      }

      op = ch;       // Save the next operator.
      num = 0;       // Reset the current number.
      hasNum = false;
      continue;
    }

    throw new SyntaxError("Invalid expression");
  }

  // Combine all signed terms.
  return stack.reduce((total, value) => total + value, 0);
}
```

- Time: `O(n)`
- Extra space: `O(n)`

If normal JavaScript floating-point division is required, remove `Math.trunc()`. This version assumes minus is a binary operator and all numeric tokens are non-negative.

<a id="calculator-parentheses"></a>
#### D. Follow-up: Parentheses

Parentheses make the expression recursive. A clean solution is recursive-descent parsing with this grammar:

```text
expression = term (("+" | "-") term)*
term       = factor (("*" | "/") factor)*
factor     = number | "(" expression ")"
```

```js
function calculateParentheses(expr) {
  let i = 0; // Current parser position.

  function skipSpaces() {
    while (expr[i] === " ") {
      i += 1;
    }
  }

  function parseExpression() {
    let value = parseTerm(); // Parse the first term.

    while (true) {
      skipSpaces();
      const op = expr[i];

      if (op !== "+" && op !== "-") {
        return value;
      }

      i += 1;                 // Consume the operator.
      const right = parseTerm(); // Parse the right term.
      value = op === "+" ? value + right : value - right;
    }
  }

  function parseTerm() {
    let value = parseFactor(); // Parse the first factor.

    while (true) {
      skipSpaces();
      const op = expr[i];

      if (op !== "*" && op !== "/") {
        return value;
      }

      i += 1;                   // Consume the operator.
      const right = parseFactor(); // Parse the right factor.

      if (op === "/" && right === 0) {
        throw new RangeError("Division by zero");
      }

      value = op === "*" ? value * right : value / right;
    }
  }

  function parseFactor() {
    skipSpaces();

    if (expr[i] === "(") {
      i += 1;                        // Consume "(".
      const value = parseExpression(); // Parse the nested expression.
      skipSpaces();

      if (expr[i] !== ")") {
        throw new SyntaxError("Missing closing parenthesis");
      }

      i += 1; // Consume ")".
      return value;
    }

    let num = 0;
    let hasDigit = false;

    while (expr[i] >= "0" && expr[i] <= "9") {
      num = num * 10 + Number(expr[i]); // Build the number.
      hasDigit = true;
      i += 1;
    }

    if (!hasDigit) {
      throw new SyntaxError("Expected a number");
    }

    return num;
  }

  const result = parseExpression(); // Parse the full expression.
  skipSpaces();

  if (i !== expr.length) {
    throw new SyntaxError("Unexpected trailing input");
  }

  return result;
}
```

- Time: `O(n)`
- Call stack: `O(d)`, where `d` is the maximum parenthesis depth

<a id="edit-distance"></a>
### 1.2 Levenshtein Distance / Edit Distance

<a id="edit-distance-why-dp"></a>
#### Why dynamic programming?

The problem has:

- **Optimal substructure:** the best answer for two prefixes is built from best answers for smaller prefixes.
- **Overlapping subproblems:** naive recursion repeatedly evaluates the same prefix pairs.

Dynamic programming computes each `(i, j)` state once, reducing exponential repeated work to `O(mn)`.

<a id="edit-distance-2d-dp"></a>
#### A. Main interview solution: 2D DP

`dp[i][j]` is the minimum edits needed to convert the first `i` characters of `a` into the first `j` characters of `b`. The table is `(m + 1) × (n + 1)` because prefix length zero represents the empty string.

```js
function editDistance(a, b) {
  const m = a.length; // Length of the first string.
  const n = b.length; // Length of the second string.

  // Add one row and column for empty prefixes.
  const dp = Array.from(
    { length: m + 1 },
    () => Array(n + 1).fill(0),
  );

  // Delete i characters to reach an empty target.
  for (let i = 0; i <= m; i += 1) {
    dp[i][0] = i;
  }

  // Insert j characters from an empty source.
  for (let j = 0; j <= n; j += 1) {
    dp[0][j] = j;
  }

  for (let i = 1; i <= m; i += 1) {
    for (let j = 1; j <= n; j += 1) {
      // Prefix lengths are one-based; string indexes are zero-based.
      if (a[i - 1] === b[j - 1]) {
        // Matching final characters require no new edit.
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        const remove = dp[i - 1][j];     // Delete from a.
        const insert = dp[i][j - 1];     // Insert into a.
        const replace = dp[i - 1][j - 1]; // Replace in a.

        // Pay for one operation after the best smaller state.
        dp[i][j] = 1 + Math.min(remove, insert, replace);
      }
    }
  }

  return dp[m][n]; // Distance between the full strings.
}
```

```js
editDistance("cat", "cut");        // 1
editDistance("kitten", "sitting"); // 3
editDistance("", "abc");           // 3
```

- Time: `O(mn)`
- Space: `O(mn)`

Remember the transitions:

```text
Up:       source loses one character  -> delete
Left:     target loses one character  -> insert
Diagonal: both lose one character     -> match or replace
```

<a id="edit-distance-space-optimized"></a>
#### B. Follow-up: Space optimization

```js
function editDistanceOptimized(a, b) {
  // Keep b as the shorter string to minimize row space.
  if (a.length < b.length) {
    [a, b] = [b, a];
  }

  // Empty source to every prefix of b.
  let previous = Array.from(
    { length: b.length + 1 },
    (_, j) => j,
  );

  for (let i = 1; i <= a.length; i += 1) {
    const current = Array(b.length + 1);
    current[0] = i; // Delete i source characters.

    for (let j = 1; j <= b.length; j += 1) {
      if (a[i - 1] === b[j - 1]) {
        current[j] = previous[j - 1]; // Reuse the diagonal.
      } else {
        const remove = previous[j];        // Cell above.
        const insert = current[j - 1];     // Cell to the left.
        const replace = previous[j - 1];   // Diagonal cell.
        current[j] = 1 + Math.min(remove, insert, replace);
      }
    }

    previous = current; // Advance the rolling rows.
  }

  return previous[b.length];
}
```

- Time: `O(mn)`
- Space: `O(min(m, n))`

The trade-off is that rolling rows do not retain enough information for easy backtracking.

<a id="edit-distance-steps"></a>
#### C. Follow-up: Return one optimal edit sequence

```js
function editDistanceWithSteps(a, b) {
  const m = a.length;
  const n = b.length;

  // Build the full table for backtracking.
  const dp = Array.from(
    { length: m + 1 },
    () => Array(n + 1).fill(0),
  );

  for (let i = 0; i <= m; i += 1) {
    dp[i][0] = i; // Delete the source prefix.
  }

  for (let j = 0; j <= n; j += 1) {
    dp[0][j] = j; // Insert the target prefix.
  }

  for (let i = 1; i <= m; i += 1) {
    for (let j = 1; j <= n; j += 1) {
      if (a[i - 1] === b[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = 1 + Math.min(
          dp[i - 1][j],     // Delete.
          dp[i][j - 1],     // Insert.
          dp[i - 1][j - 1], // Replace.
        );
      }
    }
  }

  const steps = []; // Backtracking produces steps in reverse.
  let i = m;
  let j = n;

  while (i > 0 || j > 0) {
    if (i > 0 && j > 0 && a[i - 1] === b[j - 1]) {
      i -= 1; // Matching characters need no operation.
      j -= 1;
    } else if (
      i > 0 &&
      j > 0 &&
      dp[i][j] === dp[i - 1][j - 1] + 1
    ) {
      steps.push(`Replace "${a[i - 1]}" with "${b[j - 1]}"`);
      i -= 1;
      j -= 1;
    } else if (i > 0 && dp[i][j] === dp[i - 1][j] + 1) {
      steps.push(`Delete "${a[i - 1]}"`);
      i -= 1;
    } else {
      steps.push(`Insert "${b[j - 1]}"`);
      j -= 1;
    }
  }

  return {
    distance: dp[m][n],
    steps: steps.reverse(), // Restore forward operation order.
  };
}
```

- Time: `O(mn)`
- Space: `O(mn)`

When multiple optimal paths exist, the condition order chooses one valid path.

<a id="edit-distance-dictionary"></a>
#### D. Follow-up: Closest dictionary word

```js
function findClosestWord(word, dictionary) {
  if (dictionary.length === 0) {
    return null;
  }

  let bestWord = null;         // Closest candidate so far.
  let bestDistance = Infinity; // Smallest distance so far.

  for (const candidate of dictionary) {
    // A length gap is already a lower bound on edit distance.
    if (Math.abs(word.length - candidate.length) > bestDistance) {
      continue;
    }

    const distance = editDistanceOptimized(word, candidate);

    if (distance < bestDistance) {
      bestWord = candidate;
      bestDistance = distance;
    }
  }

  return {
    word: bestWord,
    distance: bestDistance,
  };
}
```

For `k` dictionary words of average length `n` and an input length `m`:

- Time: `O(kmn)` in the straightforward analysis
- Per-comparison space: `O(min(m, n))`

At larger scale, discuss a BK-tree, trie-based pruning, frequency ranking, normalization, and a tie-breaking rule.

<a id="maximum-contiguous-subarray"></a>
### 1.3 Maximum Contiguous Subarray Sum

<a id="maximum-subarray-sum-only"></a>
#### A. Main interview solution: Sum only

```js
function maxSubarraySum(nums) {
  if (!Array.isArray(nums)) {
    throw new TypeError("nums must be an array");
  }

  if (nums.length === 0) {
    throw new RangeError("nums must not be empty");
  }

  let current = nums[0]; // Best sum ending at this index.
  let best = nums[0];    // Best sum found anywhere.

  for (let i = 1; i < nums.length; i += 1) {
    const value = nums[i];

    // Start here or extend the previous subarray.
    current = Math.max(value, current + value);

    // Update the global maximum.
    best = Math.max(best, current);
  }

  return best;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> At each index, the best subarray ending there either starts with the current number or extends the previous subarray. `current` stores the best sum ending here, while `best` stores the largest sum seen anywhere.

Initializing from `nums[0]`, rather than zero, correctly handles arrays containing only negative values.

<a id="maximum-subarray-indices"></a>
#### B. Follow-up: Return indices

```js
function maxSubarrayWithIndices(nums) {
  if (!Array.isArray(nums)) {
    throw new TypeError("nums must be an array");
  }

  if (nums.length === 0) {
    throw new RangeError("nums must not be empty");
  }

  let current = nums[0]; // Best sum ending here.
  let best = nums[0];    // Best sum overall.
  let start = 0;         // Start of the current candidate.
  let bestStart = 0;     // Start of the best subarray.
  let bestEnd = 0;       // End of the best subarray.

  for (let i = 1; i < nums.length; i += 1) {
    const value = nums[i];

    if (value > current + value) {
      current = value; // Start a new candidate.
      start = i;       // Record its start.
    } else {
      current += value; // Extend the candidate.
    }

    if (current > best) {
      best = current;    // Save the new best sum.
      bestStart = start; // Save its start.
      bestEnd = i;       // Save its end.
    }
  }

  return {
    sum: best,
    start: bestStart,
    end: bestEnd,
  };
}
```

- Time: `O(n)`
- Extra space: `O(1)`

This version keeps the earlier result when equal maximum sums occur. Clarify tie-breaking if it matters.

<a id="maximum-subarray-values"></a>
#### C. Follow-up: Return the actual values

```js
function maxSubarrayValues(nums) {
  // Reuse the boundary-producing solution.
  const result = maxSubarrayWithIndices(nums);

  return {
    ...result,
    // slice excludes its ending index.
    values: nums.slice(result.start, result.end + 1),
  };
}
```

If the result contains `k` values:

- Total time: `O(n + k)`
- Output space: `O(k)`

<a id="maximum-subarray-circular"></a>
#### D. Follow-up: Circular maximum subarray

```js
function maxCircularSubarray(nums) {
  if (!Array.isArray(nums)) {
    throw new TypeError("nums must be an array");
  }

  if (nums.length === 0) {
    throw new RangeError("nums must not be empty");
  }

  let total = nums[0];  // Sum of the entire array.
  let maxEnd = nums[0]; // Maximum sum ending here.
  let maxSum = nums[0]; // Maximum normal subarray.
  let minEnd = nums[0]; // Minimum sum ending here.
  let minSum = nums[0]; // Minimum normal subarray.

  for (let i = 1; i < nums.length; i += 1) {
    const value = nums[i];

    total += value; // Update the total sum.
    maxEnd = Math.max(value, maxEnd + value);
    maxSum = Math.max(maxSum, maxEnd);
    minEnd = Math.min(value, minEnd + value);
    minSum = Math.min(minSum, minEnd);
  }

  // Excluding the minimum would be empty when all values are negative.
  if (maxSum < 0) {
    return maxSum;
  }

  // Compare the normal result with the wrapping result.
  return Math.max(maxSum, total - minSum);
}
```

- Time: `O(n)`
- Extra space: `O(1)`

> A wrapping maximum equals the total array sum minus the minimum contiguous middle section. I compare that result with the standard non-wrapping maximum.

---

<a id="reported-linkedin"></a>
## 2. Previously Reported LinkedIn Questions

These questions or closely matching variants have appeared in public LinkedIn interview reports or in the earlier interview-question collection.

<a id="fibonacci"></a>
### Fibonacci

#### Recursive version

```js
function fibonacci(n) {
  if (!Number.isInteger(n) || n < 0) {
    throw new RangeError("n must be a non-negative integer");
  }

  if (n < 2) {
    return n;
  }

  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

The recursive version repeatedly solves the same subproblems.

- Time: `O(2^n)` as a common upper-bound explanation
- Call stack: `O(n)`

#### Iterative version

```js
function fibonacci(n) {
  if (!Number.isInteger(n) || n < 0) {
    throw new RangeError("n must be a non-negative integer");
  }

  if (n < 2) {
    return n;
  }

  let previous = 0;
  let current = 1;

  for (let i = 2; i <= n; i += 1) {
    const next = previous + current;
    previous = current;
    current = next;
  }

  return current;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> The recursive solution is simple, but it repeatedly computes the same values. If the requirement is only to calculate Fibonacci efficiently, I would use the iterative version, which runs in linear time and constant space.

---

---

<a id="memoization"></a>
### Memoization

#### Single-argument memoize

```js
function memoize(fn) {
  const cache = new Map();

  return function memoized(arg) {
    if (cache.has(arg)) {
      return cache.get(arg);
    }

    const result = fn.call(this, arg);
    cache.set(arg, result);
    return result;
  };
}
```

Use `cache.has()` rather than testing the cached value’s truthiness. Valid cached results can be `0`, `false`, `""`, `null`, or `undefined`.

#### Generalized interview baseline

```js
function memoize(fn) {
  const cache = new Map();

  return function memoized(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}
```

This is a reasonable interview baseline, but `JSON.stringify()` is not a universal key resolver. Limitations include:

- Circular references
- Functions and symbols
- Serialization cost
- Loss of object identity
- Unsupported or ambiguous values

#### Identity-based memoize

```js
function createNode() {
  return {
    primitives: new Map(),
    objects: new WeakMap(),
    hasValue: false,
    value: undefined,
  };
}

function isObject(value) {
  return (
    value !== null &&
    (typeof value === "object" || typeof value === "function")
  );
}

function memoize(fn) {
  const root = createNode();

  return function memoized(...args) {
    let node = root;
    const keys = [this, ...args];

    for (const key of keys) {
      const children = isObject(key) ? node.objects : node.primitives;

      if (!children.has(key)) {
        children.set(key, createNode());
      }

      node = children.get(key);
    }

    if (node.hasValue) {
      return node.value;
    }

    node.value = fn.apply(this, args);
    node.hasValue = true;
    return node.value;
  };
}
```

Including `this` in the key prevents different receivers from incorrectly sharing results when the function depends on receiver state.

#### Memoized Fibonacci

Recursive calls must go through the memoized function:

```js
const fibonacci = memoize(function calculate(n) {
  if (!Number.isInteger(n) || n < 0) {
    throw new RangeError("n must be a non-negative integer");
  }

  if (n < 2) {
    return BigInt(n);
  }

  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(100));
```

- Time: `O(n)`
- Cache: `O(n)`
- Call stack: `O(n)`

#### Async memoize

```js
function memoizeAsync(fn, getKey) {
  const cache = new Map();

  return function memoized(...args) {
    const key = getKey(...args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const promise = Promise.resolve(fn.apply(this, args)).catch((error) => {
      cache.delete(key);
      throw error;
    });

    cache.set(key, promise);
    return promise;
  };
}
```

Caching the Promise immediately deduplicates concurrent calls. Removing rejected Promises allows later calls to retry.

Possible cache controls:

- Maximum size
- LRU eviction
- TTL expiration
- Manual invalidation
- Weak object keys

---

---

<a id="capturing-bubbling-and-delegation"></a>
### Capturing, Bubbling, and Delegation

An event normally travels:

1. From the document toward the target during capture
2. At the target
3. Back through ancestors during bubbling

`event.target` is where the event originated. `event.currentTarget` is the element whose listener is currently running.

#### Event delegation

```js
const list = document.querySelector("#list");

function handleClick(event) {
  const button = event.target.closest("button[data-action]");

  if (!button || !list.contains(button)) {
    return;
  }

  const item = button.closest("[data-item-id]");

  if (!item) {
    return;
  }

  const action = button.dataset.action;
  const itemId = item.dataset.itemId;

  handleAction(action, itemId);
}

list.addEventListener("click", handleClick);

// Cleanup:
list.removeEventListener("click", handleClick);
```

**Spoken answer**

> Event delegation attaches one listener to a stable ancestor. I use `closest` because the user may click a nested icon or span inside the button. The containment check ensures the match belongs to this component.

---

---

<a id="dom-traversal"></a>
### DOM Traversal

#### Implement `getElementsByClassName`

This version searches descendants only and requires all supplied classes.

```js
function getElementsByClassName(root, classNames) {
  if (!(root instanceof Element) && !(root instanceof Document)) {
    throw new TypeError("root must be an Element or Document");
  }

  const required = classNames.trim().split(/\s+/).filter(Boolean);

  if (required.length === 0) {
    return [];
  }

  const results = [];
  const stack = [...root.children].reverse();

  while (stack.length > 0) {
    const element = stack.pop();
    const matches = required.every((name) =>
      element.classList.contains(name),
    );

    if (matches) {
      results.push(element);
    }

    for (let i = element.children.length - 1; i >= 0; i -= 1) {
      stack.push(element.children[i]);
    }
  }

  return results;
}
```

Let `n` be the number of descendants and `c` the number of required classes:

- Time: `O(n × c)`
- Auxiliary traversal space: up to `O(n)`

#### Is one node a descendant of another?

Native version:

```js
function isDescendant(parent, child) {
  return parent !== child && parent.contains(child);
}
```

Manual version:

```js
function isDescendant(parent, child) {
  let current = child.parentNode;

  while (current !== null) {
    if (current === parent) {
      return true;
    }

    current = current.parentNode;
  }

  return false;
}
```

Walking upward follows one parent chain; searching downward may inspect the entire subtree.

---

---

<a id="maximum-subarray"></a>
### Maximum Subarray

```js
function maximumSubarray(values) {
  if (!Array.isArray(values)) {
    throw new TypeError("values must be an array");
  }

  if (values.length === 0) {
    return null;
  }

  let currentSum = values[0];
  let currentStart = 0;
  let bestSum = values[0];
  let bestStart = 0;
  let bestEnd = 0;

  for (let i = 1; i < values.length; i += 1) {
    const value = values[i];

    if (value > currentSum + value) {
      currentSum = value;
      currentStart = i;
    } else {
      currentSum += value;
    }

    if (currentSum > bestSum) {
      bestSum = currentSum;
      bestStart = currentStart;
      bestEnd = i;
    }
  }

  return {
    sum: bestSum,
    start: bestStart,
    end: bestEnd,
  };
}
```

- Time: `O(n)`
- Extra space: `O(1)`

**Spoken answer**

> At each index, I decide whether to extend the previous subarray or start a new one. I track the temporary start index and copy it when I find a new global maximum.

Initializing from the first element correctly handles all-negative arrays.

---

---

<a id="reverse-a-doubly-linked-list"></a>
### Reverse a Doubly Linked List

```js
class ListNode {
  constructor(value) {
    this.value = value;
    this.previous = null;
    this.next = null;
  }
}

function reverseDoublyLinkedList(head) {
  let current = head;
  let newHead = null;

  while (current !== null) {
    const originalNext = current.next;

    current.next = current.previous;
    current.previous = originalNext;

    newHead = current;
    current = originalNext;
  }

  return newHead;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

> I save the original next pointer before swapping the links. After the swap, `current.next` points backward, so I need the saved pointer to continue along the original forward direction.

---

---

<a id="string-repeat"></a>
### String Repeat

#### Straightforward version

```js
function repeatString(value, times) {
  const count = Number(times);

  if (!Number.isInteger(count) || count < 0) {
    throw new RangeError("times must be a non-negative integer");
  }

  let result = "";

  for (let i = 0; i < count; i += 1) {
    result += value;
  }

  return result;
}
```

#### Doubling version

```js
function repeatString(value, times) {
  let count = Number(times);

  if (!Number.isInteger(count) || count < 0) {
    throw new RangeError("times must be a non-negative integer");
  }

  let result = "";
  let chunk = String(value);

  while (count > 0) {
    if (count % 2 === 1) {
      result += chunk;
    }

    count = Math.floor(count / 2);

    if (count > 0) {
      chunk += chunk;
    }
  }

  return result;
}
```

The doubling approach uses `O(log times)` high-level iterations, but total work cannot be less than the output size.

---

---

<a id="tuple-parser"></a>
### Tuple Parser

Given:

```js
const result = tuple("(1, 2, 3), (4, 5, 6), (7, 8, 9)");
result.multiply(2); // 2 * 5 * 8 = 80
```

Assume `multiply()` uses one-based positions.

```js
function tuple(input) {
  if (typeof input !== "string") {
    throw new TypeError("input must be a string");
  }

  const tuples = [];
  const pattern = /\(([^()]*)\)/g;
  let match;

  while ((match = pattern.exec(input)) !== null) {
    const values = match[1].split(",").map((token) => {
      const value = Number(token.trim());

      if (!Number.isFinite(value)) {
        throw new SyntaxError(`Invalid value: ${token}`);
      }

      return value;
    });

    tuples.push(values);
  }

  if (tuples.length === 0) {
    throw new SyntaxError("No tuples found");
  }

  return {
    values: tuples.map((values) => [...values]),

    multiply(position) {
      if (!Number.isInteger(position) || position < 1) {
        throw new RangeError("position must be a positive integer");
      }

      const index = position - 1;

      return tuples.reduce((product, values) => {
        if (index >= values.length) {
          throw new RangeError(`Tuple has no position ${position}`);
        }

        return product * values[index];
      }, 1);
    },
  };
}
```

- Parsing: linear in the input length for this simplified grammar
- `multiply`: `O(t)`, where `t` is the number of tuples

Nested tuples and quoted commas require a tokenizer or state-machine parser.

---

---

<a id="infinite-scroll-with-intersectionobserver"></a>
### Infinite Scroll with IntersectionObserver

Use a CoderPad HTML/CSS/JavaScript environment.

#### `index.html`

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Infinite Scroll</title>
    <link rel="stylesheet" href="./style.css" />
    <script src="./script.js" defer></script>
  </head>
  <body>
    <main>
      <ul id="feed" aria-label="Feed"></ul>
      <p id="status" aria-live="polite"></p>
      <button id="retry" type="button" hidden>Retry</button>
      <div id="sentinel" aria-hidden="true"></div>
    </main>
  </body>
</html>
```

#### `style.css`

```css
* {
  box-sizing: border-box;
}

#feed {
  margin: 0;
  padding: 0;
  list-style: none;
}

#feed li {
  min-height: 100px;
  margin-bottom: 10px;
  padding: 16px;
  border: 1px solid #ccc;
}

#status {
  min-height: 24px;
}

#sentinel {
  height: 1px;
}
```

#### `script.js` with a runnable mock API

```js
const data = Array.from({ length: 35 }, (_, i) => ({
  id: i + 1,
  title: `Item ${i + 1}`,
}));

function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function fetchPage(page) {
  await wait(400);

  const size = 10;
  const start = page * size;
  const items = data.slice(start, start + size);

  return {
    items,
    hasMore: start + size < data.length,
  };
}

const feed = document.querySelector("#feed");
const status = document.querySelector("#status");
const retry = document.querySelector("#retry");
const sentinel = document.querySelector("#sentinel");

let page = 0;
let loading = false;
let done = false;

function render(items) {
  const fragment = document.createDocumentFragment();

  for (const item of items) {
    const li = document.createElement("li");
    li.textContent = item.title;
    fragment.append(li);
  }

  feed.append(fragment);
}

async function loadMore() {
  if (loading || done) {
    return;
  }

  loading = true;
  retry.hidden = true;
  status.textContent = "Loading...";

  try {
    const { items, hasMore } = await fetchPage(page);

    render(items);
    page += 1;
    done = !hasMore;
    status.textContent = done ? "No more items." : "";

    if (done) {
      observer.disconnect();
    }
  } catch (error) {
    console.error(error);
    status.textContent = "Failed to load.";
    retry.hidden = false;
  } finally {
    loading = false;
  }
}

const observer = new IntersectionObserver(
  ([entry]) => {
    // We observe one target, so use the first reported entry.
    if (entry.isIntersecting) {
      loadMore();
    }
  },
  {
    root: null, // Use the browser viewport.
    rootMargin: "0px 0px 300px 0px", // Preload 300px early.
    threshold: 0, // Trigger when any part intersects.
  },
);

retry.addEventListener("click", loadMore);
observer.observe(sentinel);
loadMore();
```

The sentinel is not fixed to the viewport. It is a normal element after the feed. Appending items increases the feed’s height and naturally pushes the sentinel downward.

#### Real HTTP version of `fetchPage`

Use this only when the environment supplies an API:

```js
async function fetchPage(page) {
  const url = new URL("/api/feed", window.location.origin);
  url.searchParams.set("page", String(page));
  url.searchParams.set("limit", "10");

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  const result = await response.json();

  return {
    items: result.items,
    hasMore: result.hasMore,
  };
}
```

The caller remains:

```js
const { items, hasMore } = await fetchPage(page);
```

`fetchPage()` encapsulates the HTTP response check and JSON parsing, returning normalized application data.

---

---

<a id="infinite-scroll-with-a-throttled-scroll-handler"></a>
### Infinite Scroll with a Throttled Scroll Handler

Replace the observer setup with:

```js
function throttle(fn, delay) {
  let lastRun = 0;

  return function throttled(...args) {
    const now = Date.now();

    if (now - lastRun < delay) {
      return;
    }

    lastRun = now;
    return fn.apply(this, args);
  };
}

const handleScroll = throttle(() => {
  // Full height of the page, including off-screen content.
  const pageHeight = document.documentElement.scrollHeight;

  // Position of the viewport's bottom edge within the page.
  const viewportBottom = window.scrollY + window.innerHeight;

  // Remaining pixels between the viewport and the page bottom.
  const distance = pageHeight - viewportBottom;

  if (distance < 300) {
    loadMore();
  }
}, 200);

window.addEventListener("scroll", handleScroll, {
  // The handler will not cancel scrolling with preventDefault().
  passive: true,
});

loadMore();
```

Cleanup:

```js
window.removeEventListener("scroll", handleScroll);
```

Throttle and the `loading` guard solve different problems:

- Throttle limits layout calculations caused by frequent scroll events.
- `loading` prevents another network request while one is already pending.

**Spoken answer**

> The page height is the full document height. `scrollY + innerHeight` gives the current viewport’s bottom position. Their difference is the remaining distance to the page bottom. I throttle this calculation because scroll events fire frequently, and I still keep the loading guard because a network request can take longer than the throttle interval.

---

---

<a id="predicted-high-probability"></a>
## 3. Predicted / High-Probability Questions

These topics strongly match the JavaScript, asynchronous programming, performance, and frontend-utility patterns emphasized in LinkedIn frontend interviews.

<a id="promises-and-the-event-loop"></a>
### Promises and the Event Loop

JavaScript executes synchronous code on the call stack. Promise reactions and `queueMicrotask()` callbacks enter the microtask queue. Timer callbacks run as tasks. After the current stack is empty, the runtime drains microtasks before starting the next task.

```js
console.log("A");

setTimeout(() => console.log("B"), 0);

Promise.resolve().then(() => console.log("C"));

console.log("D");
```

Output:

```text
A
D
C
B
```

#### Implement `Promise.all`

```js
function promiseAll(iterable) {
  return new Promise((resolve, reject) => {
    const values = Array.from(iterable);
    const results = new Array(values.length);

    if (values.length === 0) {
      resolve([]);
      return;
    }

    let completed = 0;

    values.forEach((value, index) => {
      Promise.resolve(value).then((result) => {
        results[index] = result;
        completed += 1;

        if (completed === values.length) {
          resolve(results);
        }
      }, reject);
    });
  });
}
```

Important behavior:

- Accepts an iterable
- Accepts Promises and plain values
- Preserves input order, not completion order
- Resolves empty input to `[]`
- Rejects when any input rejects

**Spoken answer**

> I store each result at its original index so the output preserves input order. I use `Promise.resolve` to normalize plain values and Promises.

---

---

<a id="abortcontroller"></a>
### AbortController

`AbortController` can cancel the previous request when a newer search begins:

```js
let controller = null;

async function search(query) {
  controller?.abort();
  controller = new AbortController();

  const response = await fetch(
    `/api/search?q=${encodeURIComponent(query)}`,
    { signal: controller.signal },
  );

  if (!response.ok) {
    throw new Error(`Request failed: ${response.status}`);
  }

  return response.json();
}
```

When handling the error, distinguish cancellation:

```js
try {
  const results = await search(query);
  render(results);
} catch (error) {
  if (error.name !== "AbortError") {
    showError(error);
  }
}
```

---

---

<a id="retry-with-exponential-backoff"></a>
### Retry with Exponential Backoff

`await` pauses for Promises. `setTimeout()` returns a timer ID, so it must be wrapped in a Promise to become awaitable.

```js
function wait(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function retry(
  operation,
  {
    retries = 3,
    initialDelay = 200,
    factor = 2,
    shouldRetry = () => true,
  } = {},
) {
  let lastError;
  let delay = initialDelay;

  for (let attempt = 0; attempt <= retries; attempt += 1) {
    try {
      return await operation(attempt);
    } catch (error) {
      lastError = error;
      const finalAttempt = attempt === retries;

      if (finalAttempt || !shouldRetry(error)) {
        throw error;
      }

      await wait(delay);
      delay *= factor;
    }
  }

  throw lastError;
}
```

Production follow-ups:

- Add random jitter to prevent synchronized retries
- Retry only transient failures
- Respect cancellation
- Consider a maximum total duration

---

---

<a id="debounce"></a>
### Debounce

Debounce waits until calls have stopped for a specified period.

```js
function debounce(fn, delay) {
  let timer = null;
  let lastArgs;
  let lastThis;

  function debounced(...args) {
    lastArgs = args;
    lastThis = this;

    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
      fn.apply(lastThis, lastArgs);
    }, delay);
  }

  debounced.cancel = function cancel() {
    clearTimeout(timer);
    timer = null;
  };

  debounced.flush = function flush() {
    if (timer === null) {
      return undefined;
    }

    clearTimeout(timer);
    timer = null;
    return fn.apply(lastThis, lastArgs);
  };

  return debounced;
}
```

This version stores the latest context and arguments so `flush()` invokes the actual pending call.

---

---

<a id="throttle"></a>
### Throttle

Throttle limits how frequently a function executes while calls continue.

#### Simple leading-only version

```js
function throttle(fn, delay) {
  let lastRun = 0;

  return function throttled(...args) {
    const now = Date.now();

    if (now - lastRun < delay) {
      return;
    }

    lastRun = now;
    return fn.apply(this, args);
  };
}
```

#### Leading and trailing version

```js
function throttle(fn, interval) {
  let lastRun = 0;
  let timer = null;
  let latestArgs;
  let latestThis;

  function invoke() {
    lastRun = Date.now();
    timer = null;
    fn.apply(latestThis, latestArgs);
    latestArgs = undefined;
    latestThis = undefined;
  }

  return function throttled(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastRun);

    latestArgs = args;
    latestThis = this;

    if (remaining <= 0) {
      clearTimeout(timer);
      timer = null;
      invoke();
    } else if (timer === null) {
      timer = setTimeout(invoke, remaining);
    }
  };
}
```

**Debounce vs. throttle**

> Debounce waits for activity to stop. Throttle allows periodic execution while activity continues. I would debounce a search input and throttle a scroll handler.

---

---

<a id="timeout-manager"></a>
### Timeout Manager

Browsers do not provide a standard “clear every timeout” API. The application must track the timers it creates.

```js
function createTimeoutManager({
  setTimeoutFn = setTimeout,
  clearTimeoutFn = clearTimeout,
} = {}) {
  const active = new Set();

  function set(callback, delay, ...args) {
    let id;

    id = setTimeoutFn(() => {
      active.delete(id);
      callback(...args);
    }, delay);

    active.add(id);
    return id;
  }

  function clear(id) {
    active.delete(id);
    clearTimeoutFn(id);
  }

  function clearAll() {
    for (const id of active) {
      clearTimeoutFn(id);
    }

    active.clear();
  }

  return {
    setTimeout: set,
    clearTimeout: clear,
    clearAllTimeouts: clearAll,
    getActiveCount: () => active.size,
  };
}

const timers = createTimeoutManager();

timers.setTimeout(() => console.log("A"), 1000);
timers.setTimeout(() => console.log("B"), 2000);
timers.clearAllTimeouts();
```

---

---

<a id="event-emitter"></a>
### Event Emitter

#### Array-based interview version

```js
class EventEmitter {
  constructor() {
    this.events = new Map();
  }

  on(name, fn) {
    if (typeof fn !== "function") {
      throw new TypeError("fn must be a function");
    }

    const list = this.events.get(name) ?? [];
    list.push(fn);
    this.events.set(name, list);

    return () => this.off(name, fn);
  }

  off(name, fn) {
    const list = this.events.get(name);

    if (!list) {
      return false;
    }

    const index = list.indexOf(fn);

    if (index === -1) {
      return false;
    }

    list.splice(index, 1);

    if (list.length === 0) {
      this.events.delete(name);
    }

    return true;
  }

  once(name, fn) {
    const wrapper = (...args) => {
      this.off(name, wrapper);
      fn.apply(this, args);
    };

    return this.on(name, wrapper);
  }

  emit(name, ...args) {
    const list = this.events.get(name);

    if (!list) {
      return false;
    }

    for (const fn of [...list]) {
      fn.apply(this, args);
    }

    return true;
  }

  listenerCount(name) {
    return this.events.get(name)?.length ?? 0;
  }

  removeAllListeners(name) {
    if (name === undefined) {
      this.events.clear();
    } else {
      this.events.delete(name);
    }
  }
}
```

Why use a snapshot in `emit()`?

> A listener may subscribe or unsubscribe during emission. Iterating over a copy gives the current emission stable behavior.

Why remove a once-listener before invoking it?

> If the listener synchronously emits the same event, removing the wrapper first guarantees true once-only behavior.

Array semantics:

- Allows duplicate listener registrations
- `on`: `O(1)`
- `off`: `O(n)`
- `emit`: `O(n)`

With a `Set`, duplicate registrations are ignored and deletion is expected `O(1)`.

---

---

<a id="lru-cache"></a>
### LRU Cache

Invariant:

```text
Beginning of Map → least recently used
End of Map       → most recently used
```

```js
class LRUCache {
  constructor(capacity) {
    if (!Number.isInteger(capacity) || capacity <= 0) {
      throw new RangeError("capacity must be a positive integer");
    }

    this.capacity = capacity;
    this.cache = new Map();
  }

  get size() {
    return this.cache.size;
  }

  has(key) {
    return this.cache.has(key);
  }

  get(key) {
    if (!this.cache.has(key)) {
      return undefined;
    }

    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }

    this.cache.set(key, value);

    if (this.cache.size > this.capacity) {
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }

    return this;
  }

  delete(key) {
    return this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }
}
```

`Map` iterates in insertion order. A successful `get()` deletes and reinserts the entry, moving it to the end. `map.keys()` returns an iterator, and its first `next()` result contains the oldest key.

- Expected `get`: `O(1)`
- Expected `set`: `O(1)`
- Space: `O(capacity)`

---

---

<a id="supporting-drills"></a>
## 4. Possible Omissions / Supporting Drills

These drills complete the original guide and cover useful fundamentals or follow-ups that are less directly tied to the recruiter’s samples.

<a id="once"></a>
### Once

```js
function once(fn) {
  let called = false;
  let result;

  return function onceWrapper(...args) {
    if (!called) {
      result = fn.apply(this, args);
      called = true;
    }

    return result;
  };
}
```

This version counts only a successful return as the first call. If `fn` throws, a later call may try again.

---

---

<a id="reverse-a-string-in-place"></a>
### Reverse a String In Place

Assume the input is a mutable character array:

```js
function reverseString(chars) {
  let left = 0;
  let right = chars.length - 1;

  while (left < right) {
    [chars[left], chars[right]] = [chars[right], chars[left]];
    left += 1;
    right -= 1;
  }

  return chars;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

---

---

<a id="palindrome"></a>
### Palindrome

```js
function isPalindrome(value) {
  let left = 0;
  let right = value.length - 1;

  while (left < right) {
    if (value[left] !== value[right]) {
      return false;
    }

    left += 1;
    right -= 1;
  }

  return true;
}
```

- Time: `O(n)`
- Extra space: `O(1)`

Clarify case sensitivity, punctuation, whitespace, and Unicode normalization.

---

---

<a id="flatten-a-nested-array"></a>
### Flatten a Nested Array

```js
function flatten(values) {
  const result = [];
  const stack = [...values].reverse();

  while (stack.length > 0) {
    const value = stack.pop();

    if (Array.isArray(value)) {
      for (let i = value.length - 1; i >= 0; i -= 1) {
        stack.push(value[i]);
      }
    } else {
      result.push(value);
    }
  }

  return result;
}
```

The reverse initialization and reverse child insertion preserve left-to-right order.

---

---

<a id="group-by"></a>
### Group By

```js
function groupBy(values, getKey) {
  const groups = new Map();

  for (const value of values) {
    const key = getKey(value);

    if (!groups.has(key)) {
      groups.set(key, []);
    }

    groups.get(key).push(value);
  }

  return groups;
}
```

- Time: `O(n)` expected
- Space: `O(n)`

---

---

<a id="interview-communication-checklist"></a>
### Interview Communication Checklist

#### Before coding

> Let me restate the requirements to make sure I understand the problem correctly.

> Should invalid input throw, return an empty value, or can I assume valid input?

> Should object arguments be compared by identity or by value?

> I’ll start with a simple correct implementation, test it, and then discuss improvements.

#### While coding

> I’m using a `Map` because I need efficient lookup and must correctly preserve falsy values.

> I’m setting the loading flag before the first `await` so another call cannot pass the guard.

> I’m iterating over a snapshot because listeners may unsubscribe during emission.

#### After coding

> Let me walk through normal input, empty input, invalid input, duplicates, and boundary cases.

> Let `n` be the number of input elements. The algorithm visits each element once, so its time complexity is `O(n)`.

> This is the simplest working version. In production, I would additionally consider cancellation, cache limits, cleanup, accessibility, and observability.
