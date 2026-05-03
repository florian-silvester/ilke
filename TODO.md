# Implementation To-Do List

Source: [website-notes.md](website-notes.md) (client feedback + your clarifications)
Target: `animations.js` and Webflow class adjustments
Order: simplest → most complex

Legend: `[ ]` open · `[x]` done · `[~]` in progress · `🟢` low risk · `🟡` medium · `🔴` high

---

## Quick wins (CSS / Webflow only — minutes)

### [ ] T1 · Index hover — remove darkening 🟢
**From:** notes #13
**Issue:** `.index_thumb_overlay` shows at 15% opacity by default and fades to 0 on hover. Client wants the slight darkening gone — keep the zoom.
**Code:** [animations.js:632-638](animations.js:632), [animations.js:653-659](animations.js:653), [animations.js:675-681](animations.js:675)
**Plan:**
- Change `opacity: 0.15` → `opacity: 0` on lines 637 and 677. The hover-to-0 tween becomes a no-op (harmless), or strip both overlay tweens entirely for cleanliness.
- Zoom logic on `.index_thumb_wrap img` (1.03 idle → 1.0 hover) stays untouched.
**Risk:** None. Pure visual.
**Notes:**

---

### [ ] T2 · Overview grid — fix the "small thumbs" bug 🟢
**From:** notes #6
**Issue:** Overview mode squashes thumbs because the CSS sets grid-template-* but the element stays `display: flex`. Inert grid props on a flex container.
**Code:** [css/ilke-2-0.webflow.css:3071-3084](css/ilke-2-0.webflow.css:3071), [animations.js:1856-1930](animations.js:1856)
**Webflow class to change:** `.swiper-wrapper.is-overview` → set `Display` to **Grid** in Webflow.
**Then in Webflow on that same combo class, replace the 12-col template with:**
```
grid-template-columns: repeat(auto-fill, minmax(8rem, 1fr));
grid-template-rows: unset;
grid-auto-rows: 1fr;
height: auto;
max-height: 100vh;
overflow-y: auto;
```
**Also override on `.swiper-wrapper.is-overview .swiper-slide`:**
```
width: auto;
height: auto;
aspect-ratio: 3 / 2;   /* tune to her image ratio */
```
**Risk:** `.swiper-slide { height: 100vh }` ([css:3328](css/ilke-2-0.webflow.css:3328)) needs the override above. `8rem` and `3/2` are starting values — tune once we see a project with many images.
**Notes:**

---

## Small JS fixes (one function each)

### [ ] T3 · Barba scroll bleed — reset scroll on first visit 🟢
**From:** notes #8 (NEW)
**Issue:** Scrolling down on page A → clicking to page B → page B opens at the same Y. The Barba `after()` hook restores `scrollPositions[next.url.path]` if defined, but does nothing on first visit, so the browser's scrollY from page A leaks into page B.
**Code:** [animations.js:1144-1152](animations.js:1144)
**Plan:** Change the conditional so undefined falls through to `(0, 0)`:
```js
after(data) {
  const storedPosition = scrollPositions[data.next.url.path];
  window.scrollTo(0, storedPosition !== undefined ? storedPosition : 0);
}
```
**Risk:** None — restores prior behavior for known pages, fixes first-visit. May want to do this earlier (in `enter` before fade-in) to avoid a perceptible scroll jump after the fade — note for testing.
**Notes:**

---

### [ ] T4 · Mobile logo split-SVG bug 🟡
**From:** notes #4 + #8 (original)
**Issue:** On mobile scroll-up, sometimes only `.penzlien_svg` reappears, not `.studio_svg`. The current scroll handler animates both as one stagger group ([animations.js:2152-2169](animations.js:2152)), but rapid scroll direction changes can leave one SVG in a half-finished tween while the `svgsVisible` flag flips, so the next scroll event bails out without animating the orphaned element.
**Code:** [animations.js:2114-2207](animations.js:2114) (full scroll handler)
**Plan:**
1. Before each new scroll-triggered tween, kill any in-flight tweens on the SVGs:
   ```js
   gsap.killTweensOf(['.studio_svg', '.penzlien_svg']);
   ```
2. Make the visibility state a function of *actual* opacity rather than a flag, OR force-set both elements' opacity at the start of each handler so the stagger always starts from a known state.
3. Verify with `window.matchMedia('(max-width: 991px)')` if the bug is mobile-only — if yes, the touch-scroll burst rate is the trigger; the fix above still solves it.
**Risk:** Touch the scroll handler that's already complex. Test desktop scroll thoroughly after — could regress the working desktop behavior if state tracking changes.
**Notes:**

---

## Medium JS work

