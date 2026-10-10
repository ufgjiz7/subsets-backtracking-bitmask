# subsets leetcode: 4 ways to solve it, the backtracking template that also handles Subsets II, and when each one breaks

The Subsets problem (LeetCode 78) is usually the first time you meet a problem whose output is exponential by definition. That trips people up, because the instinct trained by every other problem is to look for a smarter algorithm. There isn't one here. You have to produce 2ⁿ lists, so 2ⁿ work is the floor, and the actual skill being tested is whether you can generate all of them without duplicates, without mutating a list you already handed to the result, and without writing code that falls apart the moment the input contains repeated numbers.

That last part is what separates a solution you memorized from a pattern you actually own. Subsets II (LeetCode 90) is the same problem with one pruned line added.

## What the problem actually asks

Given an array of unique integers, return every possible subset — including the empty one and the full array.


Input:  nums = [1,2,3]
Output: [[], [1], [2], [3], [1,2], [1,3], [2,3], [1,2,3]]


Order doesn't matter. What does matter is this constraint:


1 <= nums.length <= 10


Ten elements means at most 1,024 subsets. Compare that to 300-length array problems where you need an O(n log n) solution or you time out. Here, brute-force recursion over every include/exclude decision is the intended answer. When you see small bounds on the input but an exponential output, that's the problem telling you which tool to reach for.

## The mental model: a decision tree, not a search

Every element gets two fates — it's in the subset, or it isn't. Run that decision down the array and you get a binary tree with n levels and 2ⁿ leaves. Each root-to-leaf path is one subset.

That framing matters more than the code. Once you see it as a tree, these questions answer themselves:

- **How many subsets?** 2ⁿ, one per leaf.
- **Why does the recursion depth equal n?** Because the depth is the number of decisions you've made.
- **Why is there no base case you have to think hard about?** Because the tree ends when you run out of elements, not when you "find" something.

The two approaches below are the same tree traversed in slightly different shapes. Both are correct. One generalizes further.

## Solution 1: include/exclude recursion

The most direct translation of the tree. At index `i`, branch into taking `nums[i]` or skipping it.

python
def subsets(nums):
    res = []
    path = []

    def dfs(i):
        if i == len(nums):
            res.append(path[:])   # copy — not path
            return
        # branch 1: skip nums[i]
        dfs(i + 1)
        # branch 2: take nums[i]
        path.append(nums[i])
        dfs(i + 1)
        path.pop()

    dfs(0)
    return res


The `path[:]` is not optional. Append `path` itself and every entry in `res` ends up pointing at the same list, so your output becomes eight copies of whatever `path` held at the end. This is the single most common bug in this problem, and it produces output that looks almost right, which makes it worse.

You can also record the subset at every node instead of only at the leaves — you get the same 2ⁿ results, just appended in a different order:

python
def dfs(i):
    res.append(path[:])          # record current state, every node
    for j in range(i, len(nums)):
        path.append(nums[j])
        dfs(j + 1)
        path.pop()


That version is quietly the more useful one.

## Solution 2: the start-index backtracking template

Same tree, written as "try each candidate from index `start` onward," with the record step at the top of the function:

python
def subsets(nums):
    res = []
    path = []

    def dfs(start):
        res.append(path[:])
        for i in range(start, len(nums)):
            path.append(nums[i])
            dfs(i + 1)          # i + 1, not start + 1
            path.pop()

    dfs(0)
    return res


Two details worth burning in:

- **`dfs(i + 1)`, not `dfs(start + 1)`.** Using `start + 1` lets the loop re-add elements you already moved past, and you'll generate duplicates like `[1,2]` and `[2,1]`. If your subsets output has more than 2ⁿ entries, this is why.
- **`i + 1` is what makes it a combination, not a permutation.** Passing `i + 1` means "only look forward." For permutations, you'd instead track a `used` array and loop from 0 every time.

This template is the one to keep, because three other problems are the same skeleton with one line changed:

