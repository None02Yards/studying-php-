# Arrays — Deep Dive
أنا بفكّر في Array في PHP كـ `Ordered Map`: خليط من `hash table` (سرعة الوصول بالمفتاح) و`list` (تحافظ على ترتيب الإدراج).  

**نِقَاطي الذهبية:**
- **Insertion order** بيتحافظ عليه عمومًا؛ بعض الدوال بتعيد فهرسة الأرقام—ده طبيعي.
- **Key types:** `int` أو `string`. لو المفتاح `"42"` (string رقمية) PHP بتحوّله `int 42`—دي من أشهر الـgotchas وقت الدمج.
- **Copy-on-write:** مفيش نسخ فعلي غير عند التعديل—ده أحسن للأداء. لكن لو اشتغلت **by reference** هتلمس الأصل.
- **Union (`+`) vs `array_merge`:** لو عايزة أحافظ على اليسار كمرجع، بروح للـ `+`. لو عايزة أدمج فعليًا بقواعد اليمين وإعادة فهرسة، بروح لـ `array_merge`.
- **Sorting families:**
  - `sort` / `rsort` للقوائم (وبيُعيد فهرسة).
  - `asort` / `arsort` لو يهمني المفتاح وأرتّب بالقيمة.
  - `ksort` / `krsort` لو الترتيب بالمفتاح أهم.

**Multidimensional:** حلوة للجداول—بس لو الشكل ثابت، الرفع لـ `class/DTO` بيخلي القراءة والاختبار أحسن.

**Guardrail شخصي:** بعد أي تعديل كبير، بعمل “health check” على المفاتيح والتسلسل—لو في شك، باستخدم `array_values` / `array_keys` لإعادة الضبط (ذهنيًا أو بالكود لو محتاجة).


####  sort/merge/unset، معنى كدا إنه بعد أي 
**array_values/array_keys بستخدم لإعادة الضبط** 



---

# Functions & Methods — Deep Dive 
أنا بتعامل مع الـ **function** كـ **contract** واضح:

- **Types:** بحدد `parameter/return types` (nullable/union) عشان أقلل أي confusion أو لخبطة ممكن تحصل في الـ output.  
- **Defaults/Variadic:** بستخدمهم لما يقلّلوا الضوضاء.
- **Return types:** لما مش هرجع حاجة—المعنى بالنسبة لي “`void`” ذهنيًا؛ ولو المسار لا يعود (`throw/exit`) يبقى “`never`”.


**Nested Function Definition** → تعريف function جوه function.  
2. **Anonymous Functions / Closures** → function بدون اسم، ممكن تتخزن أو تترجع.  
3. **Returning a Function** → outer function ترجع inner function.  
4. **Passing a Function** → outer تاخد function كـ parameter.  
5. **Recursive Functions** → function تنادي نفسها

- Functions (Predefined & User-defined).  
- User-defined: Normal, Anonymous, Callback, Recursive.  



**Design مبني على purity:**
- الـ **core logic** = **pure** (ترجع قيمة، من غير I/O).
- الـ **I/O** (ملفات/DB/echo) بخليه في الأطراف—ده بيرفع **testability**.

**ملخص لفكرة function nested in function**: اعتبرها **pipeline**  
الفكرة: ابني الـlogic كسلسلة `map → filter → reduce` (**pure**).  
ليه مهم؟ **modular pipeline** وقابل لإعادة الاستخدام.  
بنطبق كل ده بإننا بنحوّل خطوات المعالجة لمجموعة **callables** صغيرة وركّبيهم.

**Methods (OOP):**
- **Instance** لما في state خاصّة بالكائن.
- **Static** لو العملية خدمية وما بتعتمدش على state.
- **Visibility** بتحدد الجمهور (`public`/`private`/`protected`).
- **Immutability** عندي default: أفضّل أرجّع نسخة جديدة بدل ما أعدّل في المكان.

---

# Built-in toolbox — “شنطة العِدّة” بتاعتي
**Arrays:**  
`Merge/Union (array_merge / +)`، `Search (in_array/array_key_exists)`، `Transform (array_map/filter/reduce)`، `Ordering (sort/asort/ksort)`.

**Strings:** `strlen`, `substr`, `strtolower/upper`, `str_replace`.  
لو لغات غير إنجليزي، بافكّر في `mb_*` عشان الـUnicode.

**Date/Time:** ذهنيًا أميل لـ `DateTimeImmutable` بدل الدوال الإجرائية البسيطة—أوضح في الـtimezones.

**Validation:** `filter_var` كبوابة أمان أولى (emails/URLs…).

**JSON:** `json_encode` / `json_decode` مع انتباه للترميز.

**DB:** ي `PDO + prepared statements` كستاندرد ضد 


###  String Functions
- `strlen`, `strtolower`, `strtoupper`, `substr`, `str_replace`.

###  Array Functions
- `count`, `array_merge`, `array_push`, `in_array`.

###  Math Functions
- `abs`, `round`, `rand`, `max`, `min`.

###  Date & Time Functions
- `date`, `time`, `strtotime`, `mktime`.

