# init

**When:** Once per project, to initialize the design system.

**Pre-requisites (check before anything else):**

1. **shadcn installed?** — Look for `components.json` in the project root.
   - If missing → stop and tell the designer: "shadcn is required. Run `bunx --bun shadcn@latest init` first."
2. **Paper MCP active?** — Attempt a call to Paper MCP (e.g., `get_selection()`).
   - If connection fails → stop and tell the designer: "Paper MCP is not connected. Make sure Paper Desktop is running and the MCP is configured: `claude mcp add paper --transport http http://127.0.0.1:29979/mcp --scope user`"

If both are OK, proceed silently — don't mention them.

**Environment detection (run once, save to DESIGN.md):**

After pre-requisites pass, detect the project environment:

1. **Framework** — detect from config files:
   - `next.config.*` → Next.js (check for `app/` vs `pages/` to determine App Router or Pages Router)
   - `vite.config.*` → Vite
   - `astro.config.*` → Astro
   - `nuxt.config.*` → Nuxt
   - If none found → ask the designer: "What framework are you using?"

2. **Icon library** — read from `components.json` → `iconLibrary` field (e.g., `"lucide"`, `"icons"`)

3. **Headless UI library** — read from `package.json` dependencies:
   - `@radix-ui/*` packages found → Radix UI
   - `@base-ui-components/*` packages found → Base UI
   - `react-aria` or `@react-aria/*` found → React Aria
   - None found → no headless library (Tailwind + HTML only)

These are saved in DESIGN.md section 0 and used by all other commands.

**Two initialization paths:**

Call `get_selection()` to check if the designer has a frame selected in Paper.

**If a frame IS selected → classify its contents:**

Call `get_children()` to inspect what's inside the frame:
- **Images found** → **Path A** (mood board — interpret visually)
- **Drawn elements found** (frames, shapes, text nodes styled as UI) → **Path C** (style from components — read exact styles)
- **Both images and drawn elements** → **Path A** (treat as mood board, images take priority)
- **Only `Prompt:` text** → ask the designer to add visual references or choose Path B

**If nothing is selected → ask the designer:**

> "How do you want to initialize the design system?"
>
> **A** — From a mood board: Create a frame in Paper, add your reference images and a `Prompt:` text inside, then run `/papercraft init` again
>
> **B** — From existing project: I'll read your current theme (globals.css, installed components, tokens) and generate a DESIGN.md from what's already configured

**Path B — Initialize from existing code:**

1. Read `globals.css` → extract existing color variables, radius, shadows, font definitions
2. Read `@theme inline` → extract registered Tailwind tokens
3. Read `components/ui/` → list installed components, detect spacing patterns
4. Reverse-engineer the 5 patterns from what's in code:
   - Colors → extract `--primary`, `--brand-*`, `--chart-*` values
   - Density → infer from Tailwind classes in component files (card padding, control heights)
   - Borders → infer from `--radius` value
   - Typography → detect font-family from CSS or `next/font` imports
   - Weight → detect shadow variables
5. Skip to step 8 (generate DESIGN.md) and step 9 (create frame in Paper) with the detected values
6. Report: "Initialized DESIGN.md from existing project. Detected: [primary color], [density level], [radius style], [font]. Review the DESIGN.md frame in Paper and run `/papercraft design.md` to adjust."

**Path C — Initialize from drawn components:**

The designer drew UI elements directly in Paper (buttons, cards, inputs) with the visual style they want. The agent reads exact computed styles instead of interpreting images.

**Minimum required elements:** 1 Button + 1 Card (with content) + 1 Input.
**Optional:** Badge, text nodes — help confirm detections.

