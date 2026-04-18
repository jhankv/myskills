# screen

**When:** The designer wants to create a full page or view — a dashboard, settings page, login screen, etc. Unlike `component` (which creates reusable pieces in `components/`), `screen` produces complete pages that compose existing components.

**Two input modes:**

**Mode A — Prompt:**
```
/papercraft screen sales dashboard with KPI cards, revenue chart, and recent orders table
```

**Mode B — Reference in Paper:**
The designer selects a frame in Paper that contains a reference image and/or a `Prompt:` text node, then runs `/papercraft screen`.

1. `get_selection()` → get the frame
2. `get_children()` → classify contents
3. If images → `get_screenshot()` to analyze the visual reference
4. If `Prompt:` → read the designer's instructions
5. Combine reference + prompt to understand what to build, but **always apply the project's own style from DESIGN.md** — the reference is for layout and functionality, not for colors or typography

**Empty invocation — guide the designer:**

If `/papercraft screen` is run with no prompt AND nothing selected in Paper:

> "To create a screen, I need one of these:"
>
> **Option A** — Describe it: `/papercraft screen sales dashboard with KPI cards and orders table`
>
> **Option B** — Select a reference frame in Paper with an image and/or a `Prompt:` text, then run `/papercraft screen` again

**Flow:**

1. **Determine output destination FIRST:**

   If the prompt already specifies the destination (e.g., "create a dashboard in Paper", "build a settings page in code"), skip this question. Otherwise ask:
   > "Where do you want the output?"
   > `[ Paper ]  [ Code ]  [ Both ]`

   - **Paper** — fast, cheap in tokens. Draws the screen in Paper using `write_html()`. Good for ideation.
   - **Code** — generates the page file and any needed components directly in the project.
   - **Both** — generates in code and draws in Paper simultaneously.

2. **Read only what the destination requires:**

   | Destination | Read |
   |-------------|------|
   | **Paper** | `DESIGN.md` + `density-reference.md` + `globals.css` (globals.css is source of truth for colors; density-reference for sizing) |
   | **Code** | `DESIGN.md` + `density-reference.md` + `globals.css` + `components/ui/` + `components/` + layout files |
   | **Both** | All of the above |

   For Code/Both, also read existing layouts (e.g., `layout.tsx`, sidebar, header). The screen must compose INSIDE the existing layout, not create a new one.

3. **Plan the layout — if complex (~5+ sections), show it first:**

   Before executing, propose the structure:

   ```
   +-------------------------------------+
   | Header (existing layout)            |
   +--------+--------+--------+----------+
   | KPI 1  | KPI 2  | KPI 3  | KPI 4   |  <- Card (existing)
   +--------+--------+--------+----------+
   | Revenue Chart                       |  <- Chart (existing)
   +-----------------+-------------------+
   | Recent Orders   | Top Products      |  <- Table (existing) + NEW: TopProducts
   +-----------------+-------------------+

   Components: 4 existing (Card, Chart, Table, Badge), 1 new (TopProducts)
   Proceed?
   ```

   For simple screens (< 5 sections), skip the preview and execute directly.

4. **Build using the component hierarchy:**

   When the screen needs a UI element, follow this order strictly:

   ```
   1. Use existing component from components/ui/ or components/
   2. Compose with existing components (e.g., Card + Table + Badge)
   3. Search shadcn registry (bunx --bun shadcn@latest search <name>)
   4. Create a custom component (last resort) -> place in components/
   ```

   **Never hardcode JSX inline in the page file.** Every section must be a component import.

5. **Generate the output using the container structure:**

   Screens in Paper follow the same nested frame pattern as components:

   ```
   [Frame contenedor: "Sales Dashboard - pc"]
     |-- [Frame: Visual]    <- the full screen rendered
     +-- [Frame: Info]      <- composition, status, prompt, changes
   ```

   **If Paper (or Both):**

   **Visual frame:**
   - `write_html()` with the full screen rendered using DESIGN.md tokens
   - **Resolve all tokens to real OKLCH values** — read `globals.css` and substitute `var()` references. Paper has no CSS variable context, so colors like `--muted: oklch(0.97 0 0)` must be used directly, not as `var(--muted)`. If a token references a brand variable (e.g., `--accent: var(--brand-50)`), resolve it to the actual OKLCH value
   - **Font sizes must use Tailwind's text scale only** — use 12px, 14px, 16px, 18px, 24px, 30px (matching text-xs through text-3xl). Never hardcode in-between values like 13px or 11px
   - Use realistic mock data — real names, real numbers, real dates. Never "Lorem ipsum"
   - Generate responsive layout (mobile-first)

   **Info frame:**
   ```
   STATUS: Paper only

   COMPOSITION
   Card x 4 (existing) — KPI section
   Chart (existing) — Revenue chart
   Table + Badge (existing) — Recent orders
   TopProducts (NEW — components/top-products.tsx)

   ROUTE
   Suggested: app/dashboard/page.tsx

   SECTIONS
   3 rows: KPI grid (4 cols), Chart (full), Orders + Products (2 cols)
   Layout: responsive grid, mobile stacks vertically

   PROMPT
   Write here what you want to change, then run /papercraft screen

   CHANGES
   (empty)
   ```

   **STATUS** values: `Paper only`, `Code only`, `Paper + Code`

   **ROUTE** — `Suggested:` when Paper only, `->` when in code

   **If Code (or Both):**
   - Create the page file in the appropriate route directory (read framework from DESIGN.md section 0)
   - Import all components — never inline
   - Include TypeScript types for data structures
   - Include a mock data array with realistic sample data
   - Generate responsive layout (mobile-first)

