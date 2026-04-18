# reset

**When:** The designer wants to start fresh — remove the current theme and go back to shadcn defaults. Custom components (not from shadcn) are preserved.

**Flow:**

1. **Confirm with the designer:**
   > "This will remove DESIGN.md, restore globals.css to shadcn defaults, and delete all papercraft frames from Paper. Custom components will NOT be touched. Continue?"
   > `[ Reset ]  [ Cancel ]`

   If cancelled, stop.

2. **Restore globals.css to shadcn defaults:**
   ```bash
   bunx --bun shadcn@latest init --force
   ```
   This overwrites globals.css with the default shadcn tokens (neutral gray, default radius, no brand scale). It also restores `@theme inline` to defaults.

3. **Remove DESIGN.md:**
   ```bash
   rm DESIGN.md
   ```
   This removes the design system memory. The next `/papercraft init` will start from scratch.

4. **Clean Paper frames** (if Paper MCP is connected):
   - `get_children()` on root → find all frames with `- pc` suffix
   - Delete each one (DESIGN.md - pc, Button - pc, etc.)
   - If Paper MCP is not connected → skip silently, only clean code side

5. **Report:**
   > "Reset complete. globals.css restored to shadcn defaults, DESIGN.md removed, N papercraft frames deleted from Paper. Run `/papercraft init` to start a new theme."

**What this does NOT do:**
- Does NOT delete or modify custom components (anything not from shadcn registry)
- Does NOT uninstall shadcn components — they stay in `components/ui/` with shadcn default styles
- Does NOT modify `package.json`, `tailwind.config`, or `components.json`
- Does NOT touch git history

**What about component spacing?**
After reset, shadcn components in `components/ui/` will still have the density classes from the last codemod (e.g., Compact h-8 values). `shadcn init --force` only restores globals.css, not individual component files. This is fine — the spacing classes are valid Tailwind classes and work without DESIGN.md. The next `/papercraft init` will re-run the codemod with the new density tier.
