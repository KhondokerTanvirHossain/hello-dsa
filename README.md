# DSA Cheat-Sheet

> Dense quick reference. Scan before every problem. Re-read before every mock interview.

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
   - Output: boolean / indices / value?

Then: brute force → optimized → code → trace.

---

## HashMap pattern

**Signature:** searching inside a loop → HashMap is the first hypothesis.
**Rule:** what you look up *fast* must be the **key**.

### Two Sum (boolean)

```java
boolean hasTwoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) return true;
        seen.put(nums[i], i);
    }
    return false;
}
```

**Order rule:** check first, then put. Map only contains elements *already seen*.

### Contains Nearby Duplicate (stale-index lesson)

```java
boolean containsNearbyDuplicate(int[] nums, int k) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        if (seen.containsKey(nums[i]) && i - seen.get(nums[i]) <= k) return true;
        seen.put(nums[i], i);   // overwrite — stale indices are useless
    }
    return false;
}
```

**Lesson:** when a stored entry can no longer help future queries, overwrite with the fresher one.

### Variants

| Variant         | Type                       | When                                       |
|-----------------|----------------------------|--------------------------------------------|
| Existence only  | `Set<T>`                   | Don't care about index or count            |
| Value → index   | `Map<T, Integer>`          | Need *where* you saw it                    |
| Frequency map   | `Map<T, Integer>`          | Counting (anagrams, majority, k-distinct)  |
| Prefix-sum map  | `Map<Long, Integer>`       | Subarray-sum problems                      |

### Traps

- `containsValue()` is **O(N)**. Never use inside a loop.
- Autoboxing cost: for bounded character sets, prefer `int[26]` / `int[128]`.

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
int lengthOfLongestSubstring(String s) {
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
}
```

### Complexity argument (say this aloud)

> *"Each pointer moves at most N times across the entire algorithm. Total work ≤ 2N → O(N). This is amortized analysis."*

---

## Two pointers — sorted convergence template

```java
boolean hasPairWithSum(int[] sortedNums, int target) {
    int left = 0, right = sortedNums.length - 1;
    while (left < right) {
        int sum = sortedNums[left] + sortedNums[right];
        if (sum == target) return true;
        if (sum < target) left++;
        else right--;
    }
    return false;
}
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
| `Math.max(a, b)` for running max              | one line, idiomatic                             |
| `getOrDefault` for sentinel lookup            | one map call instead of two                     |
| Comments explain *why*, not *what*            | no `// increment i`                             |

---

## Trace format (whiteboard-ready)

```
nums = [1, 0, 1, 1], k = 1
seen = {}

i = 0: nums[0] = 1
       j = -1, skip
       seen = {1: 0}

i = 1: nums[1] = 0
       j = -1, skip
       seen = {1: 0, 0: 1}

i = 2: nums[2] = 1
       j = 0, i - j = 2, 2 <= 1? NO
       seen = {1: 2, 0: 1}   (overwrite)

i = 3: nums[3] = 1
       j = 2, i - j = 1, 1 <= 1? YES
       return true ✓
```

**Rules:**
- Compute arithmetic explicitly (`2 - 0 = 2`, not `0`).
- Show data structure state at every step.
- Show non-updates too.
- Pick examples that exercise the tricky path.

---

## Communication phrases (speak aloud daily)

**Complexity justification:**
- *"For each of N elements, I do O(1) work, so total is O(N)."*
- *"Each pointer moves at most N times total — amortized O(N)."*
- *"Sort dominates: O(N log N) for sort plus O(N) for scan."*

**Tradeoffs:**
- *"I'm trading O(N) memory for O(N) time vs. O(N²) brute force."*
- *"Lowercase-only would let me use int[26] instead of a HashMap — faster, no boxing."*

**Borderline:**
- *"N³ with N=1000 is 10⁹ ops — borderline. C++ fine, Java may TLE."*

**Stuck:**
- *"Let me think about what information I'm recomputing that I already have."*
- *"Can I ask a clarifying question to narrow this?"*

**Never** go silent > 30 seconds. Narrate partial thinking.

---

## Code self-review checklist (before declaring done)

- [ ] Read problem one more time. Match every constraint.
- [ ] Trace on the given example. Compute arithmetic explicitly.
- [ ] Trace on **one adversarial case** you invented (duplicates, empty, single, all-same, edges).
- [ ] Re-read code as if it were a junior's PR. Anything redundant? Over-nested? Two lines collapsible into one?
- [ ] State the complexity in work-done terms. Justify it aloud.

---

## My recurring mistakes (update weekly)

- [ ] Misreading operators (`≤` vs `=`)
- [ ] Substring / subsequence confusion
- [ ] `maxLegth` and similar typos
- [ ] `containsValue()` instead of `containsKey()`
- [ ] Hedging on self-evaluation ("one or two")
- [ ] Skipping the trace when code "looks right"
- [ ] Redundant `if` wrapping a `while`

*(add new ones as discovered)*

---

*End of cheat-sheet.*
