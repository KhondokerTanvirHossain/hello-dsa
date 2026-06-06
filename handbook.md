# DSA Interview Handbook

> Personal reference, built progressively. Re-read weekly. Add your own notes inline.
> Last updated: end of HashMap unit (6 problems completed).

---

## How to Use This Document

This is a **living reference**, not a textbook. Three rules:

1. **Re-read it often.** Twice a week minimum. Spaced repetition is the entire game.
2. **Speak the phrases aloud.** Interview communication is graded. If you can say a complexity justification cleanly in your own voice, you have it.
3. **Add your own notes.** When you discover a new mistake you make, write it under [Recurring Mistakes](#11-recurring-mistakes). Personal mistakes are more valuable than generic ones.

---

## Table of Contents

1. [The Engineer-Beginner Distinction](#1-the-engineer-beginner-distinction)
2. [The Pre-Coding Ritual](#2-the-pre-coding-ritual)
3. [Big O Analysis](#3-big-o-analysis)
4. [Reading Constraints as Hints](#4-reading-constraints-as-hints)
5. [Pattern: HashMap for O(1) Lookup](#5-pattern-hashmap-for-o1-lookup)
6. [Pattern: Sliding Window](#6-pattern-sliding-window)
7. [Pattern: Two Pointers (intro)](#7-pattern-two-pointers-intro)
8. [Java Code Quality Standards](#8-java-code-quality-standards)
9. [Trace Discipline](#9-trace-discipline)
10. [Interview Communication Phrases](#10-interview-communication-phrases)
11. [Recurring Mistakes](#11-recurring-mistakes)
12. [Self-Evaluation Habit](#12-self-evaluation-habit)
13. [Progress Targets](#13-progress-targets)

---

## 1. The Engineer-Beginner Distinction

You are a Senior Software Engineer with six years of distributed systems experience. In **DSA / competitive algorithms**, you are a beginner. In **engineering, architecture, debugging, and system thinking**, you are senior.

This distinction matters because:

- Your DSA gap closes faster than a true beginner's. You already think in invariants, edge cases, and tradeoffs.
- Your engineering instincts bleed into algorithmic work in good ways (variable scoping, method signatures, defensive coding) — but DSA also has **its own vocabulary** that must be drilled.
- Your interview is three-part: **coding (DSA), system design, behavioral**. System design and behavioral will be your strongest rounds. DSA is the gap. 80% of training time goes there.

**Internalize:** you are not learning to think. You are learning the *vocabulary* and *patterns* of one specific domain.

---

## 2. The Pre-Coding Ritual

Before writing one line of code on any problem, do these four steps. **Skipping them is the single biggest cause of failed interviews for engineers like you.**

### Step 1 — Read the problem twice

Out loud if you must. Underline operators: `≤`, `<`, `=`, `>`. They are not interchangeable. *"≤ k"* and *"= k"* are completely different problems.

### Step 2 — Restate in one precise sentence

In your own words, in writing, using the **technical terms from the problem**. Not synonyms.

Strong restatements you've produced:
- *"Given a string, return the length of the longest contiguous slice in which all characters are unique."*
- *"Given two strings, return true if both contain the exact same characters with the same frequencies."*
- *"Check every character from the first string exists with the same or greater frequency in the second string."*

Each uses **technical terms** (contiguous, unique, frequencies, same or greater) — not vague synonyms.

### Step 3 — Walk the given example

Trace through it before designing anything. Confirm your restatement matches the expected output. **This walk often reveals the brute-force algorithm naturally**, so it doubles as algorithm design.

### Step 4 — Ask clarifying questions (value-changing ones only)

A clarifying question is *valuable* only if its answer would change your approach. Junk questions hurt your signal.

**High-value clarifying questions:**

- Can the input be empty or null? What do I return?
- What's the maximum size of the input? (Constraint → complexity budget)
- Character set / value range? (Bounded set → array; unbounded → HashMap)
- Are duplicates allowed? Is the input sorted?
- Two-input problems: equal lengths? One bounded by the other?
- Case-sensitivity for strings? Mixed case?
- What's the output format — boolean, indices, the actual answer?

**State your hypothesis alongside the question** when natural:
- *"Empty input — I'd return true since two empty strings are trivially anagrams. Confirm?"*

This is collaborative, not interrogative. Senior signal.

**Low-value (don't ask):** *"Are all numbers positive?"* — when sign doesn't affect the algorithm.

Only after these four steps: brute force → optimized → code → trace.

---

## 3. Big O Analysis

### The mental model

Big O describes the **shape of growth**, not absolute speed. If input doubles, does time double? Quadruple? Stay the same?

### CPU benchmark (memorize)

> **A modern CPU does roughly 10⁸ simple operations per second.**

This is the conversion factor between "ops" and "seconds". Whenever you compute Big O for a given N, convert it to seconds by dividing by 10⁸.

### Reference table — max N per complexity (1-second budget)

| Complexity   | Max N      | Notes                                  |
|--------------|------------|----------------------------------------|
| O(log N)     | unlimited  | Binary search                          |
| O(N)         | ~10⁸       | Linear scan, sliding window            |
| O(N log N)   | ~10⁶       | Sorting, balanced trees                |
| O(N √N)      | ~10⁵       | Mo's algorithm, sqrt decomposition     |
| O(N²)        | ~10⁴       | Pairwise loops, basic DP               |
| O(N³)        | ~500       | Floyd-Warshall, interval DP            |
| O(2ᴺ)        | ~25        | Bitmask, subset enumeration            |
| O(N!)        | ~10        | Permutation brute force                |

### Work-done justification (not "nested loop")

When stating complexity, justify it in terms of **work done**, not code shape.

| Weak (junior)                       | Strong (senior)                                                            |
|-------------------------------------|----------------------------------------------------------------------------|
| "O(N²) because nested loops"        | "For each of N elements, we compare against N others → N × N = N² ops"     |
| "O(N) because one loop"             | "We visit each element once and do O(1) work per visit → O(N) total"       |
| "O(N log N) because sorting"        | "Sort dominates: N log N for sorting + N for the scan → O(N log N)"        |

**Always frame complexity as work done.** This is the single most important communication habit for the coding round.

### Always state space complexity too

Time complexity alone is incomplete. Always include space:

> *"Time: O(R + M) — each string traversed once. Space: O(1) — count array is bounded at 26 regardless of input size."*

The **O(1) for a bounded character set** is interview gold. Saying "O(26)" is technically correct but signals less polish. Use **"O(1) because the alphabet size is constant"**.

### Hidden costs to watch for

- `HashMap.containsValue()` — **O(N)**, not O(1). Only keys are fast.
- `list.contains(x)` for ArrayList — **O(N)** linear scan.
- `String + String` in a loop — each concatenation is O(N), making the loop O(N²). Use `StringBuilder`.
- Calling `.length()`, `.size()`, `.charAt()` — these are O(1) in Java, no hidden cost.

---

## 4. Reading Constraints as Hints

The problem's constraint on N is a **direct hint** about which algorithm class is intended.

| Constraint        | Likely target complexity     | Typical techniques                                       |
|-------------------|------------------------------|----------------------------------------------------------|
| N ≤ 20 to 25      | O(2ᴺ) — exponential intended | Bitmask, backtracking, subset generation                 |
| N ≤ 500           | O(N³)                        | Floyd-Warshall, interval DP, matrix chain                |
| N ≤ 5000          | O(N²)                        | Pairwise DP, LCS, edit distance                          |
| N ≤ 10⁵           | O(N log N) or O(N)           | Sorting, binary search, two-pointer, sliding window      |
| N ≤ 10⁶ to 10⁷    | O(N) or O(N log log N)       | Linear scan, sieve                                       |
| N ≤ 10⁹           | O(log N) or O(1)             | Math, formula, matrix exponentiation                     |

**Reflex to build:** the moment you read the constraints, your brain should propose an algorithm class before you finish reading the problem twice.

**Two-dimensional inputs:** when the input has size N × K (e.g., N strings of length K), always think about the **product**. Group Anagrams: N=10⁴, K=100 → N×K = 10⁶. Brute force N²×K = 10¹⁰ dies; HashMap-by-signature does O(N×K) = 10⁶ easily.

---

## 5. Pattern: HashMap for O(1) Lookup

### Mental signature

> *"I'm searching for something inside a loop. Can I store seen items in a HashMap and check in O(1) instead?"*

Whenever you see search inside a loop → HashMap is the **first hypothesis**.

### Key insight

**Keys are looked up fast. Values are along for the ride.** If you need to look up something quickly, it must be the **key**.

### Subpatterns (all six HashMap problems fall into these)

| Subpattern                  | Example                       | Key insight                                                        |
|-----------------------------|-------------------------------|---------------------------------------------------------------------|
| Fast lookup                 | Two Sum                       | `containsKey` is O(1); `containsValue` is O(N)                      |
| Stale-state overwrite       | Contains Nearby Duplicate     | Old stored data may be useless; overwrite with fresher              |
| HashMap as window state     | Longest Substring No Repeat   | HashMap holds window contents; same shape as a Set + count          |
| Group-by-signature          | Group Anagrams                | If two inputs produce the same canonical form, group them by key    |
| Frequency array fingerprint | Group Anagrams, Valid Anagram | `int[26]` of counts is a fingerprint of any anagram class           |
| Increment/decrement trick   | Valid Anagram, Ransom Note    | Single counter array: +1 for one input, -1 for the other            |

Full templates and examples live in `patterns/hashmap.md`.

### Variants quick reference

| Variant         | Type                       | When                                       |
|-----------------|----------------------------|--------------------------------------------|
| Existence only  | `Set<T>`                   | Don't care about index or count            |
| Value → index   | `Map<T, Integer>`          | Need *where* you saw it                    |
| Frequency map   | `Map<T, Integer>`          | Counting (anagrams, majority, k-distinct)  |
| Frequency array | `int[26]` or `int[128]`    | Bounded character set — faster than HashMap|
| Prefix-sum map  | `Map<Long, Integer>`       | Subarray-sum problems                      |

### Traps

- **`containsValue()` is O(N).** Never use inside a loop. If you find yourself reaching for it, your keys and values are inverted — swap them.
- **`HashMap` vs. `Map` on the LHS.** Declare with the interface: `Map<K,V> m = new HashMap<>();`. Idiomatic Java.
- **Autoboxing cost.** `Map<Integer, Integer>` boxes every key/value. For bounded character sets, prefer `int[26]` or `int[128]` — faster, smaller, no boxing.
- **Character offset direction.** `c - 'a'` gives 0..25 for `a..z`. Test on `'b' - 'a' = 1` before submitting.
- **Verify length-relationship assumptions.** "Longer string contains shorter" must be confirmed with the interviewer or handled in code. Don't assume.

---

## 6. Pattern: Sliding Window

### When to use

You're looking at **contiguous subarrays or substrings** and asking one of:

- *"Longest / shortest / count of subarrays satisfying property P?"*
- *"Best value over a window of size k?"*

If the problem mentions *substring*, *subarray*, *contiguous*, or *window* — sliding window is the first hypothesis.

### The canonical template (memorize this code shape)

```java
int slidingWindow(int[] arr) {
    int left = 0, answer = 0;
    // some window state, e.g. a Set, frequency map, or running sum

    for (int right = 0; right < arr.length; right++) {
        // 1. EXPAND: add arr[right] to the window state

        while (windowIsInvalid()) {
            // 2. SHRINK: remove arr[left] from the window state
            left++;
        }

        // 3. UPDATE the answer (window [left..right] is now valid)
        answer = Math.max(answer, right - left + 1);
    }
    return answer;
}
```

**Three slots to fill** for any sliding-window problem:

1. **Expand step:** what state do I update when including `arr[right]`?
2. **Invalidity condition:** what makes the window invalid?
3. **Update step:** what do I track — the length, the count, the best value?

### Why it's O(N), not O(N²)

It *looks* like a nested loop, but the inner `while` doesn't restart `left` from scratch. **Each pointer moves at most N times across the entire algorithm.** Total moves ≤ 2N → O(N).

**Phrase for interviews (memorize):**

> *"Even though both pointers traverse the array, each pointer moves at most N times in total. So the algorithm performs at most 2N operations — O(N). This is amortized analysis."*

### Canonical example — Longest Substring Without Repeating Characters

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

Full unit and variants will live in `patterns/sliding-window.md`.

---

## 7. Pattern: Two Pointers (intro)

### Mental signature

You have a **linear structure** (array, string, linked list) and need to examine **pairs of positions** related by some condition. Two pointers either:

- **Converge** from both ends (sorted array, target sum, palindrome check).
- **Move forward together** at different speeds (cycle detection, sliding window — a special case).
- **One stays, one moves** (partition problems).

### The sorted-array convergence template

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

To be expanded in the formal two-pointer unit: 3Sum, Container With Most Water, Trapping Rain Water.

---

## 8. Java Code Quality Standards

Interview-quality Java has rules. Internalize all of them.

### Declarations

```java
Map<Integer, Integer> seen = new HashMap<>();   // interface on the LHS
Set<Character> seen = new HashSet<>();
List<Integer> result = new ArrayList<>();
Deque<Integer> stack = new ArrayDeque<>();      // ArrayDeque, NOT Stack class
Queue<Integer> queue = new ArrayDeque<>();      // same
```

### Naming

- **`camelCase`** for variables and methods. `maxLength`, not `maxlength`.
- **Meaningful names tell purpose, not type.** `seen` > `map`. `frequencies` > `m`. `complement` > `t`.
- **Single letters okay** for loop indices (`i`, `j`, `k`), pointers (`left`, `right`), and short-scope variables.
- **Never shadow loop variables.** Don't write `for (int i = 0; ...) { for (int i = 0; ...) }`. Use `j`.

### Braces — always

```java
if (condition) {            // ✓ braces even on single-line
    return true;
}

if (condition) return true; // ✗ industry style guides forbid this
```

This is **Google Java Style**, the standard at most top companies. Burn it in.

### Idioms (every one of these high-frequency in interviews)

```java
// For-each when index not needed
for (String s : strs) { ... }

// Running max
maxLength = Math.max(maxLength, candidate);

// Sentinel-default lookup (faster than containsKey + get)
int idx = map.getOrDefault(key, -1);
if (idx != -1) { ... }

// Get-or-create-then-update (one call, two operations)
groups.computeIfAbsent(key, k -> new ArrayList<>()).add(value);

// Decrement-then-check (counting problems)
if (--count[c - 'a'] < 0) return false;
```

### Things NOT to do

- Don't cache `arr.length` into a local variable for "performance" — JIT handles it. Adds noise.
- Don't write comments that restate the code. `// increment i` is noise. Comments explain *why*, not *what*.
- Don't use raw types (`HashMap map = new HashMap()`). Always parameterize generics.

---

## 9. Trace Discipline

### Why trace

Interviewers grade your trace as heavily as your code. A clean trace:

- Catches your own bugs before the interviewer does.
- Proves you understand your own algorithm (not just remember it).
- Demonstrates clear communication.

### The whiteboard-ready format

```
nums = [1, 0, 1, 1], k = 1
seen = {}

i = 0: nums[0] = 1
       j = seen.getOrDefault(1, -1) = -1
       j == -1, skip the if
       seen.put(1, 0)  →  seen = {1: 0}

i = 1: nums[1] = 0
       j = -1, skip
       seen = {1: 0, 0: 1}

i = 2: nums[2] = 1
       j = 0, i - j = 2 - 0 = 2.  Is 2 <= 1?  NO.
       seen.put(1, 2)  →  seen = {1: 2, 0: 1}     (overwrite stale index)

i = 3: nums[3] = 1
       j = 2, i - j = 3 - 2 = 1.  Is 1 <= 1?  YES.
       return true ✓
```

### Trace rules

1. **Compute arithmetic explicitly.** `2 - 0 = 2`, not `2 - 0 = 0`. Mistakes here destroy interviewer confidence.
2. **Show every variable at every step.** All of them. Especially the data structure state.
3. **Show non-updates too.** If `maxLength` didn't change at step k, *write that*. It proves you considered the case.
4. **Pick examples that exercise the tricky path.** Don't trace the easy case; trace one that involves shrinking, overwriting, or an edge condition.
5. **Internal consistency — the persistent gap.** Every character, operator, and arithmetic step must match every other line.
   - If you write `'r'`, the next line's subtraction must use `'r'`.
   - If you write `++`, the result must go up by 1. If `--`, down by 1.
   - **Run a top-to-bottom consistency read before sending the trace.**

### The 60-second self-review (do before declaring code "done")

1. **Read your code line by line as if you'd never seen it.**
2. Ask three questions:
   - *"Is anything in here doing nothing?"* (redundant `if`s, double lookups)
   - *"Could two lines collapse into one?"* (`computeIfAbsent`, `Math.max`)
   - *"Is there an arithmetic or indexing expression I haven't tested on a tiny case?"* (`'a' - char` direction, off-by-one)

This habit is **the** thing that moves your "self-eval = 0" toward "self-eval = 2 of 3".

---

## 10. Interview Communication Phrases

These are the exact phrases that win senior coding rounds. Practice saying them aloud.

### When clarifying

- *"Before I start designing, can I confirm a few things..."*
- *"Can the input be empty or null? If so, what's the expected return?"*
- *"What's the maximum size of N? I want to make sure my approach fits."*
- *"Are the values bounded? For example, lowercase letters only, or full Unicode?"*
- *"Empty input — I'd return true since two empty strings are trivially anagrams. Confirm?"*
- *"Can I assume the two inputs have equal length, or should I handle unequal?"*

### When proposing brute force

- *"My first approach is brute force, just so we agree on correctness, then I'll optimize."*
- *"For each of N elements, I compare against N others, so the total work is N² — that's O(N²)."*

### When optimizing

- *"I notice the brute force re-checks information I already computed. I want to avoid that by..."*
- *"I can trade memory for time here. If I store X in a HashMap, my lookup becomes O(1)."*
- *"Two strings produce the same int[26] iff they're anagrams — I can use that as a group key."*
- *"This is a classic sliding-window setup — let me walk through it."*

### When justifying complexity (both time AND space)

- *"Time: O(N) — single pass, O(1) work per element. Space: O(1) because the alphabet size is constant."*
- *"Each pointer moves at most N times in total, so the combined work is at most 2N — that's O(N). This is amortized analysis."*
- *"Sorting dominates: O(N log N) for the sort plus O(N) for the scan, giving O(N log N) overall."*
- *"For two inputs of sizes R and M: O(R + M) time, O(1) space."*

### When complexity is borderline

- *"This is O(N³) with N up to 1000, which gives 10⁹ ops — that's borderline. Should work in C++, may TLE in Java. If you'd like, I can find an O(N²) approach."*

### When done

- *"Let me trace through the given example to verify."* (Then actually do it.)
- *"Edge cases I want to check: empty input, single element, all same values, and the maximum-size case."*

### Mentioning alternative approaches

- *"I could have used two arrays (one count per string then compare), but I noticed I can do it in one pass with the increment/decrement trick."*
- *"An alternative is to sort each string and use the sorted string as the key — simpler but O(N×K log K) instead of O(N×K)."*

### When stuck

- *"I'm not seeing the optimal approach yet — let me think about what extra information would help me avoid rework."*
- *"Can I ask a clarifying question to narrow this down?"*

Never go silent for more than 30 seconds. Narrate your thinking, even partially. Silence is the #1 reason candidates fail rounds they could have passed.

---

## 11. Recurring Mistakes

Your personal log. Add to it whenever you catch yourself making one.

### Comprehension mistakes (highest frequency early on)

- **Misreading operators.** `≤ k` is not `= k`. `<` is not `≤`. Underline these.
- **Substring vs. subsequence.** Substring = contiguous. Subsequence = picked-from-anywhere.
- **Length vs. indices.** Read the output specification carefully.
- **Vague restatements.** "Different from what?" Be precise: "all unique within the substring".
- **Voice-dictating problem statements without proofreading.** Mangles operators and key terms.

### Algorithmic mistakes

- **Stale state in HashMap.** If a stored entry is no longer useful for future queries, overwrite it.
- **Self-comparison in pair searches.** `nums[i] == nums[j]` with `i == j` is trivially true. Use `j = i + 1` or check `i != j`.
- **`containsValue()` inside a loop.** O(N) inside O(N) = O(N²). Use keys.
- **Unverified length-relationship assumptions.** "Longer string contains shorter" must be confirmed or handled.
- **Tangled loops over different-length collections.** Use two separate clean loops instead of one shared-index loop.

### Code-style mistakes

- **Variable name typos.** `maxLegth` → `maxLength`. Proofread.
- **Recurring word typos.** *"length", "doesn't"* — proofread before sending.
- **Character offset inversion.** `'a' - char` instead of `char - 'a'`. **Test on `'b' - 'a' = 1`.**
- **Redundant `if` around `while`.** A `while` already does nothing if its condition is false on the first check.
- **Verbose put-or-create.** Use `computeIfAbsent` instead of explicit `if-containsKey-else`.
- **Missing braces** on single-line `if`s. Always braces.

### Trace mistakes

- **Trace shorthand drift.** Notation in trace doesn't match what code literally does (`"a:1#e:1"` vs. `"1#0#0#0#1#..."`).
- **Operator/character mislabeling.** Writing `++` when code does `--`, or labeling `t[5]='r'` then computing with `'a'`. **Do a top-to-bottom consistency read of the trace before sending.**

### Communication mistakes

- **"Nested loop" as Big O justification.** Use work-done framing.
- **"Little better" when comparing complexities.** Quantify: at N = 10⁶, the difference is 50,000×.
- **Missing space complexity.** Always state both time AND space.

### Self-evaluation mistakes

- **Hedging** ("one or two"). Give a number.
- **"No" without diagnosis.** Better: *"Caught: 0. What I'd need to start catching: [specific habit]."*

---

## 12. Self-Evaluation Habit

After every solved problem, answer two questions honestly:

1. **What did I get wrong before being told?**
2. **What would I have missed entirely if no one had pointed it out?**

Write the answers down. Track them over time. **The mistakes you can name are the mistakes you can stop making.**

**Upgrade your self-eval format:**

> *"Caught: 0. What I'd need to start catching: [specific habit]."*

Naming the habit makes it actionable. Pure "no" doesn't.

This habit is non-negotiable. It is the single highest-leverage practice in interview prep.

---

## 13. Progress Targets

Honest timeline curve, assuming 4 hours/day at 80% consistency:

| Phase            | What you can do                                                        |
|------------------|------------------------------------------------------------------------|
| Week 1 (now)     | Two Sum / Contains Duplicate II / sliding window first attempt         |
| Week 3           | Medium HashMap & sliding-window problems in ~20 min total              |
| Month 2          | Most LeetCode mediums in 15–20 min, including trace                    |
| Month 3          | Pattern-recognize unseen mediums in under 90 seconds                   |
| Month 5          | LeetCode hards within 35–45 min; mock interviews go well               |
| Month 6+         | FAANG-grade coding round + your existing senior-grade system design    |

**Per-problem time targets:**

| Difficulty          | Now      | Week 3   | Month 2  | Month 3  |
|---------------------|----------|----------|----------|----------|
| Easy (Two Sum class)| 30 min   | 8 min    | 4 min    | 2 min    |
| Medium (sliding win)| 40 min   | 20 min   | 12 min   | 8 min    |
| Hard                | n/a yet  | 45 min   | 35 min   | 25 min   |

The path is **volume + the templates in this document**, applied repeatedly to variations.

---

## Appendix — File Map for This Repo

- `handbook.md` — this file. Meta-lessons, rituals, mindset, Big O, code quality.
- `cheatsheet.md` — condensed quick reference.
- `patterns/`
  - `hashmap.md` — HashMap pattern deep dive (6 problems, all subpatterns).
  - `sliding-window.md` *(coming)*
  - `two-pointers.md` *(coming)*

---

*End of handbook.*
