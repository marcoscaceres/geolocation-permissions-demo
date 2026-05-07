# Design: Geolocation Permissions Demo Redesign

**Date:** 2026-05-07
**Purpose:** Redesign the approximate geolocation permissions model comparison demo for clarity and usability. Primary audience: antosart (Chromium), tomvangoethem (Chrome Privacy), Privacy WG reviewers.

## Structure

```
STICKY BAR (always visible on scroll)
─────────────────────────────────────────
[No Decision] [Approximate Only] [Precise] [Denied]
+ permissive/strict toggle

HERO
─────────────────────────────────────────
Title + one-sentence thesis

EXAMPLE CARDS (×4, sequential)
─────────────────────────────────────────
Each: prose → code (3-5 lines) → results strip (reactive)

SUMMARY TABLE (collapsed <details>)
─────────────────────────────────────────
Full 12-scenario matrix, expandable
```

## Sticky Bar

- `position: sticky; top: 0`
- One row of segmented buttons for user state: No Decision, Approximate Only, Precise, Denied
- Small secondary toggle: permissive/strict legacy behavior
- Backdrop blur so content scrolls underneath readably
- Updates all cards reactively when state changes

## Example Cards

Each card has four parts:

1. **Heading** — short name (e.g., "Upgrade Flow")
2. **Prose** — 2-3 sentences. The story: what the developer does, why it matters, what the finding is.
3. **Code** — 3-5 lines max. Zero comments. Just the API calls that prove the point. Syntax highlighted via Prism.js.
4. **Results strip** — reactive to sticky bar state. Shows:
   - Two-Permission: `query()` return value → API behavior → pass/fail
   - Single-Permission: `query()` return value → API behavior → pass/fail
   - One-line verdict (color-coded)

## The Four Examples

### 1. Weather App (approximate-only)
**Story:** A weather app only needs city-level location. It queries permission state, calls with approximate, adapts to accuracy.
**Code:** query + getCurrentPosition with approximate.
**Finding:** Both models work identically here. No difference.

### 2. Upgrade Flow (ride-sharing)
**Story:** A ride-sharing app starts with approximate (show nearby drivers), then upgrades to precise (pickup). The developer calls getCurrentPosition({precise}) when the user taps a button.
**Code:** Two getCurrentPosition calls (approximate, then precise).
**Finding:** This is where the models diverge in State 5. Two-permission predicts the prompt. Single-permission doesn't. But the developer pattern (call API, handle result) works regardless.

### 3. enableHighAccuracy Precedent
**Story:** Today, enableHighAccuracy is a hint with no queryable permission and no result attribute. query("geolocation") returning "granted" has never guaranteed high-accuracy results. accuracyMode is architecturally the same.
**Code:** query + getCurrentPosition with enableHighAccuracy: true.
**Finding:** The "prediction gap" already exists today. No one has required a separate query for it.

### 4. The Privacy Cost
**Story:** Under the two-permission model, any third-party script can passively learn the user's accuracy preference on page load without a gesture. This compounds w3c/permissions#52 (open since 2015).
**Code:** Two query() calls extracting the user's choice silently.
**Finding:** Two-permission enables dark patterns. Single-permission doesn't leak the choice.

## Summary Table

- Lives in a `<details><summary>Full 12-scenario matrix</summary>` at the bottom
- 4 states × 3 API calls = 12 rows
- Columns: State, Action, Two-Permission predicts?, Single-Permission predicts?
- Reactive to permissive/strict toggle
- Shows score: "Two-Permission: N/12, Single-Permission: N/12"

## What Gets Cut

- Three separate "Property" panels (replaced by example cards)
- Inline comments in code (moved to prose)
- Event log panel (internal debugging)
- "Run with current state" buttons (results are reactive, no manual trigger)
- The third column placeholder (never built)

## Technical Notes

- Single HTML file + external style.css
- Prism.js via CDN for syntax highlighting
- JavaScript mock API unchanged (same state machine logic)
- Mermaid not needed (no diagrams in this page)
- Results strip uses same `setValueClass()` pattern from existing code
