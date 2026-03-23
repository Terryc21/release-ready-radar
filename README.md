# Release Ready Radar

> A-F grading across 24 domains with Axiom agent scanning, trend tracking, cross-domain correlation, regression detection, and ship/no-ship recommendations.

## What It Does

Release Ready Radar is a pre-flight checklist for your app. It scans your entire codebase across 24 domains — security, performance, accessibility, crash safety, concurrency, and more — and gives each one a letter grade (A through F). At the end, it tells you whether the app is safe to ship or what needs fixing first.

## Quick Start

```
/release-ready-radar          # Full 24-domain audit with ship recommendation
```

## Installation

Copy `skills/codebase-audit/SKILL.md` to `~/.claude/skills/release-ready-radar/SKILL.md`

## Companion Skills

Release Ready Radar is part of a 4-skill audit suite. Each skill finds different issues, and findings from one feed into the next via `.agents/ui-audit/` handoff files.

| Skill | Focus | GitHub |
|-------|-------|--------|
| **UI Path Radar** | Navigation paths, dead ends, broken promises | [ui-path-radar](https://github.com/Terryc21/ui-path-radar) |
| **Roundtrip Radar** | Data safety, error handling, round-trip completeness | [roundtrip-radar](https://github.com/Terryc21/roundtrip-radar) |
| **UI Enhancer** | Visual quality, spacing, color accessibility, layout | [ui-enhancer](https://github.com/Terryc21/ui-enhancer) |
| **Release Ready Radar** (this skill) | Ship readiness, A-F grading across 24 domains | [release-ready-radar](https://github.com/Terryc21/release-ready-radar) |

### How They Work Together

```
ui-path-radar  -->  roundtrip-radar  -->  release-ready-radar
 (navigation)       (data safety)         (ship/no-ship)
      |                   |
      v                   v
         ui-enhancer
         (visual quality)
```

**Recommended audit order:**
1. **UI Path Radar** first — find all entry points and broken paths
2. **Roundtrip Radar** second — audit data flows behind those paths
3. **UI Enhancer** third — polish views after structural issues are fixed
4. **Release Ready Radar** last — final ship/no-ship grade

Release Ready Radar is the best **consumer** of the other skills' findings. It reads handoff files from all three companion skills and factors their blockers into domain grades. A CRITICAL finding from roundtrip-radar can cap a domain at C or lower.

Each skill writes a handoff YAML file to `.agents/ui-audit/`. If a companion skill isn't installed, Release Ready Radar runs its own domain scans independently.

## License

MIT
