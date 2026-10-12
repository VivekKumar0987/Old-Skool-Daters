# Old Skool Daters — UI Redesign: Final Improvements Report

**Version:** Final
**Scope:** Frontend presentation layer only
**Stack:** React 18 + TypeScript + MUI v5 + MapLibre GL + Zustand + Vite PWA
**Reference plan:** `OLD_SKOOL_DATERS_UI_REDESIGN_PLAN.md`
**Base doc:** `PROJECT_DOCUMENTATION.md`

---

## 1. What this report covers

A consolidated record of every UI improvement applied to the frontend presentation layer:
the design system, new shared primitives, and the redesigned screen components.

**Not touched (by design):** backend Express routes, SQLite schema, shared API contracts,
authentication mechanism, discovery selection logic, and external connection handoff.

`git` history was not available for this workspace, so this report reflects the file state
produced this session.

---

## 2. Design system (`theme/theme.ts`)

The theme was rewritten into a single factory `makeAppTheme(mode)` with light and dark
variants, replacing hard-coded colors scattered across components.

### 2.1 Design tokens

| Token | Value | Usage |
|---|---|---|
| Primary | `#7053A6` | Buttons, selected chips, focus |
| Primary hover | `#5B418D` | Hover states |
| Primary soft | `#F1EAF9` | Soft selected backgrounds |
| Focus ring | `#B89BE1` | Keyboard focus outline |
| Accent mauve | `#8D5C7B` | Secondary highlight / Wheel accent |
| Page background | `#F8F6FC` | Content backgrounds |
| Surface | `#FFFFFF` | Cards, panels, sheets |
| Text primary | `#25223A` | Titles and body |
| Text secondary | `#686577` | Supporting copy |
| Border | `#E7E2EC` | Borders and dividers |
| Success | `#26784D` | Online status |
| Uni accent | `#2678C8` | Uni Date mode |
| BADDate accent | `#B93737` | BADDate mode |

Dark mode uses matching semantic pairs: background `#171522`, paper `#231F30`,
divider `#3B3548`, text `#F7F3FA` / `#C2BCCD`.

### 2.2 Per-mode accents

```ts
export const modeAccent = {
  random:     '#7053A6',
  wheel:      '#8D5C7B',
  baddate:    '#B93737',
  university: '#2678C8',
  referral:   '#5B6A91',
};
```

### 2.3 Typography

Inter-first stack with a system fallback. Heading weights 750 with letter-spacing
`-0.025em` to `-0.03em`. Body `0.975rem/1.55`, secondary `0.9rem/1.5`, buttons
`0.9rem/650` with `textTransform: none`.

### 2.4 Shape, spacing, motion

- 8px spacing grid; base radius 16; controls 13–14; cards 22.
- Soft shadows (max ~`0 12px 34px`), never competing with map labels.
- Motion: 120–180ms hover/press, 180–240ms drawer/dialog.
- Named keyframes added globally: `osk-slide-up`, `osk-fade-in`, `osk-pop-in`, `osk-spin`.
- `prefers-reduced-motion` collapses animations and transitions to `0.01ms`.

### 2.5 Component overrides

Centralized defaults for: `MuiButton` (min 44px, 13 radius), `MuiOutlinedInput`
(min 48px, 14 radius, 16px text), `MuiCard`, `MuiPaper`, `MuiDialog`, `MuiDrawer`,
`MuiChip`, `MuiIconButton`, `MuiTooltip`, `MuiListItemButton`, `MuiDivider`,
`MuiAvatar`, `MuiSwitch`, `MuiSnackbar`, and `MuiCssBaseline` (box-sizing, no horizontal
overflow, `:focus-visible` ring, reduced-motion block).

### 2.6 Real dark-mode wiring (new)

`theme/colorMode.ts` adds `getColorMode` / `setColorMode` / `subscribeColorMode`.
`App.tsx` now reads and subscribes to the persisted preference, so the Settings
dark-mode toggle drives the actual theme (previously the toggle was local-only).

---

## 3. New shared UI primitives (`components/ui/`)

