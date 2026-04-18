# Density & Radius Reference

Internal lookup table for the papercraft codemod. Data sourced from shadcn/ui styles (Nova, Vega, Maia).

## shadcn Style Defaults

Each shadcn style ships with a built-in density, radius, and pill-buttons configuration. Use this table to determine what's already applied and what needs changing.

| Style | Density | Control text | `--radius` | Radius tier | pill-buttons |
|-------|---------|-------------|------------|-------------|--------------|
| **Nova** | Compact | text-xs | 0.625rem | Rounded | no |
| **Mira** | Compact | text-xs | 0.625rem | Rounded | no |
| **Lyra** | Compact | text-xs | 0.25rem | Sharp | no |
| **Vega** | Comfortable | text-sm | 0.625rem | Rounded | no |
| **Maia** | Spacious | text-sm | 1.25rem | Soft | no |
| **Luma** | Spacious | text-sm | 0.625rem | Rounded | no |

> No default shadcn style ships with pill-buttons. Pill buttons are always a designer's override.

### Skip-codemod rule

During `init`, read the style from `components.json` → `style` field (e.g., `"base-mira"`), extract the style name, and compare with the detected values:

- **Density matches style default** → skip density codemod (components already have the correct spacing/sizing/text classes)
- **Density differs** → run density codemod to apply the detected tier
- **Radius matches style default** → skip `--radius` change in globals.css
- **Radius differs** → update `--radius` in globals.css
- **pill-buttons detected but style has none** → apply pill-buttons override (rounded-full on Button/Badge/Toggle)
- **pill-buttons NOT detected and style has none** → nothing to do

Each axis (density, radius, pill-buttons) is evaluated independently. A moodboard might match density but differ on radius — only the differing axis gets modified.

---

## Density Tiers

Density controls spacing, heights, gaps, AND control text size via direct Tailwind classes — no CSS variables.
Radius is a **separate axis** (see Radius section below).

### Control Text Size per Density

Interactive controls use a text size that matches their density tier:

| Density | Controls text | Applies to |
|---------|--------------|------------|
| **Compact** | `text-xs` (12px) | Button, Input, Select trigger, Tabs trigger, Dropdown item, Menu item, Toggle |
| **Comfortable** | `text-sm` (14px) | Same components |
| **Spacious** | `text-sm` (14px) | Same components |

**Not affected by this rule:** Card body (always `text-sm`), CardTitle / DialogTitle / SheetTitle (always `text-base`), Badge (always `text-xs`), headings (own scale).

When the codemod changes density tier, it must also update the text size class on these controls.

### Compact (based on Nova)

| Component | Property | Class |
|-----------|----------|-------|
| **Button** | h default | `h-8` |
| **Button** | h xs | `h-6` |
| **Button** | h sm | `h-7` |
| **Button** | h lg | `h-9` |
| **Button** | px default | `px-2.5` |
| **Button** | px xs | `px-2` |
| **Button** | px sm | `px-2.5` |
| **Button** | px lg | `px-2.5` |
| **Button** | gap | `gap-1.5` |
| **Button** | icon | `size-8` |
| **Button** | icon-sm | `size-7` |
| **Button** | icon-lg | `size-9` |
| **Input** | h | `h-8` |
| **Input** | file h | `file:h-6` |
| **Textarea** | px | — (default) |
| **Textarea** | py | — (default) |
| **Select trigger** | h default | `h-8` |
| **Select trigger** | h sm | `h-7` |
| **Select item** | py | `py-1` |
| **Select item** | pl | `pl-1.5` |
| **Select label** | py | `py-1` |
| **Select label** | px | `px-1.5` |
| **Card** | py | `py-4` |
| **Card** | gap | `gap-4` |
| **Card** | sm py | `py-3` |
| **Card** | sm gap | `gap-3` |
| **CardHeader** | px | `px-4` |
| **CardHeader** | sm px | `px-3` |
| **CardHeader** | gap | `gap-1` |
| **CardContent** | px | `px-4` |
| **CardContent** | sm px | `px-3` |
| **CardFooter** | p | `p-4` |
| **CardFooter** | sm p | `p-3` |
| **CardTitle** | text | `text-base` |
| **Card** | text | `text-sm` |
| **Dialog** | p | `p-4` |
| **Dialog** | gap | `gap-4` |
| **Dialog title** | text | `text-base` |
| **Tabs list** | h | `h-8` |
| **Tabs trigger** | px | `px-1.5` |
| **Tabs trigger** | py | `py-0.5` |
| **Toggle** | h default | `h-8` |
| **Toggle** | h sm | `h-7` |
| **Toggle** | h lg | `h-9` |
| **Toggle** | min-w default | `min-w-8` |
| **Toggle** | min-w sm | `min-w-7` |
| **Toggle** | min-w lg | `min-w-9` |
| **Accordion trigger** | py | `py-2.5` |
| **Accordion content** | pb | `pb-2.5` |
| **Alert** | px | `px-2.5` |
| **Alert** | py | `py-2` |
| **Popover** | p | `p-2.5` |
| **Popover** | gap | `gap-2.5` |
| **Dropdown item** | py | `py-1` |
| **Dropdown item** | px | `px-1.5` |
| **Dropdown item** | gap | `gap-1.5` |
| **Dropdown label** | py | `py-1` |
| **Dropdown label** | px | `px-1.5` |
| **Table head** | h | `h-10` |
| **Table head** | px | `px-2` |
| **Table cell** | p | `p-2` |
| **Sheet header** | gap | `gap-0.5` |
| **Sheet title** | text | `text-base` |
| **Sheet close** | position | `top-3 right-3` |
| **Sidebar content** | gap | `gap-0` |
| **Sidebar menu** | gap | `gap-0` |
| **Pagination content** | gap | `gap-0.5` |
| **Pagination ellipsis** | size | `size-8` |
| **Radio group** | gap | `gap-2` |