| Problem | Change to the template |
| --- | --- |
| Subsets II (#90) | Sort first, add `if i > start and nums[i] == nums[i-1]: continue` |
| Combination Sum (#39) | Recurse with `dfs(i)` instead of `dfs(i+1)` — reuse allowed; stop when a running sum hits target |
| Permutations (#46) | Loop from 0 with a `used[]` array instead of a start index |

If you only learn one of the two solutions above, learn this one.

## Solution 3: iterative doubling

You can skip recursion entirely. Start with `[[]]`, then for each number, take everything already in the result and append a copy with that number added.

python
def subsets(nums):
    res = [[]]
    for n in nums:
        res += [subset + [n] for subset in res]
    return res


Walk it with `[1,2,3]`: `[[]]` → `[[], [1]]` → `[[], [1], [2], [1,2]]` → eight subsets. The list comprehension iterates over a snapshot of the old `res` while `+=` extends it — in Python the right-hand side is fully evaluated first, so this is safe. In languages where appending to a list you're iterating mutates it mid-loop, capture the current length before the inner loop instead.

Four lines, no stack depth concerns, and it's easy to explain out loud in an interview. It's also the version that's hardest to extend to Subsets II.

## Solution 4: bitmask enumeration

Since n ≤ 10, you can treat each integer from 0 to 2ⁿ−1 as a subset. Bit `i` set means `nums[i]` is included.

python
def subsets(nums):
    n = len(nums)
    return [[nums[i] for i in range(n) if mask & (1 << i)]
            for mask in range(1 << n)]


This is elegant and completely useless in a follow-up, because Subsets II breaks it: with duplicates in the input, two different masks can produce identical lists, and there's no clean place to prune. Use it to show you understand the 2ⁿ structure, not as your default answer.

## Complexity, and where people misstate it

| Approach | Time | Auxiliary space |
| --- | --- | --- |
| Include/exclude recursion | O(n × 2ⁿ) | O(n) recursion depth + O(n) current path |
| Start-index backtracking | O(n × 2ⁿ) | O(n) |
| Iterative doubling | O(n × 2ⁿ) | O(n) per built subset |
| Bitmask | O(n × 2ⁿ) | O(n) |

The `n` factor comes from copying each subset into the result — 2ⁿ subsets, each up to n long. Quoting "O(2ⁿ)" as the time complexity is technically describing the number of recursive calls, not the cost of the algorithm, and interviewers do notice. Space is O(n) for the stack and the working path if you exclude the output itself; including the output, storing the answer needs O(n × 2ⁿ).

## Subsets II: the line that changes everything

Now the same array is allowed to contain duplicates, and the output must not.

**Subsets II (#90):** `nums = [1,2,2]` → `[[], [1], [1,2], [1,2,2], [2], [2,2]]`. Six subsets, not eight.

python
def subsetsWithDup(nums):
    nums.sort()                     # required
    res = []
    path = []

    def dfs(start):
        res.append(path[:])
        for i in range(start, len(nums)):
            if i > start and nums[i] == nums[i - 1]:
                continue            # skip duplicates at this level only
            path.append(nums[i])
            dfs(i + 1)
            path.pop()

    dfs(0)
    return res


Two things are load-bearing, and both are commonly gotten wrong.

**The sort.** Without it, duplicates aren't adjacent, so `nums[i] == nums[i-1]` never fires and you emit `[1,2]` twice from different positions. Sorting is what turns "same value" into a detectable event.

**`i > start`, not `i > 0`.** The check is scoped to the current level of the recursion. Writing `if i > 0 and nums[i] == nums[i-1]: continue` skips duplicates globally and silently drops valid subsets — with `[1,2,2]` you'd lose `[2,2]` and `[1,2,2]`, because the second `2` gets skipped even in the branch where it's legitimately the first pick at its level. The distinction is: skip a duplicate if you've already explored a branch that started with that same value *at this level*. Deeper in the tree, taking it is fine and necessary.

> If your Subsets II output is missing the subsets that contain repeated values, the `i > start` vs `i > 0` boundary is almost always the culprit. It's the number one mistake on this problem.

## Ordering the practice so it sticks

Solving Subsets once and moving on leaves you re-deriving backtracking from scratch every time it shows up. A sequence that actually builds the pattern:

1. **Subsets (#78)** — get the tree, the copy, and the `i + 1`.
2. **Subsets II (#90)** — add sorting and the level-scoped skip.
3. **Combination Sum (#39)** and **Combination Sum II (#40)** — same skeleton, target sum as the stop condition, with and without reuse.
4. **Permutations (#46)** and **Permutations II (#47)** — same skeleton, `used[]` instead of a start index.
5. **Palindrome Partitioning (#131)**, **Non-decreasing Subsequences (#491)**, **Letter Combinations of a Phone Number (#17)** — the same shape wearing different clothes.

Five problems in, the pattern stops feeling like a pattern and starts feeling like a tool.

Something worth knowing if you're choosing where to practice: AlgoMonster publishes standalone write-ups for individual LeetCode problems that are readable without an account, and the Subsets page is one of them — it works through the decision tree, all major languages, and the complexity analysis in the same depth as the paid curriculum. Their Subsets II write-up is also public and it's where the `i > start` subtlety gets explained properly. If you're stuck on exactly one problem, you can often just read that page and move on. 👉 [Check what's available on the free tier here](https://algo.monster?fpr=li69)

## Where a structured course fits, and where it doesn't

AlgoMonster is built by ex-Google engineers around 48 explicit coding patterns rather than a problem bank — roughly 231 lessons and problems, ~700 interactive diagrams, decision flowcharts for "which pattern is this?", a Speedrun mode for timed review, and an AI assistant. Solutions come in Python, Java, JavaScript, C++, Go, and Haskell.

For Subsets specifically, the value isn't a better explanation of this one problem — it's placement. Backtracking sits inside the DFS/recursion pattern block, so you meet it with the template already in hand, then hit Subsets II, Combination Sum, and Permutations as variations rather than as unrelated puzzles. If your failure mode is "I can write the code but I don't know which approach a new problem wants," that's the gap this style of prep targets.

The limits are real, and third-party reviews are consistent about them. There's no video content at all — everything is text and diagrams, which is a dealbreaker if you learn by watching someone code. System design coverage is introductory and not enough for senior-level loops. There's no live mock interview, no mentor, and no timed pressure simulation, so pattern recognition gets built but interview-day pressure doesn't. Several reviewers also note that 231 problems is thin for volume practice and most candidates still pair it with LeetCode's company-specific sets.

### Current plans and pricing

AlgoMonster has no monthly option. At the time of writing, the subscribe page lists two paid plans plus a free tier:

| Plan | What you get | Price | Billing | Link |
| --- | --- | --- | --- | --- |
| Free | Limited access to sample patterns and problems | $0 | — | [Start with the free tier](https://algo.monster?fpr=li69) |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; real company questions updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps | $69.99/year (listed as $5.84/mo) | Billed annually, cancel or upgrade anytime | [Get Pro – Yearly](https://algo.monster?fpr=li69) |
| Pro – Lifetime | Everything in the annual plan plus lifetime updates, one payment | $89 | Billed once, yours forever | [Get Pro – Lifetime](https://algo.monster?fpr=li69) |

A caveat you should take seriously: AlgoMonster runs sales constantly, and older third-party write-ups quote different numbers — $99/year and $189 lifetime, or $119–$299/yr and $459 lifetime depending on when they were published. Those are snapshots from different sale periods, not a current comparison. What the page shows you at checkout is the only price that counts, and the gap between a $69.99 annual and an $89 lifetime is small enough that the lifetime plan is the obvious pick if you plan to interview more than once.

## So which one do you actually need?

- **You're stuck on Subsets or Subsets II right now.** Read the free problem write-up, fix the `path[:]` copy and the `i > start` condition, and you're done. You don't need a subscription to solve one problem.
- **You can solve Subsets but not recognize when to use backtracking.** That's the pattern-identification gap, and it's what the paid curriculum is for. A yearly plan covers the 4–8 week window most candidates prep in.
- **You're interviewing repeatedly over the next few years.** Lifetime at $89 is roughly one and a quarter years of the annual plan, and it includes future content.
- **You need video walkthroughs, live mock interviews, or deep system design.** Look elsewhere, or expect to pay for a second resource. AlgoMonster doesn't cover those.

One last thing worth internalizing about this problem: the reason Subsets shows up so early in interview prep is that it makes exponential output feel normal. Once you've written a recursion that deliberately produces 2ⁿ results and copies each one correctly, backtracking stops being scary and becomes something you reach for — which is exactly what the four problems after it are testing.
