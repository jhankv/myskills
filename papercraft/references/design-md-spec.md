# design-md-spec.md

Single source of truth for the DESIGN.md system. Read this file whenever you need to:
- Create DESIGN.md (text file) — during `init`
- Create or rebuild the "DESIGN.md - pc" frame in Paper — during `init` or `design.md rebuild`
- Verify token consistency between globals.css ↔ DESIGN.md ↔ Paper frame

The three formats are mirrors of the same data:

```
globals.css + @theme inline  → browser reads this
DESIGN.md (text file)        → agent reads this
"DESIGN.md - pc" (Paper)     → designer reads this
```

All three must stay in sync. When one changes, the others must update.

---

## 1. Token Map

These are ALL the tokens that must exist in globals.css, DESIGN.md, and the Paper frame. None may be omitted.

### Brand Scale (11 tokens)

| Token | Role | Example |
|-------|------|---------|
| `--brand-50` | Subtle tinted surfaces (light mode) | `oklch(0.990 0.015 H)` |
| `--brand-100` | Hover backgrounds (light mode) | `oklch(0.983 0.040 H)` |
| `--brand-200` | Accent borders, chart fill | `oklch(0.968 0.075 H)` |
| `--brand-300` | Chart secondary, decorative | `oklch(0.945 0.120 H)` |
| `--brand-400` | Primary in dark mode | `oklch(0.910 0.155 H)` |
| `--brand-500` | Primary in light mode, CTAs | `oklch(0.850 0.180 H)` |
| `--brand-600` | Hover state for primary | `oklch(0.760 0.165 H)` |
| `--brand-700` | Chart accent, pressed states | `oklch(0.660 0.140 H)` |
| `--brand-800` | Deep accent | `oklch(0.545 0.110 H)` |
| `--brand-900` | Text on light tinted surfaces | `oklch(0.440 0.080 H)` |
| `--brand-950` | Accent surfaces in dark mode | `oklch(0.310 0.055 H)` |

`H` = hue angle chosen during init. Lightness and chroma values are fixed per step.

### Base Scale (11 tokens)

The gray scale used for all neutral surfaces, text, and borders. Determined by the **base color** choice during init. This scale is NOT stored as CSS variables — it's the source palette from which semantic tokens derive their neutral OKLCH values.

| Step | neutral | slate | zinc | stone |
|------|---------|-------|------|-------|
| 50 | `oklch(0.985 0 0)` | `oklch(0.984 0.003 247)` | `oklch(0.985 0.002 247)` | `oklch(0.985 0.002 75)` |
| 100 | `oklch(0.970 0 0)` | `oklch(0.968 0.007 247)` | `oklch(0.967 0.004 264)` | `oklch(0.970 0.004 75)` |
| 200 | `oklch(0.922 0 0)` | `oklch(0.929 0.013 255)` | `oklch(0.920 0.009 264)` | `oklch(0.923 0.008 75)` |
| 300 | `oklch(0.870 0 0)` | `oklch(0.869 0.022 252)` | `oklch(0.871 0.015 264)` | `oklch(0.869 0.013 75)` |
| 400 | `oklch(0.708 0 0)` | `oklch(0.704 0.04 256)` | `oklch(0.705 0.023 286)` | `oklch(0.709 0.019 75)` |
| 500 | `oklch(0.556 0 0)` | `oklch(0.554 0.046 257)` | `oklch(0.552 0.027 286)` | `oklch(0.553 0.023 75)` |
| 600 | `oklch(0.439 0 0)` | `oklch(0.446 0.043 257)` | `oklch(0.442 0.026 286)` | `oklch(0.444 0.021 75)` |
| 700 | `oklch(0.371 0 0)` | `oklch(0.372 0.044 257)` | `oklch(0.374 0.023 286)` | `oklch(0.374 0.018 75)` |
| 800 | `oklch(0.269 0 0)` | `oklch(0.279 0.041 260)` | `oklch(0.274 0.019 286)` | `oklch(0.268 0.014 75)` |
| 900 | `oklch(0.205 0 0)` | `oklch(0.208 0.042 265)` | `oklch(0.210 0.014 286)` | `oklch(0.216 0.012 75)` |
| 950 | `oklch(0.145 0 0)` | `oklch(0.129 0.042 264)` | `oklch(0.141 0.010 286)` | `oklch(0.147 0.007 75)` |