6. **Token & variant verification (Code or Both):**

   Before writing any code, resolve the visual intent from Paper to actual component output. This catches mismatches caused by token overrides (e.g., `--primary` set to black instead of brand).

   **Step 6a — Resolve variants to real colors:**

   For every component you're about to use, trace the variant's CSS to its final color:
   ```
   Badge variant="default" → bg-primary → --primary → oklch(0.216 0.012 75) = BLACK
   Paper shows: green tinted badge → MISMATCH
   ```

   If the variant resolves to a different color than what Paper shows, do NOT silently use it. Flag it.

   **Step 6b — Primary override check:**

   Read DESIGN.md section 2 to check if `--primary` is overridden (i.e., NOT `brand-500`). If it is:
   - Any component variant that uses `bg-primary`, `text-primary`, or `border-primary` will render the override color (e.g., black), NOT the brand color
   - Components that commonly hit this: Badge `default`, Progress indicator, Button `default`, Toggle pressed state
   - When Paper shows brand-colored elements, use brand utility classes (`bg-brand-500`, `text-brand-900`) instead of semantic variants that resolve to `--primary`

   **Step 6c — Missing variant detection:**

   When Paper shows a visual treatment that doesn't map to any existing variant in the component (e.g., a green-tinted badge, a yellow-tinted "pending" badge), ask the designer:

   > "The Paper design shows visual treatments that don't map to existing component variants:"
   >
   > - `Completed` badge: green tinted bg — no matching Badge variant
   > - `Pending` badge: yellow/amber tinted bg — no matching Badge variant
   >
   > `[ Create new variants ]  [ Use utility classes ]`

   - **Create new variants** → add variants to the component's `cva()` definition (e.g., `success`, `warning`). This is the right choice when the treatment will be reused across screens.
   - **Use utility classes** → apply `className` overrides at the usage site (e.g., `className="bg-brand-50 text-brand-900 border-0"`). This is the right choice for one-off visual treatments.

   **Step 6d — Tailwind class verification:**

   Verify that all Tailwind classes used actually exist. Common traps:
   - `flex-2` → doesn't exist, use `flex-[2]`
   - `text-[13px]` → violates Tailwind text scale rule, use `text-xs` or `text-sm`

   **Step 6e — Inline non-existing UI elements:**

   When Paper shows a UI element that doesn't exist as a component in `components/ui/` or `components/` (e.g., a search input with a keyboard shortcut badge, a custom dropdown trigger), do NOT create a new component for it. Instead, write it inline in the screen page using semantic shadcn tokens (`bg-muted`, `text-muted-foreground`, `border`, `rounded-lg`, etc.) and standard HTML/Tailwind.

   The developer will decide later whether to extract it into a reusable component. Screens are the place to prototype — not to create premature abstractions.

   Example: Paper shows a search bar with `⌘K` badge → write it inline in the page:
   ```tsx
   <div className="relative">
     <SearchIcon className="absolute left-3 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" />
     <Input placeholder="Search" className="pl-9 pr-16" />
     <kbd className="pointer-events-none absolute right-3 top-1/2 -translate-y-1/2 rounded-md border bg-muted px-1.5 text-xs text-muted-foreground">⌘K</kbd>
   </div>
   ```

7. **Report what was created:**
   > "Created 'Sales Dashboard':"
   > - Output: Paper (or Code, or Both)
   > - Components used: Card, Chart, Table, Badge (existing)
   > - Components created: TopProducts (new)
   > - Layout: 3 rows, responsive grid

**Returning from Paper to Code / Iterating:**

If the designer selects a frame previously generated by `/papercraft screen` and runs the command again:

**If the PROMPT section has text** → iteration instructions:
1. Read the Prompt
2. Apply changes based on destination
3. Update the Info frame: append to CHANGES, clear PROMPT, update STATUS
4. Update the visual frame if Paper output

**If the PROMPT section is empty** → ask:
> "This screen already exists in Paper. What do you want to do?"
> `[ Send to code ]  [ Iterate in Paper ]  [ Update both ]`

## shadcn Conventions (delegate to shadcn skill)

When generating screen code, follow shadcn skill rules:

- **Component selection** → run `bunx --bun shadcn@latest search <name>` before creating custom components
- **Component docs** → run `bunx --bun shadcn@latest docs <component>` for correct API and usage
- **Composition** → read `rules/composition.md` (Items inside Groups, full Card composition, Tabs structure)
- **Forms in screens** → read `rules/forms.md` (FieldGroup + Field, never raw divs)
- **Styling** → read `rules/styling.md` (semantic colors, `gap-*` not `space-y-*`, `cn()`)
- **Icons** → read `rules/icons.md` (`data-icon`, no sizing classes)

Key rules:
- Use semantic colors (`bg-primary`, `text-muted-foreground`) — never raw values
- Every section must be a component import — never hardcode JSX inline in the page
- Use existing shadcn components before creating custom ones

---

**What NOT to do:**
- Don't modify DESIGN.md — `screen` consumes the design system, it doesn't change it
- Don't create new base components (Button, Input, etc.) — use what exists
- Don't create a new layout if one already exists in the project
- Don't ask for information that's already in DESIGN.md
- Don't generate screens without DESIGN.md — tell the designer: "Run `/papercraft init` first."
