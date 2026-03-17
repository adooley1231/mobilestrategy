# Exclusive Resorts Mobile Strategy
## Consolidated UX/UI Audit + Platform Recommendations
**Date:** 2026-03-17
**Scope:** The Club (React Native) + The Source v2 (Next.js mobile web, embedded as webview)

---

## The Core Problem

Members experience a **fractured mobile product**: a native app (The Club) that embeds the web portal (The Source v2) as a webview for many flows, but neither product knows the other exists. The native app has no design system coherence. The web app doesn't know it's running on a phone. The result is:

- Redundant navigation (native nav + web nav shown simultaneously)
- Inconsistent visual language (different color systems, type scales, spacing)
- No shared definition of "what belongs in native vs web"
- A luxury product that doesn't feel luxurious on the platform where most members will use it

---

## Platform Decision Framework: Native App vs Mobile Web

Before fixing any individual issue, the team needs a clear answer to: **"Does this belong in the native app or in the webview?"**

The framework below provides the principle — then we apply it to specific features.

### When to use the **native app** (React Native)
Use native for experiences that benefit from:
- **Device integration** — camera, GPS, push notifications, biometrics, haptics
- **Offline capability** — content that members need without connectivity
- **Speed/performance** — frequently-accessed screens where 60fps matters
- **Platform conventions** — bottom tab nav, swipe gestures, native pickers

### When to use **mobile web** (webview inside the app)
Use the webview for:
- **Complex content-heavy flows** — destination discovery, editorial content, search
- **Frequently changing content** — anything that changes regularly without an app release
- **Form-heavy flows** — profile, add traveler, account management
- **Administrative flows** — cancellation, policy review, document uploads

### The Anti-Pattern to Avoid
Duplicating the same flow in both native and web. Right now there are parallel implementations of: search/explore, trip details, and potentially profile — creating maintenance burden and inconsistent experiences.

---

## Platform Mapping: Current vs Recommended

| Feature / Flow | Current | Recommended | Rationale |
|---|---|---|---|
| **Login / Auth** | Native (Auth0) | Native | Biometrics, secure token storage |
| **Bottom tab navigation** | Native | Native | Platform convention, instant |
| **Home / Dashboard** | Web (webview) | Native | Frequently accessed, needs speed |
| **Explore / Search destinations** | Both (parallel) | **Web only** | Editorial content, frequently updated |
| **Destination details** | Both (parallel) | **Web only** | Rich content, media-heavy |
| **Property details + Calendar** | Native (complex) | **Web only** | High maintenance, web is more flexible |
| **Filters** | Native | Native or Web | Low complexity; consolidate to web |
| **Available Trips** | Web (webview) | Web | Forms, changes frequently |
| **Book / Request Trip** | Web (webview) | Web | Form-heavy, needs releases less often |
| **My Trips list** | Web (webview) | Web | Can use push notifications from native |
| **Trip Details** | Web (webview) | Web | Complex multi-section layout |
| **Get Ready / Pre-trip** | Native | **Web** | Forms, lots of edge cases |
| **Add Traveler** | Native | Web | Form-heavy, editorial |
| **Flight Info** | Native | Web | Form-heavy |
| **Cancellation** | Native | Web | Policy display + form |
| **Connect / Messaging** | Native (GetStream) | **Native** | Real-time, push notifications required |
| **Profile / Account** | Web (webview) | Web | Forms, changes frequently |
| **Community** | Web (webview) | Web | Content-heavy |
| **Push Notifications** | Native | Native | Requires device integration |

### Summary
**Keep in native:** Login, Navigation shell, Connect/Messaging, Push notifications, Home dashboard
**Move to web / consolidate to web:** Explore, Destination, Property, Get Ready flows (Add Traveler, Flight Info, Cancellation)
**Already correct:** Available Trips, My Trips, Trip Details, Profile, Community

---

## Consolidated Issue List: Both Platforms

This merges the two individual audits into a unified view across the mobile experience.

### Category 1: Design System — No Shared Language

| Issue | Native App | Mobile Web |
|-------|-----------|------------|
| Color palette | 70+ colors, 4 error reds | Different palette entirely |
| Typography scale | 23 sizes, bad naming | 16 variants, cleaner but Canela misused |
| Spacing tokens | Partially defined, ignored | Partially defined, ignored |
| Border radius | No system | No system |
| Button heights | Inconsistent (45/50px) | 40px (below minimum) |
| Font families | Raleway + Canela | Gotham + Canela |

**The Gap:** Two separate design systems with no shared tokens. Members see different visual languages depending on whether they're in native or webview.

### Category 2: Accessibility — Both Platforms Failing

| Issue | Native App | Mobile Web |
|-------|-----------|------------|
| Touch targets | Below 44px on icons | 40px buttons (below standard) |
| Labels on interactive elements | 80%+ missing | Inconsistent |
| Color contrast | brand.light fails WCAG | Some grays borderline |
| Screen reader support | No VoiceOver testing | No a11y testing documented |
| Semantic roles | Missing throughout | Missing throughout |

### Category 3: Navigation — Fractured Experience

| Issue | Native App | Mobile Web (in webview) |
|-------|-----------|------------|
| Redundant nav | Native tab bar shows | Web nav also shows in webview |
| Redundant footer | N/A | Web footer shows in webview |
| Back navigation | Inconsistent | Conflicts with native back button |
| Active state indicator | Not shown on mobile | Hidden on mobile drawer |
| Deep linking | 6/26 routes mapped | N/A (web has its own routes) |

### Category 4: Forms — Both Broken on Mobile

