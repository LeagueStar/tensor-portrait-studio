# ✨ Tensor Portrait Studio — Bloom & Brushstroke

[![MIT License](https://img.shields.io/badge/License-MIT-f6d79e.svg)](LICENSE)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-Zero-f4b8d0.svg)](#)
[![HTML5 Canvas](https://img.shields.io/badge/Engine-HTML5_Canvas_2D-ead5bd.svg)](#)
[![Live Demo](https://img.shields.io/badge/Live_Demo-GitHub_Pages-success.svg)](https://leaguestar.github.io/tensor-portrait-studio/)

> **Turn any portrait photograph into a living, structure-tensor impressionist painting with a personalized letter and 1-click standalone HTML gift export.**

**Tensor Portrait Studio** (*Bloom & Brushstroke*) is a zero-dependency, single-file web application that uses real-time computer vision and procedural non-photorealistic rendering (NPR) in vanilla JavaScript. Upload any portrait, calibrate the facial detail focus region, write a custom letter, and export a self-contained `.html` keepsake file that paints itself live in the recipient's browser.

🌐 **Live Studio:** [**https://leaguestar.github.io/tensor-portrait-studio/**](https://leaguestar.github.io/tensor-portrait-studio/)

---

## 🌟 Features

- **🧠 2D Structure-Tensor Flow Field**: Analyses pixel luminance gradients ($g_x, g_y$) and smoothed structure tensors ($S_{11}, S_{22}, S_{12}$) to compute local edge coherence and tangent vectors, guiding every brushstroke organically along facial features, hair waves, and clothing folds.
- **🎨 6-Layer Progressive Painting Engine**: Schedules thousands of curved, bidirectional streamlines across 6 passes (from coarse `r = 5.0` soft under-strokes down to `r = 0.38` high-density facial micro-strokes) over a ~13-second bloom timeline.
- **🎯 Interactive Face Focus Calibrator**: Position and resize the elliptical face region (`x, y, rx, ry`) with a live on-canvas guide ring to concentrate micro-strokes and smoothly blend feathered facial clarity after the painting completes.
- **✨ Atmospheric Post-Processing Pipeline**:
  - **Golden Dust Physics**: Automatically detects bright highlights ($L > 0.62$) outside the face—such as jewelry, embroidery, or catchlights—and spawns 110 twinkling golden and rose dust particles that drift upward.
  - **Additive Bloom & Light Sweep**: Real-time $1/7\text{th}$-scale downsampled additive glow pass, 28 drifting bokeh orbs, a breathing radial aura, subtle pointer parallax, and a periodic diagonal sheen sweep.
- **🎭 Dual Aesthetic Themes**:
  - **Royal Gold & Rose**: Metallic gold-to-rose gradient typography, pill buttons, and a plum-obsidian glass card.
  - **Editorial Ivory**: Minimalist warm ivory serif typography, sharp editorial buttons, and an espresso glass card.
- **🎁 1-Click Standalone Gift HTML Exporter**: Compiles and downloads a clean, single-file `for-<recipient>.html` with the base64 WebP image, calibrated face coordinates, theme, and letter baked in—and all studio editor controls stripped out.
- **🔒 100% Client-Side & Privacy-First**: All image analysis and HTML compilation happen locally in your browser. Ships with a procedurally generated default portrait so no personal photos are stored in the repository.

---

## 🚀 Quick Start

### 1. Run Locally
No build tools, `npm install`, or servers required. Clone the repository and open `index.html` directly in any modern browser:

```bash
git clone [https://github.com/LeagueStar/tensor-portrait-studio.git](https://github.com/LeagueStar/tensor-portrait-studio.git)
cd tensor-portrait-studio
open index.html   # or double-click index.html
```

### 2. Create & Export a Personalized Gift
1. Open the [**Live Studio**](https://leaguestar.github.io/tensor-portrait-studio/) and click **✨ Customize Gift Studio** in the top-right corner.
2. **Upload a Portrait**: Drag and drop a PNG cutout (with a transparent background) or any standard JPG/WebP portrait (toggle **Soft Vignette Cutout** for rectangular photos).
3. **Calibrate Face Focus**: Check **Show Guide Ring** and adjust the sliders (`Center X/Y`, `Radius X/Y`) so the dashed golden ellipse frames the subject's face.
4. **Pick a Theme & Write Your Letter**: Choose **Royal Gold & Rose** or **Editorial Ivory**, customize the bottom caption, button labels, salutation, letter paragraphs, botanical note, and sign-off.
5. **Export**: Click **🎁 Export Standalone Gift HTML**. Send the downloaded `for-<name>.html` file to your recipient—they can open it offline on any phone, tablet, or laptop.

---

## 🧮 How the Math & Rendering Pipeline Works

### 1. Multi-Scale Alpha-Weighted Color Pyramids
To prevent dark halo artifacts around transparent cutouts, RGB channels are pre-multiplied by alpha $A \in [0, 1]$ and convolved with a fast $O(1)$-per-pixel separable sliding-window `boxBlur(src, w, h, r)` at three scales:
- **`sharp`**: Unblurred alpha-normalized color buffer.
- **`mid`**: Radius $r = 2$ blurred buffer.
- **`soft`**: Radius $r = 5$ blurred buffer.

### 2. Structure Tensor & Tangent Flow Field
From the smoothed luminance $L_s = 0.3R + 0.59G + 0.11B$, central differences yield the gradient tensor components:

$$J_{11} = g_x^2, \quad J_{22} = g_y^2, \quad J_{12} = g_x g_y, \quad E = \sqrt{g_x^2 + g_y^2}$$

Each component is smoothed twice with a radius-3 box blur to form $S_{11}, S_{22}, S_{12}$. The local orientation angle $\theta$ and anisotropy coherence $c \in [0, 1]$ are computed as:

$$\theta = \frac{1}{2} \mathrm{atan2}(2 S_{12}, S_{11} - S_{22}), \qquad c = \frac{\sqrt{(S_{11} - S_{22})^2 + 4 S_{12}^2}}{S_{11} + S_{22} + \epsilon}$$

Brushstrokes follow the isophote tangent angle $\alpha = \theta + \frac{\pi}{2}$ in high-coherence regions and drift smoothly toward a soft diagonal ($0.7\text{ rad}$) in flat regions.

### 3. Six-Pass Brushstroke Schedule

| Pass | Brush Radius (`r`) | Max Length (`len`) | Color Source | Color Stop Threshold (`thr`) | Duration (`dur`) | Edge Bias (`edge`) | Target Pool |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **Layer 1** | `5.0` | `8` | `soft` | `0.34` | `1.3s` | `0.00` | Full Silhouette |
| **Layer 2** | `3.0` | `7` | `soft` | `0.24` | `1.6s` | `0.00` | Full Silhouette |
| **Layer 3** | `1.8` | `6` | `mid` | `0.18` | `2.0s` | `0.25` | Full Silhouette |
| **Layer 4** | `1.0` | `4` | `sharp` | `0.12` | `2.8s` | `0.55` | Edge-Weighted |
| **Layer 5** | `0.55` | `3` | `sharp` | `0.10` | `3.4s` | `0.70` | High-Detail Edges |
| **Layer 6** | `0.38` | `2.5` | `sharp` | `0.09` | `1.8s` | `0.40` | Face Focus Ellipse (`density: 3.4`) |

---

## 🌐 Deploying Your Own Studio on GitHub Pages

1. Push `index.html` and `README.md` to your GitHub repository.
2. Go to **Settings** → **Pages** in your repository.
3. Under **Build and deployment**, set **Source** to **Deploy from a branch** and select `main` (or `master`) / `/ (root)`.
4. Click **Save**. Your studio will be live at `https://<your-username>.github.io/tensor-portrait-studio/`.

---

## 📄 License

Distributed under the **MIT License**. Free to use, modify, and share for personal or open-source creative projects.

Created by [**@LeagueStar**](https://github.com/LeagueStar).
