# DSA Cheat-Sheet

> Dense quick reference. Scan before every problem. Re-read before every mock interview.
> Last updated: end of HashMap unit (6 problems completed).

---

## CPU benchmark

> **Modern CPU ≈ 10⁸ simple operations per second.**
> To convert ops → seconds, divide by 10⁸.

---

## Constraint → algorithm class (memorize)

| Constraint     | Target complexity     | Typical techniques                              |
|----------------|-----------------------|-------------------------------------------------|
| N ≤ 20–25      | O(2ᴺ)                 | Bitmask, backtracking, subset enumeration       |
| N ≤ 500        | O(N³)                 | Floyd-Warshall, interval DP                     |
| N ≤ 5000       | O(N²)                 | Pairwise DP, LCS, edit distance                 |
| N ≤ 10⁵        | O(N log N) or O(N)    | Sort, binary search, two-pointer, sliding window|
| N ≤ 10⁶–10⁷    | O(N)                  | Linear scan, sieve                              |
| N ≤ 10⁹        | O(log N) or O(1)      | Math, formula, matrix exponentiation            |

---

## Max safe N per complexity (1s budget)

| O(log N)  | O(N)   | O(N log N) | O(N√N) | O(N²) | O(N³) | O(2ᴺ) | O(N!) |
|-----------|--------|------------|--------|-------|-------|-------|-------|
| unlimited | 10⁸    | 10⁶        | 10⁵    | 10⁴   | 500   | 25    | 10    |

---

## Pre-coding ritual — 4 steps, every problem

1. **Read twice.** Underline `≤`, `<`, `=`. Note "substring" vs "subsequence".
2. **Restate in one precise sentence** using terms from the problem.
3. **Walk the example** to confirm. The walk often becomes your brute force.
4. **Clarifying questions** — value-changing only:
   - Empty/null input → what to return?
   - Max N? (sets complexity budget)
   - Value range / character set? (bounded → array; unbounded → HashMap)
   - Duplicates allowed? Sorted?
   - Two inputs: equal lengths? Case-sensitive? Output format?
   - **State hypothesis with question** when natural: *"Empty input → return true since two empty are anagrams. Confirm?"*

Then: brute force → optimized → code → trace.

---

## HashMap pattern

**Signature:** searching inside a loop → HashMap is the first hypothesis.
**Rule:** what you look up *fast* must be the **key**.

### Subpatterns

| Subpattern                  | Example                       |
|-----------------------------|-------------------------------|
| Fast lookup                 | Two Sum                       |
| Stale-state overwrite       | Contains Nearby Duplicate     |
| Window state                | Longest Substring No Repeat   |
| Group-by-signature          | Group Anagrams                |
| Frequency array fingerprint | Group Anagrams, Valid Anagram |
| Increment/decrement trick   | Valid Anagram, Ransom Note    |

### Two Sum (existence)

```java
Map<Integer, Integer> seen = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (seen.containsKey(complement)) return true;
    seen.put(nums[i], i);
}
return false;
```

**Order rule:** check first, then put.

### Contains Nearby Duplicate (stale overwrite)

```java
Map<Integer, Integer> seen = new HashMap<>();
for (int i = 0; i < nums.length; i++) {
    if (seen.containsKey(nums[i]) && i - seen.get(nums[i]) <= k) return true;
    seen.put(nums[i], i);   // always overwrite — stale indices useless
}
return false;
```

### Group Anagrams (group-by-signature)

```java
Map<String, List<String>> groups = new HashMap<>();
for (String s : strs) {
    int[] count = new int[26];
    for (int j = 0; j < s.length(); j++) count[s.charAt(j) - 'a']++;
    StringBuilder kb = new StringBuilder();
    for (int c : count) kb.append(c).append('#');
    groups.computeIfAbsent(kb.toString(), k -> new ArrayList<>()).add(s);
}
return new ArrayList<>(groups.values());
```

### Valid Anagram (single-array +/-)

