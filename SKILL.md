---
name: release-ready-radar
description: 'A-F grading across 24 domains with Axiom agent scanning, trend tracking, cross-domain correlation, regression detection, and ship/no-ship recommendations. Triggers: "release ready", "grade codebase", "/release-ready-radar".'
version: 2.1.0
author: Terry Nyberg
license: MIT
---

# Codebase Audit

**YOU MUST EXECUTE THIS WORKFLOW. Do not just describe it.**

**Required output:** Every finding MUST include Urgency, Risk, ROI, and Blast Radius ratings. Do not omit these ratings.

Hybrid codebase audit that combines graded reportcard analysis with deep
domain-specific agent scanning across 24 audit domains.

## Usage

- `/codebase-audit` — Full audit (parallel Axiom agents)
- `/codebase-audit quick` — Surface grep only, no agents
- `/codebase-audit focused` — Pick specific domains to audit

---

## Skill Introduction (MANDATORY — run before anything else)

On first invocation, ask the user two questions in a single `AskUserQuestion` call:

**Question 1: "What's your experience level with Swift/SwiftUI?"**
- **Beginner** — New to Swift. Plain language, analogies, define terms on first use.
- **Intermediate** — Comfortable with SwiftUI basics. Standard terms, explain non-obvious patterns.
- **Experienced (Recommended)** — Fluent with SwiftUI. Concise findings, no definitions.
- **Senior/Expert** — Deep expertise. Terse, file:line only, skip explanations.

**Question 2: "Would you like a brief explanation of what this skill does?"**
- **No, let's go (Recommended)** — Skip explanation, proceed to audit.
- **Yes, explain it** — Show a 3-5 sentence explanation adapted to the user's experience level (see below), then proceed.

**Experience-adapted explanations for Release Ready Radar:**

- **Beginner**: "Release Ready Radar is like a pre-flight checklist before your app goes to the App Store. It scans your entire codebase across 24 different areas — security, performance, accessibility, crash safety, and more — and gives each one a letter grade (A through F). At the end, it tells you whether the app is safe to ship or what needs fixing first. Think of it as a building inspector checking every room before you open for business."

- **Intermediate**: "Release Ready Radar grades your codebase across 24 domains (security, concurrency, accessibility, SwiftUI performance, data safety, etc.) using Axiom audit agents. It produces A-F grades per domain, tracks trends across runs, correlates cross-domain issues, detects regressions, and gives a ship/no-ship recommendation with specific blockers."

- **Experienced**: "24-domain A-F grading with Axiom agent scanning, trend tracking, cross-domain correlation, regression detection, and ship/no-ship recommendation. Domains include security, concurrency, accessibility, SwiftUI performance, data safety, networking, energy, and more."

- **Senior/Expert**: "24-domain grading. Axiom agents per domain. Trend + regression tracking. Ship/no-ship with blockers."

Store the experience level as `USER_EXPERIENCE` and apply to ALL output for the session.

---

## Terminal Width Check (MANDATORY — run first)

Before ANY output, check terminal width:
```bash
tput cols
```

