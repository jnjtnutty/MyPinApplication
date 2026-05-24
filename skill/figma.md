# Figma MCP Skill

**Platform-agnostic guidance** for pulling design specs, downloading assets, and verifying visual fidelity across all platforms (iOS, Web, Backend UI, etc.)

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

### Key spec mappings for different platforms

| Figma Property | SwiftUI | React/CSS | HTML |
|---|---|---|---|
| `fontSize` / `fontWeight` | `.font(.system(size:, weight:))` | `fontSize`, `fontWeight` CSS | `<span style="font-size:; font-weight:;">` |
| `lineHeight` | `.lineSpacing()` | `lineHeight` CSS | `line-height` CSS |
| `borderRadius` | `RoundedRectangle(cornerRadius:)` | `borderRadius` CSS | `border-radius` CSS |
| `padding` | `.padding(EdgeInsets(...))` | `padding` CSS | `padding` CSS |
| `gap` | `HStack(spacing:)` / `VStack(spacing:)` | `gap` CSS | CSS flexbox `gap` |
| `fill` hex color | `Color(red:, green:, blue:)` | `color: #XXXXXX` | `color: #XXXXXX` |
| `fill` rgba | `.opacity()` modifier | `rgba(r,g,b,a)` CSS | `rgba(r,g,b,a)` CSS |
| `fill` gradient | `LinearGradient(stops:...)` | `background: linear-gradient` | `background: linear-gradient` |
| `fill` IMAGE type | Download via `download_figma_images` | Download via `download_figma_images` | Download via `download_figma_images` |
| `strokeWeight` + `stroke` | `.strokeBorder(..., lineWidth:)` | `border` CSS | `border` CSS |
| `effects.boxShadow` | `.shadow(color:, radius:, x:, y:)` | `box-shadow` CSS | `box-shadow` CSS |

### Font substitution

When Figma specifies a font not bundled in your app/project, use the system default font and preserve the exact `fontSize` and `fontWeight` from Figma.
- iOS: SF Pro (system font)
- Web: System font stack

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

## Integrating assets into your project

**iOS**: Add to `Assets.xcassets/` folder
**Web**: Add to `public/images/` or `src/assets/` folder
**Backend**: Add to `static/images/` or appropriate asset folder

### iOS: PNG/JPG images

Add the PNG to `Assets.xcassets/<name>.imageset/` with `Contents.json`

### iOS: SVG icons (template rendering)

Add SVG to `Assets.xcassets/<name>.imageset/` with `Contents.json` and `template-rendering-intent: template`

### Web: PNG/JPG images

Place in `public/images/` or `src/assets/` and reference in code

### Web: SVG icons

Place in `src/icons/` or `public/svgs/` and import/use with color tinting support

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

| Figma | SwiftUI | React/CSS | HTML |
|---|---|---|---|
| `#FFFFFF` | `.white` | `#FFFFFF` or `rgb(255,255,255)` | `#FFFFFF` |
| `#14120F` | `Color(red: 20/255, green: 18/255, blue: 15/255)` | `#14120F` or `rgb(20,18,15)` | `#14120F` |
| `rgba(255,255,255,0.8)` | `.white.opacity(0.8)` | `rgba(255,255,255,0.8)` | `rgba(255,255,255,0.8)` |
| `rgba(0,0,0,0.18)` | `.black.opacity(0.18)` | `rgba(0,0,0,0.18)` | `rgba(0,0,0,0.18)` |

**Conversion formula**: Hex to RGB: split into pairs, convert each to decimal, divide by 255 for SwiftUI.
Example: `#14` = 20 decimal → `20/255` in SwiftUI (or just use `0x14` = 20)