### File Functions
- `fopen`, `fread`, `fwrite`, `file_get_contents`.

### Variable Handling
- `isset`, `empty`, `unset`, `gettype`


_______________________________

####  `abs`, `round`, `rand`, `max`, `min`.

```php
// ===== abs() =====
echo "abs examples:" . PHP_EOL;
echo "abs(-7) = " . abs(-7) . PHP_EOL;          // 7
echo "abs(3.5 - 10) = " . abs(3.5 - 10) . PHP_EOL; // 6.5

echo PHP_EOL;

// ===== round() =====
echo "round examples:" . PHP_EOL;
echo "round(3.14159) = " . round(3.14159) . PHP_EOL;       // 3 (افتراضي: 0 منازل)
echo "round(3.14159, 2) = " . round(3.14159, 2) . PHP_EOL; // 3.14
echo "round(2.6) = " . round(2.6) . PHP_EOL;               // 3
echo "round(2.4) = " . round(2.4) . PHP_EOL;               // 2

echo PHP_EOL;

// ===== rand() =====
echo "rand examples:" . PHP_EOL;
echo "rand(1, 10) = " . rand(1, 10) . PHP_EOL;   // رقم عشوائي بين 1 و 10
echo "rand(100, 999) = " . rand(100, 999) . PHP_EOL;

echo PHP_EOL;

// ===== max() / min() =====
$nums = [3, 7, -2, 9, 4];
echo "max([3, 7, -2, 9, 4]) = " . max($nums) . PHP_EOL; // 9
echo "min([3, 7, -2, 9, 4]) = " . min($nums) . PHP_EOL; //-2 
```

---

# Recursion — إمتى بستخدمها وليه
أنا بلجأ للـ **recursion** لما المشكلة طبيعتها هرمية (trees/graphs) أو **divide-and-conquer**.  

**القواعد اللي بمشي عليها:**
- **Base case** الأول—بوصلّه بسرعة عشان أضمن الخروج.
- PHP مفيهاش **Tail-Call Optimization** معتبرة، فلو العمق كبير بحوّل لـ **iteration** أو بستعمل **explicit stack/queue**.
- لو النتيجة كبيرة، الـ**generators** (تفكيك كسول) بتبقى لطيفة بدل تجميع ضخم في الذاكرة.

---

# الخلاصة من مفهومي (في 10 دقايق تنفيذ)
- **Filter & Map:** عندك أسعار فيها `null` وسالب—نظّف بـ`array_filter`، وبعدين اضربي ضريبة بـ`array_map`.
- **Reduce:** حوّل سلة مشتريات `[qty, price]` لمجموع إجمالي.
- **Assoc Sort:** عندك users `name=>score` — رتّب بالـscore تنازليًا مع الحفاظ على المفاتيح.
- **Function Refactor:** خدي function بتطبع—حوّليها ترجع قيمة واعملي `echo` برّه.
- **Recursion Light:** قائمة `nested categories`—طلّعي `list` مسطحة بالأسماء.

---

```php
<?php
declare(strict_types=1);
error_reporting(E_ALL);

/*
 * 1 — Nested named function inside a function
 * الفكرة: بعمل bootstrap مرة واحدة يعرّف inner function بتفلتر الأعداد الزوجية من array.
 * بعد كده أناديها طبيعي.
 */

function bootstrapArrayFilters(): void {
    // هتتبني كـ global عند أول نداء لـ bootstrapArrayFilters()
    function pickEven(array $xs): array {
        return array_values(array_filter($xs, fn($x) => is_int($x) && $x % 2 === 0));
    }
}

$nums = [3, 12, 7, 8, 5, 10];
// pickEven($nums); // لو عملتي كده قبل bootstrap → Fatal (مش متعرفة)
bootstrapArrayFilters();
echo "Ex1 - pickEven: " . implode(", ", pickEven($nums)) . PHP_EOL; // 12, 8, 10
```

________

```php
<?php
/*
 * EXAMPLE 2 — Nested **closure** inside a function (returns a callable)
 * الفكرة: outer يستلم threshold ويرجع closure بتفلتر الـ array بناءً عليه.
 * ده Nested أنظف لأننا مش بنعرّف global function.
 */

function makeThresholdFilter(int $t): callable {
    $filter = function(array $xs) use ($t): array {
        return array_values(array_filter($xs, fn($x) => is_numeric($x) && $x >= $t));
    };
    return $filter; // بنرجّع function
}

$filter20 = makeThresholdFilter(20);
echo "Ex2 - >=20: " . implode(", ", $filter20([5, 19, 20, 33, 7, 50])) . PHP_EOL; // 20, 33, 50
```

_____________

```php
<?php
/*
 * EXAMPLE 3 — Nested closure **inside a method** (helper محلي)
 * الفكرة: method بتعرّف helper محلي (closure) للتنسيق/المعالجة،
 * وتطبّقه على array داخليًا. مفيش أي global names اتضافت.
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
// مثال ناتج: count=4 | max=20 | normalized=[0.5, 1, 0.25, 1]
```

______________