- **160+ columns** → Use full 8-column Issue Rating Table. Proceed immediately.
- **Under 160 columns** → **Prompt the user first** using `AskUserQuestion`:

  **Question:** "Your terminal is [N] columns wide. The full Issue Rating Table needs 160+ columns. Want to widen it now?"
  - **"I've widened it" (Recommended)** — Re-run `tput cols` to confirm. If tput still reports the old width (terminal resize doesn't always propagate to the shell), trust the user and use full tables anyway.
  - **"Use compact tables"** — Use compact 3-column table with finding text on separate lines below each row:
    ```
    | # | Urgency | Fix Effort |
    |---|---------|------------|
    | 1 | 🟡 HIGH | Small      |
    |   `activeImporterKind` never assigned — file importer silently drops files |
    |   `EnhancedItemDetailView.swift:93` |
    | 2 | ⚪ LOW  | Trivial    |
    |   `showingAddImageMenu` declared but never used — dead code |
    |   `EnhancedItemDetailView.swift:94` |
    ```
    Full 8-column table goes to report file only (if report delivery was selected).
  - **"Skip check"** — Use full 8-column table regardless (user accepts wrapping).

  If the user chose compact mode, **after each compact table, print:**

```
📐 Compact table (terminal: [N] cols). Say "show full table" for all 8 columns.
```

If the user later says "show full table", "wide table", or "full ratings", re-render the most recent findings table in full 8-column format regardless of terminal width. Apply to ALL tables in the session.

---

## Before Starting

Ask the user:

**Question 1: "What's your experience level with Swift/SwiftUI?"**
- **Beginner** — New to Swift. Plain language, analogies, define terms on first use.
- **Intermediate** — Comfortable with SwiftUI basics. Standard terms, explain non-obvious patterns.
- **Experienced (Recommended)** — Fluent with SwiftUI. Concise findings, no definitions.
- **Senior/Expert** — Deep expertise. Terse, file:line only, skip explanations.

**Question 2: "Any specific concerns?"**
- **No specific concerns** — Balanced audit across all domains
- **Crash/stability issues** — Crashes, freezes, data loss
- **App Store rejection risk** — Privacy, accessibility, guidelines
- **Tech debt / maintainability** — Codebase slowing development

**Question 3: "Should the analysis consider CLAUDE.md?"**
- **Yes, use CLAUDE.md (Recommended)** — Include project context and standards
- **No, ignore CLAUDE.md** — Unbiased analysis

**Question 4: "Which audit mode?"**
- **Full (parallel agents) (Recommended)** — Launch Axiom auditor agents for deep scanning
- **Quick (surface grep only)** — Inline grep patterns, no agents — fastest
- **Focused (pick domains)** — Choose specific domains

**Question 5: "What is your timeline?"**
- **Pre-release** — Enables ship/no-ship recommendation
- **Post-release** — Ongoing improvement
- **Planning phase** — Gathering info for roadmap

**Question 6: "Will you be stepping away during the audit?"**
- **I'll be here (Recommended)** — Normal mode. Permission prompts may appear.
- **Hands-free (walk away safe)** — Read-only tools only (Read, Grep, Glob). No Bash, no Edit, no Write — nothing that triggers a permission prompt. Report writing deferred until you return.
- **Pre-approved** — You've configured Claude Code permissions. Full speed, no restrictions.

**How user concerns shape the audit:**
- Matching domains run in Batch 1 regardless of normal tier
- Categories matching concerns get **+3% weight boost**
- Issues matching concerns get **+2 bonus** to composite score
- Concern-related categories appear first in the report

**Concern-to-domain mapping:**
- Crash/stability → concurrency, memory, data-persistence
- App Store rejection → accessibility, security, ui-ux
- Tech debt → architecture, code-quality, modernization, testing

**If Focused mode:** Present domain picker:

| Group | Domains |
|-------|---------|
| Core | Architecture, Performance + SwiftUI Perf, Concurrency, Security |
| Quality | Testing, Accessibility, Code Quality, UI/UX |
| Infrastructure | Data/Persistence + Memory, Networking, Storage + Energy |

**If CLAUDE.md = Yes:** Read CLAUDE.md at repo root. Summarize in 3-5 bullets.

### Permission Modes

#### Normal Mode
- Read any file without asking.
- Write report to `.agents/research/` without asking.
- Build and run tests without asking.

#### Hands-Free Mode
**Guarantees no blocking prompts.** Only uses Read, Grep, Glob. No Bash, no Edit, no Write. When complete:
```
⏱ Hands-free audit complete. Analysis ready.
  Deferred: Report file writing, build verification.
  Reply to continue with supervised steps.
```

#### Pre-Approved Mode
Full speed, no restrictions. Assumes permissions are configured.

### Permission Setup (for unattended runs)

Safe to auto-approve (read-only):
```
Bash(find:*), Bash(wc:*), Bash(stat:*)
```

Keep prompted (modify state):
```
Edit, Write, Bash(rm:*), Bash(git:*)
```

### Experience-Level Adaptation

Adjust ALL output (grades, findings, recommendations, domain summaries) based on the user's experience level:

- **Beginner**: Explain what each grade means and why it matters. "Your app got a B in Security — this means most things are good, but there are a few spots where sensitive data could leak." Define terms. Use analogies for severity.
- **Intermediate**: Standard terminology, explain non-obvious findings. "The A- in Concurrency means your `@MainActor` usage is correct, but 3 `Task.detached` calls bypass actor isolation without justification."
- **Experienced** (default): Concise grades + key findings. No definitions. Focus on what needs action.
- **Senior/Expert**: Grades + file:line references only. Skip domain descriptions. Just the data.

### Context Budget

If context is running low, prioritize in this order:
1. Finish collecting results from dispatched agents
2. Complete grading for collected results
3. Write the report for graded categories
4. Skip unfinished domains and note as "Not audited"

Never dispatch agents you can't collect results from.

---

## Step 1: Project Auto-Detection

Scan the project to determine which Tier 2 domains are relevant:

| Domain | Detection Pattern |
|--------|-------------------|
| SpriteKit | `import SpriteKit` |
| Camera | `AVCaptureSession\|UIImagePickerController` |
| IAP | `import StoreKit\|SKPaymentQueue\|Product\.` |
| iCloud | `CKContainer\|NSPersistentCloudKitContainer` |
| TextKit | `NSTextStorage\|NSLayoutManager\|UITextView` |
| Foundation Models | `LanguageModelSession\|@Generable\|import FoundationModels` |
| Liquid Glass | `glassEffect\|GlassBackgroundEffect` |
| SwiftData | `@Model\|ModelContainer` |
| Core Data | `*.xcdatamodeld` (Glob) |
| Database Schema | `DatabaseMigrator\|ALTER TABLE\|VersionedSchema` |
| Navigation | `NavigationStack\|NavigationSplitView\|NavigationPath` |
| Modernization | `ObservableObject\|@StateObject\|@Published` |

Domains with zero hits are auto-skipped and noted in the report.

---

## Step 2: Previous Report Check

```
Glob pattern=".agents/research/*-codebase-audit.md"
```

If a previous report exists, parse the `GRADES_YAML` HTML comment block for
structured scores. Use for trend comparison in Step 7.

---

## Step 3: Codebase Metrics

Collect:

1. **File counts** — `Glob pattern="**/*.swift"` (exclude Tests, Pods, .build, DerivedData)
2. **LOC estimate** — file count x 150, or `wc -l` for accuracy
3. **Architecture** — detect MVVM/MVC/TCA by scanning for ViewModel, Controller, Reducer
4. **Persistence layer** — SwiftData, Core Data, GRDB, UserDefaults, Realm
5. **Test infrastructure** — Swift Testing vs XCTest, unit test count, UI test count
6. **Targets** — check xcodeproj for target count

Ensure output directories exist: `mkdir -p .agents/research`

Print one-line summary:
```
Project: {files} Swift files (~{loc} LOC) | {arch} | {persistence} | Tests: {unit}/{ui} | Domains: {detected_list}
```

---

## Step 4: Agent Dispatch

### Full Mode — Axiom Agents

Launch Task agents in priority batches using `run_in_background: true`.

**Batch 1 — CRITICAL (data corruption, crashes, security):**

| Domain | Task `subagent_type` |
|--------|---------------------|
| Security | `axiom:security-privacy-scanner` |
| Concurrency | `axiom:concurrency-auditor` |
| Data/Persistence | `axiom:swiftdata-auditor` |

**Batch 2 — HIGH (user-facing impact):**

| Domain | Task `subagent_type` |
|--------|---------------------|
| Performance | `axiom:swift-performance-analyzer` |
| SwiftUI Perf | `axiom:swiftui-performance-analyzer` |
| Memory | `axiom:memory-auditor` |
| Architecture | `axiom:swiftui-architecture-auditor` |

**Batch 3 — MEDIUM (quality + compliance):**

| Domain | Task `subagent_type` |
|--------|---------------------|
| Accessibility | `axiom:accessibility-auditor` |
| Testing | `axiom:testing-auditor` |
| Energy | `axiom:energy-auditor` |
| Networking | `axiom:networking-auditor` |
| Storage | `axiom:storage-auditor` |
| Codable | `axiom:codable-auditor` |
| UI/UX | Inline patterns (below) |
| Code Quality | Inline patterns (below) |

**Batch 4 — CONDITIONAL (only if detected in Step 1):**

| Domain | Task `subagent_type` | Condition |
|--------|---------------------|-----------|
| iCloud | `axiom:icloud-auditor` | CKContainer detected |
| Navigation | `axiom:swiftui-nav-auditor` | NavigationStack detected |
| Camera | `axiom:camera-auditor` | AVCaptureSession detected |
| IAP | `axiom:iap-auditor` | StoreKit detected |
| TextKit | `axiom:textkit-auditor` | NSTextStorage detected |
| SpriteKit | `axiom:spritekit-auditor` | SpriteKit detected |
| Foundation Models | `axiom:foundation-models-auditor` | FoundationModels detected |
| Liquid Glass | `axiom:liquid-glass-auditor` | glassEffect detected |
| Database Schema | `axiom:database-schema-auditor` | DatabaseMigrator detected |
| Modernization | `axiom:modernization-helper` | ObservableObject detected |

#### Agent Prompt Template

Each agent receives this prompt (fill in `{domain}`):

```
Audit the iOS codebase for {domain} issues. Scan all Swift files
(exclude *Tests.swift, */Pods/*, */.build/*, */DerivedData/*).

FRESHNESS RULE: Scan ONLY current source code files. Do NOT read files
in .agents/, scratch/, or any prior audit reports. Do NOT reference
cached findings from auto-memory or previous sessions. Base all
findings solely on what the source code contains right now.

After completing your analysis, return ONLY this compact summary:

DOMAIN: {domain}
SCORE: {0-100, start at 100, deduct per confirmed issue}
ISSUES: CRITICAL={n} HIGH={n} MEDIUM={n} LOW={n}
INCOMPLETE: {none | trigger description if catastrophic}
FINDINGS:
1. [{severity}/{confidence}] {file:line} — {description}
...(up to 10, sorted by severity desc)
STRENGTHS:
- {positive pattern observed}
(2-4 bullets)

Do NOT return full analysis prose. Compact summary only.
```

**Incomplete triggers for agents:** Instruct agents to flag `INCOMPLETE` if
they find (CONFIRMED + HIGH confidence only): confirmed crash path in hot path,
hardcoded production credentials, unencrypted secrets in UserDefaults/plist,
missing `transaction.finish()` in StoreKit, confirmed data loss path, missing
Privacy Manifest (iOS 17+), unrestricted `NSAllowsArbitraryLoads`.

#### Inline Patterns — Code Quality

Run directly (no agent). Apply Verification Rule (Step 5).

```
Grep pattern="// TODO|// FIXME|// HACK|// XXX" glob="**/*.swift" output_mode="count"
Grep pattern="as!" glob="**/*.swift"
Grep pattern="try!" glob="**/*.swift"
Grep pattern="@available\(.*deprecated" glob="**/*.swift"
```

#### Inline Patterns — UI/UX

```
Grep pattern="Color\(\.white\)|Color\(\.black\)|UIColor\.white|UIColor\.black" glob="**/*View*.swift"
Grep pattern="UIScreen\.main" glob="**/*.swift"
Grep pattern="\.ignoresSafeArea\(\)" glob="**/*.swift"
Grep pattern="Color\(red:" glob="**/*.swift" output_mode="count"
Grep pattern="\.font\(\.system\(size:" glob="**/*.swift"
```

### Quick Mode — Inline Only

Skip agent dispatch. Run essential grep patterns directly for all domains.
Apply Verification Rule to every hit.

**Architecture:** ViewModel file sizes (>500 lines), missing @MainActor on ObservableObject
**Security:** Hardcoded secrets, sensitive UserDefaults, HTTP URLs, logging credentials
**Performance:** Synchronous file I/O in views, Thread.sleep, .wait(), formatter allocation
**Concurrency:** Missing @MainActor on ViewModels, legacy DispatchQueue.main, Task.detached
**Accessibility:** Button count vs accessibilityLabel count, fixed font sizes
**Testing:** Test file count, framework detection, sleep in tests
**Energy:** Timer usage, continuous location, polling loops
**Code Quality:** TODO/FIXME count, force casts, force tries
**UI/UX:** Hardcoded colors, deprecated UIScreen.main, safe area blankets
**Memory:** Timer without invalidation, sink without store(in:), observer without removal
**Data/Persistence:** @Query count, UserDefaults for non-preference data

Quick Mode grep patterns:

```
# Security
Grep pattern="(api[_-]?key|secret[_-]?key|password|token)\s*[:=]\s*[\"'][^\"']+[\"']" glob="**/*.swift" -i
Grep pattern="UserDefaults.*\.(password|token|secret|apiKey)" glob="**/*.swift" -i
Grep pattern="http://(?!localhost|127\.0\.0\.1)" glob="**/*.swift"

# Performance
Grep pattern="Data\(contentsOf:\s*URL" glob="**/*View*.swift"
Grep pattern="Thread\.sleep" glob="**/*.swift"
Grep pattern="DateFormatter\(\)|NumberFormatter\(\)" glob="**/*.swift"

# Concurrency
Grep pattern="(class|struct).*ViewModel(?!.*@MainActor)" glob="**/*ViewModel.swift"
Grep pattern="DispatchQueue\.main\.(async|sync)" glob="**/*.swift"

# Accessibility
Grep pattern="Button\(" glob="**/*.swift" output_mode="count"
Grep pattern="\.accessibilityLabel" glob="**/*.swift" output_mode="count"
Grep pattern="\.font\(\.system\(size:" glob="**/*.swift"

# Testing
Grep pattern="import Testing" glob="**/*Test*.swift" output_mode="count"
Grep pattern="import XCTest" glob="**/*Test*.swift" output_mode="count"
Grep pattern="Thread\.sleep|sleep\(" glob="**/*Test*.swift"

# Memory
Grep pattern="Timer\.(scheduledTimer|publish)" glob="**/*.swift"
Grep pattern="NotificationCenter\.default\.addObserver" glob="**/*.swift"
Grep pattern="\.sink\s*\{" glob="**/*.swift" output_mode="count"
Grep pattern="\.store\(in:" glob="**/*.swift" output_mode="count"
```

### Focused Mode

Run only agents for user-selected domains, following Full Mode pattern.

---

## Step 5: Verification Rule

**Applies to inline patterns. Axiom agents perform their own verification.**

Grep patterns produce candidates, NOT confirmed issues. Before reporting:

1. **Read the flagged file** — minimum 20 lines of context
2. **Check structural context** — pattern may be safe in its actual scope
3. **Classify:** CONFIRMED, FALSE_POSITIVE, or INTENTIONAL
4. **Never report grep counts as issue counts** — classify each hit
5. **Only CONFIRMED findings appear in the report**

**Common false positives:**
- `Task {}` inside `@MainActor` class → inherits isolation
- `DispatchQueue.main.asyncAfter` for animation → intentional
- `nonisolated` on UIKit delegate methods → protocol requirement
- `http://` in XML namespaces → not an endpoint
- `[weak self]` in SwiftUI structs → value types

---

## Step 6: Result Collection + Cross-Domain Correlation

Process ONE domain at a time to manage context.

### 6.1 Collect

For each completed agent:
1. Read the compact summary (from Task result)
2. Record: DOMAIN, SCORE, issue counts, FINDINGS, STRENGTHS
3. Discard detail before processing the next domain

### 6.2 Deduplicate

If the same `file:line` appears in multiple agent results, merge into a
single finding with combined domains noted.

### 6.3 Cross-Domain Correlation

Flag compounding risks when the same file appears in multiple domains:

| Combination | Elevated Risk |
|-------------|--------------|
| Concurrency + Memory | Race causes unreproducible leak |
| Performance + Energy | Hot code path drains battery |
| Architecture + Testing | High-risk change with no safety net |
| Security + Networking | Exposed secret over insecure transport |
| Concurrency + Data Persistence | Concurrent writes corrupt data |
| SwiftUI Perf + Memory | Expensive views that also leak |

### 6.4 Regression Detection

If a previous report exists (from Step 2):
- Issues absent before but present now → **REGRESSION** (+50% severity)
- Issues present before and now fixed → **RESOLVED**

---

## Step 7: Grading

### Per-Category Scoring

Start at **100 points**. Deduct per finding:

| Severity | HIGH Conf | MED Conf | LOW Conf |
|----------|-----------|----------|----------|
| CRITICAL | -15 | -10 | -3 |
| HIGH | -8 | -5 | -1 |
| MEDIUM | -3 | -2 | 0 |
| LOW | -1 | -1 | 0 |

**Additional deductions:**
- Cross-domain compounding: **-5** per correlated pair
- Regression: **+50%** severity points

### Grade Scale

| Grade | Score | Grade | Score | Grade | Score |
|-------|-------|-------|-------|-------|-------|
| A+ | 97-100 | B+ | 87-89 | C+ | 77-79 |
| A | 93-96 | B | 83-86 | C | 73-76 |
| A- | 90-92 | B- | 80-82 | C- | 70-72 |
| | | D+ | 67-69 | D | 63-66 |
| | | D- | 60-62 | F | 0-59 |
| **I** | **n/a** | | | | |

### Incomplete (I) Grade

Overrides the numeric score when a CONFIRMED + HIGH confidence finding
represents a ship-blocking, irreversible risk:

| Trigger | Category |
|---------|----------|
| Confirmed crash path in hot path | Concurrency / Performance |
| Hardcoded production credentials | Security |
| Unencrypted secrets in UserDefaults/plist | Security |
| Missing `transaction.finish()` in StoreKit | Security (IAP) |
| Confirmed data loss/corruption path | Data/Persistence |
| Missing required Privacy Manifest (iOS 17+) | Security |
| Unrestricted `NSAllowsArbitraryLoads` | Security |

If ANY category is Incomplete → overall grade is `I (Incomplete)`.

### Category Weights (Timeline-Adjusted)

| Category | Base | Pre-Release | Post-Release |
|----------|------|-------------|--------------|
| Architecture | 12% | 10% | 12% |
| Code Quality | 8% | 8% | 8% |
| Performance | 12% | 10% | 12% |
| SwiftUI Perf | 8% | 8% | 10% |
| Concurrency | 10% | 8% | 12% |
| Security | 12% | 15% | 10% |
| Accessibility | 10% | 12% | 8% |
| Testing | 10% | 10% | 10% |
| UI/UX | 5% | 7% | 5% |
| Data/Persistence | 5% | 5% | 5% |
| Energy | 3% | 3% | 3% |
| Networking | 3% | 2% | 3% |
| Storage | 2% | 2% | 2% |

### Tier 2 Agent Mapping

Tier 2 findings feed into the closest Tier 1 category:

| Tier 2 Agent | Maps To |
|--------------|---------|
| iCloud | Data/Persistence |
| Codable | Code Quality |
| Navigation, TextKit, Liquid Glass | UI/UX |
| Camera, SpriteKit | Performance |
| IAP | Security |
| Foundation Models | Architecture |
| Database Schema | Data/Persistence |
| Modernization | Code Quality |

### Overall Grade

Weighted average using timeline-adjusted weights, mapped to grade scale.
If any category is Incomplete → overall is `I (Incomplete)`.

---

## Step 8: Output

Write report to `.agents/research/YYYY-MM-DD-codebase-audit.md`.
Write each section as generated — don't accumulate the full report first.

### Report Sections

1. **Header** — date, version, mode, agents dispatched, domains skipped
2. **User concerns** (if provided) — echoed with links to relevant findings
3. **CLAUDE.md summary** (if included) — 3-5 bullets
4. **Project metrics** — files, LOC, architecture, persistence, tests
5. **Grade summary line** — `**Overall: B+** (Arch A- [91] | Security A [95] | ...)`
6. **GRADES_YAML** — machine-readable scores in HTML comment for trend parsing
7. **Trend comparison** — vs previous report (if exists)
8. **Ship recommendation** — pre-release only (Step 9)
9. **Cross-domain correlations table**
10. **Per-category sections** — grade, score trail, strengths, CONFIRMED issues
11. **Technical debt summary** — by severity with counts
12. **Regressions** — new issues since last audit
13. **Top 10 prioritized issues** — composite: `(Urgency x 3) + (Risk x 2) + (ROI x 2) + (1/LOE)`
14. **Next steps** — Immediate / Short-term / Medium-term

### Issue Rating Format

For every finding in per-category sections:

| # | Finding | Urgency | Risk: Fix | Risk: No Fix | ROI | Blast Radius | Fix Effort |
|---|---------|---------|-----------|--------------|-----|--------------|------------|

**Blast Radius:** Always include the number of files the fix touches (e.g., `🟢 3 files`, `⚪ 1 file`). Do not use `<br>` tags. Count by grepping for callers/references before rating.

### Indicator Scale

| Indicator | General meaning | ROI meaning |
|-----------|----------------|-------------|
| 🔴 | Critical / high concern | Poor return |
| 🟡 | High / notable | Marginal return |
| 🟢 | Medium / moderate | Good return |
| ⚪ | Low / negligible | — |
| 🟠 | Pass / positive | Excellent return |

---

## Step 9: Ship Recommendation (Pre-Release Only)

| Recommendation | Criteria |
|----------------|----------|
| **SHIP** | No CRITICALs, <=2 HIGHs, overall B- or above, no Incompletes |
| **CONDITIONAL SHIP** | No CRITICALs, 3-5 HIGHs, overall C+ or above, no Incompletes |
| **DO NOT SHIP** | Any Incomplete, OR any CRITICAL, OR >5 HIGHs, OR overall C or below |

Any Incomplete = automatic **DO NOT SHIP** — no exceptions.

Format:

```
## Ship Recommendation: [SHIP / CONDITIONAL SHIP / DO NOT SHIP]

### Blockers (must fix before release)
1. [Issue] — LOE: Xh | File: path:line

### Advisories (fix soon after release)
1. [Issue] — LOE: Xh | File: path:line

**Estimated total blocker LOE:** X hours
```

---

## Step Progress Banner (CRITICAL — BLOCKING requirement)

**After EVERY step and EVERY commit, your NEXT output MUST be the progress banner followed by the next-step `AskUserQuestion`. Do not output anything else first. Do not leave a blank prompt.**

After completing each step, **always** print this banner:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Step [N] of 10 complete: [step name]

⏱  Next: Step [N+1] — [step name] (~[time estimate])
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Step time estimates:
| Step | Name | Est. Time |
|------|------|-----------|
| 1 | Project Auto-Detection | ~1 min |
| 2 | Previous Report Check | ~1 min |
| 3 | Codebase Metrics | ~2 min |
| 4 | Agent Dispatch | ~10-20 min |
| 5 | Verification | ~2-5 min |
| 6 | Result Collection | ~3-5 min |
| 7 | Grading | ~2 min |
| 8 | Output | ~2 min |
| 9 | Ship Recommendation | ~1 min |
| 10 | Follow-up | ~2 min |

**After a commit or any pause**, reprint the banner and auto-prompt. Never leave the user with a blank prompt.

---

## Step 10: Follow-up

After presenting the report, offer:

**Deep dive options** (requires Axiom):
- Performance → invoke `axiom-ios-performance`
- Concurrency → invoke `axiom-ios-concurrency`
- Accessibility → invoke `axiom-ios-accessibility`
- Memory → invoke `axiom-memory-debugging`

**Implementation plan:**
- Plan blockers only (CRITICAL + HIGH items)
- Plan all items (comprehensive roadmap)
- No plan needed

---

## Axiom Integration Reference

| Category | Axiom Skills | When |
|----------|--------------|------|
| Architecture | `axiom-swiftui-architecture`, `axiom-app-composition` | Refactoring |
| Performance | `axiom-ios-performance`, `axiom-swift-performance` | Deep dive |
| SwiftUI Perf | `axiom-swiftui-performance` | Rendering |
| Concurrency | `axiom-ios-concurrency`, `axiom-swift-concurrency` | Swift 6 |
| Memory | `axiom-memory-debugging` | Leak detection |
| Testing | `axiom-ios-testing`, `axiom-swift-testing` | Framework |
| Accessibility | `axiom-ios-accessibility` | WCAG |
| Data | `axiom-ios-data`, `axiom-swiftdata` | Persistence |
| UI | `axiom-ios-ui`, `axiom-hig` | Design |
| Energy | `axiom-energy` | Battery |
| Navigation | `axiom-swiftui-nav` | Deep linking |

---

## See Also

- `/workflow-audit` — UI workflow audit from user perspective

---

## Cross-Skill Handoff

Release Ready Radar complements **ui-path-radar** (navigation paths), **roundtrip-radar** (data safety), and **ui-enhancer** (visual quality). It consumes findings from the other skills and produces ship/no-ship recommendations.

### On Startup — Read Handoffs (Consumer)

Before Step 4 (Agent Dispatch), check for handoff files from other skills:
- `.agents/ui-audit/ui-path-radar-handoff.yaml` — blockers from path audit
- `.agents/ui-audit/roundtrip-radar-handoff.yaml` — blockers from data audit
- `.agents/ui-audit/ui-enhancer-handoff.yaml` — blockers from visual audit

If found, incorporate `blockers` entries into the relevant domain grades. A CRITICAL blocker from roundtrip-radar should cap the Data Safety domain at C or lower.

If not found, proceed normally — the other skills may not be installed or haven't been run yet.

### On Completion — Write Handoff (Producer)

After generating the ship recommendation, write `.agents/ui-audit/release-ready-radar-handoff.yaml`:

```yaml
source: release-ready-radar
date: <ISO 8601>
project: <project name>
recommendation: "ship" | "fix-first" | "no-ship"

for_roundtrip_radar:
  # Domains graded D or F that need data-level investigation
  priority_workflows:
    - domain: "<e.g., Data Safety>"
      grade: "D"
      reason: "<why this domain scored low>"

for_ui_path_radar:
  # Domains with navigation/UX concerns
  priority_areas:
    - domain: "<e.g., Navigation>"
      grade: "C"
      reason: "<specific navigation issues found>"

for_ui_enhancer:
  # Views flagged for visual quality issues
  priority_views:
    - domain: "<e.g., Accessibility>"
      grade: "D"
      reason: "<specific accessibility gaps>"
```

**Fire-and-forget:** Always write this file regardless of whether other skills are installed.

---

## REMINDER (End-of-File — Survives Context Compaction)

**CRITICAL:** After EVERY step, EVERY commit, and EVERY domain transition:
1. Print the progress banner (step-level)
2. Immediately `AskUserQuestion` for the next step
3. NEVER leave a blank prompt

This reminder is placed at the end of the file because context compaction tends to preserve the beginning and end. If you are unsure whether to print the banner, **print it**.
