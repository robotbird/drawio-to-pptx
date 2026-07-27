# Drawio → Native Editable PPTX

> Convert a **draw.io** (`.drawio`) diagram into a **native, editable** PowerPoint (`.pptx`) — every box becomes a real autoshape / text box, and every arrow becomes a real connector with arrowheads. The slide is **not a flat image**: move boxes, recolor them, edit text, and re-route arrows in PowerPoint / Keynote / WPS.

![License](https://img.shields.io/github/license/robotbird/drawio-to-pptx?color=blue)
![Python](https://img.shields.io/badge/python-3.x-3776AB?logo=python&logoColor=white)
![python-pptx](https://img.shields.io/badge/python--pptx-required-1f425f)
![lxml](https://img.shields.io/badge/lxml-required-green)

把 `.drawio` 转成**原生可编辑**的 PPT —— 每个框是真正的自选图形，每条箭头是真正的连线。不是一张图片，所有元素都能在 PPT 里改。

---

## ✨ Why this exists

Most "drawio to PPT" routes either export a **PNG/SVG and paste it** (one dumb picture) or render a screenshot. You lose the whole point of PowerPoint: **editing**. This tool walks the draw.io XML and rebuilds the diagram from **real PowerPoint primitives**:

| draw.io element | Becomes in PPT |
|---|---|
| Rectangle / `rounded=1` | Rectangle / Rounded-Rectangle autoshape |
| `ellipse`, `rhombus`, `cylinder3`, `triangle`, `hexagon`, `cloud`, `parallelogram`, `trapezoid` | Matching PPT autoshape (`OVAL`, `DIAMOND`, `CAN`, …) |
| `text`-only style | Text box (no fill / line) |
| Nested containers / swimlanes | Absolute coordinates resolved, drawn as backdrop box |
| Edge with source + target | Straight **line connector** (clipped to box edges) |
| Edge with waypoints | Open **polyline** (editable freeform) |
| `startArrow` / `endArrow` | Native arrowheads on the connector |
| Edge `value` | Label box at the edge midpoint |

Colors (`fillColor`, `strokeColor`, `fontColor`), line width, font size/style, alignment, and the slide background are all preserved.

## 📦 Prerequisites

Python 3 with `python-pptx` and `lxml`:

```bash
python3 -c "import pptx, lxml" 2>/dev/null && echo OK || pip install python-pptx lxml
```

## 🚀 Quick start

```bash
python3 scripts/drawio_to_pptx.py input.drawio -o output.pptx
```

That's the whole job for most diagrams. Defaults: **16:9** slide, fit-to-slide, 0.3" margin.

Multiple draw.io `<diagram>` pages → multiple slides. Compressed `.drawio` files (base64 + deflate payload) are decoded automatically.

### Bump text size for dense diagrams

```bash
python3 scripts/drawio_to_pptx.py big_arch.drawio -o out.pptx --font-scale 1.2
```

### Let it pick the orientation

```bash
python3 scripts/drawio_to_pptx.py flow.drawio -o out.pptx --landscape   # 16:9 if wide, else 4:3
```

## ⚙️ Options

| Flag | Default | Purpose |
|---|---|---|
| `-o / --output` | required | Output `.pptx` path |
| `--size` | `16:9` | Slide preset: `16:9`, `4:3`, `16:10`, `A4-landscape` |
| `--landscape` | off | Auto-pick `16:9` if the diagram is wider than tall, else `4:3` |
| `--margin` | `0.3` | Slide margin in inches |
| `--font-scale` | `1.0` | Multiply all text sizes (use `1.1`–`1.3` for dense diagrams) |
| `--min-font` / `--max-font` | `7` / `40` | Clamp text pt so labels stay readable |

## ✅ Verify the shapes are native (not an image)

```bash
python3 -c "from pptx import Presentation; s=Presentation('output.pptx').slides[0]; print(len(list(s.shapes)),'shapes')"
```

A high shape count (one per draw.io cell) means it converted correctly; a single `PICTURE` shape would mean it fell back to an image — that does **not** happen with this script.

## ⚠️ Known limitations (being honest)

- **Branded stencils** (`shape=mxgraph.aws4.*`, Cisco / Kubernetes icons, AI logos) render as a labeled rectangle — draw.io's stencil artwork can't be reproduced as a native PPT shape. The label and color are preserved.
- **Gradients** (`gradientColor`) collapse to a solid fill using `fillColor`. **Shadows** (`shadow=1`) are dropped (PPT shapes can re-add a shadow manually).
- **Orthogonal routing** is preserved when edges carry waypoints (rendered as a polyline); edges with only source/target are drawn as a clean straight line clipped to the box edges — re-route in PPT if a different path is wanted.
- **Font**: Chinese / CJK labels render via PowerPoint's font fallback. Bump `--font-scale` up if text looks small on a dense slide.

## 🧩 Also a Claude Code skill

This repo is structured as a [Claude Code](https://claude.com/claude-code) **skill** (see [`SKILL.md`](SKILL.md)). If you use Claude Code, point it at the repo and ask, e.g. *"把 drawio 转成原生 ppt"* — it will resolve deps, run the converter, and verify the output for you.

## 📁 Project structure

```
.
├── scripts/
│   └── drawio_to_pptx.py     # the converter (run it; no need to read unless extending)
├── references/
│   └── style-mapping.md      # full drawio style-key → PPT mapping table + extension guide
├── SKILL.md                  # Claude Code skill definition
└── README.md
```

For the complete style-key → PPT mapping table, edge-routing details, and how to **extend the mappings** (new shapes, text effects, edge behaviors, slide presets), see [references/style-mapping.md](references/style-mapping.md).

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

MIT License — see the [LICENSE](LICENSE) file for details.

## 👤 Author

**叶鹏** · [@robotbird](https://github.com/robotbird)