```php
<?php
/*
 * EXAMPLE 4 — Method تُعيد **nested function** (callable) بيستخدم state من الـobject
 * الفكرة: object عندها factor، وبتطلعلك closure بيضرب عناصر أي array في factor ده.
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

___________

```php
<?php
/*
 * EXAMPLE 5 — (تحذيري/للتوضيح) Nested **named function** داخل method
 * هتتبني كـ global بعد أول نداء للـmethod. ما تعيديش تعريفها تاني!
 */

final class Demo
{
    public function defineInnerSum(): int {
        // هتتبني global عند أول نداء للميثود دي
        function inner_sum(array $xs): int {
            return array_reduce($xs, fn($a, $b) => (int)$a + (int)$b, 0);
        }
        return inner_sum([1, 2, 3]); // 6
    }
}
$d = new Demo();
echo "Ex5 - inner_sum once: " . $d->defineInnerSum();
// $d->defineInnerSum(); //لو فكرت تشغّل تاني → Fatal: cannot redeclare inner_sum(
```

______________________________________

```php
<?php
/*
 * RECURSION — PURE LOGIC EXAMPLES
 * - كل الدوال بترجع قيم فقط (no I/O).
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
    if (!is_array($x)) return 0;          // عنصر عادي داخل array
    if ($x === []) return 1;              // array فاضية عمقها 1
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

// 5) Fibonacci (naive recursion — استخدمي قيم صغيرة)
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

// 7) Recursive map (يطبّق fn على كل عنصر، ويحافظ على البنية لو متداخلة)
function mapRecursive(callable $fn, array $xs): array {
    $out = [];
    foreach ($xs as $k => $v) {
        $out[$k] = is_array($v) ? mapRecursive($fn, $v) : $fn($v);
    }
    return $out;
}

// 8) Class + recursive methods (pure): شجرة تجمع القيم وتسطّحها
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
 * result — echo only (no side-effects elsewhere)
 * 
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

________________

## مثال على null

استخدمي `?string` فقط لما فعلاً مقبول ترجع `null`. غير كده حدّدي نوع non-null.  
مفيش قيمة مدخلة.  
مش هيَرْجع `null` لما تفرض `string` وتركّب fallback بـ `??` أو قيمة افتراضية في التوقيع.

```php
declare(strict_types=1);

// ممكن يرجّع null (مسموح في التوقيع)
function getCityMaybe(array $user): ?string {
    // لو مفيش address أو city → null
    return is_string($user['address']['city'] ?? null) ? $user['address']['city'] : null;
}

// لا يرجّع null أبداً (fallback واضح)
function getCityOrDefault(array $user, string $default = 'N/A'): string {
    // null-safe لـ objects غير متاحة هنا؛ مع arrays نستخدم ??
    $city = $user['address']['city'] ?? '';
    return $city !== '' ? $city : $default;
}

// using only echo 
$u1 = ['address' => ['city' => 'Cairo']];
$u2 = ['address' => []];

echo (getCityMaybe($u1) ?? 'NULL') . PHP_EOL; // Cairo
echo (getCityMaybe($u2) ?? 'NULL') . PHP_EOL; // NULL

echo getCityOrDefault($u1) . PHP_EOL; // Cairo
echo getCityOrDefault($u2) . PHP_EOL; // N/A
```

---

## استخدام get$

```php
declare(strict_types=1);

// id لازم يبقى رقم موجب
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT, ['options' => ['min_range' => 1]]);
$id = $id !== null && $id !== false ? $id : 0;   // default 
// email لازم يكون فورمات صحيح
$email = filter_input(INPUT_GET, 'email', FILTER_VALIDATE_EMAIL) ?: 'N/A';

// sort لازم من مجموعة مسموح بيها (whitelist)
$allowedSorts = ['name', 'price_asc', 'price_desc'];
$sort = $_GET['sort'] ?? 'name';
$sort = in_array($sort, $allowedSorts, true) ? $sort : 'name';

echo "id=$id\n";
echo "email=$email\n";
echo "sort=$sort\n";
```

____________

## Example 2 on using $get to prevent null

```php
<?php
declare(strict_types=1);

// URL: /search.php?tags[]=php&tags[]=oop&tags[]=
$tags = $_GET['tags'] ?? [];                      // لو مفيش ⇒ []
$tags = is_array($tags) ? $tags : [];             // ضمان array
$tags = array_values(array_filter(                 // نظافة: أشيل الفاضي وأضمن strings
    array_map(fn($t) => is_string($t) ? trim($t) : '', $tags),
    fn($t) => $t !== ''
));

echo "tags=[" . implode(',', $tags) . "]\n";
```

_____________

## #3

```php
<?php
declare(strict_types=1);

// URL:
// /products.php?page=2&q=iphone

$page = (int)($_GET['page'] ?? 1);          // لو مفيش page ⇒ 1
$q    = trim((string)($_GET['q'] ?? ''));   // لو مفيش q ⇒ ""

echo "page=$page\n";
echo "q=$q\n";
```

مفيش `null` بتطلع؛ عندك لان كل الـ defaults واضحة بمعنى النوع واضح: `int` للـpage، `string` للـq .