1. `get_screenshot()` → view the full frame to understand the overall look
2. `get_children()` → identify each element by structure (frames with text = buttons/cards, rectangles with borders = inputs)
3. `get_computed_styles()` → extract exact values from each element:
   - `border-radius` → detect per category (see below)
   - `padding`, `gap`, `height` → map to Compact/Comfortable/Spacious
   - `background-color` of button → primary color
   - `background-color`, `border-color` of card/input → base color (grays)
   - `font-family`, `font-size` → typography (**only from component elements** — buttons, cards, inputs, headings). **Never read `font-family` from the `Prompt:` text node** — the designer often types prompts in monospace or a system font, which is NOT a style signal)
   - `box-shadow` of card → weight (flat/elevated)
   - If heading text uses a different font than body text → preserve both (e.g., serif headings + sans body)
4. Read `Prompt:` if present → additional instructions from the designer. If the Prompt explicitly mentions a font, that overrides what was detected from components.

**Normalization rule:** The designer draws approximate values — they won't match any density tier exactly. The agent must match to the **nearest tier**, not copy literal values.

   How to match density:
   - Collect all height/padding/gap values from drawn elements
   - Compare each against the 3 tiers in density-reference.md
   - Use majority voting: if 2 of 3 elements are closer to Comfortable, pick Comfortable
   - All component values then normalize to that tier's exact classes (e.g., `h-9`, `p-6`)

   Example: Button h=34px (between 32 and 36), Card p=18px (between 16 and 24), Input h=35px
   → Button closer to h-9 (Comfortable), Card closer to p-4 (Compact), Input closer to h-9 (Comfortable)
   → Majority = Comfortable → all values normalize to Comfortable tier

   For radius — detect per category, NOT averaged:
   - Read `border-radius` from **Card** and **Input** → average these → pick nearest tier → sets `--radius`
   - Read `border-radius` from **Button** separately → if `border-radius >= button-height / 2` (fully rounded) AND cards/inputs are NOT pill → set `pill-buttons: true`
   - If all elements have the same radius tier → `pill-buttons: false`
   - Example: Card r=10px (Rounded), Input r=8px (Rounded), Button r=18px (≥ h/2 = pill) → `--radius: Rounded`, `pill-buttons: true`

   For colors: use exact values (oklch-skill generates the scale from the detected primary).

5. Continue with step 4 (choose base color) using the detected gray values from card/input borders

---

**Path A — Initialize from mood board:**

1. **Read the frame:**
   - `get_selection()` → get the active frame
   - `get_children()` → find images, drawn elements, and text nodes
   - If images exist → `get_screenshot()` of the full frame
   - Find the `Prompt:` text node → extract the designer's instruction

