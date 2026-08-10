# Text Truncation & Ellipsis Guidelines

These guidelines apply to all MORYX web projects. They ensure that truncated text is always accessible, especially on touch devices where hover-based tooltips are not available.

## Core Principle

> Don’t cut off text without providing a way for users to view it

This follows Material Design 3 guidelines: ellipsis indicates hidden content, so the UI must provide a mechanism to access it.

## Patterns

### 1. Text Wrapping (simplest)

Remove truncation and let text wrap naturally. Use `overflow-wrap: break-word` to prevent overflow on long unbroken strings.

```scss
.label {
  overflow-wrap: break-word;
}
```

**When to use:** Headers, titles, side sheet titles, card labels, property values — anywhere the container is wide enough to accommodate wrapping (typically 150px+).
This is the simplest fix and should be the default choice unless there is a specific layout reason to truncate.

### 2. Expansion Panel

Use `mat-expansion-panel` to show a summary with ellipsis in the collapsed header. Expanding reveals the full content.

**When to use:** Lists of items where each item has a summary and details (e.g. references, methods, log messages, skill chips).

### 3. Tap-to-Expand / Click-to-Show

Truncated text that expands inline or opens a dialog/drawer on tap.

**When to use:** Cards or compact list items where wrapping would break the layout grid, but a detail view or dialog is available.

### 4. Detail View Reveal

Truncated text in a list/sidebar where selecting the item shows the full text in a detail area. A decision should be made whether to truncate in the list or allow wrapping, based importance of the information.

**When to use:** Master-detail layouts (e.g. recipe sidebar, variant overview).

## Patterns to Avoid

### matTooltip alone

`matTooltip` requires hover and does not work on touch devices. It can be added as a supplementary enhancement but never as the sole reveal mechanism.

```html
<!-- NOT sufficient on its own -->
<span class="truncate" [matTooltip]="fullText">{{ fullText }}</span>

<!-- OK as enhancement alongside another mechanism -->
<span class="truncate" [matTooltip]="fullText" (click)="showDetails()">{{ fullText }}</span>
```

### Hardcoded character truncation in TypeScript

Do not truncate text in component/service code using `substring()`, `slice()`, or similar. This creates a data loss point that CSS cannot override.

```typescript
// BAD — do not do this
const label = name.length > 28 ? name.slice(0, 28) + '...' : name;

// BAD — do not do this
productName = productName.substring(0, maxLength - 4) + "...";
```

If truncation is needed, use CSS `text-overflow: ellipsis` so the browser handles it responsively based on available space.

### Template-level slice truncation

Same problem as TypeScript truncation — hardcoded character limits in templates.

```html
<!-- BAD -->
{{ text.length > 28 ? (text | slice: 0:28) + '...' : text }}

<!-- GOOD — use CSS truncation if needed, with a reveal mechanism -->
<span class="truncate">{{ text }}</span>
```

### Truncation without any reveal mechanism

CSS ellipsis on an element where there is no way (tap, click, expand, navigate) to see the full text.

```scss
// BAD — if no reveal mechanism exists
.cell-name {
  width: 150px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

## Decision Table

| Context | Truncation OK? | Action |
|---|---|---|
| Text inside an expansion panel header | Yes | Expansion reveals full text |
| Detail view shows full text when item is selected | Yes | Selection reveals full text |
| Container is wide enough for wrapping (150px+) | No | Remove truncation, allow wrapping |
| Grid/card layout where wrapping breaks alignment | Yes | Add tap-to-expand or navigate-to-details |
| None of the above | No | Rethink the layout — full text must be accessible |

## References

- [M3 Text Truncation](https://m3.material.io/foundations/writing/text-truncation) — Material Design 3 guidance on when and how to truncate text
- [M3 Layout](https://m3.material.io/) — touch target sizes, responsive layout guidance
- [Angular Component Interaction](https://angular.dev/guide/components/inputs) — passing full data to detail components
- [WCAG 1.4.4 Resize Text](https://www.w3.org/WAI/WCAG21/Understanding/resize-text.html) — text must remain accessible when truncated
