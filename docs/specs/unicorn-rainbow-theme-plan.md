# Plan: Unicorn rainbow theme

## Context

The spec in `docs/specs/unicorn-rainbow-theme.md` asks for a new look for Light mode, because today's off-white with terracotta reads as black and white. The new look has:

- a full, slowly shimmering rainbow page background
- frosted-glass panels on top of it
- a 🦄 in the header
- a theme button that cycles **Rainbow 🦄 → Dark → Auto**

Dark mode doesn't change. The budget is about 20 minutes, with no server or database changes.

The key decision is to keep the internal mode value and the `<html>` class as `light`. Then none of these need edits:

- the theme-init script in `apps/web/src/routes/__root.tsx`
- the `theme` value saved in localStorage
- `useResolvedTheme` (`src/hooks/use-resolved-theme.ts`)
- `ThemedToaster`

The no-flash reload keeps working too.

All paths below are under `apps/web/src/`.

## Phase 1: Rainbow tokens and background (`styles.css`)

1. **Retune the `:root, .light` block. Leave `.dark` untouched.**
   - `--background`: a pastel fallback, such as a light lavender. The `@apply bg-background` on `body` still runs, so the gradient sits on top of this colour.
   - `--card`, `--popover`, `--secondary`, `--muted`, `--accent`, `--input`, `--sidebar-accent`: translucent white, `oklch(1 0 0 / 0.7)`.
   - `--border`, `--sidebar-border`: `oklch(1 0 0 / 0.55)`.
   - `--primary`, `--ring`, `--sidebar-primary`, `--sidebar-ring`: deep violet, about `oklch(0.5 0.22 300)`. Set `--primary-foreground` and `--sidebar-primary-foreground` to white, which gives a contrast of at least 4.5:1.
   - `--brand-label`: darker violet, so the `.island-kicker` eyebrow text passes AA on frosted white.
   - `--chart-1` to `--chart-5`: rainbow hues.
   - `--muted-foreground`: darken it slightly if the Phase 3 contrast check fails.
2. **Add a `.light body` rule:**
   - `background-image: linear-gradient(135deg, red, orange, yellow, green, blue, violet, red)` in saturated oklch stops
   - `background-size: 400% 400%`
   - `background-attachment: fixed`
3. **Add the shimmer:** a `@keyframes rainbow-shimmer` animation on `background-position` (0% 50% → 100% 50% → 0% 50%, about 18s, `ease-in-out`, infinite). Apply it to `.light body` inside `@media (prefers-reduced-motion: no-preference)`. This follows the same reduced-motion pattern as the existing `.rise-in` rule.
4. **Add the frosted blur:** `.light :is([data-slot="card"], [data-slot="popover-content"], [data-slot="dialog-content"], [data-slot="alert-dialog-content"], [data-slot="dropdown-menu-content"], [data-slot="select-content"], [data-slot="sheet-content"], .island-shell)` with `backdrop-filter: blur(12px)`. `.island-shell` already uses `var(--card)`, so it turns translucent through the tokens alone.
5. **Fix the nav underline:** `.nav-link::after` uses `--brand-terracotta`. Under `.light`, override it to `var(--primary)` so the underline isn't terracotta on rainbow.

## Phase 2: Toggle label and unicorn (`components/`)

1. **`theme-toggle.tsx`:**
   - Set `MODE_LABEL.light` to `"Rainbow"`.
   - Render `<span aria-hidden="true">🦄</span>` in place of the `Sun` icon when `mode === "light"`. Drop `Sun` from `MODE_ICON` and the import, so `MODE_ICON` holds only `dark` and `auto`.
   - Change the `aria-label`/`title` so the light mode reads "Theme mode: rainbow".
   - The `NEXT_MODE` order already starts the cycle at Rainbow → Dark → Auto.
2. **`header.tsx`:**
   - Add `<span aria-hidden="true">🦄</span>` next to "Teamdesk" in the brand link (line ~14).
   - The header uses `bg-background/90`, which would be an opaque pastel bar. Add `light:`-scoped frosted styling, either a `.light header` rule in `styles.css` (`background: oklch(1 0 0 / 0.6)`) or a class on the header. Prefer the CSS rule so all the theme code stays in one file.

## Phase 3: Verify

1. Run `bun run check` from the repo root. It covers lint and typecheck.
2. Start the dev server with `preview_start` (`dev` in `.claude/launch.json`) and open http://localhost:3000. Then check:
   - **Rainbow mode:** a shimmering gradient, frosted cards and header, and 🦄 in both the header and the button. Screenshot the home page and `/bookings`.
   - **The button:** it cycles Rainbow → Dark → Auto. After a reload the chosen mode is kept, and the old light theme doesn't flash.
   - **Dark mode:** compare it with the current look; it should be unchanged.
   - **Reduced motion:** with `prefers-reduced-motion` emulated via `javascript_tool`/CSS check, the computed `animation-name` on `body` is `none`.
   - **Contrast:** use `javascript_tool` to compute contrast for body text, `--muted-foreground` and primary buttons on 70% white composited over the brightest gradient stop (yellow). Every value must be at least 4.5:1. Adjust the tokens in Phase 1 if any fail.
   - **Console:** no errors, using `read_console_messages`.
3. Use `resize_window` with the mobile preset to confirm the header 🦄 doesn't break the nav wrap.

## Phase 4: Contrast off-panel and cross-device support

Found while verifying Phases 1–3:

- **Text straight on the rainbow fails AA.** Muted text measures about 1.8:1 and body text about 4.2:1 over the darker gradient stops. Fix it by raising every gradient stop to OKLCH lightness ≥ 0.8, keeping chroma as high as the sRGB gamut allows. Darken `--muted-foreground` and `--brand-label` until ink, muted and label text all measure ≥ 4.5:1 against every stop, and still pass on the frosted panels.
- **iOS Safari ignores `background-attachment: fixed`,** so a 400%-sized gradient on a long `body` would stretch and scroll. Paint the gradient on a `position: fixed; inset: 0; z-index: -1` `body::before` layer instead, and animate that layer.
- **Older Safari needs `-webkit-backdrop-filter`** alongside `backdrop-filter`.
- **Check:** desktop Chromium, Android, iPhone Safari (the iOS Simulator), and iPad Safari.

## Files

- `apps/web/src/styles.css`: most of the work
- `apps/web/src/components/theme-toggle.tsx`
- `apps/web/src/components/header.tsx`
