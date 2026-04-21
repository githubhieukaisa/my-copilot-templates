# Skill: Widget Testing Guide for Desktop UX

## Purpose

Test desktop-specific UX behavior: window resize layouts, mouse hover, and keyboard shortcuts.

## 1) Set Desktop Window Size In testWidgets

```dart
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
	testWidgets('renders desktop layout at 1440x900', (tester) async {
		// Simulate a desktop window. This is critical for responsive desktop tests.
		tester.view.devicePixelRatio = 1.0;
		tester.view.physicalSize = const Size(1440, 900);

		// Always reset global test view mutations to avoid cross-test contamination.
		addTearDown(() {
			tester.view.resetPhysicalSize();
			tester.view.resetDevicePixelRatio();
		});

		await tester.pumpWidget(const MaterialApp(home: DesktopShell()));
		await tester.pumpAndSettle();

		// Example assertion: desktop shell should expose a sidebar in wide mode.
		expect(find.byType(DesktopShell), findsOneWidget);
	});
}

class DesktopShell extends StatelessWidget {
	const DesktopShell({super.key});

	@override
	Widget build(BuildContext context) => const Scaffold(body: Placeholder());
}
```

## 2) Simulate Desktop Mouse Hover

```dart
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
	testWidgets('hover changes visual state', (tester) async {
		await tester.pumpWidget(const MaterialApp(home: HoverableTile()));
		await tester.pumpAndSettle();

		final target = find.byKey(const ValueKey<String>('nav-projects'));

		// Create a mouse pointer explicitly (desktop behavior, not touch).
		final mouse = await tester.createGesture(kind: PointerDeviceKind.mouse);
		addTearDown(mouse.removePointer);

		// Add pointer near target, then move onto target to trigger MouseRegion.onEnter.
		await mouse.addPointer(location: const Offset(1, 1));
		await mouse.moveTo(tester.getCenter(target));
		await tester.pump(const Duration(milliseconds: 150));

		// Verify hover state changed.
		expect(find.text('Hovered'), findsOneWidget);
	});
}

class HoverableTile extends StatefulWidget {
	const HoverableTile({super.key});

	@override
	State<HoverableTile> createState() => _HoverableTileState();
}

class _HoverableTileState extends State<HoverableTile> {
	bool _isHovered = false;

	@override
	Widget build(BuildContext context) {
		return Center(
			child: MouseRegion(
				key: const ValueKey<String>('nav-projects'),
				onEnter: (_) => setState(() => _isHovered = true),
				onExit: (_) => setState(() => _isHovered = false),
				child: Text(_isHovered ? 'Hovered' : 'Idle'),
			),
		);
	}
}
```

## 3) Simulate Keyboard Shortcut (Ctrl+S)

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
	testWidgets('Ctrl+S triggers save action', (tester) async {
		await tester.pumpWidget(const MaterialApp(home: ShortcutHarness()));
		await tester.pump();

		// Desktop shortcut simulation in a single call.
		await tester.sendKeyEvent(
			LogicalKeyboardKey.keyS,
			isControlPressed: true,
		);
		await tester.pump();

		expect(find.text('Saved'), findsOneWidget);
	});
}

class SaveIntent extends Intent {
	const SaveIntent();
}

class ShortcutHarness extends StatefulWidget {
	const ShortcutHarness({super.key});

	@override
	State<ShortcutHarness> createState() => _ShortcutHarnessState();
}

class _ShortcutHarnessState extends State<ShortcutHarness> {
	bool _saved = false;

	@override
	Widget build(BuildContext context) {
		return Shortcuts(
			shortcuts: const <ShortcutActivator, Intent>{
				SingleActivator(LogicalKeyboardKey.keyS, control: true): SaveIntent(),
			},
			child: Actions(
				actions: <Type, Action<Intent>>{
					SaveIntent: CallbackAction<SaveIntent>(
						onInvoke: (intent) {
							setState(() => _saved = true);
							return null;
						},
					),
				},
				child: Center(child: Text(_saved ? 'Saved' : 'Not saved')),
			),
		);
	}
}
```

## Desktop Test Checklist

- Set and reset test window size for each responsive test.
- Use mouse pointer gestures for hover tests.
- Cover keyboard shortcuts for primary actions (save/find/new).
- Add regression tests for compact and wide breakpoints.