### Comfortable (based on Vega)

Only differences from Compact are listed:

| Component | Property | Class |
|-----------|----------|-------|
| **Button** | h default | `h-9` |
| **Button** | h sm | `h-8` |
| **Button** | h lg | `h-10` |
| **Button** | icon | `size-9` |
| **Button** | icon-sm | `size-8` |
| **Button** | icon-lg | `size-10` |
| **Input** | h | `h-9` |
| **Input** | file h | `file:h-7` |
| **Select trigger** | h default | `h-9` |
| **Select trigger** | h sm | `h-8` |
| **Select item** | py | `py-1.5` |
| **Select item** | pl | `pl-2` |
| **Select label** | py | `py-1.5` |
| **Select label** | px | `px-2` |
| **Card** | py | `py-5` |
| **Card** | gap | `gap-5` |
| **Card** | sm py | `py-4` |
| **Card** | sm gap | `gap-4` |
| **CardHeader** | px | `px-5` |
| **CardHeader** | sm px | `px-4` |
| **CardContent** | px | `px-5` |
| **CardContent** | sm px | `px-4` |
| **CardFooter** | p | `px-5 py-4` |
| **CardFooter** | sm p | `px-4 py-3` |
| **Dialog** | p | `p-5` |
| **Dialog** | gap | `gap-5` |
| **Tabs list** | h | `h-9` |
| **Tabs trigger** | px | `px-2` |
| **Tabs trigger** | py | `py-1` |
| **Toggle** | h default | `h-9` |
| **Toggle** | h sm | `h-8` |
| **Toggle** | h lg | `h-10` |
| **Toggle** | min-w default | `min-w-9` |
| **Toggle** | min-w sm | `min-w-8` |
| **Toggle** | min-w lg | `min-w-10` |
| **Accordion trigger** | py | `py-4` |
| **Accordion content** | pb | `pb-4` |
| **Alert** | px | `px-4` |
| **Alert** | py | `py-3` |
| **Popover** | p | `p-4` |
| **Popover** | gap | `gap-4` |
| **Dropdown item** | py | `py-1.5` |
| **Dropdown item** | px | `px-2` |
| **Dropdown item** | gap | `gap-2` |
| **Dropdown label** | py | `py-1.5` |
| **Dropdown label** | px | `px-2` |
| **Sheet header** | gap | `gap-1.5` |
| **Sheet close** | position | `top-4 right-4` |
| **Sidebar content** | gap | `gap-2` |
| **Sidebar menu** | gap | `gap-1` |
| **Pagination content** | gap | `gap-1` |
| **Pagination ellipsis** | size | `size-9` |
| **Radio group** | gap | `gap-3` |

### Spacious (based on Maia/Luma)

Only differences from Comfortable are listed:

| Component | Property | Class |
|-----------|----------|-------|
| **Select item** | py | `py-2` |
| **Select item** | pl | `pl-3` |
| **Select label** | py | `py-2.5` |
| **Select label** | px | `px-3` |
| **Card** | py | `py-6` |
| **Card** | gap | `gap-6` |
| **CardHeader** | px | `px-6` |
| **CardContent** | px | `px-6` |
| **CardFooter** | p | `px-6 py-4` |
| **Dialog** | p | `p-6` |
| **Dialog** | gap | `gap-6` |
| **CardHeader** | gap | `gap-2` |
| **Dropdown item** | py | `py-2` |
| **Dropdown item** | px | `px-3` |
| **Dropdown item** | gap | `gap-2.5` |
| **Dropdown label** | py | `py-2.5` |
| **Dropdown label** | px | `px-3` |
| **Table head** | h | `h-12` |
| **Table head** | px | `px-3` |
| **Table cell** | p | `p-3` |

> Spacious shares control heights (h-9) with Comfortable but uses larger container padding (p-6 vs p-5).
> The difference is in containers, menu items, table rows, and radius.

