# Papercraft Skill

Papercraft bridges the **Paper** design canvas with a **shadcn/ui** codebase. Designers communicate intent by writing directly in Paper frames — the agent reads those intentions via MCP and orchestrates theme generation, component creation, and full page design.

## Quick Start

```
/papercraft init          # Set up theme from a mood board
/papercraft design.md     # Iterate on the design system
/papercraft preview       # Render components in Paper
/papercraft component     # Create or edit a component
/papercraft screen        # Design a full page/view
/papercraft reset         # Restore defaults
```

---

## Prerequisites

- **Paper Desktop** running with MCP server at `http://127.0.0.1:29979/mcp`
- **shadcn/ui** initialized in the project (`components.json` must exist)
- **oklch-skill** and **shadcn skill** available (papercraft delegates to them)

---

## Commands

### `/papercraft init`

**One-time setup.** Reads a mood board from Paper and generates the complete design system: colors, typography, density, radius, and weight.

**Selection required:** Yes — select the mood board frame in Paper before running.

**What it does:**
1. Analyzes the selected frame for images, drawn elements, and `Prompt:` text nodes
2. Detects visual patterns: density tier, radius tier, colors, typography, weight
3. Generates brand color scale (50→950) using oklch-skill
4. Writes theme to `globals.css` (CSS variables) and `@theme inline` (Tailwind registration)
5. Applies density codemod to all `components/ui/*.tsx` files
6. Generates `DESIGN.md` file (agent-readable summary)
7. Creates `DESIGN.md - pc` frame in Paper (designer-readable visual reference)

**Input modes:**
- **Mood board (Path A):** Selected frame contains images → visual analysis
- **Existing code (Path B):** No selection → reverse-engineers theme from `globals.css` and components
- **Drawn components (Path C):** Selected frame has UI elements → extracts exact values

**Example:**
1. In Paper, create a frame with reference images and add a text node:
   ```
   Prompt: Create the visual style based on this moodboard
           and use blue-500 from tailwind as the primary color
   ```
2. Select the frame
3. Run `/papercraft init`

---

### `/papercraft design.md [prompt]`

**Iterate on the design system.** Adjust typography, density, radius, colors, or weight after initial setup.

**Selection required:** No — auto-finds the `DESIGN.md - pc` frame.

**What it does:**
1. Reads current theme state from code (`DESIGN.md`, `globals.css`, components)
2. Detects changes via `Prompt:` text nodes, drawn elements, or visual differences
3. Updates code (globals.css, component files) and Paper frame to match
4. Offers to cascade updates to all other `- pc` frames

**Sub-commands:**
- `design.md rebuild` — Reconstruct the Paper frame from current code state
- `design.md sync` — Batch propagate current theme to all `- pc` frames

**Example:**
```
/papercraft design.md change density to compact and radius to pill
```

Or write directly in the `DESIGN.md - pc` frame's prompt area:
```
Prompt: Switch to compact density with rounded radius
```

---

### `/papercraft preview`

**Render installed shadcn components** in Paper with the current theme applied.

**Selection required:** No.

**What it does:**
1. Reads installed components from `components/ui/`
2. Renders each component in default state plus 2-3 variants
3. Creates one `ComponentName - pc` frame per component

**Batching:**
- 7 or fewer components → draws all at once
- More than 7 → draws priority batch first (Button, Input, Card, Switch, Badge, Select, Table)
- Specify components: `/papercraft preview button,checkbox,dialog`

---

### `/papercraft component`

**Create or iterate on a component** — from a Paper frame, a name, or a text prompt.

**Selection required:** Optional.

**Input modes:**
- **Paper frame selected:** Reads images, drawn elements, and `Prompt:` text from the frame
- **Name provided:** Looks up existing component or creates new one
- **Text prompt:** Describes what to build

**Two component types:**
- **Base components** — Primitives that go in `components/ui/` (follows shadcn conventions)
- **Complex components** — Compositions that go in `components/` (combine base components)

**Iteration mode:** If the frame name matches an existing `- pc` component, papercraft reads the current source and applies only the requested changes — it never rewrites the entire component.

