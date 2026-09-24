# 🖥️ DeskCraft

> **Interactive Multi-Monitor & Desk Setup Planner / 交互式多屏显示器与工位桌面布局模拟器**

[![Live Demo](https://img.shields.io/badge/🚀%20Live%20Demo-Launch%20DeskCraft-38bdf8?style=for-the-badge)](https://pete2huan9.github.io/DeskCraft/)
[![Version](https://img.shields.io/badge/Version-1.1.0-emerald?style=for-the-badge)](https://github.com/pete2huan9/DeskCraft)
[![License: MIT](https://img.shields.io/badge/License-MIT-amber?style=for-the-badge)](LICENSE)

👉 **[在线免安装体验 / Click Here to Open DeskCraft](https://pete2huan9.github.io/DeskCraft/)**

DeskCraft 是一款高精度交互式多屏显示器与桌面工位布局规划工具。专为开发者、设计师、游戏玩家与远程办公人士打造，在购买硬件前以真实物理比例（1:1）直观模拟显示器尺寸、向内偏转角度、气动机械臂、客制化机械键盘及专业监听音箱，零成本完成工位预演。

DeskCraft is an interactive web-based simulator designed to help you plan, visualize, and optimize your multi-monitor desk setup before buying hardware. Accurately simulate monitor dimensions, aspect ratios, desk sizes, inward swivel angles, gas-spring monitor arms, mechanical keyboards, and studio audio monitors with real-world physical scaling.

---

## ✨ 核心功能 / Core Features

### 🖥️ 显示器与视角仿真
- **正视 + 俯视双视角联动**：左侧正视图展示显示器纵向高度、挂灯光效与近粗远细透视边框；右侧俯视图展示桌面进深（50~80cm）、视距黄金中心与机械臂走向。
- **真实物理比例**：支持 24"、27"、32"、34"、49" 等主流与带鱼屏尺寸，16:9、16:10、21:9、32:9 等屏幕比例，一键切换横竖屏。
- **向内偏转与环抱视角**：支持左右副屏向前偏折（±45°），俯视图定轴刚体纯旋转，屏幕物理长度严格守恒，多屏边缘无缝贴合。
- **支架与机械臂**：支持桌面底座支架与后桌夹式气动机械臂，自动计算关节走线。
- **磁吸对齐**：显示器边缘靠近时自动智能吸附拼接。

### 🪑 桌面与外设生态
- **桌子尺寸定制**：桌面宽度 100~240cm、深度 50~80cm 自由调节，带实时厘米刻度。
- **客制化机械键盘**：内置 61（60%）、87（TKL）、98（980）、104（全尺寸）四种精细真实键帽配列。
- **专业监听音箱**：真实比例立式双单元监听音箱，支持自由拖拽排布。
- **工位辅助外设**：包含无线鼠标、MacBook 笔记本及屏幕挂灯。

### 🎨 交互与使用体验
- **精选工位模板**：内置三屏横向环抱、34"带鱼屏+27"竖屏、双 27" 平放、单 32" 旗舰等多款预设方案，支持一键切换与随时恢复预设。
- **中英双语**：界面支持一键中英文无缝切换（`🌐 EN / 中文`）。
- **画布自由操控**：双画布均支持平滑滚轮缩放、拖拽平移与一键自适应居中。
- **自动保存**：所有布局修改自动持久化保存在本地浏览器（`localStorage`）。

---

## 🚀 快速上手 / Quick Start

### 1. 克隆仓库
```bash
git clone https://github.com/pete2huan9/DeskCraft.git
cd DeskCraft
```

### 2. 安装依赖并启动
```bash
npm install
npm run dev
```
启动后在浏览器中打开 `http://localhost:5173` 即可。

### 3. 构建发布
```bash
npm run build
```
打包文件将输出在 `dist/` 目录中。

---

## 🛠️ 技术栈 / Tech Stack

- **框架**：[Vue 3](https://vuejs.org/) (Composition API, `<script setup>`)
- **构建工具**：[Vite](https://vite.dev/)
- **图形渲染**：SVG 矢量几何计算 + CSS3 硬件加速变换
- **数据存储**：HTML5 `localStorage` 本地离线存储

---

## 📄 开源许可 / License

MIT License © 2026 [pete2huan9](https://github.com/pete2huan9)