2. **Analyze the mood board — detect exactly these 5 patterns:**

   The mood board can contain many visual signals, but only extract these 5. Ignore everything else.

   **Pattern 1 — Density (spacing):**
   - **Compact** → content is tight, many elements per screen, little whitespace
   - **Comfortable** → standard balance (default)
   - **Spacious** → lots of breathing room, few elements, generous whitespace

   Detect by: amount of empty space between elements in the mood board images.

   **Pattern 2 — Borders (radius):**
   - **Sharp** → straight or nearly straight corners (0-4px)
   - **Rounded** → soft corners (8-12px)
   - **Soft** → very rounded corners (16-24px)
   - **Pill** → fully rounded on ALL elements (buttons, inputs, cards)

   **Detection — two signals, not one:**

   Radius is detected per **category**, not as a single global value:

   1. **Base radius** — read from **cards and inputs** (the stable elements). This sets `--radius`.
   2. **Button radius** — read from **buttons and badges**. If buttons are pill (`border-radius >= height/2`) but inputs/cards are NOT pill → set `pill-buttons: true`.

   | Cards/Inputs | Buttons | Result |
   |-------------|---------|--------|
   | Sharp | Sharp | `--radius: Sharp`, pill-buttons: no |
   | Rounded | Rounded | `--radius: Rounded`, pill-buttons: no |
   | Rounded | Pill | `--radius: Rounded`, pill-buttons: **yes** |
   | Soft | Pill | `--radius: Soft`, pill-buttons: **yes** |
   | Pill | Pill | `--radius: Pill`, pill-buttons: no (everything is pill) |
   | Sharp | Pill | `--radius: Sharp`, pill-buttons: **yes** |

   **Key rule:** `--radius` is ALWAYS determined by cards/inputs. Buttons can override with pill independently.

   When `pill-buttons` is true, the codemod applies `rounded-full` to Button, Badge, and Toggle — see `density-reference.md` "Pill Buttons Override" section. All other components (Input, Select, Card, Dialog, etc.) use the base `--radius` scale normally.

   **Pattern 3 — Colors (primary + charts):**

   **If Prompt exists and specifies colors** → use exactly what the designer wrote. Prompt always wins.

   **If no Prompt or Prompt doesn't mention colors** → analyze the mood board images automatically:
   1. Detect the dominant accent/brand color → use as **primary color**
   2. Look for a secondary color that contrasts well with the primary:
      - If found → use it as the **chart base color** and generate chart-1 through chart-5 from its OKLCH scale
      - If NOT found (only one color in the mood board) → charts use the **primary color's own scale** (chart-1 = brand-100, chart-2 = brand-300, chart-3 = brand-500, chart-4 = brand-700, chart-5 = brand-900)

   Don't ask the designer to choose — decide automatically. The designer can always adjust later via `/papercraft component`.

   When generating chart colors from a secondary color, use the oklch-skill to produce a 5-step scale with good contrast between steps. The chart colors should be visually harmonious with the primary — same or analogous hue family works best.

   **Pattern 4 — Typography:**
   - **Geometric** → clean, modern sans-serif (Inter, Poppins, Geist)
   - **Humanist** → warm, friendly sans-serif (Nunito, Source Sans)
   - **Monospace** → technical, editorial (JetBrains Mono, Fira Code)
   - **Serif** → elegant, editorial (Playfair, Merriweather)

   **Detection priority:**
   1. If the `Prompt:` text or command prompt **explicitly mentions a font** → use it (Prompt always wins)
   2. Detect from **component text** (button labels, card titles, input placeholders) — see exclusion rule below
   3. For mood board images → detect visually: serif vs sans-serif vs monospace
   4. If nothing is detected → keep the project's current font — don't change it

   **⚠️ Prompt text exclusion rule:** The `Prompt:` text node is an instruction for the agent, NOT a design element. **Completely ignore its `font-family`** when detecting typography. The designer often types the prompt in monospace or a system font — that does NOT mean the design uses monospace. Only read `font-family` from actual UI elements (buttons, cards, headings, body text). If the only text nodes in the frame are `Prompt:` nodes, skip font detection from text entirely and rely on image analysis (step 3) or keep the current font (step 4).

   **Heading ≠ Body:** If headings use a different font category than body text (e.g., serif headings + sans-serif body), preserve that combination. Record both in DESIGN.md section 3.

   **Pattern 5 — Weight (surface contrast):**
   - **Flat** → little contrast between surfaces, no shadows (Linear, Vercel style)
   - **Elevated** → soft shadows, layered depth (Airbnb, Apple style)

   Detect by: presence of shadows in the mood board images.

   **What NOT to detect:** iconography, animations, layout/grid, dark/light mode preference. These are out of scope for `init`.

   **Detection hierarchy — what to look at in the mood board:**

   Not all elements are equally reliable for detection. Use this priority order:

   For **density**: padding inside cards > space between elements > amount of content per screen
   For **radius**: card corners + input corners → base radius. Button corners → pill-buttons flag if they diverge.

   Cards and inputs are the most stable elements — they set the base radius. Buttons may diverge (pill buttons with rounded inputs is very common).

   **If nothing is detected for a pattern → use Comfortable as default.** Never leave a pattern undefined.

   **Exception — Density:** If density cannot be determined with confidence from the mood board, do NOT default silently. Instead, ask the designer:
   > "I couldn't determine the density from the mood board. Which style do you prefer?"
   > `[ Compact ]  [ Comfortable ]  [ Spacious ]`

   **Density and radius are independent axes.**

   Density controls spacing, heights, and gaps. Radius controls corner curvature. They are configured separately — a project can be compact with pill radius, or spacious with sharp corners. shadcn's Lyra style proves this: it's compact AND sharp (`rounded-none`).

   The parent-child radius rule (see Rules section in SKILL.md) still applies for nested containers, but density does not constrain radius choices.

