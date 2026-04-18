# design.md

**When:** After `/papercraft init`. The designer wants to iterate on the existing style — change typography, density, radius, colors, or any other token. Also used to rebuild the "DESIGN.md - pc" frame layout when sections are missing or outdated.

**Frame resolution (no selection required):**

1. Call `get_selection()` first — if the designer already selected a "DESIGN.md - pc" frame, use it
2. If nothing is selected → search all frames in Paper for ones named "DESIGN.md - pc" using `get_children()` on the root
3. **One found** → use it automatically
4. **Multiple found** → ask the designer which one
5. **None found** → tell the designer:
   > "No DESIGN.md - pc frame found. Run `/papercraft init` first to create one."

**Flow:**

1. **Read the "DESIGN.md - pc" frame** using the resolution logic above
2. **Read current theme state from code:**
   - Read `DESIGN.md` file → full design system memory
   - Read `globals.css` → current brand scale, radius, chart colors
   - Read `@theme inline` → registered Tailwind tokens
   - Read `components/ui/` → installed components and their current spacing classes
3. **Read the designer's changes:**
   - Look for a `Prompt:` text node → specific instructions (e.g., "change font to Inter", "make it spacious", "use green-500 as primary")
   - Look for drawn elements or modified visuals → `get_computed_styles()` if present
   - Look for visual changes in the frame itself (e.g., designer may have modified a color swatch)
4. **Apply only what changed:**
   - If the designer changed **typography** → update `--font-sans` or `--font-heading` in globals, install the new font via `next/font`
   - If the designer changed **density** → re-run the density codemod: read density-reference.md, look up the new tier's classes, and rewrite spacing/sizing Tailwind classes in all `components/ui/*.tsx` files. Report: "Updated N components from [old] → [new] density."
   - If the designer changed **radius** → update `--radius` in globals
   - If the designer changed **primary color** → regenerate the brand scale 50→950 with oklch-skill, update globals, recalculate chart colors if they were derived from brand
   - If the designer changed **chart colors** → update `--chart-1` through `--chart-5` in globals
   - If the designer changed **weight** (flat/elevated) → add or remove shadow variables
   - Don't regenerate things that didn't change
5. **Update both DESIGN.md (file) and "DESIGN.md - pc" (Paper frame):**
   - Update the `DESIGN.md` file in code with the new values (follow template from `references/design-md-spec.md` Section 2)
   - Update the same frame in Paper (don't create a new one) following the layout from `references/design-md-spec.md` Section 3
   - Use `write_html()` to refresh the frame with updated values
   - Both must stay in sync — they are the same document in two formats

6. **Cascade to other frames — ask the designer:**

   After updating DESIGN.md, scan Paper for all frames with `- pc` suffix using `get_children()` on root. If other papercraft frames exist, report and ask:

   > "Updated brand color from rose-500 to lime-500. Found 10 other papercraft frames in Paper:"
   >
   > Components: Button - pc, Card - pc, Input - pc
   > Screens: Sales Dashboard - pc, User Settings - pc
   >
   > `[ Update all now ]  [ Select specific ]  [ Skip — run /papercraft design.md sync later ]`

   If no other `- pc` frames exist, skip this step.

7. **Report what changed:**
   > "Updated: brand color → lime-500, density unchanged. DESIGN.md refreshed in code and Paper."

**`/papercraft design.md rebuild` — reconstruct the Paper frame:**

Re-renders the entire "DESIGN.md - pc" frame from scratch using the current DESIGN.md file data. Use when sections are missing (e.g., semantic tokens not showing) or the layout is outdated.

1. Read `DESIGN.md` file for all current values
2. Read `globals.css` for exact OKLCH values of every token in `:root` and `.dark`
3. Read `references/design-md-spec.md` → Section 3 (Paper Frame Layout) — this defines the exact structure, all 7 sections, and the 26-token semantic tokens grid
4. Find the existing "DESIGN.md - pc" frame in Paper
5. Replace the frame content entirely following the spec. Use `write_html()` to render.
6. Report: "Rebuilt DESIGN.md - pc frame with all 7 sections (26 semantic tokens)."

This does NOT change any values — it only reconstructs the visual layout in Paper from the existing DESIGN.md and globals.css data.

---

**`/papercraft design.md sync` — batch propagation:**

1. Read current `DESIGN.md` file
2. `get_children()` on root → find all frames with `- pc` suffix
3. For each frame:
   - Read its Info frame to know the type (component or screen)
   - Re-render the visual frame with current DESIGN.md tokens (colors, radius, density, typography)
   - Update STATUS in Info frame if needed
   - Append to CHANGES: `✓ Synced with DESIGN.md (brand color: lime-500)`
4. Report:
   > "Synced 10 frames with current DESIGN.md: 5 components, 5 screens updated."

**What NOT to do:**
- Don't create a new frame — update the existing "DESIGN.md - pc" frame
- Don't regenerate the brand scale if the color didn't change
- Don't re-apply component spacing if density didn't change
- Don't update one without the other — file and frame must stay in sync
