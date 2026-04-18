# Papercraft — Design memory between canvas and code

Paper and your code don't know about each other. By now, they should.

Papercraft is a skill that connects both sides through a `DESIGN.MD` — a file that acts as shared memory between canvas and code.

You put together a moodboard in Paper (reference images, drawn components, whatever you want), the agent reads it and extracts the full design system. If you drew components directly, it pulls the exact values. It writes everything to code and renders your components with your theme directly on the canvas. From there, everything stays consistent.

## Commands

- `/papercraft init` — reads your moodboard and generates the full design system — both the `DESIGN.MD` file in code and a visual frame in Paper with all tokens and colors
- `/papercraft design.md` — if you modified something in the `DESIGN.MD` frame in Paper (a color, typography, a note with instructions), this command reads those changes and updates the code and components
- `/papercraft preview` — your shadcn/ui components rendered on the canvas with your theme applied
- `/papercraft component` / `/papercraft screen` — components or full screens in Paper, in code, or both. If it already exists, the agent applies only your changes
- `/papercraft reset` — start from scratch

## Prerequisites

- [Paper Desktop](https://paper.design) running with MCP server
- [shadcn/ui](https://ui.shadcn.com) initialized in your project
- Install required skills:
  ```
  npx skills add oklch-skill
  npx skills add shadcn
  ```

## Install

```
npx skills add jhankevi/my-skills/papercraft
```

## Stack

Claude Code · Paper MCP · shadcn/ui · Next.js · Tailwind v4 · OKLCH