**How it's used:** The semantic tokens pick specific steps from this scale. For example, with neutral base color:
- `--background` light = base-50 area (`oklch(1 0 0)` — pure white, even lighter than 50)
- `--border` light = base-200 (`oklch(0.922 0 0)`)
- `--muted-foreground` light = base-500 (`oklch(0.556 0 0)`)
- `--foreground` light = base-950 (`oklch(0.145 0 0)`)

The designer sees this scale in Paper and can say "use base-300 for border" — the agent then updates the `--border` token to that step's OKLCH value.

### Semantic Tokens (26 tokens)

Organized in 6 groups. Each token has a light and dark value.

**Core (2)**
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--background` | `oklch(1 0 0)` | `oklch(0.145 0 0)` | Page background |
| `--foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` | Default text |

**Surfaces (4)**
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--card` | `oklch(1 0 0)` | `oklch(0.205 0 0)` | Card surfaces |
| `--card-foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` | Card text |
| `--popover` | `oklch(1 0 0)` | `oklch(0.205 0 0)` | Popover/dropdown bg |
| `--popover-foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` | Popover text |

**Actions (4)**
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--primary` | `brand-500` | `brand-400` | Primary buttons, CTAs |
| `--primary-foreground` | depends on contrast | depends on contrast | Text on primary |
| `--secondary` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` | Secondary actions |
| `--secondary-foreground` | `oklch(0.205 0 0)` | `oklch(0.985 0 0)` | Text on secondary |

**States (5)**
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--muted` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` | Muted backgrounds |
| `--muted-foreground` | `oklch(0.556 0 0)` | `oklch(0.708 0 0)` | Muted text, placeholders |
| `--accent` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` | Hover/accent states |
| `--accent-foreground` | `oklch(0.205 0 0)` | `oklch(0.985 0 0)` | Text on accent |
| `--destructive` | `oklch(0.577 0.245 27.325)` | `oklch(0.704 0.191 22.216)` | Error, delete |

**Borders (3)**
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--border` | `oklch(0.922 0 0)` | `oklch(1 0 0 / 10%)` | Borders, dividers |
| `--input` | `oklch(0.922 0 0)` | `oklch(1 0 0 / 15%)` | Input borders |
| `--ring` | `brand-500` | `brand-400` | Focus rings |

**Sidebar (8)**
| Token | Light | Dark | Usage |
|-------|-------|------|-------|
| `--sidebar` | `oklch(0.985 0 0)` | `oklch(0.205 0 0)` | Sidebar background |
| `--sidebar-foreground` | `oklch(0.145 0 0)` | `oklch(0.985 0 0)` | Sidebar text |
| `--sidebar-primary` | `brand-600` | `brand-400` | Sidebar active item |
| `--sidebar-primary-foreground` | `oklch(1 0 0)` | `oklch(0.145 0 0)` | Text on sidebar active |
| `--sidebar-accent` | `oklch(0.97 0 0)` | `oklch(0.269 0 0)` | Sidebar hover + active bg |
| `--sidebar-accent-foreground` | `oklch(0.205 0 0)` | `oklch(0.985 0 0)` | Text on sidebar hover + active |
| `--sidebar-border` | `oklch(0.922 0 0)` | `oklch(1 0 0 / 10%)` | Sidebar dividers |
| `--sidebar-ring` | `brand-500` | `brand-400` | Sidebar focus rings |

The values above are defaults for **neutral** base color. Other base colors (slate, zinc, stone) change the neutral OKLCH values (chroma > 0 with a tint hue). Brand-referenced tokens always use the brand scale regardless of base color.

**⚠️ Origin rule — base scale vs brand scale:**

