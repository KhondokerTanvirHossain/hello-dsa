# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **documentation-only repository** — a personal DSA (Data Structures & Algorithms) interview-prep reference. There is no build system, no source tree, no tests, and no dependencies. Work here is editing prose, tables, and embedded code snippets.

Files, each serving a distinct role (the distinction matters when deciding where new content belongs):

- [README.md](README.md) — **index / landing page**. Short; links to the other files and explains how to use them. Not a content file.
- [cheatsheet.md](cheatsheet.md) — the **condensed cheat-sheet**: dense quick reference, scanned before each problem. New patterns get a terse entry here.
- [handbook.md](handbook.md) — the **living handbook**: fuller explanations, rituals, mindset, Big O reasoning, communication phrases, and a personal recurring-mistakes log. Content here is taught, not just listed.
- `patterns/` — **planned, not yet created**. Per-pattern files (`hashmap.md`, `sliding-window.md`, `two-pointers.md`) to be added as each pattern is drilled. The handbook's appendix and the README both reference this; creating it is deferred intentionally.

The same patterns appear in the cheat-sheet and handbook at different depth (HashMap O(1) lookup, sliding window, two pointers, Java code-quality rules, trace discipline). When adding or correcting a pattern, **keep both files consistent** — update the terse cheatsheet entry and the explanatory handbook section together.

The handbook's owner is described in it as a senior engineer (distributed systems) who is a beginner specifically at competitive DSA; explanations are pitched accordingly.

## Conventions enforced by the cheat-sheet (apply these when editing)

The document is opinionated and self-consistent. Preserve these when adding or modifying content:

- **Code snippets are Java**, and must follow the "Java code quality — quick rules" table in the README: interface types on the LHS (`Map<K,V> m = new HashMap<>()`), `ArrayDeque` over `Stack`, `camelCase`, always-braces, `getOrDefault`/`Math.max` idioms, no variable shadowing, comments explain *why* not *what*.
- **Complexity is stated in "work done" terms**, with the amortized-analysis phrasing the document models (e.g. "each pointer moves at most N times → O(N)").
- **Templates fill named slots.** Sliding-window and two-pointer sections present a reusable skeleton plus the specific decisions to fill in. Keep new patterns in that template-then-canonical-example shape.
- **Traces are whiteboard-ready**: arithmetic computed explicitly, data-structure state shown at every step, non-updates shown too. Match the existing trace format exactly when adding examples.
- The README uses `---` horizontal rules between major sections and GitHub-flavored markdown tables. Keep formatting parallel to neighboring sections.

## Editing notes

- The "My recurring mistakes" section is a living checklist meant to be appended to over time — add to it rather than rewriting it.
- Snippets are illustrative reference code, not a compiled project; there is nothing to run or test. Verify Java correctness by reading, and trace new examples by hand the way the document demonstrates.
