# Design System — FixGo
 
> The design system is the shared visual language between design and the Flutter/Android
> frontend. It prevents inconsistencies and gives every screen in `navigation-map.md` the
> same building blocks. **Rule:** before creating a new component, check here if it
> already exists.
 
---
 
## Design tokens
 
Tokens are the design system's variables. Changing a token changes the entire system.
FixGo ships both a light and a dark theme (**RF6.2** — must apply without re-login and
persist across app restarts, **NFR-08**).
 
### Colors
 
```css
/* Base palette — light theme */
--color-primary-50:  #E3F2FD;
--color-primary-500: #1976D2;   /* Default — app bar, primary actions */
--color-primary-900: #0D47A1;   /* Pressed state */
 
--color-neutral-50:  #FAFAFA;   /* Page background */
--color-neutral-900: #212121;   /* Primary text */
 
/* Base palette — dark theme */
--color-primary-500-dark: #64B5F6;
--color-neutral-50-dark:  #121212;
--color-neutral-900-dark: #ECECEC;
 
/* Semantic colors */
--color-success:  #388E3C;   /* COMPLETED, verified mechanic, ACTIVE vehicle */
--color-warning:  #F57C00;   /* PENDING, ON_THE_WAY */
--color-error:    #D32F2F;   /* CANCELLED, INACTIVE vehicle, rejected mechanic */
--color-info:     #1976D2;   /* ACCEPTED, IN_PROGRESS */
--color-neutral:  #757575;   /* OFFLINE, BUSY (mechanic) */
 
/* Text */
--color-text-primary:   #212121;
--color-text-secondary: #757575;
--color-text-disabled:  #BDBDBD;
 
/* Backgrounds */
--color-bg-page:    var(--color-neutral-50);
--color-bg-card:    #FFFFFF;
--color-bg-overlay: rgba(0,0,0, 0.5);
```
 
### Typography
 
```css
/* Family — platform default, no custom font (keeps FCM notifications consistent, RF4.5) */
--font-family-sans: 'Roboto', sans-serif;
 
/* Sizes */
--font-size-caption: 0.75rem;   /* 12px — timestamps, helper text */
--font-size-body:    1rem;      /* 16px — general text, form labels */
--font-size-h2:      1.25rem;   /* 20px — card/section titles */
--font-size-h1:      1.75rem;   /* 28px — screen titles */
 
/* Weights */
--font-weight-regular: 400;
--font-weight-medium:  600;
--font-weight-bold:    700;
 
/* Line height */
--line-height-normal: 1.5;
```
 
### Spacing
 
```css
/* 4px system */
--space-xs: 0.25rem;   /* 4px  — icon-to-label */
--space-sm: 0.5rem;    /* 8px  — compact list rows */
--space-md: 1rem;      /* 16px — default card/screen padding */
--space-lg: 1.5rem;    /* 24px — between sections on a screen */
```
 
### Borders and shadows
 
```css
--radius-sm:   4px;    /* form fields */
--radius-md:   8px;    /* cards */
--radius-full: 9999px; /* status badge (pill) */
 
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);   /* card resting state */
--shadow-md: 0 4px 6px rgba(0,0,0,0.1);    /* live-tracking card, raised */
```
 
---
 
## Components
 
### Status badge
 
The single most-reused component — a pill (`--radius-full`) showing `ServiceRequest.status`
or `Mechanic.status` (see `02-domain/entities-and-rules.md`).
 
| State | Color token | Used on |
|-------|-------------|---------|
| PENDING | `--color-warning` | Tracking screen, Mechanic dashboard |
| ACCEPTED / IN_PROGRESS | `--color-info` | Both |
| ON_THE_WAY | `--color-warning` | Tracking screen |
| COMPLETED | `--color-success` | Request history |
| CANCELLED | `--color-error` | Request history |
| Mechanic: AVAILABLE | `--color-success` | Mechanic dashboard header |
| Mechanic: BUSY / OFFLINE | `--color-neutral` | Mechanic dashboard header |
 
### Buttons
  
| Variant | Use | Disabled state |
|---------|-----|-----------------|
| Primary | Main action ("Request assistance", "Approve") | 40% opacity, not tappable |
| Secondary (outlined) | "Cancel request" | same |
| Danger | "Reject" mechanic, deactivate vehicle | same, requires confirmation modal |
| Ghost/text | "Forgot password?" | same |
 
**Usage rules:**
- Only one Primary action per screen (e.g. only one "Request assistance" button on the Driver dashboard).
- Danger only with modal confirmation (RF2.4, RF3.4).
- Buttons show a loading spinner during async calls (e.g. while `AssignMechanic` runs).
### Forms
 
| Component | When to use |
|-----------|-------------|
| Input text | Plate, brand, model (Vehicle registration) |
| Select | Vehicle type, vehicle picker on "Create request" |
| Toggle | Mechanic AVAILABLE/OFFLINE switch, dark mode |
| DatePicker | Audit log date-range filter (mandatory, RF7.1) |
 
**Error messages:** shown below the field, in `--color-error`, and say how to fix it —
e.g. "This plate is already registered" (RF2.1 uniqueness), not "Invalid plate".
 
### Feedback
 
| Component | When | Duration |
|-----------|------|----------|
| Toast/Snackbar | "Request cancelled", "Diagnostic submitted" | 4 seconds |
| Inline alert | "No mechanic available nearby" (HU-07) | Until dismissed |
| Modal | Cancel request, deactivate vehicle, reject mechanic | Until the user decides |
| Loading spinner | Matching in progress (target: within the SLA) | Until finished |
| Skeleton | Request history list loading | Until loaded |
 
### Data table (Admin only)
 
| Aspect | Behavior |
|--------|----------|
| Pagination | 20 rows per page |
| Filters | Mandatory date range above the table (RF7.1 — no unbounded query) |
| Columns | User, action, module, result, timestamp |
| Empty state | "No log entries for this range" |
 
---
 
## UX patterns
 
### Principles
 
1. **Confirm before destroying:** cancelling a request or deactivating a vehicle always shows a confirmation modal (RF2.4, RF3.4).
2. **Immediate feedback:** every tap has a visual response in < 100ms, even if only a loading state.
3. **Prevent rather than correct:** plate uniqueness and active-vehicle checks happen as the driver types, not only on submit.
4. **Status is always visible:** the badge for the active request never scrolls out of view on the tracking screen.
### Error handling
 
| Scenario | What to show |
|----------|---------------|
| Network error | Toast "No connection. Retrying..." with automatic retry (supports offline queueing, NFR-07) |
| 401 | Redirect to `/auth/login` — "Your session expired" |
| 403 | "You don't have permission to view this" (e.g. Driver hitting `/admin/*`) |
| No mechanic available | Inline alert, not a generic error (HU-07 acceptance criterion) |
| Timeout on matching | Toast "This is taking longer than expected" with a manual retry |
 
---
 
## Accessibility guide minimums
 
| Aspect | Minimum required |
|--------|-------------------|
| Text contrast | WCAG AA (4.5:1 normal text, 3:1 large text) — checked in both themes |
| Touch targets | Minimum 44×44px (buttons, list rows) |
| Form labels | Every field has an associated label |
| Status badge | Never color-only — always paired with the status text (e.g. "ACCEPTED", not just a colored dot) |
 
---
 
## Correlations
 
- Navigation map → `12-ux-ui/navigation-map.md`
- Wireframes → `12-ux-ui/wireframes.md`
- UX non-functional requirements → `04-requirements/non-functional.md`