Following shadcn's defaults, tokens split into two groups:

- **Base scale (neutral grays):** `--background`, `--foreground`, `--card`, `--card-foreground`, `--popover`, `--popover-foreground`, `--secondary`, `--secondary-foreground`, `--muted`, `--muted-foreground`, **`--accent`**, **`--accent-foreground`**, `--border`, `--input`, `--sidebar`, `--sidebar-foreground`, **`--sidebar-accent`**, **`--sidebar-accent-foreground`**, `--sidebar-border`
- **Brand scale:** `--primary`, `--primary-foreground`, `--ring`, `--sidebar-primary`, `--sidebar-primary-foreground`, `--sidebar-ring`
- **Fixed:** `--destructive` (red, independent of both scales)

Do NOT inject the brand hue into base-scale tokens. This is the most common source of inconsistency between code and Paper.

### Chart Colors (5 tokens)

| Token | Light | Dark |
|-------|-------|------|
| `--chart-1` | `brand-500` | `brand-400` |
| `--chart-2` | `brand-700` | `brand-300` |
| `--chart-3` | `brand-300` | `brand-600` |
| `--chart-4` | `brand-900` | `brand-200` |
| `--chart-5` | `brand-200` | `brand-800` |

Source is either "derived from brand scale" (when no secondary color) or "derived from secondary color" (when a second hue is detected).

### Weight

| Value | CSS |
|-------|-----|
| Flat | `--shadow-card: none` |
| Elevated | `--shadow-card: 0 1px 3px oklch(0 0 0 / 10%), 0 1px 2px oklch(0 0 0 / 6%)` |

---

## 2. DESIGN.md Text Template

The DESIGN.md file at the project root. Format based on Google Stitch (5 base sections) + 2 papercraft sections.

Purpose:
- **Memory between sessions** — Claude Code doesn't persist context. DESIGN.md gives the agent full project context at the start of any session.
- **Design decisions** — records the choices (brand hue, base color, density, radius, typography) that can't be derived from code alone.
- **Ideation canvas** — when the agent draws in Paper, it reads DESIGN.md for non-color decisions (density, radius, typography, project context).

**Color source of truth:** `globals.css` — NOT DESIGN.md. DESIGN.md records the design *decisions* (brand hue, base color name, token origins) but the agent reads actual OKLCH values from `globals.css`.

**What NOT to put in DESIGN.md:** implementation instructions, detection rules, skill logic, or duplicated OKLCH color values (those live in globals.css). DESIGN.md is decisions, not data.

