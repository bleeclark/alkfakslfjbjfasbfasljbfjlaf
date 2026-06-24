# Risk Anomaly Detection React App - Pre-Implementation TDD

**Status:** Proposed  
**Last updated:** 2026-06-24  
**App:** `so_BUI_pickulationts`  
**Target page:** `/app/so_BUI_pickulationts/risk`  
**Audience:** Engineering, Splunk admins, product reviewers

## 1. Overview

This document describes the proposed design for a React-based Risk Anomaly Detection page inside the Splunk app. The goal is to replace a rough dashboard experience with a cleaner investigation workflow that supports filters, time range selection, tables, a line chart, a histogram, loading states, and optional hiding of empty panels.

The page will be delivered as a Splunk HTML view that loads a React bundle from `appserver/static/pages/risk.js`.

## 2. Goals

- Provide a clean React dashboard for risk anomaly investigation.
- Make the filter area easy to understand and submit-driven.
- Support a Splunk-style time range selector with presets, relative time, real-time, date range, and advanced token input.
- Show risk data in simple tables, one line chart, and one histogram.
- Show a spinner and progress bar while Splunk searches run.
- Allow users to hide panels that do not have data.
- Keep panel spacing even and proportional when panels are hidden or empty.
- Persist filter state in the URL so links can be shared.
- Support mock data for local development and Splunk REST data for real searches.

## 3. Non-Goals

- This design will not replace the entire Splunk app.
- This design will not build a new visualization framework.
- This design will not require real indexes to exist for local development.
- This design will not auto-refresh panels on every field edit; refresh will happen after Submit.

## 4. User Flow

1. User opens the Risk Anomaly Detection page.
2. User selects a time range and optional filters.
3. User clicks **Submit**.
4. The app applies the draft filters.
5. All panels refresh using the applied filters.
6. If Splunk mode is enabled, each panel shows loading progress while its search runs.
7. If **Hide empty panels** is enabled, panels with no data are removed from the layout.
8. Remaining panels resize so the page does not leave blank gaps.

## 5. Proposed Page Layout

The dashboard should render panels in this order:

1. Risk Scores Table
2. Risk Score Over Time line chart
3. Entity Category Table and Domain Distribution Histogram on the same row
4. Calendar Risk Table
5. Severity Breakdown Table and Anomaly Rows Table on the same row

The page should use one shared vertical gap between panels. Panels should not add their own extra bottom margins.

Two-column rows should behave like this:

- If both panels have data, show them side by side.
- If one panel is hidden, the remaining panel should fill the row.
- If a panel is empty but not hidden, show a compact empty state.

## 6. Filter Design

### 6.1 Filter Card

The page should have a single filter card near the top.

Controls:

- Time range
- Business unit
- Severity
- Domain
- Entity type
- Entity
- Hide empty panels
- Submit
- Reset

The first row should include:

- Time range
- Business unit
- Severity

The second row should include:

- Domain
- Entity type
- Entity

### 6.2 Submit Behavior

The app should maintain two filter states:

- **Draft filters:** values currently being edited.
- **Applied filters:** values currently driving panels.

Only **Submit** should copy draft filters into applied filters and refresh the panels.

Required loading UI:

- Wait spinner
- "Running search..." message
- Dispatch state when available
- Progress bar when Splunk reports progress
- Timeout error if the job takes too long

Mock mode may not show loading because fixture data should be available immediately.

## 14. Styling Requirements

The dashboard should feel like a clean operational tool.

Requirements:

- No overlapping controls.
- No floating overlay for the time picker.
- Consistent panel gaps.
- Consistent table header and cell padding.
- Fixed table layout for even spacing.
- Compact empty panel sizing.
- Proportional two-column rows.
- Cards should use restrained borders and shadows.

## 15. Testing Plan

Add focused tests for:

- Filter dependency clearing.
- Time range preset serialization.
- Real-time time range URL round-trip.
- Advanced time range URL round-trip.
- Hide empty panels URL round-trip.
- Splunk token mapping.
- Invalid entity selection without entity type.
- Mock severity filtering.
- Splunk job progress parsing.
- Search abort behavior.
- Unknown search handling.

Primary command:

```bash
yarn test:risk
```

## 16. Build and Verification Plan

After implementation:

```bash
yarn test:risk
yarn build
yarn verify:risk-dashboard
```

Expected generated files:

```text
stage/appserver/static/pages/risk.js
stage/appserver/templates/risk.html
```

## 17. Splunk Cache Plan

For visible UI changes, the implementation must update the asset cache key in `risk.html`.

Example:

```text
page_asset_version = "YYYYMMDDHHMM"
```

After build:

1. Confirm `stage/appserver/static/pages/risk.js` changed.
2. Confirm `stage/appserver/templates/risk.html` has the new asset key.
3. Hard refresh the browser.
4. Restart Splunk Web if the old bundle is still cached.

## 18. Acceptance Criteria

The work is complete when:

- The Risk page renders without layout gaps.
- Time range, Business unit, and Severity are on the same row.
- The time selector opens inline and has Presets, Relative, Real-time, Date Range, and Advanced sections.
- Submit applies all draft filters.
- Reset restores defaults.
- Filters persist in the URL.
- Panels show spinner/progress in Splunk mode.
- Empty panels are compact when shown.
- Empty panels disappear when Hide Empty Panels is enabled.
- Remaining panels resize proportionally after empty panels are hidden.
- Tests and build verification pass.

## 19. Implementation Sequence

1. Create or update filter state objects.
2. Add URL parsing and serialization.
3. Build the filter card.
4. Build the inline time range selector.
5. Build the shared panel shell.
6. Build table panels.
7. Build the line chart panel.
8. Build the domain histogram.
9. Add mock data support.
10. Add Splunk REST search support.
11. Add loading and progress states.
12. Add hide-empty-panel behavior.
13. Normalize layout spacing.
14. Add tests.
15. Build, verify, and cache-bust.

## 20. Risks

- Splunk static asset caching may show an old bundle after changes.
- Real-time search tokens may behave differently depending on Splunk configuration.
- Empty panels can leave blank grid cells if wrappers are not collapsed correctly.
- Advanced earliest/latest token input should be validated enough to avoid obvious blank values.
- Splunk REST mode requires a valid Splunk Web session and `splunkd` base URL.

## 21. Open Questions

- Should Hide Empty Panels default to on or off?
- Should empty panels show a count of why they are empty?
- Should Advanced time tokens be validated before Submit?
- Should Splunk mode be exposed as a visible toggle or stay URL-only?
- Should each panel use separate saved searches or share a combined search result?
