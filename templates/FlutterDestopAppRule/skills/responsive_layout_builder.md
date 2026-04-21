# Skill: Responsive LayoutBuilder for Desktop Windows

## Purpose

Build desktop layouts that stay stable during continuous window resizing.

## Hard Rule

- Do not use `MediaQuery` as the primary mechanism for basic desktop layout sizing.
- Use `LayoutBuilder` constraints as the source of truth for adaptive desktop composition.

## Production Baseline: Sidebar + Expanded Main + Hover States

```dart
import 'package:flutter/material.dart';

/// Root shell that reacts to actual parent constraints.
class DesktopShell extends StatefulWidget {
	const DesktopShell({super.key});

	@override
	State<DesktopShell> createState() => _DesktopShellState();
}

class _DesktopShellState extends State<DesktopShell> {
	int? _hoveredIndex;
	int _selectedIndex = 0;

	// Static navigation metadata stays const for lower allocation churn.
	static const navItems = <_NavItemData>[
		_NavItemData(label: 'Dashboard', icon: Icons.dashboard_outlined),
		_NavItemData(label: 'Projects', icon: Icons.folder_outlined),
		_NavItemData(label: 'Settings', icon: Icons.settings_outlined),
	];

	@override
	Widget build(BuildContext context) {
		return LayoutBuilder(
			builder: (context, constraints) {
				final width = constraints.maxWidth;

				// Breakpoints are explicit and easy to tune per product.
				final isCompact = width < 960;
				final isWide = width >= 1440;

				// Fixed sidebar width; main content always uses Expanded.
				final sidebarWidth = isCompact ? 76.0 : (isWide ? 280.0 : 220.0);
				final contentPadding = isWide ? 32.0 : 20.0;

				return Row(
					children: [
						SizedBox(
							width: sidebarWidth,
							child: DecoratedBox(
								decoration: const BoxDecoration(
									color: Color(0xFF121820),
									border: Border(
										right: BorderSide(color: Color(0xFF2A3442)),
									),
								),
								child: ListView.builder(
									padding: const EdgeInsets.symmetric(vertical: 12),
									itemCount: navItems.length,
									itemBuilder: (context, index) {
										final item = navItems[index];
										final isHovered = _hoveredIndex == index;
										final isSelected = _selectedIndex == index;

										return MouseRegion(
											cursor: SystemMouseCursors.click,
											onEnter: (_) => setState(() => _hoveredIndex = index),
											onExit: (_) => setState(() => _hoveredIndex = null),
											child: InkWell(
												onTap: () => setState(() => _selectedIndex = index),
												child: AnimatedContainer(
													duration: const Duration(milliseconds: 120),
													margin: const EdgeInsets.symmetric(horizontal: 8, vertical: 4),
													padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 10),
													decoration: BoxDecoration(
														color: isSelected
																? const Color(0xFF1F6FEB)
																: (isHovered ? const Color(0xFF202C3C) : Colors.transparent),
														borderRadius: BorderRadius.circular(10),
													),
													child: Row(
														children: [
															Icon(item.icon, size: 18, color: Colors.white),
															if (!isCompact) ...[
																const SizedBox(width: 10),
																Expanded(
																	child: Text(
																		item.label,
																		maxLines: 1,
																		overflow: TextOverflow.ellipsis,
																		style: const TextStyle(color: Colors.white),
																	),
																),
															],
														],
													),
												),
											),
										);
									},
								),
							),
						),
						Expanded(
							child: Padding(
								padding: EdgeInsets.all(contentPadding),
								child: _MainPane(
									title: navItems[_selectedIndex].label,
									isWide: isWide,
								),
							),
						),
					],
				);
			},
		);
	}
}

class _MainPane extends StatelessWidget {
	const _MainPane({required this.title, required this.isWide});

	final String title;
	final bool isWide;

	@override
	Widget build(BuildContext context) {
		return DecoratedBox(
			decoration: BoxDecoration(
				color: const Color(0xFFF4F6FA),
				borderRadius: BorderRadius.circular(14),
			),
			child: Padding(
				padding: const EdgeInsets.all(16),
				child: Column(
					crossAxisAlignment: CrossAxisAlignment.start,
					children: [
						Text(title, style: Theme.of(context).textTheme.headlineSmall),
						const SizedBox(height: 12),
						Text(isWide ? 'Wide desktop mode' : 'Standard desktop mode'),
						const SizedBox(height: 12),
						const Expanded(
							child: Placeholder(),
						),
					],
				),
			),
		);
	}
}

class _NavItemData {
	const _NavItemData({required this.label, required this.icon});

	final String label;
	final IconData icon;
}
```

## Why This Pattern Works On Desktop

- Constraint-driven adaptation handles infinite resize smoothly.
- Fixed sidebar + `Expanded` content keeps layout predictable.
- `MouseRegion` provides desktop-grade hover feedback.
- Const static metadata reduces rebuild overhead.