```markdown
# DESIGN.md
> Generated by papercraft

## 0. Project Context
- Framework: [Next.js (App Router) | Vite | Astro | Nuxt | ...]
- Icon library: [lucide | icons | ...]
- Headless UI: [Radix UI | Base UI | React Aria | none]
- shadcn style: [radix-nova | radix-vega | ...]
- Package manager: [bun | npm | pnpm | yarn]

## 1. Visual Theme & Atmosphere
Description of the mood detected from the mood board (or reverse-engineered from existing code).

## 2. Color Palette & Roles

- Brand hue: [number] ([color name])
- Base color: **[neutral | slate | zinc | stone | custom]**
- Primary: brand-500 (light) / brand-400 (dark)
- Primary foreground: [dark text | light text] (L > 0.6 threshold)
- Weight: [Flat | Elevated]

### Token origins
- **Brand scale →** --primary, --primary-foreground, --ring, --sidebar-primary, --sidebar-primary-foreground, --sidebar-ring
- **Base scale →** --background, --foreground, --card, --card-foreground, --popover, --popover-foreground, --secondary, --secondary-foreground, --muted, --muted-foreground, --accent, --accent-foreground, --border, --input, --sidebar, --sidebar-foreground, --sidebar-accent, --sidebar-accent-foreground, --sidebar-border
- **Fixed →** --destructive (red, independent of both scales)

### Chart Colors
- Source: [derived from brand scale | derived from secondary color]
- chart-1: brand-[X], chart-2: brand-[X], chart-3: brand-[X], chart-4: brand-[X], chart-5: brand-[X]

> Actual OKLCH values for all tokens → read from `globals.css` (source of truth)

## 3. Typography Rules
- Font: [family] ([category]) — both headings and body (or separate if different)
- Display: [size] / [weight] / [letter-spacing]
- Heading: [size] / [weight] / [letter-spacing]
- Body: [size] / [weight] / [letter-spacing]
- Code: [family or "same font"]

**Sizes must come from Tailwind's text scale only:**
text-xs=12px, text-sm=14px, text-base=16px, text-lg=18px, text-xl=20px, text-2xl=24px, text-3xl=30px, text-4xl=36px.
Never use in-between values (13px, 11px, 15px). This ensures consistency between Paper renders, component code, and screen code — all three read this section to determine font sizes.

## 4. Component Stylings
- Buttons: [radius — rounded-full if pill-buttons, else from --radius scale] / [color] / [shadow] / [height description]
- Cards: [radius from --radius scale] / [shadow] / [border] / [padding description]
- Inputs: [radius from --radius scale] / [border] / [focus ring] / [height description]
- Tables: [row height] / [font description]

## 5. Layout Principles
- Density: [Compact | Comfortable | Spacious]
- Control text: [text-xs (12px) for Compact | text-sm (14px) for Comfortable/Spacious]
- Control heights: h-[x] (default), h-[x] (sm), h-[x] (lg)
- Card: py-[x], gap-[x], px-[x] (sections)
- Menu items: py-[x], px-[x]
- Table: h-[x] head, p-[x] cells
- Method: direct Tailwind classes (see density-reference.md)

Control text size applies to: Button, Input, Select, Tabs trigger, Dropdown/Menu items, Toggle.
Not affected: Card body (text-sm), CardTitle/DialogTitle/SheetTitle (text-base), Badge (text-xs).

## 6. Radius
- Value: --radius: [value] ([Tier name])
- Tier: [Sharp | Rounded | Soft | Pill]
- Pill buttons: [yes | no] — if yes, Button/Badge/Toggle use rounded-full regardless of --radius
- Scale: sm=0.6x, md=0.8x, lg=1x, xl=1.4x, 2xl=1.8x, 3xl=2.2x, 4xl=2.6x

## 7. Generated Components
| Component | Variants | States | Date |
|-----------|----------|--------|------|
```

Section 7 starts empty. Updated automatically by `/papercraft component`.

---

## 3. Paper Frame Layout — "DESIGN.md - pc"

The "DESIGN.md - pc" frame is the visual mirror of the DESIGN.md file in Paper. It uses `write_html()` to render 7 sections in order.

### Frame creation

- Name: **"DESIGN.md - pc"** (the `- pc` suffix identifies papercraft frames)
- If frame already exists → update in place, never create a duplicate
- Use `create_artboard()` for first creation
- Default frame height: **1900px** — prevents content from being clipped in Paper

### Theme-aware rendering

The frame must render using the project's own semantic tokens for its chrome (headings, labels, backgrounds). **Paper has no CSS variable context — all values must be resolved to actual OKLCH before passing to `write_html()`.** Read `globals.css` and substitute every `var()` reference to its final OKLCH value.

- **Background:** use the resolved `--background` OKLCH value (e.g., `oklch(1 0 0)` for light, `oklch(0.145 0 0)` for dark)
- **Headings & labels:** use the resolved `--foreground` OKLCH value
- **Muted labels** (like "DENSITY", "RADIUS"): use the resolved `--muted-foreground` OKLCH value
- **Accent values** (like "Spacious", "1.25rem", "Flat"): use the resolved `brand-500` OKLCH value (light) or `brand-400` (dark)

**Never approximate:** if a token has chroma > 0 (brand tint), use that exact chroma in Paper. Don't substitute with plain gray (`oklch(X 0 0)`) when the real value is tinted (`oklch(X 0.015 114)`).

