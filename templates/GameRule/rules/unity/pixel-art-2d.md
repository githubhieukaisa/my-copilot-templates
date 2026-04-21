---
paths:
  - "**/*.cs"
  - "**/*.csx"
  - "Assets/**/*.png"
  - "Assets/**/*.psd"
---

# Unity 2D Pixel Art Rules (Strict)

These rules are mandatory when generating Unity 2D pixel-art code, setup steps, or asset import guidance.

## 1. Texture Import Settings (Mandatory)

For all pixel-art textures and sprite sheets:

- `Texture Type`: `Sprite (2D and UI)`
- `Filter Mode`: `Point (no filter)`
- `Compression`: `None`
- `Generate Mip Maps`: `Off`
- `sRGB (Color Texture)`: `On` for color art, `Off` only for authored data textures
- `Max Size`: set to the smallest value that still preserves the source resolution

Additional sprite import constraints:

- `Sprite Mode`: `Single` for standalone sprites, `Multiple` for sheets
- `Pixels Per Unit (PPU)`: keep consistent across all gameplay sprites
- `Mesh Type`: `Full Rect` for pixel art
- `Wrap Mode`: `Clamp` for UI/icons; `Repeat` only when intentional for tiling

Never use bilinear/trilinear filtering for pixel-art gameplay assets.

## 2. Pixel-Perfect Camera Setup (Mandatory)

Use `UnityEngine.U2D.PixelPerfectCamera` on the main gameplay camera.

Required setup:

- `Assets Pixels Per Unit`: must match imported sprite PPU
- `Reference Resolution`: project-defined native pixel resolution (example: `320x180`)
- `Upscale Render Texture`: enabled when preserving integer scaling is required
- `Pixel Snapping`: enabled for moving sprites that must remain crisp
- `Crop Frame`: configure intentionally (`X`, `Y`, or both) based on target framing strategy

Do not mix inconsistent PPU values between camera and sprites.

## 3. Sprite Sheet Handling (Mandatory)

When using sprite sheets:

- Set `Sprite Mode = Multiple` and slice with exact pixel cell sizes.
- Use consistent pivot rules per category (characters, props, UI).
- Keep padding/spacing consistent with source export settings.
- Validate every slice index and naming convention before animation hookup.

Atlas and packing constraints for pixel art:

- Disable any packing option that rotates sprites.
- Disable tight packing when it introduces pixel distortion.
- Preserve point filtering and no-compression expectations in atlases.

## 4. Movement and Rendering Alignment Rules

- Prefer integer-aligned positions for pixel-perfect gameplay presentation.
- Avoid sub-pixel camera drift for orthographic pixel-art scenes.
- Keep world scale and PPU relationships documented and consistent.

## 5. Prohibited Practices

- No default texture imports for pixel art.
- No compressed pixel-art textures.
- No filtered pixel-art textures.
- No mixed PPU values without an explicit technical reason.

## 6. Example Setup Code

```csharp
using UnityEngine;
using UnityEngine.U2D;

[RequireComponent(typeof(Camera), typeof(PixelPerfectCamera))]
public sealed class PixelPerfectBootstrap : MonoBehaviour
{
    [SerializeField] private int _assetsPixelsPerUnit = 16;
    [SerializeField] private Vector2Int _referenceResolution = new Vector2Int(320, 180);

    private PixelPerfectCamera _pixelPerfectCamera;

    private void Awake()
    {
        _pixelPerfectCamera = GetComponent<PixelPerfectCamera>();
        Debug.Assert(_pixelPerfectCamera != null, "PixelPerfectCamera is required.", this);

        _pixelPerfectCamera.assetsPPU = _assetsPixelsPerUnit;
        _pixelPerfectCamera.refResolutionX = _referenceResolution.x;
        _pixelPerfectCamera.refResolutionY = _referenceResolution.y;
        _pixelPerfectCamera.upscaleRT = true;
        _pixelPerfectCamera.pixelSnapping = true;
    }
}
```