| Primitive | Purpose |
|---|---|
| `AppBottomSheet` | Responsive drawer: full-width bottom sheet on mobile, centered ≤640px on desktop. Drag handle, titled header, scrollable body, optional sticky footer. |
| `AppDialog` | Centered modal shell (max 460px) with accent icon, title, subtitle, close. Reused by all discovery states. |
| `AppEmptyState` | Iconed zero-content state; used for no contacts, no results. |
| `SectionHeader` | Title + description + trailing action pattern. |
| `StatusPill` | Online/offline pill — conveys status by dot **and** text, never color alone. |
| `MapActionButton` | Consistent 48px round map control with required `aria-label` + tooltip. |

Barrel export added at `components/ui/index.ts`.

---

## 4. Foundation and viewport hygiene

- `index.html`: `viewport-fit=cover` added for iOS safe areas.
- `styles/global.css` (new): margin/min-height reset, box-sizing, `:focus-visible` ring,
  MapLibre control offsets so zoom/fullscreen controls clear the search pill and menu,
  reduced-motion block.
- `main.tsx`: imports `global.css`.
- `App.tsx`: dark mode via `colorMode` module; reduced unused imports.

---

## 5. Component-by-component improvements

### 5.1 AuthPage (login / signup)

- Soft lavender gradient backdrop; centered card capped at **420px**, padding 24–32px.
- Gradient logo tile + gradient title treatment; clear subtitle.
- Inputs use theme height (48px) and 16px text.
- Solid primary button (safer contrast) at 48px with inline spinner while loading.
- Password toggle now has `aria-label="Show password" / "Hide password"`.
- Inline error alert with `role="alert"`.
- Form fields, validation, and submit handlers unchanged.

### 5.2 MapPage (shell / layout)

- Full-bleed map at `100dvh` with `overflow: hidden` (removes horizontal scroll).
- Search pill centered at top with safe-area padding; single, non-duplicated top control.
- Discovery modal now driven purely by `currentMode` (removed the redundant
  `showDiscoveryModal` flag path).
- `handleTalkToMe` typed against `MapUser | DiscoveryModeResult['user']`.
- Fixed a misplaced `import { useState }` that sat at the bottom of the file.
- Exported `BOTTOM_NAV_HEIGHT` so overlays position against the real nav height.

### 5.3 MapView (map chrome)

- Markers use design tokens: online ring `#26784D`, offline `#686577`.
- Markers carry `role="button"` and `aria-label` (name + status).
- Avatar fallback shows a gradient initial tile.
- Clusters use the brand gradient with a white ring and shadow.
- Zoom control moved to top-left; fullscreen to top-right; both offset via CSS.
- Map height is `100%` of its container (no forced `100vh`).
- Drag, zoom, cluster click, fly-to, fetch, and selection behavior unchanged.

### 5.4 BottomNavigation

- Five destinations preserved: **Map / Discover / Random / BADDate / Uni Date**.
- Active tab gets primary color + bold label; state syncs with `currentMode`.
- Bar height `72px + safe-area-inset-bottom`; min 56px touch targets.
- Floating action stack (Filters / Search / My location) uses `MapActionButton`,
  positioned above the real nav height.
- Account menu (Edit Profile / Settings / Log out) is an accessible `Menu` popover.

### 5.5 SearchBar

- Pill-shaped 48px field, 16px text, `id="osk-search"` (focusable from the map action).
- Selecting a result now selects the user, centering the map and opening the preview card.
- Distinct loading / no-results / results states in an anchored surface card.
- `StatusPill` + distance shown per row; clear button has `aria-label`.
- Autocomplete debounce and search endpoint unchanged.

### 5.6 UserCard

- Rebuilt on `AppBottomSheet` with drag handle, `@username` title, close.
- Avatar, `StatusPill`, optional distance chip, and line-clamped description.
- **Talk to Me** as the full-width primary action.
- No invented Save/Like actions.

### 5.7 ConnectionSheet

- Rebuilt on `AppBottomSheet`; compact, content-driven height (no giant blank panel).
- Header shows avatar + "Connect with @username".
- Fetches the full profile via the existing `GET /users/username/:username` endpoint so
  **social handles and links actually load** (the map/list payload only carries a summary).
- Rows: platform icon, label, handle, plus copy and open actions (`noopener,noreferrer`).
- Real empty state via `AppEmptyState`: "No connection methods yet" with a Close action.
- Copy gives inline check feedback; touch targets ≥44px.