The mode (light/dark) is determined by the mood board during init — if the mood board has a dark look, the generated theme is dark-first, and the frame renders in dark mode. Read the `--background` value from DESIGN.md section 2 to determine the mode: if `--background` light is the dominant one used (L close to 1), render in light mode; if the dark value was chosen as primary (L close to 0.145), render in dark mode.

### Section 1 — Brand Scale

Color swatches for brand-50 through brand-950. Each swatch:
- Small colored rectangle (32x32px) filled with the actual OKLCH value
- Label below: token name + OKLCH value

Layout: horizontal row of 11 swatches.

### Section 2 — Base Scale

Color swatches for the gray/neutral scale (base-50 through base-950). Each swatch:
- Small colored rectangle (32x32px) filled with the actual OKLCH value
- Label below: step name + OKLCH value

Layout: horizontal row of 11 swatches, same format as Brand Scale.
Include a label showing which base color is active (e.g., "neutral", "slate").

This section lets the designer visually pick gray steps for token adjustments. For example, they can see base-300 and say "use that for --border."

### Section 3 — Semantic Tokens

This is the most critical section. It ensures Paper designs use the exact same colors as code. **Render ALL 26 tokens. Do not simplify, do not skip foregrounds, do not omit sidebar tokens.**

Layout: two columns — LIGHT MODE (left) and DARK MODE (right).
Group tokens with a visible group heading.

Each row has three lines:
1. `[color swatch 24x24]  token-name   oklch(value)` — main line, bold token name
2. `                       base-XXX` or `brand-XXX` — subtitle in muted-foreground color, smaller font size

The subtitle shows the **scale origin** — which step from the Base Scale or Brand Scale this token's value comes from. This lets the designer trace every semantic token back to the raw palette above.

**Scale origin rules:**
- Match the token's OKLCH value against the Base Scale steps (50→950). If it matches → show `base-XXX` (e.g., `base-950`)
- If the token references a brand variable (e.g., `var(--brand-500)`) → show `brand-XXX` (e.g., `brand-400`)
- If the value is pure white `oklch(1 0 0)` → show `white`
- If the value doesn't match any scale step (e.g., `--destructive`) → no subtitle

```
SEMANTIC TOKENS

LIGHT MODE                                    DARK MODE

Core
[███] --background   oklch(1 0 0)             [███] --background   oklch(0.145 0 0)
       white                                           base-950

[███] --foreground   oklch(0.145 0 0)         [███] --foreground   oklch(0.985 0 0)
       base-950                                        base-50

Surfaces
[███] --card   oklch(1 0 0)                   [███] --card   oklch(0.205 0 0)
       white                                           base-900

[███] --card-foreground   oklch(0.145 0 0)    [███] --card-foreground   oklch(0.985 0 0)
       base-950                                        base-50

[███] --popover   oklch(1 0 0)               [███] --popover   oklch(0.205 0 0)
       white                                           base-900

[███] --popover-foreground   oklch(0.145 0 0) [███] --popover-foreground   oklch(0.985 0 0)
       base-950                                        base-50

Actions
[███] --primary   oklch(0.850 0.180 114)      [███] --primary   oklch(0.910 0.155 114)
       brand-500                                       brand-400

[███] --primary-foreground   oklch(...)       [███] --primary-foreground   oklch(...)
       base-950                                        base-950

[███] --secondary   oklch(0.97 0 0)           [███] --secondary   oklch(0.269 0 0)
       base-100                                        base-800

[███] --secondary-foreground   oklch(0.205 0 0) [███] --secondary-foreground   oklch(0.985 0 0)
       base-900                                        base-50

States
[███] --muted   oklch(0.97 0 0)              [███] --muted   oklch(0.269 0 0)
       base-100                                        base-800

[███] --muted-foreground   oklch(0.556 0 0)  [███] --muted-foreground   oklch(0.708 0 0)
       base-500                                        base-400

[███] --accent   oklch(0.97 0 0)              [███] --accent   oklch(0.269 0 0)
       base-100                                        base-800

[███] --accent-foreground   oklch(0.205 0 0)     [███] --accent-foreground   oklch(0.985 0 0)
       base-900                                        base-50

[███] --destructive   oklch(0.577 0.245 27)  [███] --destructive   oklch(0.704 0.191 22)


Borders
[███] --border   oklch(0.922 0 0)            [███] --border   oklch(1 0 0 / 10%)
       base-200

[███] --input   oklch(0.922 0 0)             [███] --input   oklch(1 0 0 / 15%)
       base-200

[███] --ring   oklch(0.850 0.180 114)         [███] --ring   oklch(0.910 0.155 114)
       brand-500                                       brand-400

Sidebar
[███] --sidebar   oklch(0.985 0 0)           [███] --sidebar   oklch(0.205 0 0)
       base-50                                         base-900

[███] --sidebar-foreground   oklch(0.145 0 0) [███] --sidebar-foreground   oklch(0.985 0 0)
       base-950                                        base-50

[███] --sidebar-primary   oklch(0.760 0.165 114) [███] --sidebar-primary   oklch(0.910 0.155 114)
       brand-600                                       brand-400

[███] --sidebar-primary-fg   oklch(1 0 0)    [███] --sidebar-primary-fg   oklch(0.145 0 0)
       white                                           base-950

[███] --sidebar-accent   oklch(0.97 0 0)       [███] --sidebar-accent   oklch(0.269 0 0)
       base-100                                        base-800

[███] --sidebar-accent-fg   oklch(0.205 0 0)     [███] --sidebar-accent-fg   oklch(0.985 0 0)
       base-900                                        base-50

[███] --sidebar-border   oklch(0.922 0 0)    [███] --sidebar-border   oklch(1 0 0 / 10%)
       base-200

[███] --sidebar-ring   oklch(0.850 0.180 114) [███] --sidebar-ring   oklch(0.910 0.155 114)
       brand-500                                       brand-400
```