3. **Generate color scales:**
   - Use the oklch-skill to generate a complete 50→950 scale for the **primary color**
   - If a secondary color was detected for charts → generate a 5-step chart scale from it using oklch-skill
   - If no secondary color → chart colors will come from the primary brand scale (see step 4)
   - The oklch-skill handles gamut clamping, perceptual uniformity, and chroma distribution for all scales

4. **Choose base color (gray scale):**

   The base color determines all neutral tokens: `--background`, `--foreground`, `--card`, `--card-foreground`, `--popover`, `--popover-foreground`, `--secondary`, `--secondary-foreground`, `--muted`, `--muted-foreground`, `--accent`, `--accent-foreground`, `--border`, `--input`, `--sidebar`, `--sidebar-foreground`, `--sidebar-accent`, `--sidebar-accent-foreground`, `--sidebar-border`. Ask the designer:

   > "Which gray scale do you want?"
   > `[ neutral (pure gray) ] [ slate (blue tint) ] [ zinc (cool) ] [ stone (warm) ] [ custom (brand tint) ]`

   If the mood board clearly shows tinted grays, suggest the closest match. If grays are pure, default to `neutral`.

   - **Tailwind scale** (neutral, slate, zinc, stone, gray) → use the predefined OKLCH values from Tailwind
   - **Custom** → use oklch-skill to generate a gray scale with chroma ~0.01-0.02 and hue matching the brand color

   Save the base color choice in DESIGN.md section 2 under "Semantic Tokens".

5. **Write the theme to code:**

   In `globals.css`, add the brand scale, chart colors, and shadcn mappings:

   ```css
   :root {
     /* Brand scale */
     --brand-50: oklch(...);
     --brand-100: oklch(...);
     /* ... through 950 */

     /* Point shadcn primary to brand */
     --primary: var(--brand-500);
     --primary-foreground: oklch(...); /* white or black, calculated by L threshold */

     /* ⚠️ Accent/muted/secondary — ALWAYS from base scale, NEVER brand */
     --accent: oklch(...);            /* base-100 value from chosen gray scale */
     --accent-foreground: oklch(...); /* base-900 value from chosen gray scale */
     --muted: oklch(...);             /* base-100 value */
     --muted-foreground: oklch(...);  /* base-500 value */
     --secondary: oklch(...);         /* base-100 value */
     --secondary-foreground: oklch(...); /* base-900 value */

     /* Chart colors — from secondary color scale, or from brand if no secondary */
     --chart-1: var(--brand-100); /* or oklch(...) from secondary scale */
     --chart-2: var(--brand-300);
     --chart-3: var(--brand-500);
     --chart-4: var(--brand-700);
     --chart-5: var(--brand-900);
   }
   ```

   **Critical:** `--accent`, `--muted`, `--secondary` and their foregrounds must use base-scale gray values (from the chosen gray scale: neutral, slate, zinc, stone). Do NOT set them to `var(--brand-*)`. Only `--primary`, `--ring`, and `--sidebar-primary` point to the brand scale. See design-md-spec.md "Origin rule" for the full split.

   In the `@theme inline` block, register the brand scale for Tailwind:

   ```css
   @theme inline {
     --color-brand-50: var(--brand-50);
     --color-brand-100: var(--brand-100);
     /* ... through 950 */
   }
   ```

   This makes `bg-brand-200`, `text-brand-700`, etc. available throughout the project.

