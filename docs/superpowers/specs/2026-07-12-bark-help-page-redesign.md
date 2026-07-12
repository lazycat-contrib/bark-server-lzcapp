# Bark Help Page Redesign

## Goal

Upgrade the packaged Bark Server help page so every push example uses the current LazyCat application origin and the interface feels polished, responsive, and accessible without external dependencies.

## Scope

- Replace the existing `dist/tours.html` with one self-contained HTML document.
- Keep the existing `/help` static upstream and package layout unchanged.
- Remove the manifest-level `usage` field; help remains available through the `/help` route and launcher entry.
- Do not change Bark Server APIs, authentication, deployment parameters, or container startup behavior.

## URL behavior

- Read the active instance base URL from `window.location.origin`.
- Build basic and advanced push examples from that origin.
- Preserve placeholders such as `YourKey`, title, and content while encoding example path segments safely.
- Display the active origin near the page title so users can verify which Bark Server instance they are configuring.
- Provide copy buttons whose copied text is exactly the currently displayed instance-specific example.
- Select Simplified Chinese for `zh*` browser locales and English for all other locales; localize headings, instructions, parameter descriptions, example path text, and copy feedback.

## Visual direction

- Use a light, editorial tool-page layout with Bark green as the primary accent.
- Present a compact hero, a three-step setup flow, copyable request examples, a parameter reference grid, and security guidance.
- Use restrained borders, soft shadows, rounded surfaces, and strong typographic hierarchy.
- Remain responsive from narrow mobile screens to desktop widths.
- Use inline SVG icons where useful; load no remote fonts, scripts, images, or styles.

## Motion and interaction

- Animate only initial section entry, hover elevation, and copy feedback.
- Keep UI transitions below 250ms with strong ease-out curves.
- Add subtle `scale(0.97)` press feedback to buttons.
- Avoid `transition: all`, `scale(0)`, continuous decorative motion, and layout-property animation.
- Disable positional movement under `prefers-reduced-motion`, retaining only short opacity/color feedback.
- Copy feedback changes the button label temporarily and remains usable when the Clipboard API is unavailable by falling back to a temporary text selection.

## Accessibility

- Use semantic headings, ordered steps, lists, code elements, and buttons.
- Maintain visible keyboard focus styles and sufficient contrast.
- Use `aria-live` for copy status without creating repeated motion.
- Gate hover-only effects behind `(hover: hover) and (pointer: fine)`.

## Verification

- Parse or inspect the generated HTML and confirm no `api.day.app` push example remains.
- Confirm example URLs are derived from `window.location.origin`.
- Confirm the page contains no external runtime assets.
- Build the LPK and confirm the redesigned `tours.html` is present in `content.tar`.
- Run HTML/JavaScript syntax checks available in the workspace.
- Commit and push the implementation, then manually dispatch the existing LazyCat publishing workflow after the user-deleted tag is recreated by the workflow.
