# Arrays — Deep Dive
I think of a PHP array as an **Ordered Map**: a mix of a **hash table** (fast key lookup) and a **list** (preserves insertion order).

**My golden points:**
- **Insertion order** is generally preserved; some functions will **reindex numeric keys**—that’s expected.
- **Key types:** `int` or `string`. If a key is the numeric string `"42"`, PHP will cast it to `int 42`—a classic gotcha during merges.
- **Copy-on-write:** No physical copy happens until you mutate—great for performance. If you pass **by reference**, you’ll touch the original.
- **Union (`+`) vs `array_merge`:** If you want to **keep left-hand keys** and only add non-existing keys from the right, use `+`. If you want a true **merge** that **overwrites from the right** and may **reindex numeric keys**, use `array_merge`.
- **Sorting families:**
  - `sort` / `rsort` for plain lists (**reindexes**).
  - `asort` / `arsort` when you care about **keys**, sort by **value**.
  - `ksort` / `krsort` when you care about **keys**, sort **by key**.

**Multidimensional arrays:** Great for table-like data; if the shape is fixed and fields are known, promoting to a `class`/DTO can improve readability and testability.

**Personal guardrail:** After any big change—**sort/merge/unset**—I do a “health check” on keys and order. If in doubt, I “reset” with `array_values` / `array_keys` (mentally or in code).

> **Note:** After any **sort/merge/unset**, it’s normal to “reset” using `array_values/array_keys`.

---

# Functions & Methods — Deep Dive
I treat a **function** as a clear **contract**:

- **Types:** I specify `parameter/return types` (nullable/union) to reduce confusion in the output.
- **Defaults/Variadic:** I use them when they cut noise for callers.
- **Return types:** If the function doesn’t return anything, that’s conceptually **`void`**; if a path never returns (throws/exits), that’s conceptually **`never`**.

**Common patterns**
1. **Nested Function Definition** → define a function inside another function.  
2. **Anonymous Functions / Closures** → no name; can be stored or returned.  
3. **Returning a Function** → outer function returns an inner function.  
4. **Passing a Function** → outer function accepts a function (callable) as a parameter.  
5. **Recursive Functions** → a function calls itself.

- **Functions:** both **Predefined (built-in)** and **User-defined**.  
- **User-defined flavors:** Normal, Anonymous, Callback, Recursive.

**Design anchored in purity**
- **Core logic** should be **pure** (compute a value, **no I/O**).
- Push **I/O** (files/DB/echo) to the **edges**—that boosts **testability**.

**Nested functions as a pipeline:**  
Build logic as a **pure** chain `map → filter → reduce`. Why? A **modular pipeline** is easy to reuse—model steps as small **callables** and compose them.

**Methods (OOP)**
- **Instance** when you rely on per-object **state**.
- **Static** when it’s a stateless, utility-style operation.
- **Visibility** controls audience (`public`/`private`/`protected`).
- **Immutability** (my default bias): return a **new** value instead of mutating in place.

**Arrow functions:** `fn (...) => expr` — tiny one-liners for callbacks.  
**Object operator:** `->` — used to call a method on an object.

```php
<?php
foreach ($scores as $name => $score) echo "$name:$score ";
```
---

# Built-in toolbox (my go-to set)
**Arrays:**  
`array_merge / +` (merge/union), `in_array/array_key_exists` (search), `array_map/filter/reduce` (transform), `sort/asort/ksort` (ordering).

**Strings:** `strlen`, `substr`, `strtolower/strtoupper`, `str_replace`.

**Date/Time:** I prefer `DateTimeImmutable` over bare procedural functions—time zones are clearer.

**Validation:** `filter_var` as a first safety gate (emails/URLs…).

**JSON:** `json_encode` / `json_decode`.

---

### String Functions
- `strlen`, `strtolower`, `strtoupper`, `substr`, `str_replace`.

### Array Functions
- `count`, `array_merge`, `array_push`, `in_array`.