---

## Radius

Radius is independent of density. It's an aesthetic choice set via `--radius` in `globals.css`.

| Name | `--radius` value | Description |
|------|-----------------|-------------|
| **Sharp** | `0.25rem` | Straight or nearly straight corners |
| **Rounded** | `0.625rem` | Soft corners (default shadcn) |
| **Soft** | `1.25rem` | Very rounded corners |
| **Pill** | `1.5rem` | Fully rounded on buttons/badges |

Tailwind v4 derives the full scale from `--radius`:

```
--radius-sm:  calc(var(--radius) * 0.6)
--radius-md:  calc(var(--radius) * 0.8)
--radius-lg:  var(--radius)
--radius-xl:  calc(var(--radius) * 1.4)
--radius-2xl: calc(var(--radius) * 1.8)
--radius-3xl: calc(var(--radius) * 2.2)
--radius-4xl: calc(var(--radius) * 2.6)
```

Components use Tailwind classes (`rounded-md`, `rounded-lg`, etc.) which resolve to this scale. When `--radius` changes, all components update automatically.

### Radius per component (which class to use)

These are the default radius classes. The `--radius` value determines how much curve each produces.

| Component | Container | Inner elements |
|-----------|-----------|---------------|
| **Card** | `rounded-xl` | — |
| **Button** | `rounded-lg` | — |
| **Button xs/sm** | `rounded-[min(var(--radius-md),10px)]` | — |
| **Input** | `rounded-lg` | — |
| **Select trigger** | `rounded-lg` | — |
| **Select content** | `rounded-lg` | — |
| **Select item** | — (no radius) | — |
| **Textarea** | `rounded-lg` | — |
| **Dialog** | — (default) | — |
| **Accordion** | `rounded-lg` | — |
| **Toggle** | `rounded-lg` | — |
| **Alert** | `rounded-lg` | — |
| **Popover** | `rounded-lg` | — |
| **Dropdown content** | `rounded-lg` | — |
| **Dropdown item** | `rounded-md` | — |
| **Tabs list** | — (default) | — |
| **Tabs trigger** | — (default) | — |
| **Badge** | `rounded-4xl` | — |
| **Checkbox** | `rounded-[4px]` | — |
| **Sidebar floating** | `rounded-lg` | — |

> These are the Nova/Compact defaults. For Spacious/Pill styles, the skill should upgrade radius classes proportionally (e.g., `rounded-lg` → `rounded-2xl`, `rounded-xl` → `rounded-4xl`).

---

## Pill Buttons Override

When `pill-buttons: true` in DESIGN.md, the following components get `rounded-full` regardless of `--radius`:

| Component | Normal radius | Pill override |
|-----------|--------------|---------------|
| **Button** (all sizes) | `rounded-lg` / `rounded-[min(...)]` | `rounded-full` |
| **Badge** | `rounded-4xl` | `rounded-full` |
| **Toggle** (all sizes) | `rounded-lg` | `rounded-full` |

**Everything else keeps the base `--radius` scale** — Input, Select, Card, Dialog, Textarea, Accordion, Alert, Popover, Dropdown, Tabs, Sidebar, etc. are NOT affected by this flag.

The codemod checks `pill-buttons` in DESIGN.md section 6. If true, it replaces the radius class on Button/Badge/Toggle with `rounded-full`. This is independent of density — pill buttons can be Compact, Comfortable, or Spacious.

---

## Components NOT Affected by Density

These have identical sizing across all styles — only radius changes:

- **Checkbox** — always `size-4`
- **Radio Group Item** — always `size-4`, `rounded-full`
- **Switch** — minimal size variation, always `rounded-full`
- **Badge** — always `h-5`, `px-2`, `py-0.5`
- **Separator** — no spacing/sizing properties

---

## Codemod Rules

1. **Idempotent**: Running the codemod twice with the same density must produce identical output.
2. **Only touch spacing/sizing classes**: `h-*`, `p-*`, `px-*`, `py-*`, `gap-*`, `size-*`, `min-w-*`, `text-*` (size only), `top-*`, `right-*` (positioning). For `text-*`, only use Tailwind's standard scale (`text-xs`, `text-sm`, `text-base`, `text-lg`, etc.) — never hardcode custom pixel sizes like `text-[13px]` or `text-[11px]`.
3. **Never touch**: colors, hover/focus/active states, accessibility attributes (`aria-*`, `data-*`), `cva()` variant logic, event handlers, conditional rendering.
4. **Preserve `--shadow-card`**: Shadow belongs to the weight system (flat/elevated), not density.
5. **Parent-child radius**: When a parent wraps a child with radius, `parentRadius = childRadius + parentPadding`. Use `calc()` for dynamic cases.
6. **Card horizontal padding**: Card wrapper uses `py-*` only (vertical). Sections (Header, Content, Footer) handle `px-*` individually to avoid double horizontal padding.