### [ ] T5 · Home intro — scroll into masonry section after logo animation 🟡
**From:** notes #1
**Issue:** After the entrance logo animation finishes, client wants the next section (`.project_masonry_wrap.u-section`) scrolled into view. Currently the hero animates to 70vh and stops there.
**Code:** [animations.js:2076-2107](animations.js:2076) (homepage animation), [index.html:139](index.html:139) (.hero_wrap), [index.html:379](index.html:379) (.project_masonry_wrap.u-section)
**Plan:**
- Add a chained GSAP scroll tween at the end of the hero_wrap animation timeline (in the `onComplete` of the hero `gsap.fromTo`).
- Use `gsap.to(window, { duration: 1.2, scrollTo: { y: '.project_masonry_wrap.u-section', offsetY: 0 }, ease: 'power2.inOut' })` — needs the `ScrollToPlugin`. **Verify it's loaded** — current HTML loads `gsap.min.js` and `ScrollTrigger.min.js` but not `ScrollToPlugin`. May need to add it via CDN.
- Only trigger on first session load (gate behind the existing `heroAnimationPlayed` flag, before it flips), so subsequent home-page entries don't auto-scroll.
- If user has already scrolled or interacted, **abort** the auto-scroll (check `window.scrollY > 50` or set up a one-time wheel/touchstart listener that cancels the tween).
**Risk:** Auto-scrolling can feel hostile if user already started scrolling. The abort logic matters more than the tween itself.
**Notes:**

---

### [ ] T6 · Cursor — faster hover detection during scroll 🟡
**From:** notes #5
**Issue:** When user scrolls fast, mouse stays still in viewport but new images move under it. Browsers don't always fire `mouseenter`/`mouseleave` reliably during fast scroll, so the cursor label stays on the previous image's location text instead of updating to the one now under the cursor.
**Code:** [animations.js:94-181](animations.js:94) (hover handlers), [animations.js:101-165](animations.js:101) (project link mouseenter logic)
**Plan:**
- Track last known mouse coords in `customCursorState` (already partly there — `mousemove` updates `cursor.x/y` via gsap; capture raw event.clientX/Y separately).
- Add a passive scroll listener that, on each scroll event, calls `document.elementFromPoint(lastMouseX, lastMouseY)` and walks up to find the closest `.project_link`.
- If the result differs from the currently-labeled element, manually re-run the same location-extraction logic and call `updateCursorLabel(...)`.
- Throttle to ~16ms (one frame) with `requestAnimationFrame` to avoid spamming on every scroll tick.
**Risk:** New code path can desynchronize from the existing mouseenter/mouseleave handlers. Need a single source of truth for "what is the cursor currently labeling" to avoid flicker.
**Notes:**

---

## CMS-side (mostly Webflow, possibly small JS)

### [ ] T8 · Index — sort chronologically by year + auto-number 🟢
**From:** notes #14
**CMS schema (verified via Webflow MCP):**
- Site: Ilke 2.0 (`688b84e1d55545ae80e8ab02`)
- Collection: Projects (`688b86535785c8eea36d65c7`)
- Date field: `year` (DateTime)
- Number field: `list-position` (Number, integer) — will become obsolete
**DOM structure (verified in project-index.html:357-364):**
```
.index_item
  .index_item_title_wrap
    .project_number   ← write auto-number here
```
**Plan:**
- **Webflow side:** Open the Index page Collection List in Designer → Sort by `Year`, descending. (Newest at top.)
- **JS side:** Add a `numberIndexItems()` function in `animations.js`:
  ```js
  function numberIndexItems() {
    const items = document.querySelectorAll('.index_item');
    items.forEach((item, i) => {
      const numberEl = item.querySelector('.project_number');
      if (numberEl) {
        numberEl.textContent = String(i + 1).padStart(2, '0'); // "01", "02", ...
      }
    });
  }
  ```
- Call it from `animateIndexPage()` (the function called on Barba enter for namespace "index" / "studio" — [animations.js:1167-1177](animations.js:1167)). Also call from `initializeIndexThumbHoverAnimations()` so direct loads work too.
- The `list-position` field can stay in the CMS (ignored) or be removed by you in Designer — JS doesn't care.
**Decision:** Numbering ascending (01 = newest at top) — confirm if you'd rather have descending (highest = newest).
**Risk:** Webflow's `w-dyn-bind-empty` class will be on `.project_number` until JS writes text. Should be fine since JS runs after render. If Webflow re-renders dynamically (unlikely here), the number could blink — easy fix is a MutationObserver, but probably not needed.
**Notes:**

---

## Out of scope (handled by you elsewhere)

- Mobile gallery tap+swipe (#9) — already works
- Mobile back button (#10) — pending the right-nav close-X, deferred
- Mobile nav overlap (#11) — Webflow
- Mobile info text cut-off (#12) — Webflow
- Contact page items (#15-19) — Webflow
- Original "back button on project page" (was #3) — dropped
- Original "movement on scroll down" (was #2) — dropped (covered by T5)
- Gallery loop polish (originally T7) — dropped, current loop-back-to-start behavior stays

---

## Suggested order

1. T1 (5 min, Webflow — instant gratification)
2. T2 (10 min, Webflow — visible win)
3. T3 (5 min, JS — one-line fix)
4. T4 (30 min, JS — needs mobile testing)
5. T5 (45 min, JS — needs ScrollToPlugin + abort logic)
6. T6 (1 hr, JS — careful integration with existing cursor state)
7. T8 (15 min Webflow + 30 min JS — needs DOM inspection first)