The values in this example are for neutral base color. Replace with actual values from globals.css.
Dark mode border/input tokens use alpha values (`oklch(1 0 0 / 10%)`) that don't map to a scale step — no subtitle for those.

### Section 4 — Chart Colors

chart-1 through chart-5 as color swatches with OKLCH values. Show both light and dark mode values.

### Section 5 — Typography

Font family, sizes for display/heading/body. Use the actual font from the project for rendering the text in Paper.

**Font sizes must use Tailwind's standard text scale only:** 12px (`text-xs`), 14px (`text-sm`), 16px (`text-base`), 18px (`text-lg`), 20px (`text-xl`), 24px (`text-2xl`), 30px (`text-3xl`), 36px (`text-4xl`). Never use in-between values like 13px, 11px, or 15px — these create inconsistency between Paper and code.

### Section 6 — Layout & Density

Current density tier (Compact/Comfortable/Spacious), control heights, card spacing values, and radius info including `pill-buttons` flag. This is a quick reference — the full density spec lives in `density-reference.md`.

Show:
- Density tier + key values (control heights, card padding, menu items)
- Radius: `--radius` value + tier name
- Pill buttons: yes/no — if yes, note that Button/Badge/Toggle use `rounded-full`

### Section 7 — Prompt

Visually distinct area for the designer to write instructions.

```
┌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┐
╎ PROMPT                                   ╎
╎                                          ╎
╎ Write here what you want to change,      ╎
╎ then run /papercraft design.md           ╎
╎                                          ╎
╎ AVAILABLE PRESETS                         ╎
╎ Density: compact, comfortable, spacious  ╎
╎ Radius: sharp, rounded, soft, pill       ╎
╎ Pill buttons: yes/no (pill buttons only) ╎
╎ Typography: geometric, humanist,         ╎
╎             monospace, serif              ╎
╎ Weight: flat, elevated                   ╎
╎ Color: any Tailwind color or hex         ╎
└╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌╌┘
```

Styling:
- Font: project's font family (from DESIGN.md section 3)
- Border: `1px dotted` with brand primary color (brand-500)
- Background: transparent or very subtle brand tint
