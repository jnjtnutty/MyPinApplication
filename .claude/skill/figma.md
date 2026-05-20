mcp# Figma MCP Skill

How to use the Figma MCP server (`mcp__figma__get_figma_data`, `mcp__figma__download_figma_images`) in this project.

## Setup

The Figma MCP server is pre-configured. No authentication step is needed at runtime.

## Getting design data

### From a Figma URL

URLs follow this pattern:

```
https://www.figma.com/design/<fileKey>/<fileName>?node-id=<nodeId>
```

- `fileKey` — alphanumeric string (e.g. `RbxZunWIJGyF1YrWcgE54q`)
- `nodeId` — use format `X-Y` or `X:Y` (e.g. `5-2` or `5:2`)

### Fetch layout, typography, colors, spacing

```
mcp__figma__get_figma_data
  fileKey: "RbxZunWIJGyF1YrWcgE54q"
  nodeId: "5-2"
```

This returns:
- **Node tree** — hierarchy of frames, text, images, vectors
- **Layout styles** — dimensions, padding, gap, alignment, position
- **Text styles** — fontFamily, fontWeight, fontSize, lineHeight, textAlign
- **Fill styles** — hex colors, gradients, opacity, image references
- **Stroke/border** — strokeWeight, borderRadius
- **Effects** — shadows, blur

### Key spec mappings for SwiftUI

| Figma Property | SwiftUI Equivalent |
|---|---|
| `fontSize` / `fontWeight` | `.font(.system(size:, weight:))` |
| `lineHeight` | `.lineSpacing()` or custom |
| `borderRadius` | `RoundedRectangle(cornerRadius:)` |
| `padding` | `.padding(EdgeInsets(...))` |
| `gap` | `HStack(spacing:)` / `VStack(spacing:)` |
| `fill` hex color | `Color(red:, green:, blue:)` — convert hex to 0-1 range |
| `fill` rgba | `.opacity()` modifier |
| `fill` gradient | `LinearGradient(stops:...)` |
| `fill` IMAGE type | Download via `download_figma_images` |
| `strokeWeight` + `stroke` | `.strokeBorder(..., lineWidth:)` |
| `effects.boxShadow` | `.shadow(color:, radius:, x:, y:)` |
| `alignItems: center` | `.alignment: .center` or frame alignment |

### Font substitution

When Figma specifies a font not bundled in the app (e.g. Instrument Sans), use SF Pro (system font) and preserve the exact `fontSize` and `fontWeight` from Figma.

## Downloading images

### Background / raster images

For nodes with `imageRef` fills:

```
mcp__figma__download_figma_images
  fileKey: "RbxZunWIJGyF1YrWcgE54q"
  nodes: [
    {
      "nodeId": "5-3",
      "fileName": "HeroCompass.png",
      "imageRef": "b486ba1eaf69a6e4015cfd1dc2619275158a5ddd"
    }
  ]
  localPath: "design-refs"
  pngScale: 2
```

Then add the PNG to `iOS-MCP-xcode26/Assets.xcassets/<name>.imageset/` with a `Contents.json`:

```json
{
  "images": [
    { "idiom": "universal", "scale": "1x" },
    { "idiom": "universal", "scale": "2x" },
    { "filename": "image.png", "idiom": "universal", "scale": "3x" }
  ],
  "info": { "author": "xcode", "version": 1 }
}
```

### SVG icons

For nodes of type `IMAGE-SVG`:

```
mcp__figma__download_figma_images
  fileKey: "RbxZunWIJGyF1YrWcgE54q"
  nodes: [
    { "nodeId": "5-13", "fileName": "icon_mail.svg" },
    { "nodeId": "5-18", "fileName": "icon_lock.svg" }
  ]
  localPath: "design-refs"
```

Then add each SVG to `iOS-MCP-xcode26/Assets.xcassets/<name>.imageset/` with this `Contents.json`:

```json
{
  "images": [
    { "filename": "icon_name.svg", "idiom": "universal" }
  ],
  "info": { "author": "xcode", "version": 1 },
  "properties": {
    "preserves-vector-representation": true,
    "template-rendering-intent": "template"
  }
}
```

The `template` rendering intent allows `foregroundStyle` to tint the icon color at runtime.

### Exporting a frame as reference screenshot

Export the full design frame at 2x for visual comparison:

```
mcp__figma__download_figma_images
  fileKey: "RbxZunWIJGyF1YrWcgE54q"
  nodes: [{ "nodeId": "5-2", "fileName": "login_figma.png" }]
  localPath: "design-refs"
  pngScale: 2
```

## Visual comparison workflow

1. Export Figma frame → `design-refs/<screen>_figma.png`
2. Build & run on simulator → screenshot to `design-refs/<screen>_impl.png`
3. Use `Read` tool on both PNGs to visually compare
4. Walk through each element: position, size, color, typography, radius, stroke, shadow, spacing, alignment
5. Score ≥ 80% required before marking task done

## Common color conversions

| Figma | SwiftUI |
|---|---|
| `#FFFFFF` | `.white` |
| `#14120F` | `Color(red: 20/255, green: 18/255, blue: 15/255)` |
| `rgba(255,255,255,0.8)` | `.white.opacity(0.8)` |
| `rgba(0,0,0,0.18)` | `.black.opacity(0.18)` |

Hex to decimal: split into pairs, convert each to decimal, divide by 255.
Example: `#14` = 20 decimal → `20/255`.
