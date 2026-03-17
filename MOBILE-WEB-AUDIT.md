# The Source v2 — Mobile Responsive UX/UI Audit
**App:** Next.js member portal (mobile responsive view)
**Date:** 2026-03-17
**Context:** Viewed on mobile browser AND embedded as webview inside The Club native app

---

## Executive Summary

The Source v2 has a solid mobile foundation — thoughtful navigation patterns, responsive typography, and drawer-based modals. However, critical gaps in touch targets, keyboard avoidance, image optimization, and webview context awareness fall short of luxury product standards. Most significantly: the app has **no awareness that it runs inside The Club native app**, meaning it shows redundant navigation and footer inside the webview. Touch targets are below standard (40px buttons), forms don't account for the soft keyboard, and there are no sticky CTAs on conversion-critical flows like search and booking.

---

## Mobile Navigation

**Strengths:**
- Hamburger menu correctly hidden on mobile; drawer pattern appropriate
- Nav height: 64px mobile (reasonable)
- Logo/icon sizing appropriate at 48x48px

**Issues:**
1. **No active indicator in mobile drawer** — users can't see which section is active (`display: isMobile ? 'none' : 'flex'` hides underline on mobile) — `Nav/index.tsx` line 456
2. **No breadcrumb or back path in nested states** — destination details → modal → back is disorienting
3. **Profile icon lacks touch affordance** — no visual indication it's tappable on mobile — `Nav/index.tsx` line 496–505

---

## Typography on Mobile

**Strengths:**
- Typography scales responsively across breakpoints
- 16 custom variants covering display, body, labels

**Issues:**
1. **Body text at 14px** — below 16px minimum for mobile legibility — `styles/theme.ts` line 122–127
2. **Canela used at h6 (16px on mobile)** — serif at 16px loses elegance; CLAUDE.md requires 28px+ — `styles/theme.ts` line 117–121
3. **eyebrowSmall and bodyXSmall at 12px** — too small on mobile; no responsive bump — `styles/theme.ts` line 188–193
4. **No max line-length constraint** — text stretches edge-to-edge on mobile

---

## Layout & Spacing on Mobile

**Strengths:**
- Horizontal padding: 25px on mobile (adequate)
- Footer collapses to centered column on mobile
- Bottom-drawer pattern used for modals (correct)

**Issues:**
1. **No explicit 1-column grid collapse** — search results, destination grids lack confirmed responsive breakpoints
2. **Sticky nav may overlap content** — StickyNav height offset not verified for mobile — `Nav/StickyNav/styles.ts` line 42–67
3. **Carousel scroll containment unclear** — Swiper instances may trap horizontal scroll

---

## Touch Targets & Interaction

**Issues:**
1. **`size-md` buttons are 40px** — below 44px iOS/Android minimum — `components/Common/Button/styles.ts` line 141–144
2. **Phone input at 40px height** — `components/Form/InternationalPhoneInput/styles.ts`
3. **Nav drawer tap targets unconfirmed** — no explicit minimum height on list items
4. **No hover-only interactions** (good — all `onClick`)
5. **Swipe gestures absent** beyond carousels (acceptable for now)

---

## Key Flow Analysis

### Search → Filter → Property
- Search form not sticky; keyboard covers CTA button
- No sticky bottom CTA; user must scroll to book
- Filter pills may truncate on narrow screens

### Available Trips / Booking
- No mobile-specific layout adjustments confirmed
- Prospect flow not simplified for mobile

### My Trips
- Sticky TabNav may overlap content; offset calculation not verified for mobile
- Trip cards likely don't stack to 1 column

### Trip Details
- Multi-section layout likely doesn't collapse gracefully on mobile
- Dialog/drawer pattern is correct

### Profile
- Sticky TabNav with multiple tabs may truncate on 375px
- `MobileNavCtaButton` present for Favorites tab only (good pattern, should extend)

---

## Images & Media

**Issues:**
1. **Cloudinary widths not confirmed responsive** — images likely fetched at desktop size on mobile (bandwidth waste + layout shift)
2. **`next/Image` disabled** (`images: { unoptimized: true }` in `next.config.mjs`) — mobile optimization is entirely manual
3. **No `loading="lazy"`** on images below fold
4. **Carousel images not responsive** — full container width, may cause layout shift
5. **Hero image aspect ratio** — constants defined but not verified for correct mobile crop

