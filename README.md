# 🖥️ DeskCraft Pro

[![Live Demo](https://img.shields.io/badge/🚀%20Live%20Demo-Launch%20DeskCraft-blue?style=for-the-badge)](https://pete2huan9.github.io/DeskCraft/)

👉 **[Click Here to Open DeskCraft Pro in your Browser](https://pete2huan9.github.io/DeskCraft/)**

> **Interactive Multi-Monitor & Desk Setup Planner / 交互式桌面显示器布局模拟器**

DeskCraft Pro is an interactive web-based simulator designed to help developers, creators, and remote workers plan, visualize, and optimize their desk setups before buying hardware. Accurately simulate monitor dimensions, aspect ratios, desk sizes, gas-spring monitor arms, mechanical keyboards, mice, and studio audio monitors with real-world physical scaling.

---

## ✨ Features

### 🖥️ Display & Ergonomics
- **Real-World Proportions**: Accurate centimeter-to-pixel scaling for standard and ultrawide screen sizes (24", 27", 32", 34", 49", etc.).
- **Flexible Aspect Ratios**: Full support for 16:9, 16:10, 21:9 ultrawide, and 32:9 super ultrawide displays.
- **Orientation Control**: Toggle between Landscape and Portrait modes with automatic dimension inversion.
- **Mounts & Arms**: Choose between standard heavy-duty desktop stands with telescoping poles or articulated gas-spring monitor arms with desk edge clamps.
- **Screen Light Bars**: Optional downward ambient light bar mounted atop monitors.
- **Magnetic Snapping**: Monitors automatically snap edge-to-edge for multi-monitor alignment.

### 🪑 Desk & Workspace Customization
- **Adjustable Desk Width**: Configure desks from 100 cm up to 240 cm with live centimeter scale markings.
- **Studio Audio Monitors**: Flanking left/right active speakers with customizable position.
- **Mechanical Keyboards**: Select between 4 distinct keyboard layouts:
  - **61 Keys** (Compact 60% layout, ~29 cm)
  - **87 Keys** (Tenkeyless TKL, ~36 cm)
  - **98 Keys** (Compact 1800 with numpad, ~38.5 cm)
  - **104 Keys** (Full-size standard, ~44.5 cm)
- **Independent Mouse**: Ergonomic wireless mouse with independent dragging and positioning.
- **Laptop Stand**: Optional laptop (e.g. MacBook Pro) integration on the desk.

### 🎨 User Experience & Controls
- **Bilingual Interface**: Seamless toggle between **English** and **中文** (`🌐 EN / 中文`).
- **Camera Navigation**: Smooth mouse wheel zooming, middle-click panning, and canvas navigation.
- **Mobile Touch Ready**: Full touch support with pinch-to-zoom and one-finger canvas dragging.
- **Instant Presets**: One-click layout templates (Classic 32" Elevated, Dual 27" Productivity, 34" Ultrawide + Vertical Coder, Triple 27" Surround, Laptop Docking).
- **Auto-Save**: Automatic layout persistence in your browser via `localStorage`.

---

## 🚀 Quick Start

### Prerequisites
- [Node.js](https://nodejs.org/) (v18.0 or newer recommended)
- `npm` (comes with Node.js)

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
Open your browser and navigate to `http://localhost:5173`.

### 4. Build for production
```bash
npm run build
```
Production assets will be generated in the `dist/` directory.

---

## 🌐 Deploying Online (Free Hosting)

### Option 1: Vercel (Recommended - Fastest)
1. Go to [Vercel](https://vercel.com/) and log in with your GitHub account.
2. Click **"Add New Project"** and select `DeskCraft`.
3. Vercel will automatically detect Vite. Click **Deploy**.
4. You will get a live public URL (e.g., `https://deskcraft.vercel.app`) in seconds!

### Option 2: GitHub Pages
1. In your GitHub repository, go to **Settings** → **Pages**.
2. Under **Build and deployment** → **Source**, choose **GitHub Actions**.
3. Select the standard **Static HTML** or **Vite** starter workflow to deploy automatically on every push.

---

## 🛠️ Tech Stack

- **Framework**: [Vue 3](https://vuejs.org/) (Composition API, `<script setup>`)
- **Build Tool**: [Vite](https://vite.dev/)
- **Graphics**: SVG vector rendering for mounts, arms, and keycaps
- **Storage**: Client-side HTML5 `localStorage`

---

## 📄 License

MIT License © 2026 [pete2huan9](https://github.com/pete2huan9)