```java
if (s.length() != t.length()) return false;
int[] count = new int[26];
for (int i = 0; i < s.length(); i++) {
    count[s.charAt(i) - 'a']++;
    count[t.charAt(i) - 'a']--;
}
for (int c : count) if (c != 0) return false;
return true;
```

### Ransom Note (count then subtract)

```java
int[] count = new int[26];
for (int i = 0; i < magazine.length(); i++) count[magazine.charAt(i) - 'a']++;
for (int i = 0; i < ransomNote.length(); i++) {
    if (--count[ransomNote.charAt(i) - 'a'] < 0) return false;
}
return true;
```

### Variants

| Variant         | Type                       | When                                       |
|-----------------|----------------------------|--------------------------------------------|
| Existence only  | `Set<T>`                   | Don't care about index or count            |
| Value → index   | `Map<T, Integer>`          | Need *where* you saw it                    |
| Frequency map   | `Map<T, Integer>`          | Counting (anagrams, majority, k-distinct)  |
| Frequency array | `int[26]` / `int[128]`     | Bounded set — faster than HashMap          |
| Prefix-sum map  | `Map<Long, Integer>`       | Subarray-sum problems                      |

### Traps

- `containsValue()` is **O(N)**. Never use inside a loop.
- Autoboxing cost: for bounded character sets, prefer `int[26]` / `int[128]`.
- `c - 'a'` (not `'a' - c`). Test on `'b' - 'a' = 1`.
- Verify length-relationship assumptions before using them.

---

## Sliding window template

**Signature:** "longest / shortest / count" of substring/subarray/contiguous window.

```java
int slidingWindow(int[] arr) {
    int left = 0, answer = 0;
    // window state (Set, frequency map, running sum, etc.)

    for (int right = 0; right < arr.length; right++) {
        // 1. EXPAND: include arr[right]
        while (windowIsInvalid()) {
            // 2. SHRINK: remove arr[left]
            left++;
        }
        // 3. UPDATE answer (window [left..right] is valid)
        answer = Math.max(answer, right - left + 1);
    }
    return answer;
}
```

**Three slots to fill:**
1. What state changes on expand?
2. What makes the window invalid?
3. What do I track — length, count, best value?

### Canonical — Longest Substring Without Repeating Characters

```java
int left = 0, maxLength = 0;
Set<Character> seen = new HashSet<>();
for (int right = 0; right < s.length(); right++) {
    while (seen.contains(s.charAt(right))) {
        seen.remove(s.charAt(left));
        left++;
    }
    seen.add(s.charAt(right));
    maxLength = Math.max(maxLength, right - left + 1);
}
return maxLength;
```

### Complexity argument (say this aloud)

> *"Each pointer moves at most N times across the entire algorithm. Total work ≤ 2N → O(N). This is amortized analysis."*

---

## Two pointers — sorted convergence template

```java
int left = 0, right = sortedNums.length - 1;
while (left < right) {
    int sum = sortedNums[left] + sortedNums[right];
    if (sum == target) return true;
    if (sum < target) left++;
    else right--;
}
return false;
```

Each step eliminates at least one position. → O(N) after sorting.

---

## Java code quality — quick rules

| Rule                                          | Example                                         |
|-----------------------------------------------|-------------------------------------------------|
| Interface on LHS                              | `Map<K,V> m = new HashMap<>();`                 |
| `ArrayDeque` not `Stack`                      | `Deque<Integer> stack = new ArrayDeque<>();`    |
| `camelCase`                                   | `maxLength`, not `maxlength`                    |
| Always braces                                 | even on single-line `if`                        |
| Meaningful names                              | `seen`, `complement`, `frequencies`             |
| No shadowing                                  | `for(i...) for(j...)` — never reuse `i`         |
| `Math.max(a, b)`                              | one line, idiomatic running max                 |
| `getOrDefault(k, -1)`                         | one map call instead of two                     |
| `computeIfAbsent(k, k -> new ArrayList<>())`  | get-or-create-then-update, single call          |
| `--count[c - 'a'] < 0`                        | decrement-then-check, counting problems         |
| For-each when index not needed                | `for (String s : strs)`                         |
| Comments explain *why*, not *what*            | no `// increment i`                             |

