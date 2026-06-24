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