**Example — create from prompt:**
```
/papercraft component a stats card with icon, value, label, and trend indicator
```

**Example — iterate from Paper:**
1. Select an existing `StatsCard - pc` frame
2. Add text: `Prompt: Add a sparkline chart below the value`
3. Run `/papercraft component`

---

### `/papercraft screen [prompt]`

**Design a full page or view** by composing existing components.

**Selection required:** Optional — accepts a prompt or a reference image in Paper.

**What it does:**
1. Plans layout structure (proposes ASCII layout for complex screens with 5+ sections)
2. Prioritizes existing components → composes existing → searches shadcn registry → creates custom
3. Renders in Paper with realistic mock data
4. Creates `ScreenName - pc` frame with Visual and Info sub-frames

**Output destinations:**
- **Paper** — Fast ideation, visual output only
- **Code** — Full implementation with page file, component imports, TypeScript types
- **Both** — Paper preview + code implementation

**Example:**
```
/papercraft screen a dashboard with sidebar navigation, stats row, chart area, and recent activity table
```

---

### `/papercraft reset`

**Restore project to shadcn defaults** and clean up Paper frames.

**Selection required:** No.

**What it does:**
1. Asks for confirmation before proceeding
2. Restores `globals.css` via `bunx --bun shadcn@latest init --force`
3. Removes `DESIGN.md` file
4. Deletes all `- pc` frames from Paper

**What it does NOT touch:** Custom components, installed shadcn components, and density classes from the last codemod remain intact.

---

## How Designer Intent Works

The designer communicates by writing inside Paper frames. The agent reads — it never assumes.

### The `Prompt:` contract

Any text node in a Paper frame that starts with `Prompt:` is treated as an instruction:

```
[Frame: "My Design"]
  - Images (visual references)
  - Drawn elements (exact styles to extract)
  - Text: "Prompt: Use this moodboard, make primary color teal"
```

### Detection logic

When papercraft reads a frame, it classifies its contents:

| Content type | How it's read | What it provides |
|-------------|---------------|-----------------|
| **Images** | `get_screenshot()` on the frame | Visual references, mood, color inspiration |
| **Drawn elements** | `get_computed_styles()` | Exact colors, spacing, radius values |
| **`Prompt:` text** | `get_node_info()` | Explicit designer instructions |

All signals are combined to form the complete intent.

---

## The Design System

Papercraft manages three independent visual axes plus a color system.

### Density (spacing & sizing)

Three tiers applied via Tailwind classes directly in component files:

| Tier | Button height | Card padding | Control text | Menu item padding |
|------|--------------|-------------|-------------|------------------|
| **Compact** | `h-8` (32px) | `p-4` | `text-xs` (12px) | `py-1 px-1.5` |
| **Comfortable** | `h-9` (36px) | `p-6` | `text-sm` (14px) | `py-1.5 px-2` |
| **Spacious** | `h-9` (36px) | `p-6` | `text-sm` (14px) | `py-2 px-3` |

Comfortable and Spacious share control heights — they differ in menu items, table rows, and overall breathing room.

### Radius (corner rounding)

Four tiers controlled by a single `--radius` CSS variable:

| Tier | `--radius` value | Visual character |
|------|-----------------|-----------------|
| **Sharp** | `0.25rem` | Minimal rounding |
| **Rounded** | `0.625rem` | Balanced, default feel |
| **Soft** | `1.25rem` | Generous rounding |
| **Pill** | `1.5rem` | Maximum rounding, buttons get `rounded-full` |

Tailwind v4 derives the full scale (`rounded-sm` through `rounded-4xl`) from this single value. Each component uses a specific radius class (e.g., Card: `rounded-xl`, Button: `rounded-lg`).

**Parent-child radius rule:** When a container wraps an element with border-radius:
```
parentRadius = childRadius + parentPadding
```

### Weight (shadows)

| Tier | Effect |
|------|--------|
| **Flat** | No shadows |
| **Elevated** | Soft shadow via `--shadow-card` variable |

### Color system