---

## Trace format (whiteboard-ready)

```
nums = [1, 0, 1, 1], k = 1
seen = {}

i = 0: nums[0] = 1
       j = -1, skip
       seen = {1: 0}

i = 2: nums[2] = 1
       j = 0, i - j = 2 - 0 = 2.  2 <= 1? NO
       seen = {1: 2, 0: 1}   (overwrite)

i = 3: nums[3] = 1
       j = 2, i - j = 3 - 2 = 1.  1 <= 1? YES
       return true ✓
```

**Rules:**
- Compute arithmetic explicitly (`2 - 0 = 2`, not `0`).
- Show data structure state at every step.
- Show non-updates too.
- Pick examples that exercise the tricky path.
- **Internal consistency check** — top-to-bottom read before sending:
  - If you write `'r'`, the next line must subtract `'r'`, not `'a'`.
  - If you write `++`, the result must go up. If `--`, down.

---

## Communication phrases (speak aloud daily)

**Complexity (always both time AND space):**
- *"Time: O(N), space: O(1) since alphabet size is constant."*
- *"For each of N elements, I do O(1) work, so total is O(N)."*
- *"Each pointer moves at most N times total — amortized O(N)."*
- *"Sort dominates: O(N log N) for sort plus O(N) for scan."*

**Clarifying with hypothesis:**
- *"Empty input — I'd return true. Confirm?"*

**Tradeoffs:**
- *"I could use two arrays then compare, but the increment/decrement trick does it in one pass."*
- *"Sorting each string also works but is O(K log K) per string vs. O(K) for the count-based key."*

**Borderline:**
- *"N³ with N=1000 is 10⁹ ops — borderline. C++ fine, Java may TLE."*

**Stuck:**
- *"Let me think about what information I'm recomputing that I already have."*

**Never** go silent > 30 seconds. Narrate partial thinking.

---

## Code self-review checklist (60 seconds before declaring done)

- [ ] Read problem one more time. Match every constraint.
- [ ] Read code as if it were a junior's PR:
  - [ ] Anything doing nothing? (redundant `if` around `while`, double map lookup)
  - [ ] Could two lines collapse? (`computeIfAbsent`, `Math.max`)
  - [ ] Arithmetic/indexing expression untested? (`'a' - c` direction, off-by-one)
- [ ] Trace on the given example. Compute arithmetic explicitly.
- [ ] Trace on **one adversarial case** you invented (duplicates, empty, single, all-same).
- [ ] Trace internal consistency: characters and operators in the trace match across lines.
- [ ] State complexity in work-done terms, **both time AND space**.

---

## My recurring mistakes (update weekly)

### Comprehension
- [ ] Misreading operators (`≤` vs `=`)
- [ ] Substring / subsequence confusion
- [ ] Voice-dictating without proofreading

### Algorithm
- [ ] Stale state — forget to overwrite
- [ ] Unverified length assumptions (longer/shorter)
- [ ] Tangled single loop over different-length collections

### Code style
- [ ] Typos: `maxLegth`, `length`, `doesn't`
- [ ] Character offset inversion: `'a' - char` instead of `char - 'a'`
- [ ] `containsValue()` instead of `containsKey()`
- [ ] Verbose put-or-create instead of `computeIfAbsent`
- [ ] Redundant `if` wrapping a `while`

### Trace
- [ ] Trace shorthand drift (notation doesn't match code)
- [ ] Operator/character mislabeling (write `++`, result goes down)

### Communication
- [ ] Skipping space complexity
- [ ] Hedging on self-evaluation

*(add new ones as discovered)*

---

## Self-eval format (use after every problem)

> *"Caught: [integer]. What I'd need to start catching: [specific habit]."*

Pure "no" or "zero" without a diagnostic action is wasted self-eval.

---

*End of cheat-sheet.*