### Math Functions
- `abs`, `round`, `rand`, `max`, `min`.

### Date & Time Functions
- `date`, `time`, `strtotime`, `mktime`.

### File Functions
- `fopen`, `fread`, `fwrite`, `file_get_contents`.

### Variable Handling
- `isset`, `empty`, `unset`, `gettype`.

---

### Math mini-demo (`abs`, `round`, `rand`, `max`, `min`)
```php
<?php
// ===== abs() =====
echo "abs examples:" . PHP_EOL;
echo "abs(-7) = " . abs(-7) . PHP_EOL;                 // 7
echo "abs(3.5 - 10) = " . abs(3.5 - 10) . PHP_EOL;     // 6.5

echo PHP_EOL;

// ===== round() =====
echo "round examples:" . PHP_EOL;
echo "round(3.14159) = " . round(3.14159) . PHP_EOL;       // 3 (default: 0 decimals)
echo "round(3.14159, 2) = " . round(3.14159, 2) . PHP_EOL; // 3.14
echo "round(2.6) = " . round(2.6) . PHP_EOL;               // 3
echo "round(2.4) = " . round(2.4) . PHP_EOL;               // 2

echo PHP_EOL;

// ===== rand() =====
echo "rand examples:" . PHP_EOL;
echo "rand(1, 10) = " . rand(1, 10) . PHP_EOL;   // random between 1 and 10
echo "rand(100, 999) = " . rand(100, 999) . PHP_EOL;

echo PHP_EOL;

// ===== max() / min() =====
$nums = [3, 7, -2, 9, 4];
echo "max([3, 7, -2, 9, 4]) = " . max($nums) . PHP_EOL; // 9
echo "min([3, 7, -2, 9, 4]) = " . min($nums) . PHP_EOL; // -2
```
---

# Recursion — When and why I use it
I reach for **recursion** when the problem is **hierarchical** (trees/graphs) or fits **divide-and-conquer**.

**Rules I follow:**
- Put the **base case** first—reach it quickly to guarantee termination.
- PHP has no significant **Tail-Call Optimization**; for deep recursion I switch to **iteration** or use an **explicit stack/queue**.
- For large outputs, **generators** can provide lazy iteration instead of building huge arrays.

---

# TL;DR for function patterns
- **Filter & Map** — Clean the data (drop null/negatives), then transform the remainder (tax/discount) to keep the pipeline tidy.
- **Reduce** — Fold a list into one value (e.g., sum `qty * price` across items) for a single, clear total.
- **Assoc Sort** — Sort by value without losing keys (e.g., `name => score` descending using `arsort`) to preserve identity.
- **Function Refactor** — Return values from functions and do `echo` outside to improve purity, testability, and reuse.
- **Recursion Light** — Walk nested structures (categories) with a simple base case to produce a clean, flat list.

```php
<?php
// 1) Filter & Map — clean then transform
$prices   = [100, null, -20, 50];
$clean    = array_filter($prices, fn($p) => is_numeric($p) && $p >= 0);
$withTax  = array_map(fn($p) => $p * 1.14, $clean);
echo "Filtered+Tax: " . implode(", ", $withTax);

// 2) Reduce — list -> single number (total)
$cart  = [[2, 50], [1, 100]]; // [qty, price]
$total = array_reduce($cart, fn($acc, $it) => $acc + $it[0] * $it[1], 0);
echo "Cart total: $total" . PHP_EOL;

// 3) Assoc Sort — keep keys, sort by value (desc)
$scores = ['Eman' => 80, 'Yard' => 95, 'Omar' => 60];
arsort($scores); // preserves keys
echo "Scores (desc): ";
foreach ($scores as $name => $score) echo "$name:$score ";
echo PHP_EOL;

// 4) Function Refactor — return value; echo outside
function formatSum(array $xs): string {
    return "Sum=" . array_sum($xs); // pure return (no echo)
}
echo formatSum([1, 2, 3]);

// 5) Recursion Light — flatten nested categories
$cats = ['Hardware', ['Laptops', ['Ultrabook']], 'Accessories'];
function flatten(array $xs): array {
    $out = [];
    foreach ($xs as $v) {
        $out = is_array($v) ? array_merge($out, flatten($v)) : array_merge($out, [$v]);
    }
    return $out;
}
echo "Flat: " . implode(", ", flatten($cats));
```
> **Aside:** `PHP_EOL` = “end of line” (portable newline).

