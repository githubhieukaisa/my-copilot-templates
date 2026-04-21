# Skill: FFI C and Rust Integration (Desktop)

## Purpose

Use Dart FFI to call native C/C++/Rust code from Flutter Desktop apps (Windows, macOS, Linux) with low overhead and predictable performance.

## When To Use FFI Instead of MethodChannel

- Choose FFI when you need:
  - High-frequency native calls with low call overhead.
  - Synchronous or CPU-heavy native functions (hashing, compression, codecs, numeric kernels).
  - Tight data interop with native pointers/arrays.
- Choose MethodChannel when you need:
  - Platform service APIs (permissions, OS dialogs, system APIs exposed by platform SDK).
  - Plugin-style integration where native runtime lifecycle is managed by host platform code.

## Production Baseline: Dynamic Library Loader + Typed Bindings

```dart
import 'dart:ffi' as ffi;
import 'dart:io';
import 'dart:typed_data';

import 'package:ffi/ffi.dart';

// Native signature example:
// int32_t crc32_bytes(const uint8_t* data, int32_t length);
typedef _Crc32Native = ffi.Int32 Function(ffi.Pointer<ffi.Uint8>, ffi.Int32);
typedef _Crc32Dart = int Function(ffi.Pointer<ffi.Uint8>, int);

/// Centralized loader for desktop dynamic libraries.
/// Keep naming and candidate paths explicit so failures are debuggable.
final class NativeLibraryLoader {
	const NativeLibraryLoader._();

	static ffi.DynamicLibrary open() {
		// Flutter desktop executables can load side-by-side native binaries.
		final executableDir = File(Platform.resolvedExecutable).parent.path;
		final sep = Platform.pathSeparator;

		// Candidate order is intentional: app-local first, then system resolution.
		final candidates = switch (Platform.operatingSystem) {
			'windows' => <String>[
					'$executableDir${sep}native${sep}mylib.dll',
					'$executableDir${sep}mylib.dll',
					'mylib.dll',
				],
			'linux' => <String>[
					'$executableDir${sep}lib${sep}libmylib.so',
					'$executableDir${sep}libmylib.so',
					'libmylib.so',
				],
			'macos' => <String>[
					'$executableDir${sep}Frameworks${sep}libmylib.dylib',
					'$executableDir${sep}libmylib.dylib',
					'libmylib.dylib',
				],
			_ => throw UnsupportedError('Unsupported OS: ${Platform.operatingSystem}'),
		};

		Object? lastError;
		for (final path in candidates) {
			try {
				return ffi.DynamicLibrary.open(path);
			} catch (error) {
				// Continue trying next candidate path.
				lastError = error;
			}
		}

		throw StateError(
			'Unable to load native library. '
			'Tried: $candidates. Last error: $lastError',
		);
	}
}

/// Strongly-typed binding wrapper.
/// Keep all symbol lookups here, never scattered across UI code.
final class NativeBindings {
	NativeBindings(ffi.DynamicLibrary library)
			: _crc32 = library.lookupFunction<_Crc32Native, _Crc32Dart>('crc32_bytes');

	final _Crc32Dart _crc32;

	int crc32(Uint8List input) {
		// Allocate native memory only for the duration of this call.
		final ptr = calloc<ffi.Uint8>(input.length);
		try {
			final nativeView = ptr.asTypedList(input.length);
			nativeView.setAll(0, input);

			// FFI call is synchronous; avoid calling this directly from UI frame code.
			return _crc32(ptr, input.length);
		} finally {
			calloc.free(ptr);
		}
	}
}
```

## Production Baseline: Offload Heavy FFI Work To Isolate

```dart
import 'dart:isolate';
import 'dart:typed_data';

/// Runs native work off the main isolate to avoid UI frame drops.
Future<int> crc32OffMainThread(Uint8List input) async {
	// Defensive copy prevents accidental mutation races from callers.
	final safeInput = Uint8List.fromList(input);

	return Isolate.run(() {
		// Re-open library inside isolate. DynamicLibrary handles are isolate-local.
		final library = NativeLibraryLoader.open();
		final bindings = NativeBindings(library);

		return bindings.crc32(safeInput);
	});
}
```

## Integration Checklist

- Keep native ABI stable and documented.
- Validate symbol names on all target OS builds.
- Package `.dll` / `.so` / `.dylib` into desktop bundle output.
- Add smoke test for native load + one known deterministic function result.