### 5.8 DiscoveryModal

- Single shared shell (`AppDialog`) with five accent variants.
- Mode header: icon, title, and UI-only subtitle per mode.
- **Wheel of Love now fetches a result** (previously it spun forever because the fetch was
  never triggered); the spin animation still plays while loading, with no artificial delay.
- Result: accent-ringed avatar, `@username`, status pill + mode chip, clamped bio.
- Primary **Talk to Me** uses the mode accent; **Discover Another** is outlined.
- Loading and error keep a stable minimum height (no layout jumps); error offers Try Again.
- All endpoint calls and handlers preserved.

---

## 6. Files changed this session

| File | Change |
|---|---|
| `src/theme/theme.ts` | Rewritten: tokens, `makeAppTheme`, overrides, keyframes |
| `src/theme/colorMode.ts` | **New** — persisted dark-mode module |
| `src/styles/global.css` | **New** — viewport and MapLibre hygiene |
| `src/main.tsx` | Import global styles |
| `index.html` | `viewport-fit=cover` |
| `src/App.tsx` | Dark mode via `colorMode` module |
| `src/components/ui/AppBottomSheet.tsx` | **New** |
| `src/components/ui/AppDialog.tsx` | **New** |
| `src/components/ui/AppEmptyState.tsx` | **New** |
| `src/components/ui/SectionHeader.tsx` | **New** |
| `src/components/ui/StatusPill.tsx` | **New** |
| `src/components/ui/MapActionButton.tsx` | **New** |
| `src/components/ui/index.ts` | **New** — barrel |
| `src/pages/AuthPage.tsx` | Redesigned |
| `src/pages/MapPage.tsx` | Redesigned shell |
| `src/components/MapView.tsx` | Redesigned chrome/markers |
| `src/components/BottomNavigation.tsx` | Redesigned |
| `src/components/SearchBar.tsx` | Redesigned |
| `src/components/UserCard.tsx` | Redesigned |
| `src/components/ConnectionSheet.tsx` | Redesigned (+ full-profile fetch) |
| `src/components/DiscoveryModal.tsx` | Redesigned (+ wheel fetch fix) |

---

## 7. Behavior preserved (regression contract)

- Login, registration, session persistence, logout.
- All five discovery endpoints and their result selection.
- Map pan/zoom/cluster/fly-to/selection and geolocation updates.
- Talk to Me → external connection handoff (no in-app chat).
- Profile editing fields, social handles, external links, status, search, filters.
- Backend routes, shared contracts, SQLite schema, and seed data untouched.

---

## 8. Verification status

**Not yet run in this pass.** Before shipping, run:

```bash
cd packages/frontend
pnpm build      # tsc + vite build
```

Then smoke-test at 320 / 390 / 768 / 1440px for zero horizontal scroll, keyboard
focus order, Escape-to-close, dark mode, and the five discovery modes.

---

## 9. Remaining work (not completed)

These items from the plan were started but not finished:

1. **FilterBar** — chips, radius slider/value pairing, sticky Reset/Apply footer.
2. **ProfileSheet** — grouped sections and sticky Save footer (restyle only).
3. **SettingsSheet** — grouped rows, desktop panel vs mobile sheet, disabled/coming-soon
   treatment for unbacked rows.
4. **§8 horizontal-overflow audit** across all breakpoints.
5. **Phase E QA** — before/after screenshots, a11y smoke tests, build/test run.

---

## 10. Known issues to verify (out of UI scope)

- **Auth response unwrapping:** `src/api/index.ts` `request()` returns the full envelope
  (`{ success, data }`), but `useAuth` reads `data.token` / `data.user` at the top level.
  If confirmed, authenticated flows (wheel opposite-gender, status/location persistence)
  are affected. This is a frontend logic fix, not styling, and was intentionally left for
  a separate change.
- **Profile save persistence:** `ProfileSheet` saves through the Zustand store rather than
  the `PUT /auth/me` endpoint. Restyle should not silently change this; confirm intent.
- **`GET /users/:id`:** mounted at `/api/users/users/:id`, so `api.users.getById` 404s.
  The redesigned ConnectionSheet avoids this by using `/users/username/:username`.
