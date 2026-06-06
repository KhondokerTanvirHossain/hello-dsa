# Pattern: HashMap for O(1) Lookup

> Deep dive on the HashMap pattern family. 6 canonical problems, 6 subpatterns.
> When you sit down to interview prep, study this file in full once. After that, return for spot reference.

---

## When to reach for a HashMap

The **mental signature** that should fire instantly:

> *"I'm searching for something inside a loop. Can I store seen items in a HashMap and check in O(1) instead?"*

Search inside a loop → HashMap is the first hypothesis.

You're trading **space for time**: O(N) memory in exchange for O(N²) → O(N) speedup. For a senior backend engineer, this tradeoff is identical to how you use caching in production — same instinct, new vocabulary.

---

## The non-negotiable rule

**Keys are looked up fast. Values are along for the ride.**

If you need to look up something quickly, that thing must be the **key**. This is the most common HashMap mistake among DSA-cold engineers.

| You want to look up...           | Key                          | Value                         |
|----------------------------------|------------------------------|-------------------------------|
| Whether a number was seen        | the number                   | (anything — index, count)     |
| The index of a previous element  | the element                  | its index                     |
| Whether two strings are equal-as-multisets | the multiset signature | the strings sharing it        |
| The frequency of a character     | the character                | its count                     |

**Trap: `containsValue()` is O(N).** It iterates every value. If you find yourself calling it inside a loop, your keys and values are inverted. Swap them.

---

## The six subpatterns

| # | Subpattern                  | Problem                           | Difficulty |
|---|-----------------------------|-----------------------------------|------------|
| 1 | Fast existence lookup       | Two Sum                           | Easy       |
| 2 | Stale-state overwrite       | Contains Nearby Duplicate         | Easy       |
| 3 | HashMap as window state     | Longest Substring No Repeat       | Medium     |
| 4 | Group-by-signature          | Group Anagrams                    | Medium     |
| 5 | Frequency-array fingerprint | Valid Anagram                     | Easy       |
| 6 | Increment/decrement trick   | Ransom Note                       | Easy       |

---

## Subpattern 1 — Fast existence lookup (Two Sum)

> Given an array `nums` and target `target`, return `true` if any two numbers sum to `target`.

### Approach

For each element, compute its **complement** (`target - nums[i]`). If the complement has already been seen, we have a pair. Otherwise, record the current element.

### Code

```java
boolean hasTwoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return true;
        }
        seen.put(nums[i], i);
    }
    return false;
}
```

### Why "check, then put" (not the reverse)

The map only ever contains elements you've **already passed**. This guarantees you never accidentally pair an element with itself. If you put first, then `[3, 3]` with target 6 would falsely match index 0 with itself.

### Edge-case trace: `nums = [3, 3]`, `target = 6`

```
i = 0: nums[0] = 3
       complement = 6 - 3 = 3
       seen.containsKey(3)? No.
       seen.put(3, 0)  →  seen = {3: 0}

i = 1: nums[1] = 3
       complement = 6 - 3 = 3
       seen.containsKey(3)? Yes (from i=0).
       return true ✓
```

### Complexity

- Time: **O(N)** — one pass, O(1) work per element.
- Space: **O(N)** — map holds up to N entries.

---

## Subpattern 2 — Stale-state overwrite (Contains Nearby Duplicate)

> Given `nums` and `k`, return `true` if there exist two distinct indices `i, j` with `nums[i] == nums[j]` and `|i - j| ≤ k`.

### The insight

When you've stored "value → last index seen" and you encounter the value again at index `i`, two things can happen:

1. The stored index is within `k` of `i` → return true.
2. The stored index is **outside** `k` → too far. **But that old index is now useless** for any future element, because future positions will be even further from it. So overwrite with the current index.

This generalizes: **when stored state can no longer help future queries, replace it with fresher state.**

### Code

```java
boolean containsNearbyDuplicate(int[] nums, int k) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        if (seen.containsKey(nums[i]) && i - seen.get(nums[i]) <= k) {
            return true;
        }
        seen.put(nums[i], i);   // overwrite — old index never helps again
    }
    return false;
}
```

### Why overwrite is essential — `nums = [1, 2, 3, 1, 4, 1]`, `k = 2`

Without overwrite, at `i = 5` (third `1`), the map still says `1 → 0`. Distance = 5. > k. Returns false. But indices 3 and 5 are 2 apart — correct answer is true.

With overwrite, the map at i=3 becomes `1 → 3`. At i=5, distance = 5 - 3 = 2. ≤ k. Returns true. ✓

