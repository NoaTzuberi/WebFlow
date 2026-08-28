# AI Usage Note — CTA & Quote Section Refactor

## Context
While cleaning up the `call-to-action-section` and `quote-section` components (removing redundant wrapper IDs, consolidating CSS), I reviewed my own changes first, then used Claude to do a second-pass review of the same code. This note compares what I caught independently against what the AI review surfaced.

---

## What I spotted on my own

- **`call-to-action-outer-container` ID was redundant** — the wrapper div's styling could live directly on the parent `#call-to-action-section`, so I removed the ID and merged the rules.

- **Duplicate padding shorthand** — `padding: 0 80px 0 80px` was redundant and could be written as `padding: 0 80px`.

- **`quote-content` ID was redundant** — same pattern as the CTA section; merged its rules into `#quote-section` and removed the wrapper ID.

- **Centering logic for the CTA section** — reasoned that `margin: 60px auto` would center the container given the fixed `width: 70vw`.

## What the AI review caught that I missed

- **`margin: auto` on `#quote-section` doing nothing** — I'd carried over the same centering approach from the CTA section, but `#quote-section` has no explicit `width` set. `margin: auto` only centers a block element horizontally when it has a constrained width to center *within*. Without that, the rule is inert. I'd assumed it was "just centering the same way" without checking that the precondition (a set width) was actually met.

- **Fixed `height: 150px` on the CTA section conflicting with `flex-wrap: wrap`** — I'd added `flex-wrap` anticipating responsive behavior, but hadn't connected that a fixed height on the same container would fight against wrapped content at smaller viewports (risk of overflow/squashing). This wasn't something I'd tested at mobile widths.

- **`width: 70vw` + `padding: 0 80px` interaction at small viewports** — flagged that on a narrow screen, the fixed horizontal padding eats a large proportion of the available width, leaving little room for content. I'd only reasoned about the desktop case.
