# Angular Material Dialog Guidelines

## Button Type

All buttons inside a dialog should use `type="button"`. HTML buttons default to `type="submit"`, which causes the browser to treat Enter keypresses as form submissions. This can re-trigger the button that opened the dialog.

```html
<!-- Bad -->
<button mat-button mat-dialog-close>Cancel</button>
<button mat-flat-button [mat-dialog-close]="result">Save</button>

<!-- Good -->
<button mat-button mat-dialog-close type="button">Cancel</button>
<button mat-flat-button [mat-dialog-close]="result" type="button">Save</button>
```

## Focus Management

Every dialog should have a `cdkFocusInitial` attribute on exactly one element. This ensures focus is moved inside the dialog when it opens, preventing Enter keypresses from leaking back to the trigger button behind the overlay.

Place `cdkFocusInitial` on the canceling/closing button by default. For creation/input dialogs, it may be placed on the first input field instead.

```html
<div mat-dialog-actions align="end">
  <button mat-button mat-dialog-close type="button" cdkFocusInitial>Cancel</button>
  <button mat-flat-button [mat-dialog-close]="result" type="button">Save</button>
</div>
```

## Dialog Result Pattern

Use `[mat-dialog-close]` to return results and handle them via `afterClosed()` at the call site. Avoid passing callback functions through dialog data.

```typescript
// Good - declarative result
const dialogRef = this.dialog.open(MyDialog, { data: inputData });
dialogRef.afterClosed().subscribe((result) => {
  if (result) {
    // handle confirmed result
  }
});
```

```html
<!-- In the dialog template -->
<button mat-button mat-dialog-close type="button" cdkFocusInitial>Cancel</button>
<button mat-flat-button [mat-dialog-close]="result()" type="button">Confirm</button>
```

## References

- [Angular Material Dialog - Official Documentation](https://material.angular.dev/components/dialog/overview)
- [Angular CDK Accessibility - Focus Trap & cdkFocusInitial](https://material.angular.dev/cdk/a11y/overview)
- [HTML button type attribute - MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button#type)
