# Geography Viz Kit · 地理天文可视化套件

An open-source collection of 12 interactive 3D visualizations for Chinese K-12 and university geography and astronomy education. Every module is a single self-contained HTML file — no installation, no build step, no server required. Open in any modern browser.

[English](#english) · [中文](#chinese)

---

## Live Modules

| # | File | Topic (中文) | Grade Level |
|---|------|-------------|-------------|
| 01 | `01-earth-rotation.html` | 地球自转 · 地方时 · 时区 · 晨昏线 | 高中必修一 |
| 02 | `02-earth-revolution.html` | 地球公转 · 黄赤交角 · 季节成因 | 高中必修一 |
| 03 | `03-solar-altitude.html` | 正午太阳高度角 H = 90° − \|φ − δ\| | 高中必修一 |
| 04 | `04-shadow.html` | 影子方向与长度 L = h / tan(H) | 高中必修一 |
| 05 | `05-solar-path.html` | 太阳视运动路径 · 天球穹顶 | 高中必修一 |
| 06 | `06-day-length.html` | 昼夜长短变化 · 全年热力图 | 高中必修一 |
| 07 | `07-atmospheric-circulation.html` | 气压带风带 · 科里奥利力 | 高中必修一 |
| 08 | `08-ocean-currents.html` | 世界洋流 · 动画粒子流 | 高中必修二 |
| 09 | `09-celestial-coordinates.html` | 赤道坐标系 · 赤经赤纬 · 黄道 | 大学天文 |
| 10 | `10-kepler-orbits.html` | 开普勒三定律 · 等面积扫描 | 大学天文 |
| 11 | `11-solar-system-scale.html` | 太阳系比例模型 · 对数/线性切换 | 初中/大学 |
| 12 | `12-magnetosphere.html` | 磁层与太阳风 · 极光 · Kp指数 | 大学/竞赛 |

---

## Quick Start

```bash
# Clone
git clone https://github.com/edu-ai-builders/geography-viz-kit.git
cd geography-viz-kit

# Option 1: open any file directly (most modules work fine)
open index.html

# Option 2: local server (needed only if browser blocks file:// CDN)
python3 -m http.server 8080
# then visit http://localhost:8080
```

No npm, no webpack, no dependencies to install.

---

<a name="english"></a>

## English Overview

### Design Principles

Each module follows the same three-panel layout:

```
┌─────────────────┬──────────────────────┬─────────────────┐
│  Control Panel  │   3D Canvas (Three.js)│   Data Readout  │
│  (parameters,   │   interactive, drag   │   (live numbers │
│   layer toggles,│   to rotate, zoom,    │   synced every  │
│   knowledge pts)│   slider-driven)      │   frame)        │
└─────────────────┴──────────────────────┴─────────────────┘
```

Every module includes:
- **知识点 dropdown** — switches the in-canvas explanation text
- **Layer toggles** — show/hide individual visual elements
- **Right-panel live readout** — dual-coding: same concept shown numerically and spatially
- **Productive struggle hook** — a slider whose extreme value creates a deliberate cognitive conflict (e.g., drag axial tilt to 0° and watch seasons disappear)

### Photorealistic Earth Rendering

Modules that show Earth load five textures progressively from CDN (Earth renders immediately with a placeholder; textures upgrade as they arrive):

| Layer | Source |
|-------|--------|
| Day map | three.js r160 CDN (`earth_atmos_2048.jpg`) |
| Specular (ocean mask) | three.js r160 CDN (`earth_specular_2048.jpg`) |
| Normal map (terrain relief) | three.js r160 CDN (`earth_normal_2048.jpg`) |
| Night lights (city glow) | vasturiano/three-globe CDN |
| Cloud layer (5 MB, async) | turban/webgl-earth (loads last, no blocking) |

Custom GLSL shader handles day/night blending, normal-mapped terrain, specular ocean sheen, and city light glow in the terminator band.

### Technology Stack

- **Three.js r160** via importmap from `cdn.jsdelivr.net` — no bundler
- **EffectComposer + UnrealBloomPass** — post-processing bloom on all modules
- **CSS2DRenderer** — HTML label overlays, unaffected by WebGL post-processing
- **CatmullRomCurve3** — smooth ocean current flowlines from lat/lon waypoints
- Single HTML file per module, ~20–55 KB each

---

<a name="chinese"></a>

## 中文说明

### 项目介绍

这是一套为中国 K-12 及大学地理/天文教育设计的交互式 3D 可视化工具，共 12 个模块。

**核心设计理念：**
- **双重编码**：每个概念同时用 3D 空间画面和精确数值呈现
- **认知冲突触发**：每个模块至少有一个"生产性挣扎"钩子，让学生通过意外结果重建正确认知
- **渐进式披露**：贴图、说明文字、图层分步加载，不阻塞操作
- **零门槛使用**：单文件 HTML，浏览器直接打开，无需安装任何环境

### 课程覆盖

| 阶段 | 模块 | 对应课程 |
|------|------|---------|
| Phase 1 | 01–05 | 人教版高中地理必修一（地球运动基础） |
| Phase 2 | 06–08 | 高中地理必修一/二（大气与海洋） |
| Phase 3 | 09–12 | 大学天文学基础 / 地理竞赛 |

完整课程知识体系见 [`知识体系.md`](知识体系.md)，包含每个模块的：知识点、年级、核心误区、关键公式、认知冲突触发器。

### 教学亮点举例

- **模块02**：把黄赤交角滑杆拖到 0°，季节消失 → 学生理解"距离不是季节的原因"
- **模块06**：全年×全纬度昼长热力图，一张图看清极昼极夜分布规律
- **模块10**：开普勒等面积扇形实时高亮，近日点/远日点速度差异肉眼可见
- **模块12**：调 Kp 指数，极光圈向低纬度扩展 → 理解磁暴与极光的关系

### 适用对象

- 高中地理老师：课堂演示直接用
- 备考学生：直观理解高频考点
- 初中学生：地球运动、太阳系比例等入门概念
- 大学天文/地理专业：坐标系、开普勒定律、磁层物理

---

## File Structure

```
geography-viz-kit/
├── index.html                    # Landing page (module card grid)
├── 01-earth-rotation.html        # Earth rotation & time zones
├── 02-earth-revolution.html      # Earth revolution & seasons
├── 03-solar-altitude.html        # Solar noon altitude
├── 04-shadow.html                # Shadow direction & length
├── 05-solar-path.html            # Solar path diagrams (celestial dome)
├── 06-day-length.html            # Day length heatmap
├── 07-atmospheric-circulation.html  # Pressure belts & wind bands
├── 08-ocean-currents.html        # Ocean currents (animated particles)
├── 09-celestial-coordinates.html # Equatorial coordinate system
├── 10-kepler-orbits.html         # Kepler's three laws
├── 11-solar-system-scale.html    # Solar system scale model
├── 12-magnetosphere.html         # Magnetosphere & aurora
├── 知识体系.md                   # Full Chinese curriculum knowledge map
└── CLAUDE.md                     # Developer guide (architecture, conventions)
```

---

## Contributing

The shared Earth rendering block (textures, GLSL shader, cloud mesh, bloom setup) is defined in `01-earth-rotation.html` between the markers:
```
// ╔══ BEGIN: SHARED EARTH RENDER BLOCK v2 ══╗
// ╚══ END: SHARED EARTH RENDER BLOCK v2   ══╝
```
If you improve the Earth shader, copy the updated block to all other Earth-showing modules (`02`, `03`, `04`, `05`, `06`).

See `CLAUDE.md` for full architecture notes, coordinate conventions, UV mapping details, and the Module 02 two-group axial tilt pattern.

---

## License

MIT — free to use in classrooms, schools, and self-study. Attribution appreciated.