---

```php
<?php
declare(strict_types=1);
error_reporting(E_ALL);

/*
 * 1 — Nested named function inside a function
 * Idea: bootstrap once to define an inner function that filters even numbers.
 * Then call it normally.
 */

function bootstrapArrayFilters(): void {
    // Will be defined in the global function namespace on first call
    function pickEven(array $xs): array {
        return array_values(array_filter($xs, fn($x) => is_int($x) && $x % 2 === 0));
    }
}

$nums = [3, 12, 7, 8, 5, 10];
// pickEven($nums); // Calling before bootstrap → Fatal (undefined)
bootstrapArrayFilters();
echo "Ex1 - pickEven: " . implode(", ", pickEven($nums)) . PHP_EOL; // 12, 8, 10
```
---

```php
<?php
/*
 * EXAMPLE 2 — Nested closure inside a function (returns a callable)
 * Idea: outer receives a threshold and returns a closure that filters an array accordingly.
 * Cleaner than defining a global named function.
 */

function makeThresholdFilter(int $t): callable {
    $filter = function(array $xs) use ($t): array {
        return array_values(array_filter($xs, fn($x) => is_numeric($x) && $x >= $t));
    };
    return $filter; // return a function
}

$filter20 = makeThresholdFilter(20);
echo "Ex2 - >=20: " . implode(", ", $filter20([5, 19, 20, 33, 7, 50])) . PHP_EOL; // 20, 33, 50
```
---

```php
<?php
/*
 * EXAMPLE 3 — Nested closure inside a method (local helper)
 * Idea: method defines a local helper (closure) for normalization/formatting,
 * applies it internally. No global function names added.
 */

final class Formatter
{
    public function summarize(array $xs, int $precision = 2): string {
        $normalize = function(array $arr) use ($precision): array {
            if (!$arr) return [];
            $max = max($arr);
            if ($max == 0) return $arr;
            return array_map(fn($x) => round($x / $max, $precision), $arr);
        };

        $norm = $normalize($xs);
        return "count=" . count($xs)
             . " | max=" . ($xs ? max($xs) : "NA")
             . " | normalized=[" . implode(", ", $norm) . "]";
    }
}

$f = new Formatter();
echo "Ex3 - summarize: " . $f->summarize([10, 20, 5, 20]) . PHP_EOL;
// Example output: count=4 | max=20 | normalized=[0.5, 1, 0.25, 1]
```
---

```php
<?php
/*
 * EXAMPLE 4 — Method returns a nested function (callable) that uses object state
 * Idea: object holds a factor; it returns a closure that scales array numbers by that factor.
 */

final class VectorOps
{
    public function __construct(private float $factor) {}

    public function makeScaler(): callable {
        $k = $this->factor; // capture
        $scale = function(array $xs) use ($k): array {
            return array_map(fn($x) => is_numeric($x) ? $x * $k : $x, $xs);
        };
        return $scale;
    }
}

$ops = new VectorOps(2.5);
$scaleBy2_5 = $ops->makeScaler();
echo "Ex4 - scale x2.5: " . implode(", ", $scaleBy2_5([1, 4, 7])) . PHP_EOL; // 2.5, 10, 17.5
```
---