---

## Forms on Mobile

**Issues:**
1. **Phone input 40px** — below 44px minimum
2. **No keyboard avoidance** — forms obscured by soft keyboard; no `scrollIntoView()` or padding adjustment
3. **Validation errors hidden by keyboard** — `ErrorStyled` renders below input, not above
4. **No mobile-specific input types** — missing `type="tel"`, `type="email"` for correct keyboards
5. **TextField `marginBottom: 32px` hardcoded** — excessive on small screens

---

## Performance Signals

**Strengths:**
- `dynamic()` imports with `ssr: false` on heavy components
- Lazy loading patterns on pages

**Issues:**
1. **~15 context providers** nested in `_app.tsx` — may impact initial mobile load
2. **No `loading="lazy"`** on image components
3. **global.css is large** (~61KB of overrides including Hotjar, Google Places hacks)

---

## Missing Mobile Patterns

1. **No sticky CTA button** on search, booking, trip details flows
2. **No webview detection** — app runs blind inside The Club native app
3. **No pull-to-refresh** (acceptable if not required)
4. **No swipe-to-dismiss** on drawers (enhancement opportunity)
5. **No native app bridge** — can't call native features (share, haptic, push)

---

## Webview Considerations

**Critical — App runs inside The Club native mobile app as a webview:**

1. **No webview detection** — footer and full nav render inside webview (redundant, wastes vertical space)
2. **No native bridge** — can't access auth tokens, push notifications, native share
3. **External links unhandled** — may open inside webview instead of external browser
4. **Android back button conflicts** — web history management may conflict with native back
5. **No testing confirmed** inside actual The Club app

**Recommended detection:**
```typescript
const isWebview = typeof window !== 'undefined' && !!window.ReactNativeWebView;
// or check user agent for The Club app identifier
```

---

## Top 10 Priority Mobile Issues

| # | Issue | Location | Fix |
|---|-------|----------|-----|
| 1 | Button touch targets below 44px (size-md = 40px) | `components/Common/Button/styles.ts` | Enforce 44px minimum across all interactive elements |
| 2 | No sticky CTA on search/booking flows | `pages/search.tsx`, booking modal | Add position:sticky bottom container with CTA + safe-area-inset-bottom |
| 3 | No webview detection — redundant nav/footer inside native app | `pages/_app.tsx` | Add isWebview utility; conditionally hide footer, simplify nav |
| 4 | Cloudinary images not optimized for mobile | All CloudinaryImage components | Add responsive `w_` transforms; cap at 90vw on mobile |
| 5 | Forms don't account for soft keyboard | All form pages | Scroll-to-field on focus; padding-bottom adjustment when keyboard open |
| 6 | Body text at 14px (below mobile minimum) | `styles/theme.ts` | Increase body to 16px on xs/sm breakpoints |
| 7 | No active tab indicator in mobile nav drawer | `components/Nav/index.tsx` | Add background or underline for active nav item |
| 8 | Tab navigation truncation on ≤375px | TabNav on Profile, MyTrips | Responsive font size or label abbreviation for xs |
| 9 | Canela used at h6 (16px mobile) | `styles/theme.ts` | Restrict Canela to h1–h3 only; always Gotham below 28px |
| 10 | No back navigation in nested mobile flows | Nav/drawer stacking | Add back button/breadcrumb to nested states |

---

## Quick Wins

- Change `size-md` button from 40px → 44px (5 min)
- Increase phone input from 40px → 44px (5 min)
- Add active indicator to mobile nav drawer (10 min)
- Add `const isWebview` utility and hide footer in webview (15 min)
- Restrict Canela to h1–h3 in theme (5 min)
- Bump body to 16px on xs/sm breakpoints (5 min)
- Add `loading="lazy"` to CloudinaryImage below fold (15 min)
- Add `type="tel"` to phone inputs, `type="email"` to email inputs (10 min)
- Add aspect-ratio CSS to hero images (10 min)
- Sticky CTA on search page (30 min)
