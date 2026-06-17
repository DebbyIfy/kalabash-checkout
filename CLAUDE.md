# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the project

These are static files — open any `.dc.html` directly in a browser (no build step, no server required):

```
open "Kalabash Checkout.dc.html"
open "Kalabash Checkout Mobile.dc.html"
open "Kalabash Checkout (export).dc.html"
```

`support.js` is the compiled dc-runtime. If you need to rebuild it: `cd dc-runtime && bun run build`. **Do not edit `support.js` directly.**

---

## dc-runtime template format

`.dc.html` files wrap content in `<x-dc>...</x-dc>`. A `<script type="text/x-dc" data-dc-script>` block at the bottom holds the component logic. The runtime parses the document, compiles the template, and renders it using `window.React`.

### Template syntax

| Syntax | Meaning |
|---|---|
| `{{ varName }}` | Interpolation — resolved from `renderVals()` |
| `<sc-if value="{{ expr }}">` | Conditional block (truthy = shown) |
| `<sc-for list="{{ arr }}" as="item">` | List rendering |
| `<x-import component-from-global-scope="Name" from="./file.jsx">` | Import a JSX component into the template |
| `<dc-import name="OtherDC">` | Import another `.dc.html` component |
| `<helmet>` | Inject into `<head>` (styles, meta, fonts) |
| `style-focus="..."` | Pseudo-class styles applied as a class at runtime (`:focus`, `:hover`, etc.) |

`hint-placeholder-val` on `<sc-if>` is the value shown during streaming mode only — it has no effect when rendered normally.

### Component logic

```js
class Component extends DCLogic {
  state = { screen: 'signin', ... };

  renderVals() {
    // Return the flat object the template renders against.
    // All {{ expressions }} resolve from this object.
    return { isSignin: this.state.screen === 'signin', ... };
  }

  // Standard React-style lifecycle
  componentDidMount() {}
  componentDidUpdate(prevProps, prevState) {}
  componentWillUnmount() {}
}
```

`setState` works identically to React class component `setState`. `renderVals()` is the equivalent of `render()` — it produces the prop bag that all `{{ }}` interpolations and `<sc-if>` expressions evaluate against.

### Configurable props

The `data-props` JSON attribute on the `<script>` tag declares externally-settable props:

```json
{
  "$preview": { "width": 1200 },
  "balanceLow": { "editor": "boolean", "default": true }
}
```

`$preview` controls the preview viewport width. All other keys become props accessible via `this.props` inside the component.

---

## File map

| File | Purpose |
|---|---|
| `Kalabash Checkout.dc.html` | Main checkout — desktop + responsive (renders mobile layout via `isMobile` breakpoint at 768px) |
| `Kalabash Checkout Mobile.dc.html` | Checkout wrapped inside an iOS 26 device frame for mobile preview |
| `Kalabash Checkout (export).dc.html` | Print/export variant |
| `Kalabash Wallet Checkout (1).html` | Plain HTML prototype (not dc-runtime) |
| `ios-frame.jsx` | iOS 26 Liquid Glass device frame; exports `IOSDevice`, `IOSStatusBar`, `IOSNavBar`, `IOSGlassPill`, `IOSList`, `IOSListRow`, `IOSKeyboard` to `window` |
| `support.js` | Compiled dc-runtime — do not edit |
| `assets 2/kalabash-logo.png` | Brand logo |
| `_ds/` | Kalabash design system — tokens, component JSX, and documentation |

---

## Checkout flow

The `screen` state value drives which panel is shown:

```
signin → checkout
              ↓ (sufficient balance)        ↓ (insufficient balance)
             pin                         topup → otp → pin
              ↓
           success

pin → resetOtp → newPin → pin   (PIN reset path)
```

The `balanceLow` prop (boolean) simulates wallet balance state:
- `true` → wallet ₦20,000, order ₦25,000 → insufficient path
- `false` → wallet ₦30,000 → sufficient path

---

## Design system (`_ds/`)

Token and component source lives in `_ds/kalabash-design-system-*/`. Key design rules:

- **Primary blue**: `#07478C` — CTAs, active states, brand surfaces
- **Fonts**: Sora (headings, display numbers, bold UI labels) · Inter (body, captions, form labels)
- **Spacing**: 4-point scale: `4 · 8 · 12 · 16 · 24 · 32`
- **Radii**: `8px` inputs/badges, `12px` buttons/cards, `16px` modals/major surfaces, `9999px` pills
- **Icons**: Lucide SVGs inline — 24×24 viewbox, `stroke-width 2`, `stroke-linecap round`. Stroke only, no fill. Color via `currentColor`.
- **Voice**: Sentence case everywhere. No title case, no exclamation marks, no emoji in product UI. Verbs first on CTAs ("Sign in to pay", not "Payment sign-in").
- **Semantic colors**: Success `#22C55E`, Error `#EF4444`, Warning `#FACC15`, Info `#54D8FF`

See `_ds/.../README.md` for the full token reference and component documentation.
