# Design Skills — VibeKit Agent

You are building for a user who expects premium quality. Follow these principles for every UI/frontend task.

## Core Philosophy
- Simplicity is architecture, not style. If it feels complicated, the design is wrong.
- Every element must justify its existence. If it can be removed without losing meaning, remove it.
- The best interface is the one the user never notices.

## Visual Hierarchy
- Every screen has ONE primary action. Make it unmissable.
- Secondary actions support — they never compete.
- If everything is bold, nothing is bold.
- Visual weight must match functional importance.

## Spacing & Layout
- Whitespace is a feature, not empty space. Breathing room feels premium.
- Every element sits on a consistent grid. No exceptions.
- Mobile-first. Design for thumbs, then cursors.
- Every screen must feel intentional at every viewport size.

## Typography
- Clear hierarchy with type sizes. Max 2-3 font weights per screen.
- The type should feel calm, not chaotic.

## Color
- Use color with restraint and purpose.
- Colors guide attention, not scatter it.
- Sufficient contrast for accessibility.

## Components & Consistency
- Same component styled identically everywhere it appears.
- Interactive elements must be obviously interactive.
- Account for all states: default, hover, focus, disabled, loading, error, empty.

## Empty & Loading States
- Every screen should look intentional with no data.
- Guide the user toward their first action.
- Use skeleton screens or subtle loaders — the app should feel alive while waiting.

## Responsive Design
- Fluid across all viewport sizes — not just functional at breakpoints.
- Touch targets sized for thumbs on mobile (min 44px).
- No screen size should feel like an afterthought.

## Accessibility
- Keyboard navigation, focus states, ARIA labels.
- Color contrast ratios meet WCAG AA minimum.
- Screen reader flow makes sense.

## The Jobs Filter
For every element, ask:
1. "Would a user need to be told this exists?" — if yes, redesign until obvious
2. "Can this be removed without losing meaning?" — if yes, remove it
3. "Does this feel inevitable?" — if no, it's not done

## Implementation Rules
- Use a design system / consistent tokens (colors, spacing, radii, shadows).
- No hardcoded values — reference variables/tokens.
- Transitions should feel like physics, not decoration.
- The app should feel like it respects the user's time and attention.