- **Brand scale (50→950):** 11 OKLCH tokens derived from the primary color. Registered as `--brand-50` through `--brand-950`.
- **Base scale (50→950):** Neutral foundation (neutral/slate/zinc/stone or custom). Used for `--muted`, `--muted-foreground`, etc.
- **Semantic tokens (26 total):** Organized in 6 groups — Core, Surfaces, Actions, States, Borders, Sidebar. All mapped in `globals.css`.
- **Chart colors (5 tokens):** Derived from brand or secondary color scale.

**Source of truth:** `globals.css` contains the actual OKLCH values. `DESIGN.md` records decisions. The `DESIGN.md - pc` Paper frame is the visual representation.

---

## Frame Structure

All papercraft-generated frames follow this convention:

### Naming — `- pc` suffix

Every frame gets the `- pc` suffix:
```
DESIGN.md - pc
Button - pc
Sales Dashboard - pc
```

This enables discovery via a single `get_children()` call on root.

### Nested structure

```
[Container: "Name - pc"]
  ├── [Visual]   ← rendered output (colors, typography, layout)
  └── [Info]     ← metadata (status, composition, variants, prompt, changes)
```

- Visual and Info are separate frames — never mix technical text inside the visual
- The `- pc` suffix goes on the container, not inner frames

### Info frame contents

| Field | Purpose |
|-------|---------|
| STATUS | Paper only / Code only / Paper + Code |
| COMPOSITION | Which base components are used |
| TYPE | Base or Complex |
| Variants | Available variants, sizes, props |
| PROMPT | Designer's current instruction |
| CHANGES | History of modifications |

---

## Typography Rules

Papercraft uses **only** Tailwind's standard text scale — never custom sizes:

| Class | Size |
|-------|------|
| `text-xs` | 12px |
| `text-sm` | 14px |
| `text-base` | 16px |
| `text-lg` | 18px |
| `text-xl` | 20px |
| `text-2xl` | 24px |
| `text-3xl` | 30px |

- **In code:** Use Tailwind classes (`text-xs`, `text-sm`, etc.)
- **In Paper:** Use the corresponding px values (`12px`, `14px`, etc.) — Paper has no CSS variable context
- The mood board informs font **family** and **weight**, not sizes

---

## Rendering in Paper

When drawing with `write_html()`, CSS variables don't work — Paper has no runtime CSS context.

**Resolution process:**
1. Read `globals.css` to get actual OKLCH values
2. Resolve variable chains: if `--accent: var(--brand-50)` and `--brand-50: oklch(0.990 0.015 114)`, use `oklch(0.990 0.015 114)` directly
3. Every color in Paper must exactly match its code counterpart

This prevents Paper showing plain grays while the app shows brand-tinted neutrals.

---

## Integration

Papercraft is the **orchestrator**. It knows *when* and *why* to call each tool, but delegates the *how*:

| Dependency | Used for |
|-----------|----------|
| **oklch-skill** | Color conversion, palette generation (50→950), contrast checking, gamut clamping |
| **shadcn skill** | Component search, installation, docs lookup, composition patterns, variant conventions |
| **Paper MCP** | All reads from and writes to the design canvas |

---

## Typical Workflows

### Starting a new project theme

1. Create a mood board frame in Paper with reference images
2. Add a `Prompt:` text node with color/style preferences
3. Select the frame → `/papercraft init`
4. Review the generated `DESIGN.md - pc` frame and code
5. Iterate with `/papercraft design.md` as needed

### Previewing components with the theme

1. Install shadcn components: `bunx --bun shadcn@latest add button card input`
2. Run `/papercraft preview` to see them rendered in Paper
3. Iterate on individual components with `/papercraft component`

### Designing a page

1. Run `/papercraft screen` with a description or select a reference frame
2. Review the layout in Paper
3. Iterate by adding `Prompt:` text to the screen's frame
4. Send to code when satisfied

### Updating the theme later

1. Open the `DESIGN.md - pc` frame in Paper
2. Write new instructions in the prompt area
3. Run `/papercraft design.md`
4. Use `design.md sync` to propagate changes to all existing `- pc` frames
