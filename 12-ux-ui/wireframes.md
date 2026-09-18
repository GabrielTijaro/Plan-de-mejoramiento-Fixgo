# Wireframes — FixGo
 
> Low-fidelity structure of the four screens that carry the core flow. ASCII, focused on
> layout and hierarchy — not colors (see `design-system.md` for that).
 
---
 
## 1. Create service request (`/driver/requests/new`)
 
```
┌─────────────────────────────────────┐
│ ← New request                       │
├─────────────────────────────────────┤
│  Vehicle                             │
│  [ Toyota Corolla — ABC123      ▾ ]  │
│                                       │
│  ⚠ No active vehicle?                │
│    → "Register a vehicle" link        │
│                                       │
│  Location                             │
│  [  Current location detected  ]      │
│                                       │
│  ┌─────────────────────────────┐      │
│  │                               │    │
│  │         (map preview)         │    │
│  │                               │    │
│  └─────────────────────────────┘      │
│                                       │
│  [       Request assistance      ]  │  ← primary button
└─────────────────────────────────────┘
```
 
---
 
## 2. Live tracking (`/driver/requests/:requestId/tracking`)
 
```
┌─────────────────────────────────────┐
│  Your request              [PENDING]│  ← status badge
├─────────────────────────────────────┤
│  ┌─────────────────────────────┐     │
│  │                             │     │
│  │     (live map — RTDB)       │     │
│  │         driver   mechanic │       │
│  └─────────────────────────────┘     │
│                                      │
│  Mechanic: Not yet assigned          │
│  ETA: —                              │
│                                      │
│  [          Cancel request         ] │  ← outlined button
│    (only enabled if PENDING/ACCEPTED)│
└─────────────────────────────────────┘
```
 
---
 
## 3. Request detail — Mechanic (`/mechanic/requests/:requestId`)
 
```
┌─────────────────────────────────────┐
│ ← Request #4821          [ACCEPTED]│
├──────────────────────────────────── ─┤
│  Driver: Carlos M.                   │
│  Vehicle: Toyota Corolla — ABC123    │
│  Location: Cra 5 #23-10, Neiva       │
│  Distance: 1.4 km                    │
│                                      │
│  [   Mark as "On the way"   ]        │
│  [   Mark as "In progress"  ]        │
│                                      │
│  [   Submit diagnostic →    ]        │  ← enabled only when IN_PROGRESS
└─────────────────────────────────────┘
```
 
---
 
## 4. Mechanic verification — Admin (`/admin/mechanics/pending`)
 
```
┌─────────────────────────────────────┐
│  Pending verification (3)            │
├───────────────────────────────────── ┤
│  Taller El Motor           [Review →]│
│  Requested: 2 days ago               │
│  ─────────────────────────────────   │
│  AutoFix Express           [Review →]│
│  Requested: 5 hours ago              │
│  ─────────────────────────────────   │
│  Mecánica Rápida           [Review →]│
│  Requested: 1 day ago                │
└─────────────────────────────────────┘
 
   /admin/mechanics/:mechanicId
┌─────────────────────────────────────┐
│ ← Taller El Motor                    │
├───────────────────────────────────── ┤
│  Workshop name: Taller El Motor      │
│  Documents: [view attached files]    │
│                                      │
│  [   ✓ Approve   ]  [   ✕ Reject  ] │
└─────────────────────────────────────┘
```
 
## Correlations
 
- Screens with real routes and roles → `12-ux-ui/navigation-map.md`
- Visual tokens (colors, typography) applied here → `12-ux-ui/design-system.md`