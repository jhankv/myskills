# component

**When:** The designer wants to create a new component or iterate on an existing one. Can work with a selected frame, a frame name, or just a text prompt.

**Flow:**

1. **Determine output destination FIRST:**

   If the prompt already specifies the destination (e.g., "create a badge in Paper", "build a form in code"), skip this question. Otherwise ask:
   > "Where do you want the output?"
   > `[ Paper ]  [ Code ]  [ Both ]`

   - **Paper** — draws the component in Paper only (fast, for ideation)
   - **Code** — generates the component in code only
   - **Both** — generates in code and draws in Paper

2. **Read only what the destination requires:**

   | Destination | Read |
   |-------------|------|
   | **Paper** | `DESIGN.md` + `density-reference.md` + `globals.css` (globals.css is source of truth for colors) |
   | **Code** | `DESIGN.md` + `density-reference.md` + `globals.css` + `components/ui/` |
   | **Both** | All of the above |

   **Why density-reference.md for Paper too:** Drawing realistic components in Paper requires exact dimensions — input heights, card padding, button sizes, gaps between fields. DESIGN.md section 5 only has the highlights. `density-reference.md` has every value per component per tier. Convert Tailwind classes to px for `write_html()` (e.g., `h-9` → `36px`, `p-6` → `24px`, `gap-4` → `16px`).

   **Token resolution for Paper:** Paper has no CSS variable context. Before calling `write_html()`, read `globals.css` and resolve all `var()` references to their actual OKLCH values. For example, if `--accent: var(--brand-50)` and `--brand-50: oklch(0.990 0.015 114)`, use `oklch(0.990 0.015 114)` directly — NOT a plain gray approximation like `oklch(0.99 0 0)`.

   **Font sizes in Paper:** Only use Tailwind's standard text scale values: 12px, 14px, 16px, 18px, 24px, 30px. Never hardcode in-between sizes like 13px, 11px, or 15px.

   **Radius in Paper output:** Read DESIGN.md section 6 for `--radius` value AND `pill-buttons` flag. When rendering with `write_html()`:
   - Apply `--radius` scale to cards, inputs, selects, dialogs normally
   - If `pill-buttons: yes` → render Button, Badge, Toggle with `border-radius: 9999px` (fully rounded), regardless of `--radius`

3. **Resolve the input (what to build):**

   Three input modes, checked in order:
   - **Frame selected in Paper** → `get_selection()` to read it
   - **Frame name mentioned** (e.g., "iterate Button - pc") → search with `get_children()` on root
   - **Text only** (e.g., "create a registration form") → no frame, build from description

   If nothing is selected and no name or description is given → ask: "Select a frame in Paper, name an existing component, or describe what you want to create."

4. **Detect mode: iteration vs creation:**

   Check if the frame's name matches a component that already exists in `components/ui/` or `components/`. A frame created by `/papercraft preview` will have the component name (e.g., "Button", "Card").

   - **Frame name matches existing component** → enter **iteration mode**
   - **Frame name is new / doesn't match** → enter **creation mode**

5. **Read frame contents** (both modes):

   **Images present (visual reference):**
   - `get_screenshot()` of the full frame → analyze the visual reference
   - Read `Prompt:` if present → get specific instructions
   - **Analyze the layout structure** of the reference — identify every section, element, and their arrangement
   - **Propose an ASCII layout** before executing, then wait for confirmation
   - The reference dictates the **what** (structure, layout, sections). DESIGN.md dictates the **how** (colors, radius, density, typography)
   - **Respect the full layout** — don't simplify, don't remove sections, don't reinterpret the structure

   **Drawn elements, no images:**
   - `get_computed_styles()` → extract colors, borders, sizing from drawn elements
   - Read `Prompt:` if present → get specific instructions

   **Only text (Prompt only):**
   - Read the `Prompt:` text
   - If something critical is missing → ask the designer only for what's genuinely needed

---

## Iteration mode (component exists in code)

The designer is adding variants, adjusting styles, or refining an existing component.

1. **Read the current component source** in `components/ui/<component>.tsx` or `components/<component>.tsx`
2. **Understand what already exists** — variants, sizes, props, styles defined in `cva()` or equivalent
3. **Read the Prompt from the Info frame** — if the designer wrote instructions in the PROMPT section
4. **Apply the changes from the Prompt and/or drawn elements:**
   - Adding a new variant → add it to the `cva()` variants object
   - Modifying an existing variant → edit only that variant's styles
   - Adding a new size → add it to the sizes object
   - Changing colors/borders → update the relevant class strings
5. **Never rewrite the entire component.** Touch only what the designer asked for.
6. **Token & variant verification (Code or Both):**
   Before applying changes, run the same verification as creation mode step 6:
   - **6a** — Resolve any new or modified variant to its actual color. Does it match Paper?
   - **6b** — If `--primary` is overridden, check new variants don't accidentally use `bg-primary` where brand color is intended
   - **6c** — If Paper shows a treatment with no existing variant, ask: `[ Create new variant ]` vs `[ Use utility classes ]`
   - **6d** — Verify Tailwind classes exist
7. **Apply output based on the destination chosen in step 1:**
   - **Code** → apply changes to the component file
   - **Paper** → update the visual frame + update the Info frame
   - **Both** → apply to code and update both frames in Paper
8. **Update the Info frame after every iteration:**
   - Update **STATUS** to reflect the current sync state
   - Update **TYPE** — if component was just sent to code, change from `Suggested:` to `→` with actual path
   - Append to **CHANGES** log what was done
   - Clear the **PROMPT** text after reading it (the instruction was consumed)

