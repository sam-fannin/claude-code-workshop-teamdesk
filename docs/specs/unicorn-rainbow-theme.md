# Unicorn rainbow theme

Status: ready to build. Budget: about 20 minutes. CSS and two small component edits, with no server or database changes.

## Why

Light mode today is warm off-white paper with a single terracotta accent, and it reads as black and white. The rainbow theme replaces it.

## What the user sees

- **Rainbow replaces Light.** The theme button cycles **Rainbow 🦄 → Dark → Auto**. Dark is unchanged.
- **Auto** shows rainbow on a light-mode device and dark on a dark-mode device.
- People who saved "Light" before now get rainbow automatically.
- **Full rainbow page.** The whole page background is a saturated diagonal rainbow gradient (red → orange → yellow → green → blue → violet).
- **Slow shimmer.** The gradient drifts slowly across the page, about one loop every 15–20 s. It stops for users with `prefers-reduced-motion: reduce`.
- **Frosted glass panels.** Cards, popovers, dialogs, inputs and the sticky header are semi-transparent white (around 70% opacity) with a backdrop blur. The rainbow shows through softly, and dark text stays readable on top.
- **Unicorn.** A 🦄 emoji sits next to the "Teamdesk" name in the header. The theme button shows 🦄 instead of the sun icon while in Rainbow mode.

## How to build it

Keep the internal mode value and the `<html>` class as `light`; only the label and the look change. This keeps these working without edits:

- the theme-init script in `apps/web/src/routes/__root.tsx`
- the `theme` value already saved in localStorage
- `useResolvedTheme`
- the toaster

1. **`apps/web/src/styles.css`**, the `:root, .light` block:
   - Retune the tokens for rainbow:
     - `--background`: a pastel fallback colour.
     - `--card`, `--popover`, `--input`, `--secondary`, `--muted`, `--accent`: translucent white, e.g. `oklch(1 0 0 / 0.7)`.
     - `--primary` and `--ring`: a vivid magenta/violet. Keep `--primary-foreground` at a contrast of at least 4.5:1.
     - `--border`: translucent white.
     - Chart colours: rainbow hues.
   - Leave the `.dark` block alone.
   - Add a `.light body` rule with a `linear-gradient(135deg, …)` rainbow, `background-size: 400% 400%` and `background-attachment: fixed`.
   - Add a `@keyframes rainbow-shimmer` animation on `background-position`.
   - Wrap the animation in `@media (prefers-reduced-motion: no-preference)`.
   - Add `.light [data-slot="card"]` (and the popover and dialog slots, if they have them) with `backdrop-filter: blur(12px)`.
2. **`apps/web/src/components/theme-toggle.tsx`**:
   - Change `MODE_LABEL.light` to `"Rainbow"`.
   - Render 🦄 (an `aria-hidden` span) instead of the `Sun` icon for `light`.
   - Update the `aria-label`/`title` wording so it says "rainbow" rather than "light".
3. **`apps/web/src/components/header.tsx`**: add an `aria-hidden` 🦄 next to "Teamdesk".

## Done when

- A fresh browser on a light-mode device shows the shimmering rainbow page with frosted cards and 🦄 in the header.
- The button cycles Rainbow 🦄 → Dark → Auto, and a reload keeps the chosen mode with no flash of the old light theme.
- Dark mode looks exactly as before.
- With reduce-motion turned on, the rainbow is still.
- Body text and primary buttons on frosted panels pass WCAG AA (4.5:1) against the lightest part of the gradient showing through.
- `bun run check` passes.

## Out of scope

- Unicorn artwork or image files.
- Sparkles and effects.
- Unicorns in empty states or toasts.
- A separate fourth theme mode.
- Per-user theme stored on the server.
- Changes to dark mode.
