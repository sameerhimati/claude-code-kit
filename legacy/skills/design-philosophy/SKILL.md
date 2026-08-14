---
name: design-philosophy
description: Interactive design system builder. Asks about product vision, then generates a comprehensive design.md file.
disable-model-invocation: false
allowed-tools: Task, Read, Write, Edit, Glob, Grep, Bash
---

# Design Philosophy: $ARGUMENTS

## Phase 1: Discovery (Ask Before Designing)

Ask the user these questions sequentially. Adapt follow-ups based on answers.

### Product Identity
1. **What is this product?** One sentence.
2. **Who uses it?** Age range, technical comfort, context of use (mobile on-the-go, desktop focused work, etc.)
3. **What emotion should the product evoke?** (calm/trustworthy/playful/powerful/minimal/premium)
4. **Name 2-3 products whose design you admire.** What specifically about them?

### Visual Direction
5. **Color philosophy:**
   - Monochrome (like Apple, Holding, Learnt)
   - Branded with 1-2 accent colors
   - Rich/colorful palette
6. **Typography feel:** Modern sans-serif / Classic serif / Monospace-technical / Mixed
7. **Density:** Spacious and breathing / Balanced / Dense and information-rich
8. **Dark mode?** Primary dark / Primary light / Both from day one

### Interaction & Components
9. **Animation philosophy:** None (instant) / Subtle (micro-interactions) / Expressive (page transitions, delights)
10. **Forms and inputs:** Minimal (few fields, smart defaults) / Comprehensive (power users)
11. **Mobile priority?** Mobile-first / Desktop-first / Equal weight

### Constraints
12. **Accessibility level:** AA minimum / AAA target
13. **Must support:** (screen readers, keyboard nav, reduced motion, etc.)
14. **Existing brand assets?** (logos, colors, fonts already chosen)

---

## Phase 2: Generate design.md

Based on answers, generate `design.md` in the project root:

```markdown
# Design System — [Project Name]

## Philosophy
> [One paragraph capturing the design soul of this product]

### Core Principles
1. **[Principle]** — [Why this matters for this product]
2. **[Principle]** — [Why]
3. **[Principle]** — [Why]

## Color System

### Palette
| Token | Value | Usage |
|-------|-------|-------|
| `--color-bg-primary` | [hex] | Main background |
| `--color-bg-secondary` | [hex] | Cards, sections |
| `--color-text-primary` | [hex] | Body text |
| `--color-text-secondary` | [hex] | Secondary text, labels |
| `--color-accent` | [hex] | CTAs, links, active states |
| `--color-success` | [hex] | Confirmations |
| `--color-warning` | [hex] | Warnings |
| `--color-error` | [hex] | Errors, destructive actions |
| `--color-border` | [hex] | Dividers, borders |

### Dark Mode
[Mapping or note about dark mode approach]

## Typography

| Role | Font | Weight | Size | Line Height |
|------|------|--------|------|-------------|
| Display | [font] | [wt] | [size] | [lh] |
| Heading 1 | [font] | [wt] | [size] | [lh] |
| Heading 2 | [font] | [wt] | [size] | [lh] |
| Body | [font] | [wt] | [size] | [lh] |
| Caption | [font] | [wt] | [size] | [lh] |
| Code | [font] | [wt] | [size] | [lh] |

## Spacing & Layout

- **Grid:** [8pt / 4pt] base unit
- **Container max-width:** [value]
- **Spacing scale:** [4, 8, 12, 16, 24, 32, 48, 64, 96]
- **Border radius:** [none / subtle (4px) / rounded (8px) / pill]

## Components

### Buttons
- **Primary:** [description]
- **Secondary:** [description]
- **Ghost/Text:** [description]
- **Destructive:** [description]

### Cards
[Default card style, elevation, padding]

### Forms
[Input style, label placement, error display]

### Navigation
[Nav pattern, mobile behavior]

## Interaction & Motion

- **Transitions:** [duration, easing]
- **Hover states:** [behavior]
- **Loading states:** [skeleton / spinner / shimmer]
- **Page transitions:** [none / fade / slide]

## Accessibility

- **Contrast ratio:** [AA / AAA]
- **Focus indicators:** [style]
- **Reduced motion:** [respected via prefers-reduced-motion]
- **Screen reader:** [considerations]

## Do's and Don'ts

### Do
- [pattern to follow]

### Don't
- [anti-pattern to avoid]
```

## Rules
- Always ask Phase 1 questions first. Never assume the design direction.
- If the project already has a design.md, read it first and ask what to update.
- Reference the user's stated inspirations in the output.
- Be specific with hex values, font names, and pixel sizes — not vague.
- If the user has existing Tailwind config or CSS variables, align with them.