### Complexity

- Time: **O(N)**.
- Space: **O(min(N, U))** where U is the number of distinct values.

---

## Subpattern 3 — HashMap as window state (Longest Substring Without Repeating Characters)

> Given a string `s`, return the length of the longest contiguous substring in which all characters are unique.

### The setup

Two pointers — `left` and `right` — define a sliding window. A `HashSet` (or character frequency map) tracks **what's currently inside the window**. Expand `right` always; shrink `left` whenever the window becomes invalid.

Full sliding-window pattern lives in `sliding-window.md`. This problem is its canonical introduction.

### Code

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

### Complexity argument

The inner `while` does **not** restart `left` from scratch each iteration. Across the whole algorithm, `right` moves N times and `left` moves at most N times. **Total ≤ 2N → O(N).** This is amortized analysis.

Memorize the phrase:

> *"Each pointer moves at most N times in total. So the algorithm performs at most 2N operations — O(N)."*

### Complexity

- Time: **O(N)**.
- Space: **O(min(N, alphabet_size))**.

### Two implementation flavors

| Style                                  | Pros                       | Cons                                |
|----------------------------------------|----------------------------|-------------------------------------|
| HashSet + shrink-one-at-a-time         | Simpler, easier to write   | A few more operations               |
| HashMap of last-index + jump `left`    | Slightly fewer ops         | Trickier; need `max(left, last+1)`  |

Default to the HashSet version. Use the HashMap-jump variant when the problem explicitly motivates it.

---

## Subpattern 4 — Group-by-signature (Group Anagrams)

> Given a list of strings, group together the strings that are anagrams of one another.

### The insight

Two strings are anagrams **iff their character-count arrays are identical**. So the `int[26]` count *is a fingerprint* of the anagram class.

If we convert the fingerprint to a HashMap key, all anagrams collapse onto the same key. The map directly produces the groups.

This is **HashMap for grouping**, not for lookup. Same data structure, different objective. The mental signature:

> *"If two inputs can be reduced to the same canonical form, group them with a HashMap keyed by that form."*

### Code

