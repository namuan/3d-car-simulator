# Mobile Adaptation Plan — Scenic Route

## Philosophy
Game must be fully playable on iPhone (375-414px wide) in both portrait and landscape, without external hardware. Touch-first, fingers as controllers. Minimize geometry for mobile GPU, maximize touch target sizes.

---

## 1. Touch Controls *(no keyboard)*

### Virtual Gamepad Overlay
- **Left side:** Steering — two large touch zones (tap/hold for lane change left/right)
- **Right side:** Throttle — two vertical zones (top half = accelerate, bottom half = brake)
- **Space bar → bottom center area** for brake redundancy

### Lane Change
- **Tap left zone** (left of center, roughly 40% of screen) → change left lane
- **Tap right zone** (right of center, roughly 40% of screen) → change right lane
- Hold doesn't repeat — one tap, one lane change
- Visual feedback: zone briefly flashes on touch

### Acceleration / Braking
- **Press and hold upper-right quadrant → accelerate** (W/↑ equivalent)
- **Press and hold lower-right quadrant → brake** (S/↓/Space equivalent)
- Visual: quadrant highlights when active

### Toggle Controls (View, Night, Reset)
- **Swipe up from bottom edge** → toggle view (FPV/top-down)
- **Swipe down from top edge** → toggle day/night
- **Long press anywhere (1.5s)** → reset/restart
- Or: small fixed buttons at bottom

### Orientation
- **Default:** landscape orientation lock recommended (wider FOV, better controls)
- Portrait also supported (controls positioned vertically)

---

## 2. UI Adjustments for Small Screens

| Element | Current | Mobile | Notes |
|---|---|---|---|
| Top bar | Centered, 16px top | Full width, 8px top, smaller padding | `vw` units for padding/gap |
| Speed gauge | bottom:120px, right:24px, 42px font | bottom:80px, right:8px, 28px font | Keep it out of touch zones |
| Lane indicator | bottom:120px, left:24px | bottom:80px, left:8px, smaller dots | Compact |
| Controls bar | Bottom, lots of text | **Hidden by default** — show on tap of ⚙️ icon | Save screen space |
| Speed lines | Full screen | Disabled on mobile (perf) | Too much CPU for CSS animations |
| Game Over | Full overlay | Same, but larger button (min 48px height) | Touch-friendly tap target |
| Font sizes | 13-15px | Scale to ~14px minimum | Readable on small screen |
| Touch targets | Button 6-14px height | **All interactive elements ≥44px** | Apple HIG requirement |

### Responsive Breakpoints
- `max-width: 480px` — phone portrait
- `max-width: 820px` — phone landscape / small tablet
- `max-height: 480px` — landscape phone (overrides width check)

---

## 3. Performance Optimizations

### Geometry Reduction
| Change | Detail | Saving |
|---|---|---|
| Trees: halve count | Mobile 25 trees (vs 50) | ~50% tree geo |
| Mounds: reduce | Mobile 20 mounds (vs 40) | ~50% mound geo |
| Bushes: reduce | Mobile 40 bushes (vs 80) | ~50% bush geo |
| Mountains: reduce | Mobile 10 mountains (vs 16) | ~38% mountain geo |
| Guardrails: keep poles, skip rail bars | Remove the horizontal bar meshes | ~50% guardrail geo |

### Rendering
- **Pixel ratio:** `Math.min(dpr, 1.5)` on mobile (was 2)
- **Shadows:** keep `PCFSoftShadowMap` but reduce mapSize to 512 (was 1024)
- **Tone mapping:** keep ACES
- **Anti-alias:** can be disabled on low-end for extra perf

### JS
- **Speed lines:** disabled entirely on touch devices
- **CSS backdrop-filter:** disabled on mobile (high GPU cost)
- **Animations:** keep `requestAnimationFrame`, frame time clamped to 0.05s

---

## 4. Mobile-Specific Edge Cases

### Audio
- iOS requires user gesture before AudioContext starts
- Already handled: `ensureAudio()` on first keydown
- **Add:** also call `ensureAudio()` on first touchstart

### Fullscreen
- On launch, detect mobile and offer fullscreen button
- Or auto-request fullscreen on first tap (respects browser policy)

### Viewport
- Already have `<meta name="viewport" content="width=device-width, initial-scale=1.0">`
- **Add:** `maximum-scale=1.0, user-scalable=no` to prevent pinch-zoom during gameplay
- **Add:** `viewport-fit=cover` for iPhone notch area

### Safe Areas
- iPhone X+ has notch/safe areas
- Use `env(safe-area-inset-*)` in CSS for UI positioning
- Top bar: `top: calc(env(safe-area-inset-top) + 8px)`
- Bottom controls: `bottom: calc(env(safe-area-inset-bottom) + 8px)`

### Touch Events vs Mouse
- All `.hover` selectors need `@media (hover: hover)` wrapper
- `:active` states instead of `:hover` visual feedback
- Prevent `touch-action: manipulation` to avoid double-tap zoom
- Touch areas need `pointer-events: auto` while dead zones need `none`

### Double-Tap Prevention
- `touch-action: manipulation` on body
- No zoom on double-tap in game area

### Orientation
- Suggest landscape via overlay if in portrait (`@media (orientation: portrait)`)
- Auto-detect and show rotation hint

---

## 5. Implementation Order

1. **Media query scaffolding** — add mobile detection JS + CSS breakpoints
2. **Touch controls** — virtual D-pad overlay (steer left/right, gas/brake)
3. **UI resize** — scale all fixed elements, use vw/vh, safe-area padding
4. **Performance** — reduce geometry on mobile, lower pixel ratio, shadow res
5. **Polish** — orientation hint, fullscreen, audio gesture fix, remove hover effects
6. **Test** — verify on iPhone simulator or real device

---

## 6. Files Changed
- `index.html` — all CSS, HTML, JS in single file
