# Search Results Density Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Separate the search input from the results panel by `24px` and reduce single-line result rows to approximately `50px` without changing Quartz search behavior.

**Architecture:** Keep the existing Quartz search component and override only its presentation in `quartz/styles/custom.scss`. Use explicit high-specificity selectors so Quartz Themes cannot restore the joined panel or oversized result rows, then verify both the stylesheet contract and computed browser dimensions.

**Tech Stack:** Quartz v5, SCSS, PowerShell assertions, Prettier, Node test runner, local Quartz server

---

### Task 1: Define the spacing and density regression contract

**Files:**

- Test: `quartz/styles/custom.scss`

- [x] **Step 1: Run a failing stylesheet assertion**

```powershell
$source = Get-Content -LiteralPath 'quartz/styles/custom.scss' -Raw
$required = @(
  'margin-top: 1.5rem;',
  'min-height: 3.125rem;',
  'border-radius: 0.75rem;',
  'align-items: center;'
)
foreach ($rule in $required) {
  if (-not $source.Contains($rule)) {
    throw "Missing compact search rule: $rule"
  }
}
```

Expected: FAIL because the current stylesheet has no result-panel top margin or `3.125rem` result-row minimum height.

- [x] **Step 2: Record the current browser dimensions**

Start the local preview with `npx.cmd quartz build --serve`, search for `项目`, and read computed dimensions for `.search-bar`, `.search-layout.display-results`, and `.result-card.focus`.

Expected before implementation: the input and results panel touch vertically, and a single-line result card is materially taller than `50px` in the current theme.

### Task 2: Implement the separated panel and compact result rows

**Files:**

- Modify: `quartz/styles/custom.scss:201-235`
- Modify: `quartz/styles/custom.scss:270-284`
- Modify: `quartz/styles/custom.scss:335-356`

- [x] **Step 1: Make the search input an independent rounded surface**

Replace the joined input treatment with:

```scss
.page .search > .search-container > .search-space > .search-bar {
  height: 2.875rem;
  margin-bottom: 0;
  padding: 0 1rem;
  border: 1px solid var(--lightgray);
  border-radius: 0.75rem;
  background: var(--garden-page);
  box-shadow: var(--garden-search-shadow);
}
```

Keep the focus border and combine its inset focus ring with the outer shadow.

- [x] **Step 2: Move the results panel down by `24px`**

Update the displayed layout rules to include:

```scss
margin-top: 1.5rem;
border-top: 1px solid var(--lightgray);
border-radius: 0.75rem;
```

Remove the old joined-panel-only `border-top: 0` and bottom-corners-only radius.

- [x] **Step 3: Reduce result-row height**

Update `.result-card` to use:

```scss
display: flex;
align-items: center;
min-height: 3.125rem;
padding: 0.55rem 0.9rem 0.55rem calc(0.9rem - 3px);
```

Add a scoped title rule with `font-size: 1rem`, `line-height: 1.35`, and zero margin. Long titles may wrap and grow naturally beyond the minimum height.

- [x] **Step 4: Keep mobile spacing compact**

Inside the existing `max-width: 800px` media query, set the displayed search layout to `margin-top: 0.875rem` while retaining the single-column results behavior.

- [x] **Step 5: Run the stylesheet assertion again**

Run the PowerShell assertion from Task 1.

Expected: PASS.

- [x] **Step 6: Format and run repository verification**

```powershell
npx.cmd prettier quartz/styles/custom.scss --write
npx.cmd prettier quartz/styles/custom.scss --check
git diff --check
npm.cmd test
npx.cmd quartz build
```

Expected: Prettier reports all matched files formatted, `git diff --check` exits `0`, all tests pass, and Quartz finishes the static build.

- [x] **Step 7: Verify computed browser dimensions**

In a desktop viewport, search for `项目` and verify:

- input-to-results gap is approximately `24px`;
- a single-line result card is approximately `50px` high;
- results and preview remain approximately `40% / 60%`;
- light and dark selected states remain visible.

In a `390px` viewport, verify the preview remains hidden and the result panel uses the smaller mobile gap.

- [ ] **Step 8: Commit and push**

```powershell
git add -- quartz/styles/custom.scss docs/superpowers/plans/2026-07-11-search-results-density.md
git commit -m "style: refine search result spacing"
git push origin v5
```