```java
List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        int[] count = new int[26];
        for (int j = 0; j < s.length(); j++) {
            count[s.charAt(j) - 'a']++;
        }
        StringBuilder keyBuilder = new StringBuilder();
        for (int c : count) {
            keyBuilder.append(c).append('#');
        }
        groups.computeIfAbsent(keyBuilder.toString(), k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

### The `computeIfAbsent` idiom — internalize this

Instead of:

```java
if (groups.containsKey(key)) {
    group = groups.get(key);
} else {
    group = new ArrayList<>();
}
group.add(s);
groups.put(key, group);
```

Write:

```java
groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
```

One call, one HashMap lookup, idiomatic. Every Java interviewer expects to see this in grouping problems.

### Two alternative keys (worth mentioning in interview)

| Key choice                              | Time per string  | Memory          | Notes                              |
|-----------------------------------------|------------------|-----------------|------------------------------------|
| Count array stringified (`"1#0#0#..."`) | O(K)             | O(26) per key   | **Optimal**                        |
| Sorted string (`"eat"` → `"aet"`)       | O(K log K)       | O(K) per key    | Simpler, slightly slower           |

**Senior move in interviews:** code the optimal, mention the alternative aloud — *"I could also sort each string and use that as the key. Simpler code but K log K instead of K per string."*

### Complexity

- N strings, K = max length.
- Time: **O(N × K)** — for each string, O(K) to build count, O(26) = O(1) to build key.
- Space: **O(N × K)** — the result itself.

### Trap

`'a' - c` instead of `c - 'a'`. **Test on `'b' - 'a' = 1`.** Anything else, you've inverted.

---

## Subpattern 5 — Frequency-array fingerprint (Valid Anagram)

> Given two strings `s` and `t`, return `true` if they are anagrams of each other.

### Approach A — Single array, increment/decrement (optimal)

Walk through both strings simultaneously. Increment count for each character in `s`, decrement for each character in `t`. If both strings have the same multiset, the array is **all zeros** at the end.

```java
boolean validAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;
    }
    for (int c : count) {
        if (c != 0) return false;
    }
    return true;
}
```

### Approach B — Two arrays, compare (simpler)

```java
boolean validAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] countS = new int[26];
    int[] countT = new int[26];
    for (int i = 0; i < s.length(); i++) {
        countS[s.charAt(i) - 'a']++;
        countT[t.charAt(i) - 'a']++;
    }
    return Arrays.equals(countS, countT);
}
```

Both are O(N) time, O(1) space. Approach A is one pass, one array — slightly better constants and "more elegant" in an interview. Approach B is **easier to write correctly** under pressure.

**Senior move:** code Approach A, mention Approach B as the simpler alternative.

### Complexity

- Time: **O(N)**.
- Space: **O(1)** because the alphabet size is constant.

---

## Subpattern 6 — Increment/decrement trick (Ransom Note)

> Given two strings `ransomNote` and `magazine`, return `true` if `ransomNote` can be built from `magazine`'s characters (each letter used at most once).

### The insight

This is **sub-multiset containment**, not equality. The magazine must have **at least** the characters required by the ransom note. Strategy:

1. Count every character in the magazine.
2. Walk through the ransom note, decrementing each count.
3. If any count drops below zero, we ran out of that letter — return false.

### Code

```java
boolean canConstruct(String ransomNote, String magazine) {
    int[] count = new int[26];
    for (int i = 0; i < magazine.length(); i++) {
        count[magazine.charAt(i) - 'a']++;
    }
    for (int i = 0; i < ransomNote.length(); i++) {
        if (--count[ransomNote.charAt(i) - 'a'] < 0) {
            return false;
        }
    }
    return true;
}
```

### The `--count[...] < 0` idiom

Reads aloud as: *"decrement the count first, then check if it went negative."* Single expression for two operations. One of the most-used Java moves in counting problems. Memorize.

### Why two loops, not one

Tempting to write a single loop indexed by `i` over both strings. Don't. The strings have **different lengths**, so a shared index either misses characters or runs out of bounds. **Two separate loops** are cleaner, safer, and the canonical pattern:

> *"Count one. Subtract the other. If anything went negative, return false."*

### Complexity

- Let R = `ransomNote.length()`, M = `magazine.length()`.
- Time: **O(R + M)**.
- Space: **O(1)** — count array is bounded at 26.

---

## Side-by-side: when to pick which subpattern

| You need to...                                  | Use                                              |
|-------------------------------------------------|--------------------------------------------------|
| Find if two values pair up by some condition    | Fast lookup (subpattern 1)                       |
| Track the *most recent* index/state of each value | Stale-overwrite (subpattern 2)                 |
| Maintain "what's currently in the window"       | Window state with HashSet/HashMap (subpattern 3) |
| Group inputs by some canonical form             | Group-by-signature (subpattern 4)                |
| Check if two collections are equal as multisets | Frequency array, increment/decrement (subpattern 5) |
| Check if one collection is a sub-multiset of another | Count then subtract, watch for negatives (subpattern 6) |

---

## Traps and gotchas (the ones you've hit)

- **`containsValue()` is O(N).** Inside a loop, that gives O(N²). Use `containsKey`/`get`.
- **Character offset direction.** Always `c - 'a'`, not `'a' - c`. Test on `'b' - 'a' = 1`.
- **Verbose `if-containsKey-else`.** Use `computeIfAbsent` for grouping problems.
- **Length-relationship assumptions.** Don't assume one input is longer than the other. Either ask, or handle both.
- **Tangled single loop over different-length collections.** Two clean loops beat one tangled one.

---

## Variants — quick reference

| Variant         | Java type                  | When                                       |
|-----------------|----------------------------|--------------------------------------------|
| Existence only  | `Set<T>`                   | Don't care about index, count, or value    |
| Value → index   | `Map<T, Integer>`          | Need to know *where* you saw it            |
| Frequency map   | `Map<T, Integer>`          | Counting (anagrams, majority, k-distinct)  |
| Frequency array | `int[26]` or `int[128]`    | Bounded character set                      |
| Prefix-sum map  | `Map<Long, Integer>`       | Subarray-sum-equals-K problems             |

---

## Practice extensions (do these next when you revisit this pattern)

- **Two Sum II** (sorted input) — use two-pointer instead.
- **Two Sum III** (data structure design) — design `add(num)` and `find(target)`.
- **Subarray Sum Equals K** — prefix-sum map.
- **Longest Substring with K Distinct Characters** — window state + size check.
- **Find All Anagrams in a String** — frequency-array + sliding window.
- **Top K Frequent Elements** — frequency map + heap.
- **First Unique Character in a String** — frequency map, two passes.
- **Isomorphic Strings** — two HashMaps (bidirectional check).

When you're comfortable with these, the HashMap pattern is interview-ready.

---

*End of HashMap pattern document.*