---

## Creation mode (new component)

The designer wants a component that doesn't exist yet in code.

1. **Check if shadcn has it:** run `bunx --bun shadcn@latest search <component-name>`. If it exists, install it and apply the theme tokens rather than building from scratch.
2. **If shadcn doesn't have it:** read DESIGN.md section 0 to know which headless UI library the project uses (Radix, Base UI, React Aria). Use that library as headless base if the component needs accessibility/interaction. For purely visual components, build with standard HTML + Tailwind.
3. **Classify component type — base vs complex:**

   **Base component** → a primitive, reusable UI element (Button, Input, Badge, Toggle, etc.)
   - Lives in `components/ui/`
   - Has variants via `cva()`
   - Follows shadcn conventions

   **Complex component** → composes multiple base components (StatsCard, OrdersTable, PricingSection, etc.)
   - Lives in `components/` (NOT `components/ui/`)
   - Imports from `components/ui/` — never duplicates base component logic

   How to classify:
   - Does it wrap/compose 2+ existing components? → **complex**
   - Is it a single interactive element with variants? → **base**
   - Could it be used in multiple unrelated screens? → **base**
   - Is it specific to a feature or section? → **complex**

4. **Generate the component:**
   - Follow shadcn skill conventions (composition, variants, styling rules)
   - Use semantic color tokens (`bg-primary`, `text-muted-foreground`) — never raw brand values
   - For custom colors beyond semantic tokens, use brand scale utilities (`bg-brand-200`, `text-brand-700`)
5. **Default specs** (when the Prompt doesn't specify):

   | Component | Sizes | Variants | States |
   |-----------|-------|----------|--------|
   | Button | sm, md, lg | primary, secondary, outline, ghost, link, destructive | default, hover, active, disabled |
   | Input | md | default | default, focus, error, disabled |
   | Badge | md | default, secondary, destructive, outline | — |
   | Card | — | — | — |

6. **Token & variant verification (Code or Both):**

   Before writing code, verify that component variants resolve to the same colors shown in Paper. This catches mismatches caused by token overrides.

   **6a — Resolve variants to real colors:**

   For each component variant you plan to use, trace the CSS to its final value:
   ```
   Badge variant="default" → bg-primary → --primary → actual OKLCH value
   Compare with Paper: does it match?
   ```

   **6b — Primary override check:**

   Read DESIGN.md section 2. If `--primary` is NOT `brand-500` (e.g., it's stone-900 for black buttons):
   - Variants using `bg-primary` will render the override color, NOT brand
   - When the component should show brand color, use brand utilities (`bg-brand-500`) instead of `bg-primary` variants
   - Affected components: Badge `default`, Progress indicator, Toggle pressed, any custom variant using `primary` tokens

   **6c — Missing variant detection:**

   When Paper shows a visual treatment that doesn't map to any existing variant, ask the designer:

   > "The Paper design shows visual treatments that don't exist as variants:"
   >
   > - Green tinted badge for "success" state — no matching variant
   > - Yellow tinted badge for "warning" state — no matching variant
   >
   > `[ Create new variants ]  [ Use utility classes ]`

   - **Create new variants** → add to the component's `cva()` (e.g., `success: "bg-brand-50 text-brand-900"`, `warning: "bg-amber-50 text-amber-900"`). Best when the treatment will be reused.
   - **Use utility classes** → apply `className` overrides at the usage site. Best for one-off treatments.

   **6d — Tailwind class verification:**

   Verify all Tailwind classes exist. Common traps:
   - `flex-2` → use `flex-[2]`
   - `text-[13px]` → violates text scale, use `text-xs` or `text-sm`

7. **Apply output based on the destination chosen in step 1:**
   - **Code** → create the component file in `components/ui/` (base) or `components/` (complex). STATUS = `Code only`
   - **Paper** → create using the container structure. STATUS = `Paper only`. TYPE shows `Suggested:` path
   - **Both** → create in code and draw in Paper. STATUS = `Paper + Code`. TYPE shows `→` with actual path

## shadcn Conventions (delegate to shadcn skill)

Before generating or modifying component code, read these rules from the shadcn skill:

- **Creating variants** → read `rules/styling.md` (className for layout not styling, semantic colors, `cn()`, no manual `dark:` overrides)
- **Form components** → read `rules/forms.md` (FieldGroup + Field, validation with `data-invalid` + `aria-invalid`)
- **Component composition** → read `rules/composition.md` (Items inside Groups, full Card composition, Tabs structure, Avatar with fallback)
- **Icons** → read `rules/icons.md` (`data-icon="inline-start"`, no sizing classes on icons)
- **Base vs Radix APIs** → read `rules/base-vs-radix.md` (check DESIGN.md section 0 for which base the project uses)
- **Component selection** → run `bunx --bun shadcn@latest search <name>` before creating custom components
- **Component docs** → run `bunx --bun shadcn@latest docs <component>` to get correct API and usage patterns

Key rules to always follow:
- Use semantic colors (`bg-primary`, `text-muted-foreground`) — never raw values like `bg-blue-500`
- Use `gap-*` not `space-y-*` for spacing
- Use `size-*` when width and height are equal
- Never override component colors via `className` — use variants or CSS variables

---

**Both modes — update DESIGN.md:**

After creating or modifying a component, update section 7 ("Generated Components") in `DESIGN.md`:
- **New component** → add a new row
- **Modified component** → update the existing row with new variants/states and current date
- If `DESIGN.md` doesn't exist yet, warn: "Run `/papercraft init` first."
