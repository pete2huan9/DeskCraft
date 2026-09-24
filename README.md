# 🖥️ DeskCraft

<p align="center">
  <img src="public/favicon.svg" alt="DeskCraft Logo" width="80" height="80" />
</p>

<p align="center">
  <strong>Interactive Multi-Monitor & Desk Setup Planner</strong><br>
  <em>交互式多屏显示器与工位桌面布局规划模拟器</em>
</p>

<p align="center">
  <a href="https://pete2huan9.github.io/DeskCraft/"><img src="https://img.shields.io/badge/🚀%20Live%20Demo-Launch%20DeskCraft-38bdf8?style=for-the-badge" alt="Live Demo" /></a>
  <img src="https://img.shields.io/badge/Version-1.1.0-emerald?style=for-the-badge" alt="Version 1.1.0" />
  <img src="https://img.shields.io/badge/Vue-3.5-42b883?style=for-the-badge&logo=vue.js" alt="Vue 3" />
  <img src="https://img.shields.io/badge/Vite-8.0-646cff?style=for-the-badge&logo=vite" alt="Vite" />
  <img src="https://img.shields.io/badge/License-MIT-amber?style=for-the-badge" alt="License MIT" />
</p>

---

👉 **[🌐 Click Here to Open DeskCraft in your Browser / 在线免安装直接体验](https://pete2huan9.github.io/DeskCraft/)**

DeskCraft is a high-precision, interactive web application engineered to help developers, designers, gamers, and remote professionals plan, visualize, and optimize their multi-monitor desk setups before purchasing hardware. Accurately simulate monitor dimensions, aspect ratios, desk dimensions, inward swivel angles, gas-spring monitor arms, mechanical keyboards, mice, and studio audio monitors with true real-world physical scaling (15 px/inch, 1 cm ≈ 5.9 px).

DeskCraft 是一款高精度交互式多屏与桌面布局规划工具，专为开发者、设计师、游戏玩家与居家办公人士打造。支持真实物理比例仿真、正视与俯视双视角实时联动、向内偏转黄金环抱视角、机械臂线径、多种客制化机械键盘及专业监听音箱，助您在采购硬件前零成本完成人体工学工位预演。

---

## 🌟 What's New in v1.1 / 1.1 版本重磅更新

- **⚡ Dual Synchronized Viewports (正视 + 俯视双视角并排联动)**:
  - **Front Orthographic View (正视图)**: Displays vertical clearance, screen heights, light bar glow fixtures, and real-time trigonometric projected width ($W \cdot \cos\theta$) with near-thin/far-thick tapered bezel borders depicting 3D inward slant.
  - **Top-Down Plan View (俯视图)**: Visualizes real desk depth (50–80 cm), viewing sweet-spots, light cones, and multi-joint monitor arm reach.
- **📐 Invariant Screen Physics & Rigid-Body Rotation (定轴刚体旋转与屏幕长度严格守恒)**:
  - In the Top View, monitor screens undergo pure 2D Euclidean rigid-body rotation around their hinge joints, guaranteeing that physical screen length ($W$) remains **100% constant** at any swivel angle (0° to 45°).
  - Seamless zero-gap corner snapping between flat and swiveled displays in panoramic triple-monitor wrap setups.
- **⌨️ Realistic Mechanical Keyboards with Keycap Layouts (全仿真客制化机械键盘)**:
  - Vector SVG keycap mapping with distinct alphanumeric wells, function rows, navigation clusters, and arrow keys:
    - **61 Keys (60% Compact)**: Minimalist layout (~29.5 cm)
    - **87 Keys (TKL)**: Tenkeyless layout with isolated arrow & navigation block (~36 cm)
    - **98 Keys (980 / 1800-compact)**: High-efficiency layout with numpad (~39 cm)
    - **104 Keys (Full Size)**: Standard 100% layout (~44.5 cm)
  - Color-accented Esc and Enter keycaps.
- **🔊 Upright Studio Audio Monitors (扶正专业监听音箱)**:
  - Studio speakers aligned upright with dual acoustic drivers (tweeter & woofer) and realistic 13×22 cm proportions.
- **🏎️ Automatic 3D Z-Ordering (基于俯视纵深的智能前后层级遮挡)**:
  - Monitor and accessory stacking orders in the Front View are automatically computed from physical depth coordinates in the Top View.
- **🔄 Dedicated Preset Restore System (工位模板一键恢复系统)**:
  - Presets Modal with instant "Restore Preset" (恢复预设) and reset controls.
- **🎨 Right-Docked Dedicated Control Dock (分体独立侧边控制坞)**:
  - Clean layout separating tools and forms into an independent right rail with zero viewport overlap.

---

## ✨ Core Features / 核心功能

### 🖥️ Display Simulation (显示器与人体工学)
- **Real-World Proportions**: Accurate centimeter-to-pixel scaling for standard and ultrawide sizes (24", 27", 32", 34", 49", etc.).
- **Flexible Aspect Ratios**: 16:9, 16:10, 21:9 ultrawide, 32:9 super ultrawide, and custom ratios.
- **Orientation Switch**: One-click 90° toggle between Landscape and Portrait with auto-swapped physical dimensions.
- **Inward Swivel & Wrap**: Smooth swivel angle adjustment (±45°) with quick chips (⤹ 30°, Flat, 30° ⤸) and sightline sweet-spot cone.
- **Mounts & Arms**: Choose between heavy-duty desktop stands or multi-articulated gas-spring monitor arms with rear desk clamps.
- **Screen Light Bars**: Optional downward ambient light bar mounted atop monitors.
- **Magnetic Snapping**: Monitors automatically snap edge-to-edge for multi-monitor alignment.

### 🪑 Desk & Workspace Accessories (桌面与外设生态)
- **Configurable Desk Dimensions**: Adjust desk width from 100 cm to 240 cm and depth from 50 cm to 80 cm with live millimeter/centimeter scale markings.
- **Acoustic Studio Monitors**: Independent left and right studio speakers with front/top drag positioning.
- **Mechanical Keyboards**: 4 layouts (61 / 87 / 98 / 104 keys) with realistic keycaps and drag placement.
- **Ergonomic Mouse**: Precision wireless mouse with realistic proportions and independent drag.
- **MacBook / Laptop**: Optional laptop on desk with true lid and trackpad scale.

### 🎨 User Experience & Controls (交互体验)
- **Bilingual Interface**: One-click toggle between **English** and **中文** (`🌐 EN / 中文`).
- **Smooth Canvas Controls**: Independent zoom and pan for both Front View and Top View with mouse wheel and drag gestures.
- **Mobile & Touch Friendly**: Responsive layout with touch drag support.
- **Curated Presets**:
  - 🏎️ **Triple 27" Panoramic Wrap** (190×75 cm, 30° inward slant, zero speaker overlap)
  - 👨‍💻 **34" Ultrawide + 27" Vertical** (160×75 cm, programmer favorite with dual gas arms)
  - 💼 **Dual 27" Productivity** (160×75 cm, side-by-side flat setup)
  - 👑 **Classic 32" Elevated Flagship** (140×75 cm, single central arm)
  - 🚀 **49" Super Ultrawide Command Center** (180×75 cm, 32:9 curved behemoth)
- **Offline Auto-Save**: Seamless persistence to `localStorage`.

---

## 📐 Coordinate & Physical Scaling Specification / 物理几何规范

DeskCraft uses standard real-world metric conversions:
$$\text{Pixel Scale} = 15\text{ px / inch}$$
$$\text{CM to PX Ratio} = \frac{15}{2.54} \approx 5.9055\text{ px / cm}$$

- **Horizontal Orthographic Projection (正交水平投影视宽)**:
  $$W_{\text{projected}} = W_{\text{physical}} \cdot \cos(\theta)$$
- **Top View Euclidean Length (俯视图物理实长，严格守恒)**:
  $$L = \sqrt{\Delta x^2 + \Delta y^2} = W_{\text{physical}}$$
- **Sweet Spot Ergonomic Distance (人眼黄金视距中心)**:
  Centered horizontally at $\frac{\text{DeskWidth}}{2}$, situated $14\text{ cm}$ in front of the desk edge.

---

## 🚀 Quick Start / 快速上手

### Prerequisites
- [Node.js](https://nodejs.org/) (v18.0 or newer recommended, Node 20/22/24 verified)
- `npm`

### 1. Clone the repository
```bash
git clone https://github.com/pete2huan9/DeskCraft.git
cd DeskCraft
```

### 2. Install dependencies
```bash
npm install
```

### 3. Run development server
```bash
npm run dev
```
Open your browser at `http://localhost:5173`.

### 4. Build for production
```bash
npm run build
```
Production bundles will be compiled to `dist/`.

---

## 🌐 Online Deployment / 在线部署指南

### Option 1: GitHub Pages (Automatic via GitHub Actions)
DeskCraft includes a pre-configured automated CI/CD workflow at `.github/workflows/deploy.yml`.
1. Push your repository to GitHub.
2. In your repo settings, go to **Settings** → **Pages** → **Source**, and select **GitHub Actions**.
3. Every push to `main` will build and publish automatically to `https://<username>.github.io/<repo>/`.

### Option 2: Vercel (Instant)
1. Import `DeskCraft` on [Vercel](https://vercel.com/).
2. Vercel automatically detects Vite. Click **Deploy**.

---

## 🛠️ Tech Stack / 技术栈

- **Core**: [Vue 3](https://vuejs.org/) (`<script setup>`, Composition API)
- **Bundler**: [Vite 8](https://vite.dev/)
- **Graphics**: SVG 2.0 Vector Graphics + HTML5 Canvas transforms
- **Storage**: Client-Side HTML5 `localStorage` (No server or database required, 100% private)
- **Styling**: Pure CSS with Custom Properties, GPU-accelerated transforms (`translate3d`, `rotate`)

---

## 📄 License

MIT License © 2026 [pete2huan9](https://github.com/pete2huan9)
