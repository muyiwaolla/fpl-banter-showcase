# Engineering Decisions and Lessons

This document captures selected decisions that shaped FPL Banter.

## 1. Do not call FPL from page views

**Decision:** Browser requests read FPL Banter state. Background processes own upstream FPL calls.

**Why:** User traffic should not directly multiply external API traffic.

**Trade-off:** Data is refreshed on a cadence rather than instantaneously for every browser request.

## 2. Treat live and final data differently

**Decision:** Live calculations are provisional. Finished gameweeks are reconciled against official data.

**Why:** During active play, some final FPL mechanics are not fully settled.

**Trade-off:** The codebase needs separate live and final paths, but the resulting behaviour is easier to reason about.

## 3. Validate completeness before finalising

**Decision:** A finished aggregate requires evidence that the expected official league roster was processed.

**Why:** A successful job execution is not the same thing as a complete dataset.

**Lesson:** Background jobs need data-quality invariants, not just exception handling.

## 4. Scope derived data by season

**Decision:** Aggregated statistics include season context.

**Why:** FPL IDs, manager teams, and competition state change between seasons.

**Lesson:** Time-based products need explicit lifecycle boundaries.

## 5. Centralise HTTP behaviour in the frontend

The frontend moved toward a shared request client rather than repeating fetch logic across many components.

This creates one place for:

- base URL handling
- authentication headers
- error parsing
- request behaviour

A later production issue demonstrated why this matters: a hard-coded `www` backend URL caused cross-origin login failures for users visiting the non-`www` site.

The fix was to use same-origin production API requests while retaining explicit local development configuration.

## 6. Test against real product behaviour

Unit tests are valuable, but production data systems also need environment validation.

The release pattern used for higher-risk changes is:

1. local tests
2. local browser verification
3. staging deployment
4. staging behaviour check
5. production deployment
6. production sanity check against official FPL data

This caught several issues that a build-success signal alone would not have found.

## 7. Product logic should be observable

Derived features such as banter statistics are fun only if users trust them.

For each statistic, the implementation needs:

- a clear source of truth
- a defined gameweek/season scope
- predictable handling of missing data
- a way to compare the result with official FPL information

The more playful the feature, the more important the underlying calculation discipline becomes.