```php
<?php
/*
 * EXAMPLE 5 — (caution) Nested named function inside a method
 * It becomes a global function name after the first call. Do not redeclare it!
 */

final class Demo
{
    public function defineInnerSum(): int {
        // Will be defined globally on first call to this method
        function inner_sum(array $xs): int {
            return array_reduce($xs, fn($a, $b) => (int)$a + (int)$b, 0);
        }
        return inner_sum([1, 2, 3]); // 6
    }
}
$d = new Demo();
echo "Ex5 - inner_sum once: " . $d->defineInnerSum();
// $d->defineInnerSum(); // If you call again → Fatal: cannot redeclare inner_sum(
```
---

```php
<?php
/*
 * RECURSION — PURE LOGIC EXAMPLES
 * - All functions return values only (no I/O).
 */

// 1) Sum of array (recursive)
function sumRecursive(array $xs): float {
    if ($xs === []) return 0.0;
    $head = (float)array_shift($xs);
    return $head + sumRecursive($xs);
}

// 2) Flatten nested arrays (recursive 1-D)
function flattenRecursive(array $xs): array {
    $out = [];
    foreach ($xs as $x) {
        if (is_array($x)) {
            $out = array_merge($out, flattenRecursive($x));
        } else {
            $out[] = $x;
        }
    }
    return $out;
}

// 3) Max depth of a nested array ( [] => depth 1 )
function maxDepth(mixed $x): int {
    if (!is_array($x)) return 0;      // a plain element inside an array
    if ($x === []) return 1;          // empty array has depth 1
    $max = 0;
    foreach ($x as $v) {
        $max = max($max, maxDepth($v));
    }
    return 1 + $max;
}

// 4) Factorial (recursive)
function factorial(int $n): int {
    if ($n <= 1) return 1;
    return $n * factorial($n - 1);
}

// 5) Fibonacci (naive recursion — use small n)
function fib(int $n): int {
    if ($n <= 1) return $n;
    return fib($n - 1) + fib($n - 2);
}

// 6) Binary search (recursive) — returns index or -1
function binarySearch(array $sorted, int $target, int $lo = 0, ?int $hi = null): int {
    if ($hi === null) $hi = count($sorted) - 1;
    if ($lo > $hi) return -1;
    $mid = intdiv($lo + $hi, 2);
    if ($sorted[$mid] === $target) return $mid;
    if ($sorted[$mid] > $target) return binarySearch($sorted, $target, $lo, $mid - 1);
    return binarySearch($sorted, $target, $mid + 1, $hi);
}

// 7) Recursive map — apply fn to every element, preserve nested shape
function mapRecursive(callable $fn, array $xs): array {
    $out = [];
    foreach ($xs as $k => $v) {
        $out[$k] = is_array($v) ? mapRecursive($fn, $v) : $fn($v);
    }
    return $out;
}

// 8) Class + recursive methods (pure): a tree that sums and flattens values
final class Node {
    public function __construct(
        public int $value,
        /** @var array<Node> */
        public array $children = []
    ) {}

    public function sum(): int {
        $s = $this->value;
        foreach ($this->children as $c) {
            $s += $c->sum(); // recursion down the tree
        }
        return $s;
    }

    public function toArray(): array {
        $acc = [$this->value];
        foreach ($this->children as $c) {
            $acc = array_merge($acc, $c->toArray()); // recursion
        }
        return $acc;
    }
}

/* 
 * DEMO — echo only (no side-effects elsewhere)
 */

$nums          = [3, 1, 4, 1, 5, 9, 2];
$nested        = [1, [2, [3, 4], 5], [6, [7, [8]]]];
$sorted        = [1, 2, 4, 5, 7, 9, 12, 20];

$sum           = sumRecursive($nums);
$flat          = flattenRecursive($nested);
$depth         = maxDepth($nested);
$fact6         = factorial(6);    // 720
$fib10         = fib(10);         // 55
$find7         = binarySearch($sorted, 7);
$find100       = binarySearch($sorted, 100);

$doubledNested = mapRecursive(fn($x) => is_numeric($x) ? $x * 2 : $x, $nested);

$tree = new Node(5, [
    new Node(3, [new Node(2), new Node(1)]),
    new Node(4),
    new Node(6, [new Node(7)])
]);
$treeSum   = $tree->sum();
$treeArray = $tree->toArray();

echo "Sum (recursive): $sum" . PHP_EOL;
echo "Flatten: [" . implode(", ", $flat) . "]" . PHP_EOL;
echo "Max depth: $depth" . PHP_EOL;
echo "factorial(6): $fact6" . PHP_EOL;
echo "fib(10): $fib10" . PHP_EOL;
echo "binarySearch 7 -> index: $find7" . PHP_EOL;
echo "binarySearch 100 -> index: $find100" . PHP_EOL;
echo "mapRecursive (x2): [" . implode(", ", flattenRecursive($doubledNested)) . "]" . PHP_EOL;

echo "Tree sum: $treeSum" . PHP_EOL;
echo "Tree toArray: [" . implode(", ", $treeArray) . "]" . PHP_EOL;
```
---

