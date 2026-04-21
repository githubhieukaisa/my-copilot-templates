# Flutter Desktop UI Rules

## Objective

Enforce desktop-native interaction quality across Windows, macOS, and Linux.

## Non-Negotiable UX Primitives

- **MUST** provide hover affordances for actionable elements.
- **MUST** provide keyboard shortcuts for high-frequency actions.
- **MUST** provide context menus for item-level commands.
- **MUST** support continuous resize with stable, adaptive layout.
- **MUST** preserve keyboard accessibility and visible focus cues.

## Hover and Pointer Rules

- Wrap clickable regions with `MouseRegion`.
- Reflect hover state in visuals (color, elevation, outline, cursor).
- Use appropriate cursors (`SystemMouseCursors.click`, `text`, `resizeLeftRight`, etc.).

```dart
MouseRegion(
	cursor: SystemMouseCursors.click,
	child: AnimatedContainer(
		duration: const Duration(milliseconds: 120),
		color: isHovering ? hoverColor : baseColor,
		child: child,
	),
)
```

## Keyboard Mapping Rules

- Define top-level `Shortcuts` and `Actions` per screen or app shell.
- Prefer stable desktop bindings:
  - Save: Ctrl/Cmd+S
  - Find: Ctrl/Cmd+F
  - New: Ctrl/Cmd+N
- Avoid collisions with platform-reserved combinations.

```dart
Shortcuts(
	shortcuts: <ShortcutActivator, Intent>{
		const SingleActivator(LogicalKeyboardKey.keyS, control: true): const SaveIntent(),
	},
	child: Actions(
		actions: <Type, Action<Intent>>{SaveIntent: SaveAction(controller)},
		child: child,
	),
)
```

## Context Menu Rules

- Expose row/item actions via right-click or platform menu trigger.
- Keep menu labels action-oriented and consistent with toolbar actions.
- Disable unavailable actions instead of hiding critical actions silently.

## Layout and Resize Rules

- Use `LayoutBuilder` for breakpoint-free adaptive composition.
- Define minimum widths for panes and tables.
- Handle narrow width collapse gracefully (stack/split/condense).
- Configure initial and minimum window sizes with `window_manager`.

```dart
LayoutBuilder(
	builder: (context, constraints) {
		if (constraints.maxWidth >= 1200) return const ThreePaneLayout();
		if (constraints.maxWidth >= 800) return const TwoPaneLayout();
		return const SinglePaneLayout();
	},
)
```

## Feedback and Accessibility

- Show deterministic feedback for loading, success, and failure states.
- Ensure tab traversal order is logical.
- Keep touch-target-equivalent hit areas comfortable for pointer interaction.

## Forbidden Anti-Patterns

- Mobile-only gesture assumptions for critical actions.
- Hover-only actions without keyboard alternative.
- Fixed-size layout that breaks on resize.
- Hidden destructive actions without context confirmation.

## Review Checklist

- Are hover states implemented for actionable controls?
- Are shortcuts/actions present and conflict-checked?
- Are context menus available where expected?
- Does layout remain usable across wide and narrow windows?
- Is focus and keyboard navigation reliable?
