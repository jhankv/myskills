# preview

**When:** After `/papercraft init` or `/papercraft design.md`, to visually verify the theme in Paper.

**Invocation:**
- `/papercraft preview` — draws the priority batch (max 7)
- `/papercraft preview button,checkbox,dialog` — draws only the specified components

**Flow:**

1. **Read installed components:**
   - Check `components/ui/` directory to see which shadcn components are installed
   - Run `bunx --bun shadcn@latest info --json` to get the full project context

2. **Determine what to draw:**

   **If specific components were passed as arguments** (e.g., `preview button,checkbox`):
   - Draw only those components. No limit.

   **If no arguments — automatic batch:**
   - Count installed components
   - If 7 or fewer → draw all of them
   - If more than 7 → draw only the **priority batch** (max 7), selected from this priority order:
     1. Button, Input, Card, Switch, Badge, Select, Table
     2. Checkbox, Dialog, Tabs, Avatar, Alert
     3. Sidebar, Navigation, Textarea, Slider
     4. Everything else
   - Pick the top 7 that are actually installed

3. **Create one frame per component using the container structure:**

   Every component drawn in Paper follows this nested frame structure:

   ```
   [Frame contenedor: "ComponentName - pc"]   <- create_artboard() with - pc suffix
     |-- [Frame: Visual]                      <- the rendered component
     +-- [Frame: Info]                        <- technical reference (separate frame)
   ```

   **Visual frame (top):** Render the component with the project's brand tokens. Show the default variant plus 2-3 visually distinct variants side by side. Only render in default state (no hover, no focus, no active).

   **Token resolution:** Paper has no CSS variable context. Before calling `write_html()`, read `globals.css` and resolve all `var()` references to actual OKLCH values. If `--muted` has brand tint in code, it must have the same tint in Paper — never approximate with plain gray.

   **Font sizes:** Only use Tailwind text scale values in `write_html()`: 12px, 14px, 16px, 18px, 24px, 30px. Never hardcode in-between sizes.

   **Info frame (bottom — separate from visual):**

   ```
   STATUS: Paper + Code

   COMPOSITION
   Card + CardHeader + CardContent (existing)
   TrackProgressItem x 3 (existing)

   COLORS (TAILWIND DIRECT)
   blue-500  purple-500  green-500

   TYPE
   Complex component -> components/track-progress-card.tsx

   Sizes: sm, default, lg, icon
   Also in code: secondary, ghost, link variants
   States: hover, active, disabled, loading
   Props: asChild

   PROMPT
   Write here what you want to change, then run /papercraft component

   CHANGES
   (empty)
   ```

   **STATUS** — sync state indicator:
   - `Paper only` — component exists only in Paper (ideation phase)
   - `Code only` — component exists in code, drawn in Paper as preview
   - `Paper + Code` — synchronized in both

   **TYPE** — changes based on sync state:
   - If `Paper only`: `Suggested:` path
   - If `Paper + Code` or `Code only`: actual path with `->`

   **Never mix info text inside the visual frame.** They are siblings inside the container.

4. **After drawing, report to the user:**

   If the full list was drawn (7 or fewer installed):
   > "Drew all components in Paper: Button, Input, Card, Switch, Badge, Select, Table."

   If only the priority batch was drawn:
   > "Drew 7 priority components in Paper. You have 13 more installed. To draw specific ones, run `/papercraft preview checkbox,dialog,tabs`."

The preview is for the designer to verify the visual style. If something looks wrong, they can adjust via `/papercraft design.md`.