## Null example

Use `?string` **only** when returning `null` is truly acceptable. Otherwise, make the return type non-null.  
No input provided → nullable function may return `null`.  
Non-null strategy: enforce `string` and attach a **fallback** via `??` or a default parameter.

```php
<?php
declare(strict_types=1);

// May return null (nullable return type)
function getCityMaybe(array $user): ?string {
    // If 'address' or 'city' is missing → null
    return is_string($user['address']['city'] ?? null) ? $user['address']['city'] : null;
}

// Never returns null (clear fallback)
function getCityOrDefault(array $user, string $default = 'N/A'): string {
    // For arrays, use ?? to handle missing keys
    $city = $user['address']['city'] ?? '';
    return $city !== '' ? $city : $default;
}

// echo-only demo
$u1 = ['address' => ['city' => 'Cairo']];
$u2 = ['address' => []];

echo (getCityMaybe($u1) ?? 'NULL') . PHP_EOL; // Cairo
echo (getCityMaybe($u2) ?? 'NULL') . PHP_EOL; // NULL

echo getCityOrDefault($u1) . PHP_EOL; // Cairo
echo getCityOrDefault($u2) . PHP_EOL; // N/A
```
---

## Using `$_GET`
```php
<?php
declare(strict_types=1);

// id must be positive integer
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT, ['options' => ['min_range' => 1]]);
$id = $id !== null && $id !== false ? $id : 0;   // default

// email must be a valid format
$email = filter_input(INPUT_GET, 'email', FILTER_VALIDATE_EMAIL) ?: 'N/A';

// sort must be from a whitelist
$allowedSorts = ['name', 'price_asc', 'price_desc'];
$sort = $_GET['sort'] ?? 'name';
$sort = in_array($sort, $allowedSorts, true) ? $sort : 'name';

echo "id=$id\n";
echo "email=$email\n";
echo "sort=$sort\n";
```
---

## Example 2: using `$_GET` to avoid nulls
```php
<?php
declare(strict_types=1);

// URL: /search.php?tags[]=php&tags[]=oop&tags[]=
$tags = $_GET['tags'] ?? [];                      // default []
$tags = is_array($tags) ? $tags : [];             // ensure array
$tags = array_values(array_filter(                 // clean: drop empty, ensure strings
    array_map(fn($t) => is_string($t) ? trim($t) : '', $tags),
    fn($t) => $t !== ''
));

echo "tags=[" . implode(',', $tags) . "]\n";
```
---

## #3
```php
<?php
declare(strict_types=1);

// URL example: /products.php?page=2&q=iphone

$page = (int)($_GET['page'] ?? 1);          // default page = 1
$q    = trim((string)($_GET['q'] ?? ''));   // default query = ""

echo "page=$page\n";
echo "q=$q\n";
```
No `null` leaks here; defaults make the **types explicit**: `int` for `page`, `string` for `q`.