6. **Apply visual style — radius and density:**

   **First: read the shadcn style defaults** from `density-reference.md` → "shadcn Style Defaults" table. Read `components.json` → `style` field (e.g., `"base-mira"` → style is `mira`). Look up the style's built-in density, radius, and pill-buttons. Compare with the detected values from the mood board to decide what needs changing:

   | Axis | Detected == style default? | Action |
   |------|---------------------------|--------|
   | Density | Yes | **Skip** density codemod |
   | Density | No | **Run** density codemod |
   | Radius | Yes | **Skip** `--radius` change |
   | Radius | No | **Update** `--radius` in globals.css |
   | pill-buttons | Not detected, style has none | **Skip** |
   | pill-buttons | Detected, style has none | **Apply** rounded-full to Button/Badge/Toggle |

   Each axis is independent. Report what was skipped and why: "Density: Compact — matches style mira, skipped codemod. Radius: Rounded → Soft — changed --radius to 1.25rem. Pill buttons: detected — applied rounded-full to Button/Badge/Toggle."

   **Radius** — set `--radius` in `globals.css` based on detected borders (only if different from style default):
   - Sharp → `0.25rem`
   - Rounded → `0.625rem`
   - Soft → `1.25rem`
   - Pill → `1.5rem`

   The Tailwind v4 radius scale (`--radius-sm` through `--radius-4xl`) derives automatically from `--radius`. No other radius work needed.

   **Density** — run the codemod on installed components (only if different from style default):

   Read `density-reference.md` (in the same directory as this skill) to look up the correct Tailwind classes for the detected density level (Compact, Comfortable, or Spacious).

   For each component file in `components/ui/`:
   1. Look up the component in the density reference table
   2. Replace spacing/sizing classes with the values for the detected tier
   3. Leave everything else untouched (colors, states, a11y, cva logic)

   The codemod is idempotent — running it twice with the same density produces identical output. The reference file lists every class per component per tier so the skill knows exactly what to write.

   **Card spacing rule:** The Card wrapper uses `py-*` only (vertical padding). Sections (CardHeader, CardContent, CardFooter) handle `px-*` individually. This prevents double horizontal padding.

   **Typography** — if detected (Pattern 4), install the font via `next/font` and update `--font-sans` or `--font-heading` in globals.css. If not detected, don't change fonts.

   **Weight** — if detected (Pattern 5):
   - **Flat** → no shadow variables, keep shadcn defaults
   - **Elevated** → add `--shadow-card` and apply to Card component

   **What NOT to modify:** Don't touch `cva()` variant definitions, event handlers, accessibility attributes, conditional logic, colors, or state classes. Only replace spacing/sizing Tailwind classes.

7. **Generate dark mode:**
   - Derive dark mode by reversing the brand scale mapping mathematically (oklch-skill handles this)
   - In `.dark`, set `--primary: var(--brand-400)` (lighter step for dark backgrounds)
   - Set `--primary-foreground` based on contrast (L threshold from oklch-skill)
   - Derive `--background`, `--card`, `--muted`, etc. from the brand scale's darkest steps

8. **Generate `DESIGN.md` — project design system memory:**

   **If `DESIGN.md` already exists** → ask before overwriting:
   > "DESIGN.md already exists. Do you want to overwrite it with the new style?"
   > `[ Overwrite ]  [ Cancel ]`

   If cancelled, stop here — the designer can use `/papercraft design.md` to update specific properties instead.

   Read `references/design-md-spec.md` → Section 2 (DESIGN.md Text Template). Follow that template exactly, filling in the detected values. All 26 semantic tokens must be listed — see the Token Map in the same file for the complete list.

9. **Create "DESIGN.md - pc" frame in Paper:**

   If a "DESIGN.md - pc" frame already exists, update it in place — don't create a duplicate.

   Read `references/design-md-spec.md` → Section 3 (Paper Frame Layout). Follow that layout exactly — it defines all 7 sections, the 26-token semantic tokens grid, and the Prompt area. Use `create_artboard()` + `write_html()` to render.

**What NOT to do:**
- Don't override shadcn's internal state handling (hover, active, disabled, focus) — shadcn manages those
- Don't ask the designer for information that's already defined in `globals.css` or `@theme inline`