| Issue | Native App | Mobile Web |
|-------|-----------|------------|
| Keyboard avoidance | Not confirmed | No keyboard avoidance pattern |
| Field height | Default RN (ok) | Phone input 40px (too small) |
| Validation feedback | Only button disable | Errors hidden below keyboard |
| Input types | Mostly correct | Missing tel/email types |
| Labels | Default floating | Default MUI floating |

### Category 5: Loading & Error States

| Issue | Native App | Mobile Web |
|-------|-----------|------------|
| Loading indicators | Inconsistent (3 patterns) | Not audited in detail |
| Skeleton states | One-off, not reused | Skeleton component exists, needs audit |
| Error visibility | Hidden in production | Toast-based (ok) |
| Button state during load | Remains clickable | Not confirmed |

### Category 6: The Webview Gap (Critical)

The web app running inside the native app has **zero awareness** it's in a webview:
- Shows full navigation header (redundant with native app)
- Shows full footer (useless real estate)
- Doesn't share auth state with native app (or unclear if it does)
- No bridge to native features (push, haptic, share)
- Android back button behavior unhandled

---

## Priority Matrix

Issues ranked by **Impact × Effort**:

### P0 — Fix First (High impact, achievable in a sprint)

| # | Issue | Platform | Effort |
|---|-------|----------|--------|
| 1 | **Webview detection + hide nav/footer inside app** | Web | S |
| 2 | **Button touch targets: enforce 44px minimum** | Both | S |
| 3 | **Decide and document native vs web platform ownership** | Strategy | S |
| 4 | **Stop duplicating Explore/Search in both platforms** | Both | M |
| 5 | **Active nav indicator in mobile drawer** | Web | S |

### P1 — High Value, Next Quarter

| # | Issue | Platform | Effort |
|---|-------|----------|--------|
| 6 | **Unified design token layer** (shared colors, type scale, spacing) | Both | L |
| 7 | **Sticky CTA on conversion flows** (search, booking) | Web | M |
| 8 | **Keyboard avoidance on all forms** | Both | M |
| 9 | **Consolidate native Get Ready flows → web** | Native→Web | L |
| 10 | **Complete deep linking** in native app | Native | M |
| 11 | **Responsive Cloudinary images** | Web | M |
| 12 | **Typography cleanup** — reduce to 8 semantic sizes | Both | M |

### P2 — Polish & Completeness

| # | Issue | Platform | Effort |
|---|-------|----------|--------|
| 13 | **Accessibility audit + remediation** (labels, roles, contrast) | Both | L |
| 14 | **Form component library** (consistent inputs across web) | Web | L |
| 15 | **Build native button + component library** | Native | L |
| 16 | **Native → Web bridge** (auth, push, share) | Both | L |
| 17 | **Loading/skeleton states** (consistent pattern) | Both | M |
| 18 | **Remove deprecated screens from native nav** | Native | S |
| 19 | **Color system consolidation** | Both | L |
| 20 | **Error states visible in production** | Native | M |

---

## Recommended Design Approach

### Phase 1: Establish the Contract (Weeks 1–2)
**Goal:** Define ownership and eliminate duplication.

1. Document the platform map (native vs web for each feature) — ratify with product/engineering
2. Add webview detection to The Source v2 (`isWebview` flag)
3. Conditionally hide web nav + footer when running inside The Club
4. Establish a single source of truth for design tokens — shared CSS custom properties that map to both platforms
5. Remove the 8 deprecated screens from the native app

**Output:** A clear platform architecture doc + webview-aware web app

---

### Phase 2: Fix the Foundation (Weeks 3–6)
**Goal:** Close the critical UX gaps that affect all members.

1. **Touch targets** — 44px minimum enforced on both platforms
2. **Typography** — reduce to 8 semantic sizes, Canela rules enforced
3. **Sticky CTAs** — search, booking, and trip detail flows get persistent bottom CTAs
4. **Keyboard avoidance** — forms on both platforms handle soft keyboard
5. **Active nav states** — mobile drawer shows active section

**Output:** Both apps meet baseline mobile UX standards

---

### Phase 3: Consolidate Native Flows → Web (Weeks 7–12)
**Goal:** Reduce maintenance burden by migrating Get Ready flows to web.

1. Move Add Traveler → web form (retire native version)
2. Move Flight Info → web form
3. Move Cancellation → web form
4. Unify Explore/Search → web only (retire native duplicate)
5. Build webview bridge: auth token sharing, native back button handling

**Output:** Native app is a shell + messaging/notifications; web handles all content flows

---

### Phase 4: Elevate to Luxury (Ongoing)
**Goal:** Visual and interaction quality worthy of the brand.

1. Shared design token system (colors, typography, spacing) applied to both platforms
2. Accessibility remediation (WCAG AA compliance across both platforms)
3. Form component library (web) + button/component library (native)
4. Loading/skeleton states — consistent across all screens
5. Image optimization — responsive Cloudinary transforms

**Output:** Cohesive, premium feel across the entire mobile experience

---

## What "Done" Looks Like

A member using The Club app should:
1. Open the app → see their native dashboard (fast, offline-capable)
2. Tap Explore → seamless transition to web, no double nav visible, no footer
3. Search, filter, and browse destinations with sticky CTAs and responsive images
4. Select dates → book a trip → form works with keyboard visible, validates inline
5. See My Trips → tap a trip → full web trip details without friction
6. Tap Connect → native real-time messaging (never leaves native)
7. Everything at 16px+ body text, 44px+ tap targets, Canela only at 28px+

---

## Files
- `theclub-audit/UX-UI-AUDIT.md` — Native app (The Club) full audit
- `theclub-audit/MOBILE-WEB-AUDIT.md` — Web portal (The Source v2) mobile audit
- `theclub-audit/MOBILE-STRATEGY.md` — This document
