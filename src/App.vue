<template>
  <div class="app-container" :class="{ 'is-mobile': isMobile }">
    <!-- Top Navigation Bar -->
    <header class="app-header">
      <div class="brand">
        <span class="brand-icon">🖥️</span>
        <div class="brand-text">
          <span class="brand-title">DeskCraft</span>
          <span class="brand-subtitle">{{ t('brandSubtitle') }}</span>
        </div>
      </div>

      <div class="header-actions">
        <!-- View Mode Switcher (Dual vs Front vs Top) -->
        <div class="view-mode-pill">
          <button 
            class="view-mode-btn" 
            :class="{ active: viewLayoutMode === 'dual' }" 
            @click="setLayoutMode('dual')"
            :title="t('dualViewTitle')"
          >
            <span class="btn-icon">🪟📐</span>
            <span class="btn-text">{{ t('dualView') }}</span>
          </button>
          <button 
            class="view-mode-btn" 
            :class="{ active: viewLayoutMode === 'front' }" 
            @click="setLayoutMode('front')"
            :title="t('frontViewTitle')"
          >
            <span class="btn-icon">🪟</span>
            <span class="btn-text">{{ t('frontViewOnly') }}</span>
          </button>
          <button 
            class="view-mode-btn" 
            :class="{ active: viewLayoutMode === 'top' }" 
            @click="setLayoutMode('top')"
            :title="t('topViewTitle')"
          >
            <span class="btn-icon">📐</span>
            <span class="btn-text">{{ t('topViewOnly') }}</span>
          </button>
        </div>

        <!-- Language Switcher Button -->
        <button class="icon-btn lang-btn" @click="toggleLanguage" :title="lang === 'zh' ? 'Switch to English' : '切换为中文'">
          <span class="btn-icon">🌐</span>
          <span class="btn-text font-bold">{{ lang === 'zh' ? 'EN' : '中文' }}</span>
        </button>

        <!-- Quick Desk Size Chip (Width x Depth) -->
        <div class="quick-chip desk-chip" @click="openDeskSettings">
          <span class="chip-icon">🪑</span>
          <span class="chip-label">{{ desk.widthCm }} × {{ deskDepthCm }} cm</span>
        </div>

        <!-- Presets Button -->
        <button class="icon-btn" @click="showPresetsModal = true" :title="t('presetsTitle')">
          <span class="btn-icon">⚡</span>
          <span class="btn-text">{{ t('presets') }}</span>
        </button>

        <!-- Accessories Toggle Button -->
        <button class="icon-btn" @click="toggleAccessoriesModal" :title="t('accessoriesTitle')">
          <span class="btn-icon">⌨️</span>
          <span class="btn-text">{{ t('accessories') }}</span>
        </button>

        <!-- Fit View Button -->
        <button class="icon-btn" @click="fitToScreen" :title="t('fitScreenTitle')">
          <span class="btn-icon">⤢</span>
          <span class="btn-text">{{ t('fitScreen') }}</span>
        </button>

        <!-- Desktop Add Monitor Button -->
        <button v-if="!isMobile" class="primary-btn" @click="openAddSheet">
          <span>➕ {{ t('addMonitor') }}</span>
        </button>
      </div>
    </header>

    <!-- Main Workspace Layout: Canvas on the Left + Dedicated Tools Sidebar on the Right -->
    <main class="app-main-layout">
      <!-- Canvas Workspace: Side-by-Side Dual Viewports or Single Viewport -->
      <div class="canvas-workspace" :class="`view-${viewLayoutMode}`">
      
      <!-- ================= 1. LEFT PANE: FRONT ELEVATION VIEW (主视图 - 大) ================= -->
      <div 
        v-show="viewLayoutMode === 'dual' || viewLayoutMode === 'front'"
        class="canvas-pane front-pane"
        ref="frontCanvasRef"
        @wheel.prevent="onFrontWheel"
        @pointerdown="onFrontCanvasPointerDown"
        @pointermove="onFrontCanvasPointerMove"
        @pointerup="onFrontCanvasPointerUp"
        @pointercancel="onFrontCanvasPointerUp"
      >
        <!-- Front Viewport Tag Header -->
        <div class="pane-header-tag">
          <div class="pane-tag-title">
            <span class="pane-icon">🪟</span>
            <span class="pane-name">{{ t('frontView') }}</span>
            <span class="pane-badge">{{ t('frontElevationBadge') }}</span>
          </div>
          <div class="pane-tag-actions">
            <span class="pane-zoom-text">{{ Math.round(frontZoom * 100) }}%</span>
            <button class="pane-mini-btn" @click.stop="fitFrontToScreen" :title="t('fitScreenTitle')">⤢</button>
          </div>
        </div>

        <!-- Background Grid -->
        <div class="grid-overlay"></div>

        <!-- Front World Transform Layer (Zoom & Pan) -->
        <div 
          class="world-layer"
          :style="{
            transform: `translate(${frontPan.x}px, ${frontPan.y}px) scale(${frontZoom})`,
            transformOrigin: '0 0'
          }"
        >
          <!-- Monitor Stands & Arms (Rendered behind monitors and desk) -->
          <svg class="stands-svg-layer">
            <defs>
              <linearGradient id="armMetalGrad" x1="0%" y1="0%" x2="100%" y2="100%">
                <stop offset="0%" stop-color="#475569" />
                <stop offset="50%" stop-color="#1e293b" />
                <stop offset="100%" stop-color="#0f172a" />
              </linearGradient>
              <linearGradient id="poleMetalGrad" x1="0%" y1="0%" x2="100%" y2="0%">
                <stop offset="0%" stop-color="#64748b" />
                <stop offset="50%" stop-color="#334155" />
                <stop offset="100%" stop-color="#1e293b" />
              </linearGradient>
            </defs>

            <!-- Monitor Stand / Arm Elements -->
            <g v-for="m in monitors" :key="'stand-' + m.id">
              <!-- 1. Monitor Arm (气压机械臂) -->
              <template v-if="m.standType === 'arm'">
                <path 
                  :d="calculateArmPath(m)" 
                  stroke="url(#armMetalGrad)" 
                  stroke-width="12" 
                  stroke-linecap="round" 
                  stroke-linejoin="round"
                  fill="none" 
                />
                <path 
                  :d="calculateArmPath(m)" 
                  stroke="#64748b" 
                  stroke-width="2" 
                  stroke-linecap="round" 
                  fill="none" 
                  opacity="0.6"
                />
                <!-- Elbow Joint Circles -->
                <circle :cx="getArmJoints(m).jointX" :cy="getArmJoints(m).jointY" r="10" fill="#334155" stroke="#64748b" stroke-width="2" />
                <circle :cx="getArmJoints(m).jointX" :cy="getArmJoints(m).jointY" r="4" fill="#94a3b8" />
                <!-- Desk Edge Clamp -->
                <rect 
                  :x="getArmJoints(m).clampX - 12" 
                  :y="desk.y - 4" 
                  width="24" 
                  height="16" 
                  rx="3" 
                  fill="#1e293b" 
                  stroke="#475569" 
                  stroke-width="2" 
                />
              </template>

              <!-- 2. Desktop Stand (原装立柱底座) -->
              <template v-else-if="m.standType === 'stand' || !m.standType">
                <!-- Telescoping Column down to desk surface -->
                <rect 
                  :x="m.x + getProjectedWidthPx(m) / 2 - 9" 
                  :y="m.y + m.heightPx * 0.45" 
                  width="18" 
                  :height="Math.max(12, desk.y - (m.y + m.heightPx * 0.45))" 
                  fill="url(#poleMetalGrad)"
                  rx="2"
                />
                <!-- Desk Base Plate on desk tabletop -->
                <path 
                  :d="`M ${m.x + getProjectedWidthPx(m) / 2 - 48} ${desk.y} 
                       L ${m.x + getProjectedWidthPx(m) / 2 - 38} ${desk.y - 7} 
                       L ${m.x + getProjectedWidthPx(m) / 2 + 38} ${desk.y - 7} 
                       L ${m.x + getProjectedWidthPx(m) / 2 + 48} ${desk.y} Z`" 
                  fill="#1e293b" 
                  stroke="#475569" 
                  stroke-width="1.5"
                />
              </template>
            </g>
          </svg>

          <!-- Desk Visualization in Front View -->
          <div class="desk-group">
            <!-- Desk Tabletop Surface -->
            <div 
              class="desk-surface"
              :style="{
                width: deskWidthPx + 'px',
                height: deskHeightPx + 'px',
                left: desk.x + 'px',
                top: desk.y + 'px'
              }"
            >
              <!-- Desk Dimension Arrow/Ruler -->
              <div class="desk-ruler">
                <span class="ruler-line left-line"></span>
                <span class="ruler-text">◄ {{ effectiveDeskWidthCm }} cm ({{ t('deskDepth') }} {{ deskDepthCm }} cm) ►</span>
                <span class="ruler-line right-line"></span>
              </div>
            </div>

            <!-- Desk Legs -->
            <div 
              class="desk-leg left-leg"
              :style="{
                left: (desk.x + 35) + 'px',
                top: (desk.y + deskHeightPx) + 'px'
              }"
            ></div>
            <div 
              class="desk-leg right-leg"
              :style="{
                left: (desk.x + deskWidthPx - 50) + 'px',
                top: (desk.y + deskHeightPx) + 'px'
              }"
            ></div>
          </div>

          <!-- ================= Front View Table Accessories ================= -->

          <!-- 1. Keyboard (Separate & Draggable) -->
          <div 
            v-if="accessories.keyboard.enabled"
            class="table-accessory keyboard-item table-accessory-draggable"
            :class="{ 'is-selected-accessory': selectedAccessory === 'keyboard' }"
            :style="{
              left: (desk.x + accessories.keyboard.xPx) + 'px',
              top: (desk.y - keyboardHeightPx) + 'px',
              width: keyboardWidthPx + 'px',
              height: keyboardHeightPx + 'px',
              zIndex: getFrontDepthZIndex('keyboard')
            }"
            @pointerdown.stop="startAccessoryDrag('keyboard', $event)"
            :title="`${t('keyboard')} (${currentKeyboardLayout.labelEn || currentKeyboardLayout.value}) · ${t('dragHint')}`"
          >
            <div class="mech-keyboard-unit" :class="`layout-${accessories.keyboard.layout}`">
              <div class="keyboard-layout-tag">{{ accessories.keyboard.layout }}</div>
              <div class="keyboard-chassis">
                <div class="key-strip top-row">
                  <span class="cap cap-esc"></span>
                  <span class="cap" v-for="n in (accessories.keyboard.layout === '61' ? 9 : 11)" :key="'f' + n"></span>
                  <span v-if="accessories.keyboard.layout !== '61'" class="cap cap-accent"></span>
                  <span v-if="accessories.keyboard.layout === '104' || accessories.keyboard.layout === '98'" class="cap cap-num-f" v-for="x in 3" :key="'nf' + x"></span>
                </div>
                <div class="key-strip mid-row">
                  <span class="cap" v-for="n in (accessories.keyboard.layout === '61' ? 11 : 13)" :key="'q' + n"></span>
                  <span v-if="accessories.keyboard.layout === '104' || accessories.keyboard.layout === '98'" class="cap cap-numpad" v-for="x in 4" :key="'nq' + x"></span>
                </div>
                <div class="key-strip mid-row">
                  <span class="cap" v-for="n in (accessories.keyboard.layout === '61' ? 10 : 12)" :key="'a' + n"></span>
                  <span class="cap cap-enter"></span>
                  <span v-if="accessories.keyboard.layout === '104' || accessories.keyboard.layout === '98'" class="cap cap-numpad" v-for="x in 4" :key="'na' + x"></span>
                </div>
                <div class="key-strip bottom-row">
                  <span class="cap cap-ctrl"></span>
                  <span class="cap cap-space"></span>
                  <template v-if="accessories.keyboard.layout !== '61'">
                    <span class="cap cap-arrow" v-for="a in 3" :key="'ar' + a"></span>
                  </template>
                  <template v-if="accessories.keyboard.layout === '104' || accessories.keyboard.layout === '98'">
                    <span class="cap cap-numpad-zero"></span>
                    <span class="cap cap-enter"></span>
                  </template>
                </div>
              </div>
              <div class="keyboard-rgb-glow"></div>
            </div>
          </div>

          <!-- 2. Mouse (Separate & Draggable) -->
          <div 
            v-if="accessories.mouse.enabled"
            class="table-accessory mouse-item table-accessory-draggable"
            :class="{ 'is-selected-accessory': selectedAccessory === 'mouse' }"
            :style="{
              left: (desk.x + accessories.mouse.xPx) + 'px',
              top: (desk.y - mouseHeightPx) + 'px',
              width: mouseWidthPx + 'px',
              height: mouseHeightPx + 'px',
              zIndex: getFrontDepthZIndex('mouse')
            }"
            @pointerdown.stop="startAccessoryDrag('mouse', $event)"
            :title="`${t('mouse')} · ${t('dragHint')}`"
          >
            <div class="wireless-mouse-unit">
              <div class="mouse-shell">
                <div class="mouse-buttons-split"></div>
                <div class="mouse-wheel"></div>
                <div class="mouse-thumb-rest"></div>
              </div>
            </div>
          </div>

          <!-- 3. Studio Desktop Speakers (Left & Right Pair) -->
          <template v-if="accessories.speakers.enabled">
            <!-- Left Speaker -->
            <div 
              class="table-accessory speaker-box speaker-left table-accessory-draggable"
              :class="{ 'is-selected-accessory': selectedAccessory === 'speaker-left' }"
              :style="{
                left: (desk.x + accessories.speakers.leftXPx) + 'px',
                top: (desk.y - speakerHeightPx) + 'px',
                width: speakerWidthPx + 'px',
                height: speakerHeightPx + 'px',
                zIndex: getFrontDepthZIndex('speaker-left')
              }"
              @pointerdown.stop="startSpeakerDrag('left', $event)"
              :title="`${t('speakerLeft')} · ${t('dragHint')}`"
            >
              <div class="speaker-tweeter"></div>
              <div class="speaker-woofer"></div>
              <div class="speaker-label">L</div>
            </div>

            <!-- Right Speaker -->
            <div 
              class="table-accessory speaker-box speaker-right table-accessory-draggable"
              :class="{ 'is-selected-accessory': selectedAccessory === 'speaker-right' }"
              :style="{
                left: (desk.x + accessories.speakers.rightXPx) + 'px',
                top: (desk.y - speakerHeightPx) + 'px',
                width: speakerWidthPx + 'px',
                height: speakerHeightPx + 'px',
                zIndex: getFrontDepthZIndex('speaker-right')
              }"
              @pointerdown.stop="startSpeakerDrag('right', $event)"
              :title="`${t('speakerRight')} · ${t('dragHint')}`"
            >
              <div class="speaker-tweeter"></div>
              <div class="speaker-woofer"></div>
              <div class="speaker-label">R</div>
            </div>
          </template>

          <!-- 4. Laptop / MacBook -->
          <div 
            v-if="accessories.laptop.enabled"
            class="table-accessory laptop-item table-accessory-draggable"
            :class="{ 'is-selected-accessory': selectedAccessory === 'laptop' }"
            :style="{
              left: (desk.x + accessories.laptop.xPx) + 'px',
              top: (desk.y - laptopHeightPx) + 'px',
              width: laptopWidthPx + 'px',
              height: laptopHeightPx + 'px',
              zIndex: getFrontDepthZIndex('laptop')
            }"
            @pointerdown.stop="startAccessoryDrag('laptop', $event)"
            :title="`${t('laptop')} · ${t('dragHint')}`"
          >
            <div class="laptop-screen-lid">
              <div class="laptop-camera-notch"></div>
              <div class="laptop-screen-display">
                <span class="laptop-tag">MacBook {{ accessories.laptop.size }}"</span>
              </div>
            </div>
            <div class="laptop-base">
              <div class="laptop-notch-cutout"></div>
            </div>
          </div>

          <!-- Snapping Active Guide Line in Front View -->
          <div 
            v-if="activeSnapGuide" 
            class="snap-guide-line"
            :class="activeSnapGuide.type"
            :style="activeSnapGuide.style"
          ></div>

          <!-- Empty Canvas Hint -->
          <div 
            v-if="monitors.length === 0" 
            class="canvas-empty-hint"
            :style="{
              left: (desk.x + deskWidthPx / 2) + 'px',
              top: (desk.y - 120) + 'px'
            }"
          >
            <div class="empty-icon">📺</div>
            <div class="empty-title">{{ t('emptyTitle') }}</div>
            <button class="primary-btn" @click.stop="openAddSheet">{{ t('emptyBtn') }}</button>
          </div>

          <!-- ================= Front View Monitors: 严格 projectedWidth 宽与真实四边近大远小/近粗远细 ================= -->
          <div 
            v-for="m in monitors" 
            :key="m.id"
            class="monitor-card"
            :class="{ 
              'is-selected': selectedMonitor?.id === m.id,
              'is-dragging': draggingId === m.id,
              'is-portrait': m.isPortrait,
              'is-swiveled': m.swivelAngle && m.swivelAngle !== 0
            }"
            :style="{
              width: getProjectedWidthPx(m) + 'px',
              height: m.heightPx + 'px',
              left: m.x + 'px',
              top: m.y + 'px',
              zIndex: getFrontDepthZIndex('monitor', m)
            }"
            @pointerdown.stop="startMonitorDrag(m.id, $event)"
          >
            <!-- Screen Hanging Light Bar -->
            <div v-if="m.hasLightBar" class="screen-light-bar">
              <div class="light-fixture"></div>
              <div class="light-beam"></div>
            </div>

            <!-- SVG Screen Bezel & Display Panel (True Orthographic Front View: Standard rectangle with tapered borders for swiveled angle) -->
            <svg 
              class="monitor-frame-svg"
              :width="getProjectedWidthPx(m)"
              :height="m.heightPx"
              :viewBox="`0 0 ${getProjectedWidthPx(m)} ${m.heightPx}`"
            >
              <defs>
                <linearGradient :id="'screenGrad-' + m.id" :x1="m.swivelAngle < 0 ? '100%' : '0%'" y1="0%" :x2="m.swivelAngle < 0 ? '0%' : '100%'" y2="100%">
                  <stop offset="0%" :stop-color="hexToRgba(m.color, 0.22)" />
                  <stop offset="45%" stop-color="#111827" />
                  <stop offset="100%" stop-color="#090d16" />
                </linearGradient>
                <linearGradient :id="'glareGrad-' + m.id" x1="0%" y1="0%" x2="0%" y2="100%">
                  <stop offset="0%" stop-color="#ffffff" stop-opacity="0.08" />
                  <stop offset="100%" stop-color="#ffffff" stop-opacity="0" />
                </linearGradient>
                <clipPath :id="'screenClip-' + m.id">
                  <rect x="0" y="0" :width="getProjectedWidthPx(m)" :height="m.heightPx" rx="6" />
                </clipPath>
              </defs>

              <g :clip-path="'url(#screenClip-' + m.id + ')'">
                <!-- 1. Background Chassis -->
                <rect x="0" y="0" :width="getProjectedWidthPx(m)" :height="m.heightPx" fill="#090d16" />

                <!-- 2. Inner Display Glass -->
                <polygon 
                  :points="getScreenBorderData(m).innerPolyStr"
                  :fill="'url(#screenGrad-' + m.id + ')'"
                />

                <!-- 3. Screen Glare / Reflection across top half -->
                <polygon 
                  :points="getScreenBorderData(m).glarePolyStr"
                  :fill="'url(#glareGrad-' + m.id + ')'"
                />

                <!-- 4. Bezel Border Frame (近粗远细，上下由细到粗渐变) -->
                <path 
                  :d="getScreenBorderData(m).borderPath"
                  fill-rule="evenodd"
                  :fill="selectedMonitor?.id === m.id ? '#38bdf8' : m.color"
                />

                <!-- 5. Near Edge Specular Highlight (近侧高光) -->
                <line 
                  v-if="m.swivelAngle && m.swivelAngle !== 0"
                  :x1="getScreenBorderData(m).highlightX"
                  :y1="4"
                  :x2="getScreenBorderData(m).highlightX"
                  :y2="m.heightPx - 4"
                  stroke="#ffffff"
                  stroke-width="1.5"
                  stroke-linecap="round"
                  opacity="0.65"
                />
              </g>

              <!-- Outer subtle border stroke for crisp separation -->
              <rect 
                x="0.5" 
                y="0.5" 
                :width="getProjectedWidthPx(m) - 1" 
                :height="m.heightPx - 1" 
                rx="6" 
                fill="none" 
                stroke="rgba(0,0,0,0.5)" 
                stroke-width="1"
              />
            </svg>

            <!-- Screen Content Badges (Centered HTML Overlay) -->
            <div class="screen-info-overlay">
              <div class="size-pill">
                <span class="diagonal-num">{{ m.diagonal }}"</span>
                <span class="ratio-text">{{ m.isPortrait ? `${m.ratioY}:${m.ratioX}` : `${m.ratioX}:${m.ratioY}` }}</span>
                <span v-if="m.swivelAngle" class="swivel-num-pill">
                  📐 {{ Math.abs(m.swivelAngle) }}° {{ m.swivelAngle > 0 ? '⤹' : '⤸' }}
                </span>
              </div>
              <div class="physical-dim">
                {{ m.widthCm }} × {{ m.heightCm }} cm
                <div v-if="m.swivelAngle" class="proj-dim-text">
                  {{ t('projectedWidth') }}: {{ getProjectedWidthCm(m) }}cm
                </div>
              </div>
            </div>

            <!-- Bottom Chin / Logo Pip -->
            <div class="screen-chin">
              <span class="chin-dot"></span>
            </div>

            <!-- Quick Action Handles when Selected -->
            <div v-if="selectedMonitor?.id === m.id && draggingId !== m.id" class="monitor-quick-actions">
              <button class="quick-btn rotate-btn" @pointerdown.stop="toggleRotation(m)" :title="t('rotateBtn')">
                🔄
              </button>
              <button 
                class="quick-btn" 
                :class="{ 'active-light': m.hasLightBar }"
                @pointerdown.stop="toggleLightBar(m)" 
                :title="t('lightBarBtn')"
              >
                💡
              </button>
              <button class="quick-btn delete-btn" @pointerdown.stop="removeMonitor(m.id)" :title="t('delete')">
                🗑️
              </button>
            </div>
          </div>
        </div>

        <!-- Front Floating HUD (Zoom & Fit) -->
        <div class="pane-hud">
          <button class="hud-btn" @click.stop="zoomInFront" :title="t('zoomIn')">➕</button>
          <span class="hud-zoom-label" @click.stop="resetFrontView">{{ Math.round(frontZoom * 100) }}%</span>
          <button class="hud-btn" @click.stop="zoomOutFront" :title="t('zoomOut')">➖</button>
          <button class="hud-btn fit-btn" @click.stop="fitFrontToScreen" :title="t('fitScreenTitle')">⤢</button>
        </div>
      </div>

      <!-- Splitter Pane Divider in Dual Mode -->
      <div v-if="viewLayoutMode === 'dual'" class="pane-divider">
        <div class="divider-line"></div>
      </div>

      <!-- ================= 2. RIGHT PANE: TOP-DOWN VIEW (俯视图 - 小) ================= -->
      <div 
        v-show="viewLayoutMode === 'dual' || viewLayoutMode === 'top'"
        class="canvas-pane top-pane"
        ref="topCanvasRef"
        @wheel.prevent="onTopWheel"
        @pointerdown="onTopCanvasPointerDown"
        @pointermove="onTopCanvasPointerMove"
        @pointerup="onTopCanvasPointerUp"
        @pointercancel="onTopCanvasPointerUp"
      >
        <!-- Top Viewport Tag Header -->
        <div class="pane-header-tag">
          <div class="pane-tag-title">
            <span class="pane-icon">📐</span>
            <span class="pane-name">{{ t('topView') }}</span>
            <span class="pane-badge">{{ t('topPlanBadge') }}</span>
          </div>
          <div class="pane-tag-actions">
            <span class="pane-zoom-text">{{ Math.round(topZoom * 100) }}%</span>
            <button class="pane-mini-btn" @click.stop="fitTopToScreen" :title="t('fitScreenTitle')">⤢</button>
          </div>
        </div>

        <!-- Background Grid -->
        <div class="grid-overlay"></div>

        <!-- Top World Transform Layer (Zoom & Pan) -->
        <div 
          class="world-layer"
          :style="{
            transform: `translate(${topPan.x}px, ${topPan.y}px) scale(${topZoom})`,
            transformOrigin: '0 0'
          }"
        >
          <div class="top-view-container">
            <!-- Desk Tabletop Surface (from above) -->
            <div 
              class="top-desk-surface"
              :style="{
                left: desk.x + 'px',
                top: TOP_DESK_Y + 'px',
                width: deskWidthPx + 'px',
                height: deskDepthPx + 'px'
              }"
            >
              <!-- Back edge chamfer / cable trench -->
              <div class="desk-back-trench"></div>
              <div class="top-desk-grain"></div>

              <!-- Measurement Rulers -->
              <div class="top-desk-ruler-x">
                <span>◄ {{ effectiveDeskWidthCm }} cm {{ t('deskWidth') }} ►</span>
              </div>
              <div class="top-desk-ruler-y">
                <span>▲ {{ deskDepthCm }} cm {{ t('deskDepth') }} ▼</span>
              </div>

              <!-- Keyboard from above (Draggable in X and Y!) -->
              <div 
                v-if="accessories.keyboard.enabled"
                class="top-keyboard table-accessory-draggable"
                :class="{ 'is-selected-accessory': selectedAccessory === 'keyboard' }"
                :style="{
                  left: accessories.keyboard.xPx + 'px',
                  top: Math.max(10, (deskDepthPx - (14 * CM_TO_PX) - (15 * CM_TO_PX) - (accessories.keyboard.yDepthPx || 0))) + 'px',
                  width: (currentKeyboardLayout.widthCm * CM_TO_PX) + 'px',
                  height: (14 * CM_TO_PX) + 'px'
                }"
                @pointerdown.stop="startTopAccessoryDrag('keyboard', $event)"
                :title="`${t('keyboard')} (${currentKeyboardLayout.labelEn || currentKeyboardLayout.value}) · ${t('dragBothHint')}`"
              >
                <!-- SVG Realistic Keyboard with Keycaps -->
                <svg 
                  class="top-keyboard-svg"
                  :viewBox="getTopKeyboardLayout(accessories.keyboard.layout).viewBox"
                  preserveAspectRatio="none"
                >
                  <!-- Case Outer Base -->
                  <rect 
                    x="0.5" 
                    y="0.5" 
                    :width="getTopKeyboardLayout(accessories.keyboard.layout).totalW - 1" 
                    :height="getTopKeyboardLayout(accessories.keyboard.layout).totalH - 1" 
                    rx="4.5" 
                    fill="#101726" 
                    stroke="#334155" 
                    stroke-width="1.2" 
                  />

                  <!-- Keycap Switch Plate / Inner Well -->
                  <rect 
                    x="2.5" 
                    y="2.5" 
                    :width="getTopKeyboardLayout(accessories.keyboard.layout).totalW - 5" 
                    :height="getTopKeyboardLayout(accessories.keyboard.layout).totalH - 5" 
                    rx="3.5" 
                    fill="#080c14" 
                  />

                  <!-- Keycaps -->
                  <g v-for="(k, idx) in getTopKeyboardLayout(accessories.keyboard.layout).keys" :key="'kb-key-' + idx">
                    <rect 
                      :x="k.x" 
                      :y="k.y" 
                      :width="k.w" 
                      :height="k.h" 
                      rx="1.5" 
                      :fill="getKeyColor(k.type).fill" 
                      :stroke="getKeyColor(k.type).stroke" 
                      stroke-width="0.8" 
                    />
                    <!-- Subtle keycap top highlight -->
                    <rect 
                      :x="k.x + 1" 
                      :y="k.y + 0.8" 
                      :width="Math.max(1, k.w - 2)" 
                      :height="Math.max(1, k.h - 2.5)" 
                      rx="1" 
                      fill="none" 
                      stroke="rgba(255, 255, 255, 0.1)" 
                      stroke-width="0.6" 
                    />
                  </g>

                  <!-- Status Indicator LEDs for 104 and 98 -->
                  <circle 
                    v-for="(led, lIdx) in getTopKeyboardLayout(accessories.keyboard.layout).leds" 
                    :key="'led-' + lIdx" 
                    :cx="led.cx" 
                    :cy="led.cy" 
                    r="1.2" 
                    :fill="led.color" 
                    stroke="#0284c7"
                    stroke-width="0.5"
                  />
                </svg>

                <span class="top-kb-badge">{{ accessories.keyboard.layout }}</span>
              </div>

              <!-- Mouse from above (Draggable in X and Y!) -->
              <div 
                v-if="accessories.mouse.enabled"
                class="top-mouse table-accessory-draggable"
                :class="{ 'is-selected-accessory': selectedAccessory === 'mouse' }"
                :style="{
                  left: accessories.mouse.xPx + 'px',
                  top: Math.max(10, (deskDepthPx - (12 * CM_TO_PX) - (16 * CM_TO_PX) - (accessories.mouse.yDepthPx || 0))) + 'px',
                  width: (7.5 * CM_TO_PX) + 'px',
                  height: (12 * CM_TO_PX) + 'px'
                }"
                @pointerdown.stop="startTopAccessoryDrag('mouse', $event)"
                :title="`${t('mouse')} · ${t('dragBothHint')}`"
              >
                <div class="top-mouse-wheel"></div>
              </div>

              <!-- Studio Speakers from above (Draggable in X and Y!) -->
              <template v-if="accessories.speakers.enabled">
                <!-- Left Speaker -->
                <div 
                  class="top-speaker left-top-speaker table-accessory-draggable"
                  :class="{ 'is-selected-accessory': selectedAccessory === 'speaker-left' }"
                  :style="{
                    left: accessories.speakers.leftXPx + 'px',
                    top: (25 + (accessories.speakers.leftYDepthPx || 0)) + 'px',
                    width: speakerWidthPx + 'px',
                    height: (18 * CM_TO_PX) + 'px'
                  }"
                  @pointerdown.stop="startTopSpeakerDrag('left', $event)"
                  :title="`${t('speakerLeft')} · ${t('dragBothHint')}`"
                >
                  <div class="top-speaker-driver"></div>
                  <span class="top-spk-tag">L</span>
                </div>

                <!-- Right Speaker -->
                <div 
                  class="top-speaker right-top-speaker table-accessory-draggable"
                  :class="{ 'is-selected-accessory': selectedAccessory === 'speaker-right' }"
                  :style="{
                    left: accessories.speakers.rightXPx + 'px',
                    top: (25 + (accessories.speakers.rightYDepthPx || 0)) + 'px',
                    width: speakerWidthPx + 'px',
                    height: (18 * CM_TO_PX) + 'px'
                  }"
                  @pointerdown.stop="startTopSpeakerDrag('right', $event)"
                  :title="`${t('speakerRight')} · ${t('dragBothHint')}`"
                >
                  <div class="top-speaker-driver"></div>
                  <span class="top-spk-tag">R</span>
                </div>
              </template>

              <!-- Laptop from above (MacBook - Draggable in X and Y!) -->
              <div 
                v-if="accessories.laptop.enabled"
                class="top-laptop table-accessory-draggable"
                :class="{ 'is-selected-accessory': selectedAccessory === 'laptop' }"
                :style="{
                  left: accessories.laptop.xPx + 'px',
                  top: Math.max(10, (deskDepthPx - laptopHeightPx - (12 * CM_TO_PX) - (accessories.laptop.yDepthPx || 0))) + 'px',
                  width: laptopWidthPx + 'px',
                  height: laptopHeightPx + 'px'
                }"
                @pointerdown.stop="startTopAccessoryDrag('laptop', $event)"
                :title="`${t('laptop')} · ${t('dragBothHint')}`"
              >
                <div class="top-laptop-lid">
                  <div class="top-laptop-kb-well"></div>
                  <div class="top-laptop-trackpad"></div>
                  <span class="top-laptop-tag">MacBook {{ accessories.laptop.size }}"</span>
                </div>
              </div>
            </div>

            <!-- SVG layer for Monitor Arms, Viewing Cones, Angle Arcs in Top View -->
            <svg class="top-view-svg-layer">
              <defs>
                <marker id="topArrow" viewBox="0 0 10 10" refX="5" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
                  <path d="M 0 0 L 10 5 L 0 10 z" fill="#38bdf8" />
                </marker>
              </defs>

              <!-- Monitor Arms from Above -->
              <g v-for="m in monitors" :key="'top-arm-' + m.id">
                <!-- Desk Back Clamp -->
                <rect 
                  :x="getTopMonitorCoords(m).clampX - 14" 
                  :y="TOP_DESK_Y - 8" 
                  width="28" 
                  height="16" 
                  rx="3" 
                  fill="#1e293b" 
                  stroke="#475569" 
                  stroke-width="2" 
                />
                <!-- Arm Path -->
                <path 
                  :d="getTopArmPath(m)" 
                  stroke="#64748b" 
                  stroke-width="6" 
                  stroke-linecap="round" 
                  fill="none" 
                />
                <circle :cx="getTopArmJoint(m).x" :cy="getTopArmJoint(m).y" r="5" fill="#38bdf8" />

                <!-- Viewing / Light Cone forward onto desk (Pointing toward user) -->
                <polygon 
                  :points="getTopViewingCone(m)" 
                  :fill="hexToRgba(m.color, m.hasLightBar ? 0.12 : 0.06)" 
                />

                <!-- Angle Arc if swiveled (Curving forward toward user) -->
                <g v-if="m.swivelAngle && m.swivelAngle !== 0">
                  <path 
                    :d="getTopAngleArc(m)" 
                    stroke="#38bdf8" 
                    stroke-width="2" 
                    stroke-dasharray="3 3" 
                    fill="none" 
                  />
                  <text 
                    :x="getTopAngleTextPos(m).x" 
                    :y="getTopAngleTextPos(m).y" 
                    fill="#38bdf8" 
                    font-size="12" 
                    font-weight="bold" 
                    text-anchor="middle"
                  >
                    {{ Math.abs(m.swivelAngle) }}°
                  </text>
                </g>
              </g>

              <!-- Sightlines to User Sweet Spot -->
              <g class="sweet-spot-group">
                <line 
                  v-for="m in monitors" 
                  :key="'sightline-' + m.id"
                  :x1="getUserSweetSpot().x" 
                  :y1="getUserSweetSpot().y - 8" 
                  :x2="getTopMonitorCoords(m).centerX" 
                  :y2="getTopMonitorCoords(m).centerY" 
                  stroke="#38bdf8" 
                  stroke-width="1.5" 
                  stroke-dasharray="4 4" 
                  opacity="0.35" 
                />
              </g>
            </svg>

            <!-- Monitors from above (Draggable in X and Depth Y!) -->
            <div 
              v-for="m in monitors" 
              :key="'top-mon-' + m.id"
              class="top-monitor-bar"
              :class="{ 
                'is-selected': selectedMonitor?.id === m.id,
                'is-swiveled': m.swivelAngle && m.swivelAngle !== 0
              }"
              :style="{
                left: getTopMonitorLeft(m) + 'px',
                top: getTopMonitorCoords(m).topY + 'px',
                width: m.widthPx + 'px',
                transform: `rotate(${-m.swivelAngle || 0}deg)`,
                transformOrigin: getTopTransformOrigin(m),
                borderColor: selectedMonitor?.id === m.id ? '#38bdf8' : m.color
              }"
              @pointerdown.stop="startTopMonitorDrag(m.id, $event)"
              :title="m.diagonal + ' Inch · ' + t('dragBothHint')"
            >
              <!-- Front Screen Glow (Facing bottom/user) -->
              <div 
                class="top-screen-glass" 
                :style="{ background: m.color, boxShadow: `0 0 12px ${hexToRgba(m.color, 0.6)}` }"
              ></div>
              <div class="top-screen-body">
                <span class="top-screen-tag">
                  {{ m.diagonal }}" {{ m.isPortrait ? (lang === 'zh' ? '竖' : 'P') : (lang === 'zh' ? '横' : 'L') }} · {{ m.widthCm }}cm
                  <span v-if="m.swivelAngle" class="top-angle-pill">({{ Math.abs(m.swivelAngle) }}°)</span>
                </span>
                <span v-if="m.hasLightBar" class="top-lightbar-badge">💡</span>
              </div>
            </div>

            <!-- User Seating / Viewing Sweet Spot -->
            <div 
              class="user-sweet-spot"
              :style="{
                left: getUserSweetSpot().x + 'px',
                top: (getUserSweetSpot().y - 8) + 'px'
              }"
            >
              <div class="sweet-spot-label">
                <span class="chair-icon-top">💺</span>
                <span class="spot-title">{{ t('viewSweetSpot') }}</span>
                <span class="spot-divider">·</span>
                <span class="spot-sub">~65 cm</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Top Floating HUD (Zoom & Fit) -->
        <div class="pane-hud">
          <button class="hud-btn" @click.stop="zoomInTop" :title="t('zoomIn')">➕</button>
          <span class="hud-zoom-label" @click.stop="resetTopView">{{ Math.round(topZoom * 100) }}%</span>
          <button class="hud-btn" @click.stop="zoomOutTop" :title="t('zoomOut')">➖</button>
          <button class="hud-btn fit-btn" @click.stop="fitTopToScreen" :title="t('fitScreenTitle')">⤢</button>
        </div>
      </div>

      <!-- Floating Add Button for Mobile -->
      <div v-if="isMobile && !activeDrawer" class="mobile-floating-actions">
        <button v-if="selectedMonitor" class="floating-edit-btn" @click="activeDrawer = 'edit'">
          ✏️ {{ t('edit') }} {{ selectedMonitor.diagonal }}"
        </button>
        <button class="floating-acc-btn" @click="activeDrawer = 'accessories'">
          ⌨️ {{ t('accessories') }}
        </button>
        <button class="floating-add-btn" @click="openAddSheet">
          ➕ {{ t('addMonitor') }}
        </button>
      </div>
    </div>

    <!-- Dedicated Right Sidebar Panel: Tools & Inspector (Fixed Column on Desktop) -->
    <aside v-if="!isMobile" class="desktop-sidebar-panel">
      <!-- Sidebar Top Header -->
      <div class="sidebar-header">
        <div class="sidebar-title">
          <span class="sidebar-icon">🛠️</span>
          <span>{{ t('controlPanel') }}</span>
        </div>
        <div class="sidebar-mon-count">
          {{ monitors.length }} {{ t('monitorsCount') }}
        </div>
      </div>

      <!-- Current Preset & One-Click Restore Card -->
      <div class="inspector-card preset-sidebar-card">
        <div class="card-header-flex">
          <div class="card-title">⚡ {{ t('currentPresetTitle') }}</div>
          <button class="chip-button" @click="showPresetsModal = true">{{ t('switchPreset') }}</button>
        </div>
        <div class="preset-summary-box">
          <span class="preset-summary-icon">{{ currentPresetObj.icon }}</span>
          <div class="preset-summary-content">
            <span class="preset-summary-name">{{ lang === 'zh' ? currentPresetObj.nameZh : currentPresetObj.nameEn }}</span>
            <span class="preset-summary-desc">{{ lang === 'zh' ? currentPresetObj.descZh : currentPresetObj.descEn }}</span>
          </div>
        </div>
        <div class="preset-summary-actions mt-2">
          <button class="primary-btn full-btn restore-highlight-btn" @click="restoreCurrentPreset">
            ↺ {{ t('restorePresetBtn') }}
          </button>
        </div>
      </div>

      <!-- Displays List Card -->
      <div class="inspector-card">
        <div class="card-header-flex">
          <div class="card-title">🖥️ {{ t('monitorsList') }}</div>
          <button class="icon-mini-add-btn" @click="selectedId = null" :title="t('addMonitor')">➕</button>
        </div>
        <div class="monitor-chips-list">
          <div 
            v-for="m in monitors" 
            :key="'chip-' + m.id"
            class="monitor-chip-row"
            :class="{ active: selectedMonitor?.id === m.id }"
            @click="selectedId = m.id"
          >
            <div class="chip-color-dot" :style="{ backgroundColor: m.color }"></div>
            <div class="chip-info">
              <span class="chip-name">{{ m.diagonal }}" {{ m.isPortrait ? (lang === 'zh' ? '竖屏' : 'Portrait') : (lang === 'zh' ? '横屏' : 'Landscape') }}</span>
              <span class="chip-sub">
                {{ m.widthCm }}×{{ m.heightCm }}cm
                <template v-if="m.swivelAngle"> · 📐 {{ Math.abs(m.swivelAngle) }}°</template>
              </span>
            </div>
            <button class="chip-del-btn" @click.stop="removeMonitor(m.id)" :title="t('delete')">🗑️</button>
          </div>
        </div>
      </div>

      <!-- Desk Controls (Width & Depth) -->
      <div class="inspector-card">
        <div class="card-title">{{ t('deskSize') }}</div>
        
        <!-- Desk Width -->
        <div class="field-sub-label">{{ t('deskWidth') }}</div>
        <div class="input-row">
          <input 
            type="range" 
            v-model.number="desk.widthCm" 
            min="60" 
            max="300" 
            step="5"
            class="range-slider"
          />
          <div class="number-input-wrap">
            <input type="number" v-model.number="desk.widthCm" min="50" max="400" />
            <span class="unit">cm</span>
          </div>
        </div>
        <div class="quick-chips">
          <button 
            v-for="w in [100, 120, 140, 160, 180, 200]" 
            :key="w" 
            class="chip-button"
            :class="{ active: desk.widthCm === w }"
            @click="desk.widthCm = w; saveToLocalStorage()"
          >
            {{ w }}cm
          </button>
        </div>

        <!-- Desk Depth (进深: 50, 60, 70, 75, 80cm) -->
        <div class="field-sub-label mt-3">{{ t('deskDepth') }}</div>
        <div class="input-row">
          <input 
            type="range" 
            v-model.number="deskDepthCm" 
            min="40" 
            max="120" 
            step="5"
            class="range-slider"
          />
          <div class="number-input-wrap">
            <input type="number" v-model.number="deskDepthCm" min="40" max="150" />
            <span class="unit">cm</span>
          </div>
        </div>
        <div class="quick-chips">
          <button 
            v-for="dep in [50, 60, 70, 75, 80]" 
            :key="dep" 
            class="chip-button"
            :class="{ active: deskDepthCm === dep }"
            @click="deskDepthCm = dep; saveToLocalStorage()"
          >
            {{ dep }}cm
          </button>
        </div>
      </div>

      <!-- Desktop Accessories Controls -->
      <div class="inspector-card">
        <div class="card-title">{{ t('accessoriesTitle') }}</div>
        <div class="accessories-toggle-list">
          <!-- Keyboard Options -->
          <div class="accessory-block">
            <div class="toggle-row">
              <label class="toggle-label">
                <input type="checkbox" v-model="accessories.keyboard.enabled" @change="saveToLocalStorage" />
                <span>{{ t('keyboard') }}</span>
              </label>
            </div>
            <div v-if="accessories.keyboard.enabled" class="sub-layout-picker mt-2">
              <span class="picker-label">{{ t('keyboardLayout') }}</span>
              <div class="quick-chips">
                <button 
                  v-for="k in KEYBOARD_LAYOUTS" 
                  :key="k.value"
                  class="chip-button sm"
                  :class="{ active: accessories.keyboard.layout === k.value }"
                  @click="accessories.keyboard.layout = k.value; saveToLocalStorage()"
                >
                  {{ lang === 'zh' ? k.labelZh : k.labelEn }}
                </button>
              </div>
            </div>
          </div>

          <!-- Mouse Option -->
          <div class="toggle-row">
            <label class="toggle-label">
              <input type="checkbox" v-model="accessories.mouse.enabled" @change="saveToLocalStorage" />
              <span>{{ t('mouse') }}</span>
            </label>
          </div>

          <!-- Studio Speakers -->
          <div class="toggle-row">
            <label class="toggle-label">
              <input type="checkbox" v-model="accessories.speakers.enabled" @change="saveToLocalStorage" />
              <span>{{ t('speakers') }}</span>
            </label>
          </div>

          <!-- Laptop -->
          <div class="toggle-row">
            <label class="toggle-label">
              <input type="checkbox" v-model="accessories.laptop.enabled" @change="saveToLocalStorage" />
              <span>{{ t('laptop') }}</span>
            </label>
            <div v-if="accessories.laptop.enabled" class="sub-chips">
              <button 
                class="chip-button sm" 
                :class="{ active: accessories.laptop.size === 14 }"
                @click="accessories.laptop.size = 14; updateLaptopDimensions()"
              >14"</button>
              <button 
                class="chip-button sm" 
                :class="{ active: accessories.laptop.size === 16 }"
                @click="accessories.laptop.size = 16; updateLaptopDimensions()"
              >16"</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Edit Selected Monitor -->
      <div v-if="selectedMonitor" class="inspector-card edit-card">
        <div class="card-header-flex">
          <div class="card-title">{{ t('editSelected') }}</div>
          <button class="close-x" @click="clearSelection">✕</button>
        </div>

        <div class="field-group">
          <label>{{ t('screenSize') }}</label>
          <div class="quick-chips">
            <button 
              v-for="d in [24, 27, 32, 34, 49]" 
              :key="d" 
              class="chip-button"
              :class="{ active: selectedMonitor.diagonal === d }"
              @click="setMonitorDiagonal(selectedMonitor, d)"
            >
              {{ d }}"
            </button>
          </div>
          <div class="input-row mt-2">
            <input 
              type="range" 
              v-model.number="selectedMonitor.diagonal" 
              @input="updateSelected"
              min="13" 
              max="65" 
              step="1"
              class="range-slider"
            />
            <div class="number-input-wrap">
              <input 
                type="number" 
                v-model.number="selectedMonitor.diagonal" 
                @input="updateSelected" 
                min="10" 
                max="100" 
              />
              <span class="unit">"</span>
            </div>
          </div>
        </div>

        <div class="field-group">
          <label>{{ t('aspectRatio') }}</label>
          <div class="ratio-chips">
            <button 
              v-for="opt in RATIO_OPTIONS" 
              :key="opt.value"
              class="chip-button"
              :class="{ active: selectedMonitor.ratioString === opt.value }"
              @click="setMonitorRatio(selectedMonitor, opt.value)"
            >
              {{ opt.value === 'custom' ? t('customRatio') : opt.label }}
            </button>
          </div>
          <div v-if="selectedMonitor.ratioString === 'custom'" class="custom-ratio-row">
            <input type="number" v-model.number="selectedMonitor.ratioX" @input="updateSelected" min="1" />
            <span>:</span>
            <input type="number" v-model.number="selectedMonitor.ratioY" @input="updateSelected" min="1" />
          </div>
        </div>

        <!-- Stand & Mounting Options -->
        <div class="field-group">
          <label>{{ t('mountType') }}</label>
          <div class="quick-chips">
            <button 
              class="chip-button"
              :class="{ active: selectedMonitor.standType === 'stand' || !selectedMonitor.standType }"
              @click="setMonitorStand(selectedMonitor, 'stand')"
            >
              {{ t('standOriginal') }}
            </button>
            <button 
              class="chip-button"
              :class="{ active: selectedMonitor.standType === 'arm' }"
              @click="setMonitorStand(selectedMonitor, 'arm')"
            >
              {{ t('standArm') }}
            </button>
            <button 
              class="chip-button"
              :class="{ active: selectedMonitor.standType === 'none' }"
              @click="setMonitorStand(selectedMonitor, 'none')"
            >
              {{ t('standNone') }}
            </button>
          </div>
        </div>

        <!-- Light Bar Toggle -->
        <div class="field-group">
          <label class="toggle-label">
            <input type="checkbox" v-model="selectedMonitor.hasLightBar" @change="saveToLocalStorage" />
            <span>{{ t('lightBar') }}</span>
          </label>
        </div>

        <!-- Swivel / Inward Slant Angle (向前偏转折角) -->
        <div class="field-group">
          <div class="field-header-row">
            <label>{{ t('swivelAngle') }}</label>
            <span class="field-value-badge">{{ selectedMonitor.swivelAngle || 0 }}°</span>
          </div>
          <div class="quick-chips angle-chips">
            <button 
              class="chip-button"
              :class="{ active: selectedMonitor.swivelAngle === 30 }"
              @click="setMonitorSwivel(selectedMonitor, 30)"
            >
              ⤹ {{ t('swivelLeft30') }}
            </button>
            <button 
              class="chip-button"
              :class="{ active: !selectedMonitor.swivelAngle || selectedMonitor.swivelAngle === 0 }"
              @click="setMonitorSwivel(selectedMonitor, 0)"
            >
              {{ t('flat') }}
            </button>
            <button 
              class="chip-button"
              :class="{ active: selectedMonitor.swivelAngle === -30 }"
              @click="setMonitorSwivel(selectedMonitor, -30)"
            >
              {{ t('swivelRight30') }} ⤸
            </button>
          </div>
          <div class="slider-row mt-2">
            <input 
              type="range" 
              min="-45" 
              max="45" 
              step="5" 
              v-model.number="selectedMonitor.swivelAngle" 
              @input="saveToLocalStorage"
              class="range-slider"
            />
            <div class="slider-labels">
              <span>-45°</span>
              <span>0°</span>
              <span>+45°</span>
            </div>
          </div>
          <div class="help-hint">
            {{ t('swivelHint') }}
          </div>
        </div>

        <div class="field-group">
          <label>{{ t('physicalDimensions') }}</label>
          <div class="metric-badge">
            {{ selectedMonitor.widthCm }} cm (W) × {{ selectedMonitor.heightCm }} cm (H)
            <div v-if="selectedMonitor.swivelAngle" class="mt-1 text-cyan font-bold">
              {{ t('projectedWidth') }}: {{ getProjectedWidthCm(selectedMonitor) }} cm (勾股投影)
            </div>
          </div>
        </div>

        <div class="card-footer-buttons">
          <button class="secondary-btn" @click="toggleRotation(selectedMonitor)">
            🔄 {{ selectedMonitor.isPortrait ? t('toLandscape') : t('toPortrait') }}
          </button>
          <button class="danger-btn" @click="removeMonitor(selectedMonitor.id)">
            🗑️ {{ t('delete') }}
          </button>
        </div>
      </div>

      <!-- Add Monitor (when nothing selected) -->
      <div v-else class="inspector-card add-card">
        <div class="card-title">➕ {{ t('addMonitor') }}</div>

        <div class="field-group">
          <label>{{ t('quickSize') }}</label>
          <div class="quick-chips">
            <button 
              v-for="d in [24, 27, 32, 34, 49]" 
              :key="d" 
              class="chip-button"
              :class="{ active: addForm.diagonal === d }"
              @click="addForm.diagonal = d"
            >
              {{ d }}"
            </button>
          </div>
          <div class="input-row mt-2">
            <input 
              type="range" 
              v-model.number="addForm.diagonal" 
              min="13" 
              max="65" 
              step="1"
              class="range-slider"
            />
            <div class="number-input-wrap">
              <input type="number" v-model.number="addForm.diagonal" min="10" max="100" />
              <span class="unit">"</span>
            </div>
          </div>
        </div>

        <div class="field-group">
          <label>{{ t('aspectRatio') }}</label>
          <div class="ratio-chips">
            <button 
              v-for="opt in RATIO_OPTIONS" 
              :key="opt.value"
              class="chip-button"
              :class="{ active: addForm.ratioString === opt.value }"
              @click="addForm.ratioString = opt.value"
            >
              {{ opt.value === 'custom' ? t('customRatio') : opt.label }}
            </button>
          </div>
        </div>

        <div class="field-group">
          <label>{{ t('mountType') }}</label>
          <div class="quick-chips">
            <button 
              class="chip-button"
              :class="{ active: addForm.standType === 'stand' }"
              @click="addForm.standType = 'stand'"
            >
              {{ t('standOriginal') }}
            </button>
            <button 
              class="chip-button"
              :class="{ active: addForm.standType === 'arm' }"
              @click="addForm.standType = 'arm'"
            >
              {{ t('standArm') }}
            </button>
            <button 
              class="chip-button"
              :class="{ active: addForm.standType === 'none' }"
              @click="addForm.standType = 'none'"
            >
              {{ t('standNone') }}
            </button>
          </div>
        </div>

        <button class="primary-btn full-btn mt-3" @click="addMonitor">
          ➕ {{ t('putOnDesk') }}
        </button>
      </div>
    </aside>
  </main>

    <!-- Mobile Bottom Drawers -->
    <div v-if="isMobile && activeDrawer" class="mobile-drawer-overlay" @click.self="activeDrawer = null">
      <div class="mobile-drawer-sheet">
        <!-- Add Sheet -->
        <template v-if="activeDrawer === 'add'">
          <div class="drawer-header">
            <h3>➕ {{ t('addMonitor') }}</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>
          <div class="drawer-body">
            <div class="field-group">
              <label>{{ t('screenSize') }}</label>
              <div class="quick-chips">
                <button 
                  v-for="d in [24, 27, 32, 34, 49]" 
                  :key="d" 
                  class="chip-button"
                  :class="{ active: addForm.diagonal === d }"
                  @click="addForm.diagonal = d"
                >
                  {{ d }}"
                </button>
              </div>
              <div class="input-row mt-2">
                <input type="range" v-model.number="addForm.diagonal" min="13" max="65" step="1" class="range-slider" />
                <div class="number-input-wrap">
                  <input type="number" v-model.number="addForm.diagonal" min="10" max="100" />
                  <span class="unit">"</span>
                </div>
              </div>
            </div>

            <div class="field-group">
              <label>{{ t('aspectRatio') }}</label>
              <div class="ratio-chips">
                <button 
                  v-for="opt in RATIO_OPTIONS" 
                  :key="opt.value"
                  class="chip-button"
                  :class="{ active: addForm.ratioString === opt.value }"
                  @click="addForm.ratioString = opt.value"
                >
                  {{ opt.value === 'custom' ? t('customRatio') : opt.label }}
                </button>
              </div>
            </div>

            <div class="field-group">
              <label>{{ t('mountType') }}</label>
              <div class="quick-chips">
                <button 
                  class="chip-button"
                  :class="{ active: addForm.standType === 'stand' }"
                  @click="addForm.standType = 'stand'"
                >
                  {{ t('standOriginal') }}
                </button>
                <button 
                  class="chip-button"
                  :class="{ active: addForm.standType === 'arm' }"
                  @click="addForm.standType = 'arm'"
                >
                  {{ t('standArm') }}
                </button>
              </div>
            </div>

            <button class="primary-btn full-btn mt-3" @click="addMonitor">
              ➕ {{ t('putOnDesk') }}
            </button>
          </div>
        </template>

        <!-- Edit Sheet -->
        <template v-else-if="activeDrawer === 'edit' && selectedMonitor">
          <div class="drawer-header">
            <h3>✏️ {{ t('edit') }} {{ selectedMonitor.diagonal }}"</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>
          <div class="drawer-body">
            <div class="field-group">
              <label>{{ t('screenSize') }}</label>
              <div class="quick-chips">
                <button 
                  v-for="d in [24, 27, 32, 34, 49]" 
                  :key="d" 
                  class="chip-button"
                  :class="{ active: selectedMonitor.diagonal === d }"
                  @click="setMonitorDiagonal(selectedMonitor, d)"
                >
                  {{ d }}"
                </button>
              </div>
            </div>

            <div class="field-group">
              <label class="toggle-label">
                <input type="checkbox" v-model="selectedMonitor.hasLightBar" @change="saveToLocalStorage" />
                <span>{{ t('lightBar') }}</span>
              </label>
            </div>

            <!-- Swivel in mobile -->
            <div class="field-group">
              <label>{{ t('swivelAngle') }} ({{ selectedMonitor.swivelAngle || 0 }}°)</label>
              <div class="quick-chips angle-chips">
                <button class="chip-button" :class="{ active: selectedMonitor.swivelAngle === 30 }" @click="setMonitorSwivel(selectedMonitor, 30)">
                  ⤹ 30°
                </button>
                <button class="chip-button" :class="{ active: !selectedMonitor.swivelAngle || selectedMonitor.swivelAngle === 0 }" @click="setMonitorSwivel(selectedMonitor, 0)">
                  0°
                </button>
                <button class="chip-button" :class="{ active: selectedMonitor.swivelAngle === -30 }" @click="setMonitorSwivel(selectedMonitor, -30)">
                  -30° ⤸
                </button>
              </div>
            </div>

            <div class="drawer-btn-row mt-3">
              <button class="secondary-btn" @click="toggleRotation(selectedMonitor)">
                🔄 {{ selectedMonitor.isPortrait ? t('toLandscape') : t('toPortrait') }}
              </button>
              <button class="danger-btn" @click="removeMonitor(selectedMonitor.id)">
                🗑️ {{ t('delete') }}
              </button>
            </div>

            <button class="primary-btn full-btn mt-3" @click="activeDrawer = null">
              ✓ {{ t('done') }}
            </button>
          </div>
        </template>

        <!-- Accessories Sheet -->
        <template v-else-if="activeDrawer === 'accessories'">
          <div class="drawer-header">
            <h3>⌨️ {{ t('accessoriesTitle') }}</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>
          <div class="drawer-body">
            <div class="toggle-row">
              <label class="toggle-label">
                <input type="checkbox" v-model="accessories.keyboard.enabled" @change="saveToLocalStorage" />
                <span>{{ t('keyboard') }}</span>
              </label>
            </div>
            <div v-if="accessories.keyboard.enabled" class="sub-layout-picker mt-2 mb-3">
              <div class="quick-chips">
                <button 
                  v-for="k in KEYBOARD_LAYOUTS" 
                  :key="k.value"
                  class="chip-button sm"
                  :class="{ active: accessories.keyboard.layout === k.value }"
                  @click="accessories.keyboard.layout = k.value; saveToLocalStorage()"
                >
                  {{ lang === 'zh' ? k.labelZh : k.labelEn }}
                </button>
              </div>
            </div>

            <div class="toggle-row">
              <label class="toggle-label">
                <input type="checkbox" v-model="accessories.mouse.enabled" @change="saveToLocalStorage" />
                <span>{{ t('mouse') }}</span>
              </label>
            </div>

            <div class="toggle-row">
              <label class="toggle-label">
                <input type="checkbox" v-model="accessories.speakers.enabled" @change="saveToLocalStorage" />
                <span>{{ t('speakers') }}</span>
              </label>
            </div>

            <div class="toggle-row">
              <label class="toggle-label">
                <input type="checkbox" v-model="accessories.laptop.enabled" @change="saveToLocalStorage" />
                <span>{{ t('laptop') }}</span>
              </label>
            </div>

            <button class="primary-btn full-btn mt-3" @click="activeDrawer = null">
              ✓ {{ t('done') }}
            </button>
          </div>
        </template>

        <!-- Desk Settings Sheet -->
        <template v-else-if="activeDrawer === 'desk'">
          <div class="drawer-header">
            <h3>🪑 {{ t('deskSettings') }}</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>

          <div class="drawer-body">
            <div class="field-group">
              <label>{{ t('deskWidth') }}</label>
              <div class="quick-chips">
                <button 
                  v-for="w in [100, 120, 140, 160, 180, 200]" 
                  :key="w" 
                  class="chip-button"
                  :class="{ active: desk.widthCm === w }"
                  @click="desk.widthCm = w"
                >
                  {{ w }}cm
                </button>
              </div>
              <div class="input-row mt-3">
                <input 
                  type="range" 
                  v-model.number="desk.widthCm" 
                  min="60" 
                  max="300" 
                  step="5"
                  class="range-slider"
                />
                <div class="number-input-wrap">
                  <input type="number" v-model.number="desk.widthCm" min="50" max="400" />
                  <span class="unit">cm</span>
                </div>
              </div>
            </div>

            <div class="field-group mt-3">
              <label>{{ t('deskDepth') }}</label>
              <div class="quick-chips">
                <button 
                  v-for="dep in [50, 60, 70, 75, 80]" 
                  :key="dep" 
                  class="chip-button"
                  :class="{ active: deskDepthCm === dep }"
                  @click="deskDepthCm = dep"
                >
                  {{ dep }}cm
                </button>
              </div>
              <div class="input-row mt-3">
                <input 
                  type="range" 
                  v-model.number="deskDepthCm" 
                  min="40" 
                  max="120" 
                  step="5"
                  class="range-slider"
                />
                <div class="number-input-wrap">
                  <input type="number" v-model.number="deskDepthCm" min="40" max="150" />
                  <span class="unit">cm</span>
                </div>
              </div>
            </div>

            <button class="primary-btn full-btn mt-3" @click="activeDrawer = null; fitToScreen()">
              ✓ {{ t('doneAndCenter') }}
            </button>
          </div>
        </template>
      </div>
    </div>

    <!-- Presets Modal Dialog (With Dedicated Restore Action for Every Preset) -->
    <div 
      v-if="showPresetsModal" 
      class="modal-backdrop"
      @click.self="showPresetsModal = false"
    >
      <div class="modal-card presets-dialog-card">
        <div class="modal-header">
          <div class="modal-title">⚡ {{ t('presetsTitle') }}</div>
          <button class="close-x" @click="showPresetsModal = false">✕</button>
        </div>

        <div class="presets-grid">
          <div 
            v-for="p in PRESETS" 
            :key="p.id" 
            class="preset-item-card"
            :class="{ 'is-active-preset': activePresetId === p.id }"
          >
            <div class="preset-card-top">
              <div class="preset-icon-badge">{{ p.icon }}</div>
              <div class="preset-info-col">
                <div class="preset-title-row">
                  <span class="preset-name">{{ lang === 'zh' ? p.nameZh : p.nameEn }}</span>
                  <span v-if="activePresetId === p.id" class="preset-active-badge">
                    ✅ {{ t('activePresetBadge') }}
                  </span>
                </div>
                <div class="preset-desc">{{ lang === 'zh' ? p.descZh : p.descEn }}</div>
              </div>
            </div>

            <!-- Dedicated Action Buttons for Each Preset -->
            <div class="preset-card-actions">
              <button 
                class="preset-card-btn apply-btn"
                @click="applyPreset(p)"
                :title="t('applyPresetBtn')"
              >
                ⚡ {{ t('applyPresetBtn') }}
              </button>
              <button 
                class="preset-card-btn restore-btn"
                @click="restorePreset(p)"
                :title="t('restorePresetHint')"
              >
                ↺ {{ t('restorePresetBtn') }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Toast Notification for Restore / Apply feedback -->
    <transition name="toast-fade">
      <div v-if="toastMessage" class="toast-notification">
        {{ toastMessage }}
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

// --- 多语言国际化 (i18n) 与视角模式 ---
const lang = ref('zh')
const viewLayoutMode = ref('dual') // 'dual' (主俯并排) | 'front' (仅主视) | 'top' (仅俯视)

const setLayoutMode = (mode) => {
  viewLayoutMode.value = mode
  setTimeout(() => {
    fitToScreen()
  }, 60)
}

const TRANSLATIONS = {
  zh: {
    brandSubtitle: '桌面显示器与外设布局搭配',
    deskWidth: '桌宽',
    deskDepth: '进深',
    presets: '搭配预设',
    presetsTitle: '⚡ 快速布局预设',
    accessories: '外设配件',
    accessoriesTitle: '🖲️ 桌面配件与外设',
    fitScreen: '适应屏幕',
    fitScreenTitle: '自适应画布居中',
    addMonitor: '添加显示器',
    deskSize: '🪑 桌面尺寸 (cm)',
    deskSettings: '调整桌面环境',
    deskRulerText: 'cm 桌面范围',
    keyboard: '⌨️ 机械键盘',
    keyboardLayout: '键盘配列:',
    mouse: '🖱️ 独立鼠标',
    speakers: '🔊 桌面监听音箱 (对箱)',
    speakerLeft: '左声道音箱',
    speakerRight: '右声道音箱',
    laptop: '💻 MacBook 笔记本',
    editSelected: '✏️ 编辑选中显示器',
    screenSize: '屏幕尺寸 (英寸)',
    quickSize: '快速选尺寸',
    aspectRatio: '屏幕比例',
    customRatio: '自定义比例',
    mountType: '安装与支架方式',
    standOriginal: '🖥️ 原装底座',
    standArm: '🦾 气压机械臂',
    standNone: '🧱 壁挂/悬浮',
    lightBar: '💡 开启屏幕挂灯 (Screen Light Bar)',
    lightBarBtn: '切换屏幕挂灯',
    rotateBtn: '旋转 90°',
    physicalDimensions: '真实长宽：',
    projectedWidth: '投影横向占位',
    toLandscape: '切为横屏',
    toPortrait: '切为竖屏',
    delete: '删除',
    putOnDesk: '放入桌面',
    emptyTitle: '当前桌面没有显示器',
    emptyBtn: '点击添加显示器',
    dragHint: '可左右拖移',
    dragBothHint: '可拖拽移动位置与深度',
    edit: '编辑',
    done: '完成配置',
    doneAndCenter: '完成并居中适应',
    zoomIn: '放大',
    zoomOut: '缩小',
    frontView: '主视图',
    topView: '俯视图',
    dualView: '双视并排',
    frontViewOnly: '仅主视',
    topViewOnly: '仅俯视',
    frontViewTitle: '切换为仅主视平视视角',
    topViewTitle: '切换为仅俯视全景视角',
    dualViewTitle: '同时显示主视与俯视双视角 (左主右俯)',
    frontElevationBadge: '正视立面 (近大远小)',
    topPlanBadge: '俯视平面 (角度与进深)',
    swivelAngle: '向前偏转折角 (内折)',
    swivelLeft30: '左内折 30°',
    swivelRight30: '右内折 30°',
    flat: '平直 0°',
    swivelHint: '副屏向前偏转朝向用户，勾股投影减少横向占位，营造环抱沉浸包围感',
    viewSweetSpot: '视距黄金中心',
    controlPanel: '配置工具栏',
    monitorsList: '显示器列表',
    monitorsCount: '台显示器',
    currentPresetTitle: '当前搭配方案',
    activePresetBadge: '当前生效中',
    applyPresetBtn: '应用方案',
    restorePresetBtn: '恢复此预设默认',
    restorePresetHint: '将该预设的尺寸、角度及外设重置为初始出厂参数',
    switchPreset: '切换/管理预设',
    allPresetsBtn: '全部预设模版'
  },
  en: {
    brandSubtitle: 'Desktop Monitor & Battlestation Planner',
    currentPresetTitle: 'Current Setup Preset',
    activePresetBadge: 'Active',
    applyPresetBtn: 'Apply Preset',
    restorePresetBtn: 'Restore Preset Defaults',
    restorePresetHint: 'Reset monitors, dimensions and accessories to default factory layout',
    switchPreset: 'Change Preset',
    allPresetsBtn: 'Browse Presets',
    deskWidth: 'Width',
    deskDepth: 'Depth',
    presets: 'Presets',
    presetsTitle: '⚡ Quick Setup Presets',
    accessories: 'Accessories',
    accessoriesTitle: '🖲️ Desktop Accessories',
    fitScreen: 'Fit View',
    fitScreenTitle: 'Auto-Fit & Center View',
    addMonitor: 'Add Monitor',
    deskSize: '🪑 Desk Size (cm)',
    deskSettings: 'Desk Environment Settings',
    deskRulerText: 'cm Desk Surface',
    keyboard: '⌨️ Mechanical Keyboard',
    keyboardLayout: 'Keyboard Layout:',
    mouse: '🖱️ Independent Mouse',
    speakers: '🔊 Studio Monitors (Pair)',
    speakerLeft: 'Left Speaker',
    speakerRight: 'Right Speaker',
    laptop: '💻 MacBook Laptop',
    editSelected: '✏️ Edit Selected Display',
    screenSize: 'Screen Size (Inches)',
    quickSize: 'Quick Sizes',
    aspectRatio: 'Aspect Ratio',
    customRatio: 'Custom',
    mountType: 'Mounting & Stand',
    standOriginal: '🖥️ Desktop Stand',
    standArm: '🦾 Monitor Arm',
    standNone: '🧱 Wall Mount',
    lightBar: '💡 Screen Light Bar',
    lightBarBtn: 'Toggle Screen Light Bar',
    rotateBtn: 'Rotate 90°',
    physicalDimensions: 'Dimensions: ',
    projectedWidth: 'Projected Width',
    toLandscape: 'To Landscape',
    toPortrait: 'To Portrait',
    delete: 'Delete',
    putOnDesk: 'Place on Desk',
    emptyTitle: 'No displays on the desk',
    emptyBtn: 'Click to Add Monitor',
    dragHint: 'Draggable',
    dragBothHint: 'Draggable in position & depth',
    edit: 'Edit',
    done: 'Save Configuration',
    doneAndCenter: 'Done & Center',
    zoomIn: 'Zoom In',
    zoomOut: 'Zoom Out',
    frontView: 'Front View',
    topView: 'Top View',
    dualView: 'Side-by-Side',
    frontViewOnly: 'Front Only',
    topViewOnly: 'Top Only',
    frontViewTitle: 'Front Elevation View',
    topViewTitle: 'Top-Down Plan View',
    dualViewTitle: 'Simultaneous Dual Viewports (Front & Top)',
    frontElevationBadge: 'Front Elevation (Perspective)',
    topPlanBadge: 'Top Plan (Angles & Depth)',
    swivelAngle: 'Inward Swivel Angle',
    swivelLeft30: 'Left 30°',
    swivelRight30: 'Right 30°',
    flat: 'Flat 0°',
    swivelHint: 'Angle side screen forward towards user to save desk width with trigonometric projection',
    viewSweetSpot: 'Ergonomic Sweet Spot',
    controlPanel: 'Tools & Inspector',
    monitorsList: 'Displays List',
    monitorsCount: 'Displays'
  }
}

const t = (key) => {
  return TRANSLATIONS[lang.value]?.[key] || TRANSLATIONS['zh'][key] || key
}

const toggleLanguage = () => {
  lang.value = lang.value === 'zh' ? 'en' : 'zh'
  try {
    localStorage.setItem('deskcraft_lang', lang.value)
  } catch (e) {}
}

// --- 基础参数 ---
const PIXEL_SCALE = 15 
const CM_TO_PX = PIXEL_SCALE / 2.54 
const SNAP_THRESHOLD = 15 

// 固定世界坐标参考原点
const FIXED_DESK_X = 100 
const TOP_DESK_Y = 160

// --- 响应式移动端检测 ---
const isMobile = ref(false)
const checkMobile = () => {
  isMobile.value = window.innerWidth <= 768
}

// --- 桌面环境配置 (Width & Depth) ---
const desk = reactive({
  widthCm: 160, 
  heightCm: 4,  
  x: FIXED_DESK_X,         
  y: 520        
})

const deskDepthCm = ref(70)
const deskDepthPx = computed(() => deskDepthCm.value * CM_TO_PX)

const effectiveDeskWidthCm = computed(() => {
  const val = desk.widthCm
  if (val === '' || val === null || val <= 0) return 120
  return Number(val)
})

const deskWidthPx = computed(() => effectiveDeskWidthCm.value * CM_TO_PX)
const deskHeightPx = computed(() => desk.heightCm * CM_TO_PX)

// --- 显示器三角投影占位计算 (勾股定理 / Trigonometric Projection) ---
const getProjectedWidthPx = (m) => {
  const ang = Math.abs(m.swivelAngle || 0)
  if (!ang) return m.widthPx
  return Math.round(m.widthPx * Math.cos(ang * Math.PI / 180))
}

const getProjectedWidthCm = (m) => {
  const ang = Math.abs(m.swivelAngle || 0)
  if (!ang) return m.widthCm
  return Number((m.widthCm * Math.cos(ang * Math.PI / 180)).toFixed(1))
}

// --- 正交正视图边框计算：保持标准矩形高度，同一深度边框等粗，离视点近的变细 ---
const getScreenBorderData = (m) => {
  const W_p = getProjectedWidthPx(m)
  const H = m.heightPx
  const ang = m.swivelAngle || 0

  const BASE_DEPTH_THICKNESS = 6.0 // 远端/基准深度边框厚度 (中间平放显示器及偏折屏远端)
  const MIN_NEAR_THICKNESS = 2.0    // 近端（靠近视点）最细厚度

  let tL = BASE_DEPTH_THICKNESS
  let tR = BASE_DEPTH_THICKNESS

  if (ang !== 0) {
    const absAng = Math.abs(ang)
    // 随偏转角增大，离人近的一侧从 6.0px 平滑收窄至 2.0px
    const shrink = (BASE_DEPTH_THICKNESS - MIN_NEAR_THICKNESS) * Math.min(1.0, Math.sin(absAng * Math.PI / 180) / Math.sin(30 * Math.PI / 180))
    const tNear = Number((BASE_DEPTH_THICKNESS - shrink).toFixed(1))
    const tFar = BASE_DEPTH_THICKNESS

    if (ang > 0) {
      // 左侧副屏向前折：左侧近（细），右侧远（粗，与主屏同深度接缝等粗）
      tL = tNear
      tR = tFar
    } else {
      // 右侧副屏向前折（如绿色显示器）：左侧远（粗，与主屏同深度接缝等粗），右侧近（细）
      tL = tFar
      tR = tNear
    }
  }

  // 内屏四顶点 (与外边矩形平齐，仅上下边随边框粗细微倾斜)
  const pTl = { x: tL, y: tL }
  const pTr = { x: W_p - tR, y: tR }
  const pBr = { x: W_p - tR, y: H - tR }
  const pBl = { x: tL, y: H - tL }

  const innerPolyStr = `${pTl.x},${pTl.y} ${pTr.x},${pTr.y} ${pBr.x},${pBr.y} ${pBl.x},${pBl.y}`

  // 边框 Path（evenodd 填充：外矩形 [0, W_p] x [0, H] 挖空内多边形）
  const borderPath = `M 0 0 L ${W_p} 0 L ${W_p} ${H} L 0 ${H} Z M ${pTl.x} ${pTl.y} L ${pTr.x} ${pTr.y} L ${pBr.x} ${pBr.y} L ${pBl.x} ${pBl.y} Z`

  // 屏幕上方反光玻璃
  const glareMidL = tL + (H - 2 * tL) * 0.42
  const glareMidR = tR + (H - 2 * tR) * 0.42
  const glarePolyStr = `${pTl.x},${pTl.y} ${pTr.x},${pTr.y} ${pTr.x},${glareMidR} ${pTl.x},${glareMidL}`

  // 近端高光细线 X 坐标（仅偏转屏幕在离视点近的锐利薄边上显示高光）
  const highlightX = ang > 0 ? 1.5 : (W_p - 1.5)

  return {
    tL,
    tR,
    innerPolyStr,
    borderPath,
    glarePolyStr,
    highlightX
  }
}


// --- 双视口独立相机系统 (Dual Viewports Zoom & Pan) ---
const frontCanvasRef = ref(null)
const topCanvasRef = ref(null)

const frontZoom = ref(1.0)
const frontPan = reactive({ x: 0, y: 0 })

const topZoom = ref(1.0)
const topPan = reactive({ x: 0, y: 0 })

// 坐标转换：将屏幕 client 坐标转换为特定视口的世界坐标
const screenToWorld = (clientX, clientY, targetView = 'front') => {
  if (targetView === 'top') {
    if (!topCanvasRef.value) return { x: 0, y: 0 }
    const rect = topCanvasRef.value.getBoundingClientRect()
    return {
      x: (clientX - rect.left - topPan.x) / topZoom.value,
      y: (clientY - rect.top - topPan.y) / topZoom.value
    }
  }
  if (!frontCanvasRef.value) return { x: 0, y: 0 }
  const rect = frontCanvasRef.value.getBoundingClientRect()
  return {
    x: (clientX - rect.left - frontPan.x) / frontZoom.value,
    y: (clientY - rect.top - frontPan.y) / frontZoom.value
  }
}

// 主视图缩放与自适应
const zoomInFront = () => {
  frontZoom.value = Math.min(2.5, Number((frontZoom.value * 1.15).toFixed(2)))
}
const zoomOutFront = () => {
  frontZoom.value = Math.max(0.25, Number((frontZoom.value * 0.85).toFixed(2)))
}
const resetFrontView = () => fitFrontToScreen()

const fitFrontToScreen = () => {
  if (!frontCanvasRef.value) return
  const rect = frontCanvasRef.value.getBoundingClientRect()
  const cw = rect.width
  const ch = rect.height
  if (cw <= 0 || ch <= 0) return

  let minX = desk.x
  let maxX = desk.x + deskWidthPx.value
  let minY = desk.y - 140
  let maxY = desk.y + deskHeightPx.value + 60

  monitors.value.forEach(m => {
    const projW = getProjectedWidthPx(m)
    minX = Math.min(minX, m.x)
    maxX = Math.max(maxX, m.x + projW)
    minY = Math.min(minY, m.y)
    maxY = Math.max(maxY, m.y + m.heightPx)
  })

  if (accessories.speakers.enabled) {
    minX = Math.min(minX, desk.x + accessories.speakers.leftXPx)
    maxX = Math.max(maxX, desk.x + accessories.speakers.rightXPx + speakerWidthPx.value)
  }

  const contentW = maxX - minX
  const contentH = maxY - minY
  const padding = isMobile.value ? 20 : 50

  const scaleX = (cw - padding * 2) / contentW
  const scaleY = (ch - padding * 2) / contentH
  const targetZoom = Math.min(scaleX, scaleY, 1.1)

  frontZoom.value = Math.max(0.25, Math.min(1.6, Number(targetZoom.toFixed(2))))
  frontPan.x = (cw - contentW * frontZoom.value) / 2 - minX * frontZoom.value
  frontPan.y = (ch - contentH * frontZoom.value) / 2 - minY * frontZoom.value
}

// 俯视图缩放与自适应
const zoomInTop = () => {
  topZoom.value = Math.min(2.5, Number((topZoom.value * 1.15).toFixed(2)))
}
const zoomOutTop = () => {
  topZoom.value = Math.max(0.25, Number((topZoom.value * 0.85).toFixed(2)))
}
const resetTopView = () => fitTopToScreen()

const fitTopToScreen = () => {
  if (!topCanvasRef.value) return
  const rect = topCanvasRef.value.getBoundingClientRect()
  const cw = rect.width
  const ch = rect.height
  if (cw <= 0 || ch <= 0) return

  let minX = desk.x - 30
  let maxX = desk.x + deskWidthPx.value + 30
  let minY = TOP_DESK_Y - 50
  let maxY = TOP_DESK_Y + deskDepthPx.value + 160 // 包含下方视距中心与椅子

  const contentW = maxX - minX
  const contentH = maxY - minY
  const padding = isMobile.value ? 16 : 40

  const scaleX = (cw - padding * 2) / contentW
  const scaleY = (ch - padding * 2) / contentH
  const targetZoom = Math.min(scaleX, scaleY, 1.1)

  topZoom.value = Math.max(0.25, Math.min(1.5, Number(targetZoom.toFixed(2))))
  topPan.x = (cw - contentW * topZoom.value) / 2 - minX * topZoom.value
  topPan.y = (ch - contentH * topZoom.value) / 2 - minY * topZoom.value
}

// 全局双视口同时自适应
const fitToScreen = () => {
  fitFrontToScreen()
  fitTopToScreen()
}

// 主视口滚轮与平移
const onFrontWheel = (e) => {
  if (!frontCanvasRef.value) return
  const rect = frontCanvasRef.value.getBoundingClientRect()
  const mouseX = e.clientX - rect.left
  const mouseY = e.clientY - rect.top

  const factor = e.deltaY < 0 ? 1.1 : 0.9
  const oldZoom = frontZoom.value
  const newZoom = Math.max(0.25, Math.min(2.5, Number((oldZoom * factor).toFixed(2))))
  if (newZoom === oldZoom) return

  frontPan.x = mouseX - (mouseX - frontPan.x) * (newZoom / oldZoom)
  frontPan.y = mouseY - (mouseY - frontPan.y) * (newZoom / oldZoom)
  frontZoom.value = newZoom
}

let isPanningFront = false
let startPanFrontMouse = { x: 0, y: 0 }
let startPanFrontOffset = { x: 0, y: 0 }

const onFrontCanvasPointerDown = (e) => {
  clearSelection()
  isPanningFront = true
  startPanFrontMouse = { x: e.clientX, y: e.clientY }
  startPanFrontOffset = { x: frontPan.x, y: frontPan.y }
}

const onFrontCanvasPointerMove = (e) => {
  if (!isPanningFront) return
  frontPan.x = startPanFrontOffset.x + (e.clientX - startPanFrontMouse.x)
  frontPan.y = startPanFrontOffset.y + (e.clientY - startPanFrontMouse.y)
}

const onFrontCanvasPointerUp = () => {
  isPanningFront = false
}

// 俯视口滚轮与平移
const onTopWheel = (e) => {
  if (!topCanvasRef.value) return
  const rect = topCanvasRef.value.getBoundingClientRect()
  const mouseX = e.clientX - rect.left
  const mouseY = e.clientY - rect.top

  const factor = e.deltaY < 0 ? 1.1 : 0.9
  const oldZoom = topZoom.value
  const newZoom = Math.max(0.25, Math.min(2.5, Number((oldZoom * factor).toFixed(2))))
  if (newZoom === oldZoom) return

  topPan.x = mouseX - (mouseX - topPan.x) * (newZoom / oldZoom)
  topPan.y = mouseY - (mouseY - topPan.y) * (newZoom / oldZoom)
  topZoom.value = newZoom
}

let isPanningTop = false
let startPanTopMouse = { x: 0, y: 0 }
let startPanTopOffset = { x: 0, y: 0 }

const onTopCanvasPointerDown = (e) => {
  clearSelection()
  isPanningTop = true
  startPanTopMouse = { x: e.clientX, y: e.clientY }
  startPanTopOffset = { x: topPan.x, y: topPan.y }
}

const onTopCanvasPointerMove = (e) => {
  if (!isPanningTop) return
  topPan.x = startPanTopOffset.x + (e.clientX - startPanTopMouse.x)
  topPan.y = startPanTopOffset.y + (e.clientY - startPanTopMouse.y)
}

const onTopCanvasPointerUp = () => {
  isPanningTop = false
}

// --- 屏幕比例预设选项 ---
const RATIO_OPTIONS = [
  { label: '16:9', value: '16:9' },
  { label: '16:10', value: '16:10' },
  { label: '21:9', value: '21:9' },
  { label: '32:9', value: '32:9' },
  { label: '3:2', value: '3:2' },
  { label: 'Custom', value: 'custom' }
]

// 表单与状态
const addForm = reactive({
  diagonal: 27,
  ratioString: '16:9',
  customX: 16,
  customY: 9,
  standType: 'stand'
})

const monitors = ref([])
let idCounter = 1
const colors = ['#38bdf8', '#f87171', '#34d399', '#fbbf24', '#a78bfa', '#f472b6']

const selectedId = ref(null)
const selectedMonitor = computed(() => monitors.value.find(m => m.id === selectedId.value))

const clearSelection = () => {
  selectedId.value = null
  selectedAccessory.value = null
}

const setMonitorDiagonal = (m, d) => {
  m.diagonal = d
  updateSelected()
}

const setMonitorRatio = (m, r) => {
  m.ratioString = r
  updateSelected()
}

const setMonitorStand = (m, type) => {
  m.standType = type
  saveToLocalStorage()
}

const toggleLightBar = (m) => {
  m.hasLightBar = !m.hasLightBar
  saveToLocalStorage()
}

// 计算显示器物理 cm 尺寸及像素尺寸
const calculateDimensions = (diagonalInch, rx, ry, isPortrait) => {
  const diagonalPx = diagonalInch * PIXEL_SCALE
  const hypotenuse = Math.sqrt(rx * rx + ry * ry)
  let wPx = (rx / hypotenuse) * diagonalPx
  let hPx = (ry / hypotenuse) * diagonalPx

  const diagonalCm = diagonalInch * 2.54
  let wCm = (rx / hypotenuse) * diagonalCm
  let hCm = (ry / hypotenuse) * diagonalCm

  if (isPortrait) {
    return {
      widthPx: hPx,
      heightPx: wPx,
      widthCm: Number(hCm.toFixed(1)),
      heightCm: Number(wCm.toFixed(1))
    }
  }
  return {
    widthPx: wPx,
    heightPx: hPx,
    widthCm: Number(wCm.toFixed(1)),
    heightCm: Number(hCm.toFixed(1))
  }
}

const addMonitor = () => {
  const d = Number(addForm.diagonal)
  if (d <= 0) return alert("请输入有效尺寸 / Please enter valid size")

  const rx = addForm.ratioString === 'custom' ? Number(addForm.customX) : Number(addForm.ratioString.split(':')[0])
  const ry = addForm.ratioString === 'custom' ? Number(addForm.customY) : Number(addForm.ratioString.split(':')[1])

  if (rx <= 0 || ry <= 0) return alert("比例不能小于或等于0 / Ratio must be greater than 0")

  const { widthPx, heightPx, widthCm, heightCm } = calculateDimensions(d, rx, ry, false)

  const newId = idCounter++
  const newMonitor = {
    id: newId,
    diagonal: d,
    ratioString: addForm.ratioString,
    ratioX: rx,
    ratioY: ry,
    isPortrait: false,
    standType: addForm.standType,
    hasLightBar: false,
    swivelAngle: 0,
    widthPx,
    heightPx,
    widthCm,
    heightCm,
    x: desk.x + (deskWidthPx.value - widthPx) / 2,
    y: desk.y - heightPx - (8 * CM_TO_PX),
    color: colors[(newId - 1) % colors.length]
  }

  monitors.value.push(newMonitor)
  selectedId.value = newId
  activeDrawer.value = null
  saveToLocalStorage()
  setTimeout(() => fitToScreen(), 60)
}

const updateSelected = () => {
  if (!selectedMonitor.value) return
  const m = selectedMonitor.value
  let rx = m.ratioString === 'custom' ? Number(m.ratioX) : Number(m.ratioString.split(':')[0])
  let ry = m.ratioString === 'custom' ? Number(m.ratioY) : Number(m.ratioString.split(':')[1])

  const { widthPx, heightPx, widthCm, heightCm } = calculateDimensions(m.diagonal, rx, ry, m.isPortrait)
  m.widthPx = widthPx
  m.heightPx = heightPx
  m.widthCm = widthCm
  m.heightCm = heightCm
  saveToLocalStorage()
}

const toggleRotation = (m) => {
  m.isPortrait = !m.isPortrait
  const tempW = m.widthPx
  m.widthPx = m.heightPx
  m.heightPx = tempW

  const tempCm = m.widthCm
  m.widthCm = m.heightCm
  m.heightCm = tempCm

  m.y = desk.y - m.heightPx - (6 * CM_TO_PX)
  saveToLocalStorage()
}

const removeMonitor = (id) => {
  monitors.value = monitors.value.filter(m => m.id !== id)
  if (selectedId.value === id) clearSelection()
  if (activeDrawer.value === 'edit') activeDrawer.value = null
  saveToLocalStorage()
}

// --- 支架与机械臂路径计算 (Front View) ---
const getArmJoints = (m) => {
  const projW = getProjectedWidthPx(m)
  const vesaX = m.x + projW / 2
  const vesaY = m.y + m.heightPx / 2
  let clampX = vesaX - 40
  clampX = Math.max(desk.x + 30, Math.min(desk.x + deskWidthPx.value - 30, clampX))
  const jointX = (vesaX + clampX) / 2 - 50
  const jointY = (vesaY + desk.y) / 2 + 10

  return { vesaX, vesaY, jointX, jointY, clampX }
}

const calculateArmPath = (m) => {
  const { vesaX, vesaY, jointX, jointY, clampX } = getArmJoints(m)
  return `M ${clampX} ${desk.y} L ${jointX} ${jointY} L ${vesaX} ${vesaY}`
}

// --- 偏转角度与俯视图 (Top View) 几何参数与坐标计算 ---
const setMonitorSwivel = (m, angle) => {
  m.swivelAngle = angle
  saveToLocalStorage()
}

// 俯视图显示器几何信息
const getTopMonitorCoords = (m) => {
  const mountY = TOP_DESK_Y + (m.depthOffsetPx || (12 * CM_TO_PX))
  const projW = getProjectedWidthPx(m)
  const leftX = m.x
  const centerX = m.x + projW / 2
  const centerY = mountY
  
  let clampX = centerX - 35
  clampX = Math.max(desk.x + 30, Math.min(desk.x + deskWidthPx.value - 30, clampX))
  
  return { leftX, topY: mountY - 10, centerX, centerY, clampX }
}

// 俯视图元素基准 Left 坐标 (保证旋转后与主视图水平投影完全对齐)
const getTopMonitorLeft = (m) => {
  const projW = getProjectedWidthPx(m)
  if (m.swivelAngle > 0) {
    return m.x + projW - m.widthPx
  }
  return m.x
}

const getTopTransformOrigin = (m) => {
  if (m.swivelAngle > 0) return 'right center'
  if (m.swivelAngle < 0) return 'left center'
  return 'center center'
}

// 俯视图机械臂路径计算 (从后桌夹延伸至屏幕背部 VESA)
const getTopArmPath = (m) => {
  const { centerX, centerY, clampX } = getTopMonitorCoords(m)
  const jointX = (clampX + centerX) / 2 - 25
  const jointY = (TOP_DESK_Y + centerY) / 2
  return `M ${clampX} ${TOP_DESK_Y} L ${jointX} ${jointY} L ${centerX} ${centerY}`
}

const getTopArmJoint = (m) => {
  const { centerX, centerY, clampX } = getTopMonitorCoords(m)
  return {
    x: (clampX + centerX) / 2 - 25,
    y: (TOP_DESK_Y + centerY) / 2
  }
}

// 用户视距黄金中心点 (Sweet Spot)
const getUserSweetSpot = () => {
  return {
    x: desk.x + deskWidthPx.value / 2,
    y: TOP_DESK_Y + deskDepthPx.value + (14 * CM_TO_PX)
  }
}

// 俯视图屏幕光锥/视线梯形区域 (前倾朝向用户 +Y)
const getTopViewingCone = (m) => {
  const { centerY } = getTopMonitorCoords(m)
  const sweet = getUserSweetSpot()
  const w = m.widthPx
  const ang = Math.abs(m.swivelAngle || 0) * Math.PI / 180
  const projW = getProjectedWidthPx(m)
  
  let p1x, p1y, p2x, p2y
  if (m.swivelAngle > 0) {
    p2x = m.x + projW
    p2y = centerY
    p1x = m.x
    p1y = centerY + w * Math.sin(ang)
  } else if (m.swivelAngle < 0) {
    p1x = m.x
    p1y = centerY
    p2x = m.x + projW
    p2y = centerY + w * Math.sin(ang)
  } else {
    p1x = m.x
    p1y = centerY
    p2x = m.x + w
    p2y = centerY
  }
  
  return `${p1x},${p1y} ${p2x},${p2y} ${sweet.x + 20},${sweet.y - 10} ${sweet.x - 20},${sweet.y - 10}`
}

// 偏转角度虚线圆弧 (弧线向前突出朝向用户 +Y)
const getTopAngleArc = (m) => {
  const { centerY } = getTopMonitorCoords(m)
  const ang = Math.abs(m.swivelAngle || 0) * Math.PI / 180
  const r = 55
  const projW = getProjectedWidthPx(m)
  if (m.swivelAngle > 0) {
    const anchorX = m.x + projW
    const startX = anchorX - r
    const startY = centerY
    const endX = anchorX - r * Math.cos(ang)
    const endY = centerY + r * Math.sin(ang)
    return `M ${startX} ${startY} A ${r} ${r} 0 0 0 ${endX} ${endY}`
  } else {
    const anchorX = m.x
    const startX = anchorX + r
    const startY = centerY
    const endX = anchorX + r * Math.cos(ang)
    const endY = centerY + r * Math.sin(ang)
    return `M ${startX} ${startY} A ${r} ${r} 0 0 1 ${endX} ${endY}`
  }
}

const getTopAngleTextPos = (m) => {
  const { centerY } = getTopMonitorCoords(m)
  const projW = getProjectedWidthPx(m)
  if (m.swivelAngle > 0) {
    return {
      x: m.x + projW - 65,
      y: centerY + 25
    }
  } else {
    return {
      x: m.x + 65,
      y: centerY + 25
    }
  }
}

// --- 键盘配列配置 ---
const KEYBOARD_LAYOUTS = [
  { value: '61', labelZh: '61 键 (60%)', labelEn: '61-key (60%)', widthCm: 29.5 },
  { value: '87', labelZh: '87 键 (TKL)', labelEn: '87-key (TKL)', widthCm: 36 },
  { value: '98', labelZh: '98 键 (980)', labelEn: '98-key (98%)', widthCm: 39 },
  { value: '104', labelZh: '104 键 (全尺寸)', labelEn: '104-key (Full)', widthCm: 44.5 }
]

// --- 俯视图机械键盘具体键位几何计算 (61 / 87 / 98 / 104) ---
const getKeyColor = (type) => {
  switch (type) {
    case 'accent-esc':
      return { fill: '#0284c7', stroke: '#38bdf8' }
    case 'accent-enter':
      return { fill: '#0d9488', stroke: '#2dd4bf' }
    case 'space':
      return { fill: '#243048', stroke: '#3b4b68' }
    case 'mod':
      return { fill: '#161e2e', stroke: '#2b364a' }
    case 'alpha':
    default:
      return { fill: '#1e283b', stroke: '#334155' }
  }
}

const getTopKeyboardLayout = (layout) => {
  const u = 10
  const gap = 1.2
  const pitch = u + gap
  const kH = 9.2
  const padX = 5
  const padY = 5
  const keys = []
  const leds = []

  const addKey = (x, y, wUnits, type = 'alpha', hUnits = 1) => {
    const w = wUnits * u + (wUnits - 1) * gap
    const h = hUnits * kH + (hUnits - 1) * gap
    keys.push({ x: Number(x.toFixed(1)), y: Number(y.toFixed(1)), w: Number(w.toFixed(1)), h: Number(h.toFixed(1)), type })
  }

  // 15u 标准字母与主输入区 (Alpha Block)
  const generateAlphaBlock = (baseX, startRowY, is60Percent = false) => {
    // Row 1: 数字行
    let curX = baseX
    let curY = startRowY
    addKey(curX, curY, 1, is60Percent ? 'accent-esc' : 'mod')
    curX += pitch
    for (let i = 0; i < 12; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 2, 'mod')

    // Row 2: Tab + QWERTY
    curX = baseX
    curY += kH + gap
    addKey(curX, curY, 1.5, 'mod')
    curX += 1.5 * u + 0.5 * gap + gap
    for (let i = 0; i < 12; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 1.5, 'mod')

    // Row 3: Caps + ASDF
    curX = baseX
    curY += kH + gap
    addKey(curX, curY, 1.75, 'mod')
    curX += 1.75 * u + 0.75 * gap + gap
    for (let i = 0; i < 11; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 2.25, 'accent-enter')

    // Row 4: Shift + ZXCV
    curX = baseX
    curY += kH + gap
    addKey(curX, curY, 2.25, 'mod')
    curX += 2.25 * u + 1.25 * gap + gap
    for (let i = 0; i < 10; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 2.75, 'mod')

    // Row 5: 空格行 (15u)
    curX = baseX
    curY += kH + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 6.25, 'space')
    curX += 6.25 * u + 5.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
  }

  if (layout === '61') {
    // 61 键：纯 60% 5 行紧凑排列
    const totalW = 15 * pitch + padX * 2 - gap
    const totalH = 5 * kH + 4 * gap + padY * 2
    generateAlphaBlock(padX, padY, true)
    return {
      viewBox: `0 0 ${totalW.toFixed(1)} ${totalH.toFixed(1)}`,
      totalW,
      totalH,
      keys,
      leds
    }
  }

  const fRowY = padY
  const alphaStartY = fRowY + kH + 2.8

  if (layout === '87') {
    // 87 键 (TKL)
    let fx = padX
    addKey(fx, fRowY, 1, 'accent-esc')
    fx += pitch + 6.0
    for (let g = 0; g < 3; g++) {
      for (let i = 0; i < 4; i++) {
        addKey(fx, fRowY, 1, 'mod')
        fx += pitch
      }
      fx += 3.5
    }

    generateAlphaBlock(padX, alphaStartY, false)

    const navX = padX + 15 * pitch + 4.5
    let nfx = navX
    for (let i = 0; i < 3; i++) {
      addKey(nfx, fRowY, 1, 'mod')
      nfx += pitch
    }
    let ny = alphaStartY
    for (let r = 0; r < 2; r++) {
      let nx = navX
      for (let c = 0; c < 3; c++) {
        addKey(nx, ny, 1, 'mod')
        nx += pitch
      }
      ny += kH + gap
    }
    const arrowUpY = alphaStartY + 3 * (kH + gap)
    const arrowDownY = alphaStartY + 4 * (kH + gap)
    addKey(navX + pitch, arrowUpY, 1, 'alpha')
    addKey(navX, arrowDownY, 1, 'alpha')
    addKey(navX + pitch, arrowDownY, 1, 'alpha')
    addKey(navX + 2 * pitch, arrowDownY, 1, 'alpha')

    const totalW = navX + 3 * pitch + padX - gap
    const totalH = alphaStartY + 5 * kH + 4 * gap + padY
    return {
      viewBox: `0 0 ${totalW.toFixed(1)} ${totalH.toFixed(1)}`,
      totalW,
      totalH,
      keys,
      leds
    }
  }

  if (layout === '104') {
    // 104 键 (全尺寸，主区 + 编辑区 + 小键盘)
    let fx = padX
    addKey(fx, fRowY, 1, 'accent-esc')
    fx += pitch + 6.0
    for (let g = 0; g < 3; g++) {
      for (let i = 0; i < 4; i++) {
        addKey(fx, fRowY, 1, 'mod')
        fx += pitch
      }
      fx += 3.5
    }

    generateAlphaBlock(padX, alphaStartY, false)

    const navX = padX + 15 * pitch + 4.5
    let nfx = navX
    for (let i = 0; i < 3; i++) {
      addKey(nfx, fRowY, 1, 'mod')
      nfx += pitch
    }
    let ny = alphaStartY
    for (let r = 0; r < 2; r++) {
      let nx = navX
      for (let c = 0; c < 3; c++) {
        addKey(nx, ny, 1, 'mod')
        nx += pitch
      }
      ny += kH + gap
    }
    const arrowUpY = alphaStartY + 3 * (kH + gap)
    const arrowDownY = alphaStartY + 4 * (kH + gap)
    addKey(navX + pitch, arrowUpY, 1, 'alpha')
    addKey(navX, arrowDownY, 1, 'alpha')
    addKey(navX + pitch, arrowDownY, 1, 'alpha')
    addKey(navX + 2 * pitch, arrowDownY, 1, 'alpha')

    // 独立小键盘区
    const numX = navX + 3 * pitch + 5.0
    leds.push({ cx: numX + 8, cy: fRowY + 4, color: '#38bdf8' })
    leds.push({ cx: numX + 18, cy: fRowY + 4, color: '#38bdf8' })
    leds.push({ cx: numX + 28, cy: fRowY + 4, color: '#38bdf8' })

    let nRowY = alphaStartY
    addKey(numX, nRowY, 1, 'mod')
    addKey(numX + pitch, nRowY, 1, 'mod')
    addKey(numX + 2 * pitch, nRowY, 1, 'mod')
    addKey(numX + 3 * pitch, nRowY, 1, 'mod')

    nRowY += kH + gap
    addKey(numX, nRowY, 1, 'alpha')
    addKey(numX + pitch, nRowY, 1, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')
    addKey(numX + 3 * pitch, nRowY, 1, 'mod', 2)

    nRowY += kH + gap
    addKey(numX, nRowY, 1, 'alpha')
    addKey(numX + pitch, nRowY, 1, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')

    nRowY += kH + gap
    addKey(numX, nRowY, 1, 'alpha')
    addKey(numX + pitch, nRowY, 1, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')
    addKey(numX + 3 * pitch, nRowY, 1, 'accent-enter', 2)

    nRowY += kH + gap
    addKey(numX, nRowY, 2, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')

    const totalW = numX + 4 * pitch + padX - gap
    const totalH = alphaStartY + 5 * kH + 4 * gap + padY
    return {
      viewBox: `0 0 ${totalW.toFixed(1)} ${totalH.toFixed(1)}`,
      totalW,
      totalH,
      keys,
      leds
    }
  }

  if (layout === '98') {
    // 98 键 (980 紧凑布局)
    let fx = padX
    addKey(fx, fRowY, 1, 'accent-esc')
    fx += pitch + 3.0
    for (let i = 0; i < 12; i++) {
      addKey(fx, fRowY, 1, 'mod')
      fx += pitch
      if (i === 3 || i === 7) fx += 2.0
    }
    fx += 3.0
    for (let i = 0; i < 4; i++) {
      addKey(fx, fRowY, 1, 'mod')
      fx += pitch
    }

    let curX = padX
    let curY = alphaStartY
    for (let i = 0; i < 13; i++) {
      addKey(curX, curY, 1, i === 0 ? 'mod' : 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 2, 'mod')

    curX = padX
    curY += kH + gap
    addKey(curX, curY, 1.5, 'mod')
    curX += 1.5 * u + 0.5 * gap + gap
    for (let i = 0; i < 12; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 1.5, 'mod')

    curX = padX
    curY += kH + gap
    addKey(curX, curY, 1.75, 'mod')
    curX += 1.75 * u + 0.75 * gap + gap
    for (let i = 0; i < 11; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 2.25, 'accent-enter')

    curX = padX
    curY += kH + gap
    addKey(curX, curY, 2.25, 'mod')
    curX += 2.25 * u + 1.25 * gap + gap
    for (let i = 0; i < 10; i++) {
      addKey(curX, curY, 1, 'alpha')
      curX += pitch
    }
    addKey(curX, curY, 1.75, 'mod')
    curX += 1.75 * u + 0.75 * gap + gap
    addKey(curX + 1.5, curY, 1, 'alpha') // Up Arrow

    curX = padX
    curY += kH + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 1.25, 'mod')
    curX += 1.25 * u + 0.25 * gap + gap
    addKey(curX, curY, 6.25, 'space')
    curX += 6.25 * u + 5.25 * gap + gap
    addKey(curX, curY, 1, 'mod')
    curX += pitch
    addKey(curX + 0.5, curY, 1, 'alpha') // Left
    curX += pitch
    addKey(curX + 0.5, curY, 1, 'alpha') // Down
    curX += pitch
    addKey(curX + 0.5, curY, 1, 'alpha') // Right

    const numX = padX + 15 * pitch + 4.0
    let nRowY = alphaStartY
    addKey(numX, nRowY, 1, 'mod')
    addKey(numX + pitch, nRowY, 1, 'mod')
    addKey(numX + 2 * pitch, nRowY, 1, 'mod')
    addKey(numX + 3 * pitch, nRowY, 1, 'mod')

    nRowY += kH + gap
    addKey(numX, nRowY, 1, 'alpha')
    addKey(numX + pitch, nRowY, 1, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')
    addKey(numX + 3 * pitch, nRowY, 1, 'mod', 2)

    nRowY += kH + gap
    addKey(numX, nRowY, 1, 'alpha')
    addKey(numX + pitch, nRowY, 1, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')

    nRowY += kH + gap
    addKey(numX, nRowY, 1, 'alpha')
    addKey(numX + pitch, nRowY, 1, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')
    addKey(numX + 3 * pitch, nRowY, 1, 'accent-enter', 2)

    nRowY += kH + gap
    addKey(numX, nRowY, 2, 'alpha')
    addKey(numX + 2 * pitch, nRowY, 1, 'alpha')

    const totalW = numX + 4 * pitch + padX - gap
    const totalH = alphaStartY + 5 * kH + 4 * gap + padY
    return {
      viewBox: `0 0 ${totalW.toFixed(1)} ${totalH.toFixed(1)}`,
      totalW,
      totalH,
      keys,
      leds
    }
  }

  return getTopKeyboardLayout('87')
}


// --- 桌面外设与配件状态 ---
const accessories = reactive({
  keyboard: {
    enabled: true,
    layout: '98',
    xPx: 363,
    yDepthPx: 0
  },
  mouse: {
    enabled: true,
    xPx: 621,
    yDepthPx: 0
  },
  speakers: {
    enabled: true,
    leftXPx: 30,
    leftYDepthPx: 0,
    rightXPx: 830,
    rightYDepthPx: 0
  },
  laptop: {
    enabled: false,
    size: 14,
    xPx: 120,
    yDepthPx: 0
  }
})

// 键盘尺寸
const currentKeyboardLayout = computed(() => {
  return KEYBOARD_LAYOUTS.find(l => l.value === accessories.keyboard.layout) || KEYBOARD_LAYOUTS[2]
})
const keyboardWidthPx = computed(() => currentKeyboardLayout.value.widthCm * CM_TO_PX)
const keyboardHeightPx = computed(() => 3.6 * CM_TO_PX)

// 鼠标尺寸
const mouseWidthPx = computed(() => 7.5 * CM_TO_PX)
const mouseHeightPx = computed(() => 3.4 * CM_TO_PX)

// 监听音箱尺寸
const speakerWidthPx = computed(() => 13 * CM_TO_PX)
const speakerHeightPx = computed(() => 22 * CM_TO_PX)

// 笔记本尺寸
const laptopWidthPx = computed(() => (accessories.laptop.size === 16 ? 36 : 31) * CM_TO_PX)
const laptopHeightPx = computed(() => (accessories.laptop.size === 16 ? 25 : 22) * CM_TO_PX)

const updateLaptopDimensions = () => {
  saveToLocalStorage()
}

// --- 主视图外设拖动 (Front View Accessory Drag) ---
const selectedAccessory = ref(null)
let isDraggingAccessory = false
let activeDragTarget = null 
let startAccessoryMouseX = 0
let startAccessoryInitialX = 0

const startAccessoryDrag = (key, event) => {
  event.stopPropagation()
  isDraggingAccessory = true
  activeDragTarget = key
  selectedAccessory.value = key
  clearSelection()

  const worldPos = screenToWorld(event.clientX, event.clientY, 'front')
  startAccessoryMouseX = worldPos.x
  startAccessoryInitialX = accessories[key].xPx

  window.addEventListener('pointermove', onAccessoryPointerMove)
  window.addEventListener('pointerup', onAccessoryPointerUp)
  window.addEventListener('pointercancel', onAccessoryPointerUp)
}

const startSpeakerDrag = (side, event) => {
  event.stopPropagation()
  isDraggingAccessory = true
  activeDragTarget = side === 'left' ? 'speaker-left' : 'speaker-right'
  selectedAccessory.value = activeDragTarget
  clearSelection()

  const worldPos = screenToWorld(event.clientX, event.clientY, 'front')
  startAccessoryMouseX = worldPos.x
  startAccessoryInitialX = side === 'left' ? accessories.speakers.leftXPx : accessories.speakers.rightXPx

  window.addEventListener('pointermove', onAccessoryPointerMove)
  window.addEventListener('pointerup', onAccessoryPointerUp)
  window.addEventListener('pointercancel', onAccessoryPointerUp)
}

const onAccessoryPointerMove = (event) => {
  if (!isDraggingAccessory) return
  const worldPos = screenToWorld(event.clientX, event.clientY, 'front')
  const deltaX = worldPos.x - startAccessoryMouseX
  let targetX = startAccessoryInitialX + deltaX

  if (activeDragTarget === 'keyboard') {
    targetX = Math.max(0, Math.min(deskWidthPx.value - keyboardWidthPx.value, targetX))
    accessories.keyboard.xPx = targetX
  } else if (activeDragTarget === 'mouse') {
    targetX = Math.max(0, Math.min(deskWidthPx.value - mouseWidthPx.value, targetX))
    accessories.mouse.xPx = targetX
  } else if (activeDragTarget === 'laptop') {
    targetX = Math.max(0, Math.min(deskWidthPx.value - laptopWidthPx.value, targetX))
    accessories.laptop.xPx = targetX
  } else if (activeDragTarget === 'speaker-left') {
    targetX = Math.max(0, Math.min(deskWidthPx.value - speakerWidthPx.value, targetX))
    accessories.speakers.leftXPx = targetX
  } else if (activeDragTarget === 'speaker-right') {
    targetX = Math.max(0, Math.min(deskWidthPx.value - speakerWidthPx.value, targetX))
    accessories.speakers.rightXPx = targetX
  }
}

const onAccessoryPointerUp = () => {
  isDraggingAccessory = false
  activeDragTarget = null
  window.removeEventListener('pointermove', onAccessoryPointerMove)
  window.removeEventListener('pointerup', onAccessoryPointerUp)
  window.removeEventListener('pointercancel', onAccessoryPointerUp)
  saveToLocalStorage()
}

// --- 俯视图外设拖动 (Top View Accessory Drag - X & Depth Y) ---
let isDraggingTopAccessory = false
let activeTopDragTarget = null
let startTopAccMouse = { x: 0, y: 0 }
let startTopAccInit = { x: 0, y: 0 }

const startTopAccessoryDrag = (key, event) => {
  event.stopPropagation()
  isDraggingTopAccessory = true
  activeTopDragTarget = key
  selectedAccessory.value = key
  clearSelection()

  const worldPos = screenToWorld(event.clientX, event.clientY, 'top')
  startTopAccMouse.x = worldPos.x
  startTopAccMouse.y = worldPos.y
  startTopAccInit.x = accessories[key].xPx
  startTopAccInit.y = accessories[key].yDepthPx || 0

  window.addEventListener('pointermove', onTopAccessoryPointerMove)
  window.addEventListener('pointerup', onTopAccessoryPointerUp)
  window.addEventListener('pointercancel', onTopAccessoryPointerUp)
}

const startTopSpeakerDrag = (side, event) => {
  event.stopPropagation()
  isDraggingTopAccessory = true
  activeTopDragTarget = side === 'left' ? 'speaker-left' : 'speaker-right'
  selectedAccessory.value = activeTopDragTarget
  clearSelection()

  const worldPos = screenToWorld(event.clientX, event.clientY, 'top')
  startTopAccMouse.x = worldPos.x
  startTopAccMouse.y = worldPos.y
  if (side === 'left') {
    startTopAccInit.x = accessories.speakers.leftXPx
    startTopAccInit.y = accessories.speakers.leftYDepthPx || 0
  } else {
    startTopAccInit.x = accessories.speakers.rightXPx
    startTopAccInit.y = accessories.speakers.rightYDepthPx || 0
  }

  window.addEventListener('pointermove', onTopAccessoryPointerMove)
  window.addEventListener('pointerup', onTopAccessoryPointerUp)
  window.addEventListener('pointercancel', onTopAccessoryPointerUp)
}

const onTopAccessoryPointerMove = (event) => {
  if (!isDraggingTopAccessory) return
  const worldPos = screenToWorld(event.clientX, event.clientY, 'top')
  const deltaX = worldPos.x - startTopAccMouse.x
  const deltaY = worldPos.y - startTopAccMouse.y

  if (activeTopDragTarget === 'keyboard') {
    const targetX = Math.max(0, Math.min(deskWidthPx.value - keyboardWidthPx.value, startTopAccInit.x + deltaX))
    const targetY = Math.max(-80, Math.min(deskDepthPx.value - 120, startTopAccInit.y - deltaY))
    accessories.keyboard.xPx = targetX
    accessories.keyboard.yDepthPx = targetY
  } else if (activeTopDragTarget === 'mouse') {
    const targetX = Math.max(0, Math.min(deskWidthPx.value - mouseWidthPx.value, startTopAccInit.x + deltaX))
    const targetY = Math.max(-85, Math.min(deskDepthPx.value - 120, startTopAccInit.y - deltaY))
    accessories.mouse.xPx = targetX
    accessories.mouse.yDepthPx = targetY
  } else if (activeTopDragTarget === 'laptop') {
    const targetX = Math.max(0, Math.min(deskWidthPx.value - laptopWidthPx.value, startTopAccInit.x + deltaX))
    const targetY = Math.max(-60, Math.min(deskDepthPx.value - 120, startTopAccInit.y - deltaY))
    accessories.laptop.xPx = targetX
    accessories.laptop.yDepthPx = targetY
  } else if (activeTopDragTarget === 'speaker-left') {
    const targetX = Math.max(0, Math.min(deskWidthPx.value - speakerWidthPx.value, startTopAccInit.x + deltaX))
    const targetY = Math.max(-10, Math.min(deskDepthPx.value - 80, startTopAccInit.y + deltaY))
    accessories.speakers.leftXPx = targetX
    accessories.speakers.leftYDepthPx = targetY
  } else if (activeTopDragTarget === 'speaker-right') {
    const targetX = Math.max(0, Math.min(deskWidthPx.value - speakerWidthPx.value, startTopAccInit.x + deltaX))
    const targetY = Math.max(-10, Math.min(deskDepthPx.value - 80, startTopAccInit.y + deltaY))
    accessories.speakers.rightXPx = targetX
    accessories.speakers.rightYDepthPx = targetY
  }
}

const onTopAccessoryPointerUp = () => {
  isDraggingTopAccessory = false
  activeTopDragTarget = null
  window.removeEventListener('pointermove', onTopAccessoryPointerMove)
  window.removeEventListener('pointerup', onTopAccessoryPointerUp)
  window.removeEventListener('pointercancel', onTopAccessoryPointerUp)
  saveToLocalStorage()
}

// --- 主视图显示器拖拽 (Front View Monitor Drag with Projected Width Snapping) ---
const draggingId = ref(null)
let dragPointerId = null
let dragStartOffset = { x: 0, y: 0 }
const activeSnapGuide = ref(null)

// --- 根据俯视图深度坐标自动计算主视图中的前后层级 (Z-Index) ---
const getFrontDepthZIndex = (type, target = null) => {
  if (type === 'monitor') {
    const m = target
    if (!m) return 10
    if (draggingId.value === m.id || selectedMonitor.value?.id === m.id) return 1000
    // 俯视图深度：安装点基础深度 + 前折偏转伸向用户的纵深
    const baseY = m.depthOffsetPx || (12 * CM_TO_PX)
    const forwardSwing = m.swivelAngle ? Math.round(m.widthPx * Math.sin(Math.abs(m.swivelAngle) * Math.PI / 180) * 0.85) : 0
    return Math.max(10, Math.round(baseY + forwardSwing) + (m.id || 0))
  }
  if (type === 'speaker-left') {
    if (selectedAccessory.value === 'speaker-left' || activeDragTarget === 'speaker-left') return 1000
    return Math.max(5, Math.round(25 + (accessories.speakers.leftYDepthPx || 0)))
  }
  if (type === 'speaker-right') {
    if (selectedAccessory.value === 'speaker-right' || activeDragTarget === 'speaker-right') return 1000
    return Math.max(5, Math.round(25 + (accessories.speakers.rightYDepthPx || 0)))
  }
  if (type === 'keyboard') {
    if (selectedAccessory.value === 'keyboard' || activeDragTarget === 'keyboard') return 1000
    const topY = deskDepthPx.value - (14 * CM_TO_PX) - (15 * CM_TO_PX) - (accessories.keyboard.yDepthPx || 0)
    return Math.max(30, Math.round(topY + 14 * CM_TO_PX))
  }
  if (type === 'mouse') {
    if (selectedAccessory.value === 'mouse' || activeDragTarget === 'mouse') return 1000
    const topY = deskDepthPx.value - (12 * CM_TO_PX) - (16 * CM_TO_PX) - (accessories.mouse.yDepthPx || 0)
    return Math.max(30, Math.round(topY + 12 * CM_TO_PX))
  }
  if (type === 'laptop') {
    if (selectedAccessory.value === 'laptop' || activeDragTarget === 'laptop') return 1000
    const topY = deskDepthPx.value - laptopHeightPx.value - (12 * CM_TO_PX) - (accessories.laptop.yDepthPx || 0)
    return Math.max(20, Math.round(topY + laptopHeightPx.value))
  }
  return 10
}

const startMonitorDrag = (id, event) => {
  event.stopPropagation()
  draggingId.value = id
  selectedId.value = id
  selectedAccessory.value = null
  dragPointerId = event.pointerId
  event.target.setPointerCapture?.(event.pointerId)

  const m = monitors.value.find(item => item.id === id)
  if (!m) return

  const worldPos = screenToWorld(event.clientX, event.clientY, 'front')
  dragStartOffset = {
    x: worldPos.x - m.x,
    y: worldPos.y - m.y
  }

  window.addEventListener('pointermove', onMonitorPointerMove)
  window.addEventListener('pointerup', onMonitorPointerUp)
  window.addEventListener('pointercancel', onMonitorPointerUp)
}

const onMonitorPointerMove = (event) => {
  if (draggingId.value === null) return
  const current = monitors.value.find(m => m.id === draggingId.value)
  if (!current) return

  const worldPos = screenToWorld(event.clientX, event.clientY, 'front')
  let targetX = worldPos.x - dragStartOffset.x
  let targetY = worldPos.y - dragStartOffset.y

  activeSnapGuide.value = null
  const curProjW = getProjectedWidthPx(current)

  // 1. 桌面顶部吸附
  if (
    targetX + curProjW > desk.x - SNAP_THRESHOLD &&
    targetX < desk.x + deskWidthPx.value + SNAP_THRESHOLD
  ) {
    if (Math.abs((targetY + current.heightPx) - desk.y) < SNAP_THRESHOLD) {
      targetY = desk.y - current.heightPx
      activeSnapGuide.value = {
        type: 'horizontal',
        style: {
          left: `${desk.x}px`,
          top: `${desk.y}px`,
          width: `${deskWidthPx.value}px`
        }
      }
    }
  }

  // 2. 显示器之间吸附 (严格基于投影宽度计算)
  monitors.value.forEach(m => {
    if (m.id === current.id) return
    const mProjW = getProjectedWidthPx(m)

    const isYAligned = targetY < m.y + m.heightPx + SNAP_THRESHOLD && targetY + current.heightPx > m.y - SNAP_THRESHOLD
    const isXAligned = targetX < m.x + mProjW + SNAP_THRESHOLD && targetX + curProjW > m.x - SNAP_THRESHOLD

    if (isYAligned) {
      if (Math.abs(targetX - m.x) < SNAP_THRESHOLD) targetX = m.x
      if (Math.abs(targetX - (m.x + mProjW)) < SNAP_THRESHOLD) targetX = m.x + mProjW
      if (Math.abs((targetX + curProjW) - m.x) < SNAP_THRESHOLD) targetX = m.x - curProjW
      if (Math.abs((targetX + curProjW) - (m.x + mProjW)) < SNAP_THRESHOLD) targetX = m.x + mProjW - curProjW
    }

    if (isXAligned) {
      if (Math.abs(targetY - m.y) < SNAP_THRESHOLD) targetY = m.y
      if (Math.abs(targetY - (m.y + m.heightPx)) < SNAP_THRESHOLD) targetY = m.y + m.heightPx
      if (Math.abs((targetY + current.heightPx) - m.y) < SNAP_THRESHOLD) targetY = m.y - current.heightPx
      if (Math.abs((targetY + current.heightPx) - (m.y + m.heightPx)) < SNAP_THRESHOLD) targetY = m.y + m.heightPx - current.heightPx
    }
  })

  // 3. 严格防重叠与桌面防穿透
  for (let iter = 0; iter < 2; iter++) {
    monitors.value.forEach(m => {
      if (m.id === current.id) return
      const mProjW = getProjectedWidthPx(m)
      const isOverlapping = targetX < m.x + mProjW && targetX + curProjW > m.x && targetY < m.y + m.heightPx && targetY + current.heightPx > m.y
      if (isOverlapping) {
        const overlapLeft = (targetX + curProjW) - m.x
        const overlapRight = (m.x + mProjW) - targetX
        const overlapTop = (targetY + current.heightPx) - m.y
        const overlapBottom = (m.y + m.heightPx) - targetY
        const minOverlap = Math.min(overlapLeft, overlapRight, overlapTop, overlapBottom)

        if (minOverlap === overlapLeft) targetX = m.x - curProjW
        else if (minOverlap === overlapRight) targetX = m.x + mProjW
        else if (minOverlap === overlapTop) targetY = m.y - current.heightPx
        else if (minOverlap === overlapBottom) targetY = m.y + m.heightPx
      }
    })

    const isOverlappingDesk = targetX < desk.x + deskWidthPx.value && targetX + curProjW > desk.x && targetY < desk.y + deskHeightPx.value && targetY + current.heightPx > desk.y
    if (isOverlappingDesk) {
      const overlapLeft = (targetX + curProjW) - desk.x
      const overlapRight = (desk.x + deskWidthPx.value) - targetX
      const overlapTop = (targetY + current.heightPx) - desk.y
      const overlapBottom = (desk.y + deskHeightPx.value) - targetY
      const minOverlap = Math.min(overlapLeft, overlapRight, overlapTop, overlapBottom)

      if (minOverlap === overlapLeft) targetX = desk.x - curProjW
      else if (minOverlap === overlapRight) targetX = desk.x + deskWidthPx.value
      else if (minOverlap === overlapTop) targetY = desk.y - current.heightPx
      else if (minOverlap === overlapBottom) targetY = desk.y + deskHeightPx.value
    }
  }

  current.x = targetX
  current.y = targetY
}

const onMonitorPointerUp = () => {
  draggingId.value = null
  dragPointerId = null
  activeSnapGuide.value = null
  window.removeEventListener('pointermove', onMonitorPointerMove)
  window.removeEventListener('pointerup', onMonitorPointerUp)
  window.removeEventListener('pointercancel', onMonitorPointerUp)
  saveToLocalStorage()
}

// --- 俯视图显示器拖动 (Top View Monitor Drag - X & Depth Y) ---
let isDraggingTopMonitor = false
let topMonDragId = null
let startTopMonMouse = { x: 0, y: 0 }
let startTopMonInit = { x: 0, y: 0 }

const startTopMonitorDrag = (id, event) => {
  event.stopPropagation()
  isDraggingTopMonitor = true
  topMonDragId = id
  selectedId.value = id
  selectedAccessory.value = null

  const m = monitors.value.find(item => item.id === id)
  if (!m) return

  const worldPos = screenToWorld(event.clientX, event.clientY, 'top')
  startTopMonMouse.x = worldPos.x
  startTopMonMouse.y = worldPos.y
  startTopMonInit.x = m.x
  startTopMonInit.y = m.depthOffsetPx || (12 * CM_TO_PX)

  window.addEventListener('pointermove', onTopMonitorPointerMove)
  window.addEventListener('pointerup', onTopMonitorPointerUp)
  window.addEventListener('pointercancel', onTopMonitorPointerUp)
}

const onTopMonitorPointerMove = (event) => {
  if (!isDraggingTopMonitor || topMonDragId === null) return
  const current = monitors.value.find(m => m.id === topMonDragId)
  if (!current) return

  const worldPos = screenToWorld(event.clientX, event.clientY, 'top')
  const deltaX = worldPos.x - startTopMonMouse.x
  const deltaY = worldPos.y - startTopMonMouse.y

  const projW = getProjectedWidthPx(current)
  let targetX = startTopMonInit.x + deltaX
  targetX = Math.max(desk.x, Math.min(desk.x + deskWidthPx.value - projW, targetX))

  let targetDepth = startTopMonInit.y + deltaY
  targetDepth = Math.max(4 * CM_TO_PX, Math.min(deskDepthPx.value - (12 * CM_TO_PX), targetDepth))

  current.x = targetX
  current.depthOffsetPx = targetDepth
}

const onTopMonitorPointerUp = () => {
  isDraggingTopMonitor = false
  topMonDragId = null
  window.removeEventListener('pointermove', onTopMonitorPointerMove)
  window.removeEventListener('pointerup', onTopMonitorPointerUp)
  window.removeEventListener('pointercancel', onTopMonitorPointerUp)
  saveToLocalStorage()
}

// --- 移动端底部抽屉与弹窗 ---
const activeDrawer = ref(null)
const showPresetsModal = ref(false)

const openAddSheet = () => {
  activeDrawer.value = 'add'
}

const openDeskSettings = () => {
  if (isMobile.value) {
    activeDrawer.value = 'desk'
  }
}

const toggleAccessoriesModal = () => {
  if (isMobile.value) {
    activeDrawer.value = 'accessories'
  }
}

// 设置默认桌面布局 (160cm 宽 × 70cm 深: 左侧 27" 竖屏副屏 + 32" 横屏主屏 + 98 键 + 独立鼠标 + 对箱音箱 + 机械臂)
const setDefaultSetup = () => {
  desk.x = FIXED_DESK_X
  desk.widthCm = 160
  deskDepthCm.value = 70
  
  const d27 = calculateDimensions(27, 16, 9, true)
  const d32 = calculateDimensions(32, 16, 9, false)

  const totalMonitorsWidth = d27.widthPx + d32.widthPx
  const startX = desk.x + (deskWidthPx.value - totalMonitorsWidth) / 2
  
  const m27X = startX
  const m27Y = desk.y - d27.heightPx - (7.5 * CM_TO_PX)

  const m32X = startX + d27.widthPx
  const m32Y = desk.y - d32.heightPx - (13 * CM_TO_PX)

  monitors.value = [
    {
      id: 1,
      diagonal: 27,
      ratioString: '16:9',
      ratioX: 16,
      ratioY: 9,
      isPortrait: true,
      standType: 'arm',
      hasLightBar: false,
      swivelAngle: 0,
      widthPx: d27.widthPx,
      heightPx: d27.heightPx,
      widthCm: d27.widthCm,
      heightCm: d27.heightCm,
      x: m27X,
      y: m27Y,
      depthOffsetPx: 12 * CM_TO_PX,
      color: colors[1]
    },
    {
      id: 2,
      diagonal: 32,
      ratioString: '16:9',
      ratioX: 16,
      ratioY: 9,
      isPortrait: false,
      standType: 'arm',
      hasLightBar: true,
      swivelAngle: 0,
      widthPx: d32.widthPx,
      heightPx: d32.heightPx,
      widthCm: d32.widthCm,
      heightCm: d32.heightCm,
      x: m32X,
      y: m32Y,
      depthOffsetPx: 10 * CM_TO_PX,
      color: colors[0]
    }
  ]
  idCounter = 3

  accessories.keyboard.enabled = true
  accessories.keyboard.layout = '98'
  accessories.keyboard.yDepthPx = 0
  accessories.mouse.enabled = true
  accessories.mouse.yDepthPx = 0
  accessories.speakers.enabled = true
  accessories.speakers.leftYDepthPx = 0
  accessories.speakers.rightYDepthPx = 0
  accessories.laptop.enabled = false
  accessories.laptop.yDepthPx = 0

  const kbW = 39 * CM_TO_PX
  accessories.keyboard.xPx = Math.round((deskWidthPx.value - kbW) / 2)
  accessories.mouse.xPx = accessories.keyboard.xPx + Math.round(kbW) + 24

  accessories.speakers.leftXPx = 30
  accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 30
}

// --- 常用布局预设模版 ---
const PRESETS = [
  {
    id: 'dual-flagship-27v-32h',
    icon: '🌟',
    nameZh: '双屏旗舰 (左27"竖屏 + 右32"主屏 + 98键 + 双机械臂)',
    nameEn: 'Dual Flagship (27" Vertical + 32" Main + 98-Key + Dual Arms)',
    descZh: '160×70cm 桌面：左侧 27" 竖屏副屏 + 32" 悬空主屏 + 98 键键盘 + 独立鼠标 + 机械臂 + 监听音箱',
    descEn: '160×70cm desk: Left 27" vertical + Right 32" main screen with dual arms, 98-key keyboard & studio speakers',
    action: () => {
      setDefaultSetup()
    }
  },
  {
    id: 'single-32',
    icon: '🖥️',
    nameZh: '经典单屏 (32" 悬空立架 + 104键 + 鼠标 + 音箱)',
    nameEn: 'Classic Single (32" Stand + 104-Key + Mouse + Audio)',
    descZh: '140×70cm 桌面：单台 32 寸大屏悬空黄金高度 + 104 键全尺寸键盘 + 独立鼠标 + 监听音箱',
    descEn: '140×70cm desk: 32" screen at ergonomic elevated height with 104-key keyboard, mouse & studio monitors',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 140
      deskDepthCm.value = 70
      const d = calculateDimensions(32, 16, 9, false)
      const monX = desk.x + (deskWidthPx.value - d.widthPx) / 2
      const monY = desk.y - d.heightPx - (12 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 32, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'stand', hasLightBar: true, swivelAngle: 0,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: monX, y: monY, color: colors[0], depthOffsetPx: 10 * CM_TO_PX
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '104'
      accessories.keyboard.yDepthPx = 0
      accessories.mouse.enabled = true
      accessories.mouse.yDepthPx = 0
      accessories.speakers.enabled = true
      accessories.speakers.leftYDepthPx = 0
      accessories.speakers.rightYDepthPx = 0
      accessories.laptop.enabled = false
      const kbW = 44.5 * CM_TO_PX
      accessories.keyboard.xPx = Math.round((deskWidthPx.value - kbW) / 2)
      accessories.mouse.xPx = accessories.keyboard.xPx + Math.round(kbW) + 24
      accessories.speakers.leftXPx = 35
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 35
      idCounter = 2
    }
  },
  {
    id: 'dual-27',
    icon: '💼',
    nameZh: '双 27" 横向办公 (160cm 桌面无遮挡)',
    nameEn: 'Dual 27" Productivity (160cm Desk)',
    descZh: '经典双屏并排，160×70cm 宽桌面，显示器与左右监听音箱完全无重叠',
    descEn: 'Classic dual 27" side-by-side on 160×70cm desk with zero speaker overlap',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 160
      deskDepthCm.value = 70
      const d1 = calculateDimensions(27, 16, 9, false)
      const d2 = calculateDimensions(27, 16, 9, false)
      const totalW = d1.widthPx + d2.widthPx
      const startX = desk.x + (deskWidthPx.value - totalW) / 2
      const monY = desk.y - d1.heightPx - (10 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: true,
          widthPx: d1.widthPx, heightPx: d1.heightPx, widthCm: d1.widthCm, heightCm: d1.heightCm,
          x: startX, y: monY, color: colors[0], swivelAngle: 0, depthOffsetPx: 10 * CM_TO_PX
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: false,
          widthPx: d2.widthPx, heightPx: d2.heightPx, widthCm: d2.widthCm, heightCm: d2.heightCm,
          x: startX + d1.widthPx, y: monY, color: colors[1], swivelAngle: 0, depthOffsetPx: 10 * CM_TO_PX
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '98'
      accessories.keyboard.yDepthPx = 0
      accessories.mouse.enabled = true
      accessories.mouse.yDepthPx = 0
      accessories.speakers.enabled = true
      accessories.speakers.leftYDepthPx = 0
      accessories.speakers.rightYDepthPx = 0
      accessories.laptop.enabled = false

      const kbW = 39 * CM_TO_PX
      accessories.keyboard.xPx = Math.round((deskWidthPx.value - kbW) / 2)
      accessories.mouse.xPx = accessories.keyboard.xPx + Math.round(kbW) + 24
      accessories.speakers.leftXPx = 30
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 30
      idCounter = 3
    }
  },
  {
    id: 'triple-27',
    icon: '🏎️',
    nameZh: '三屏横向环抱 (27" × 3，左右前倾30°)',
    nameEn: 'Triple 27" Panoramic Wrap (30° Inward Slant)',
    descZh: '190×75cm 宽屏大桌：中间平放，左右两侧各向前倾折 30° 黄金视距环抱，左右音箱完全无遮挡',
    descEn: '190×75cm desk: Center flat with left & right angled 30° inward toward user, zero speaker overlap',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 190
      deskDepthCm.value = 75
      const d = calculateDimensions(27, 16, 9, false)
      const projW = Math.round(d.widthPx * Math.cos(30 * Math.PI / 180))
      const totalW = projW + d.widthPx + projW
      const startX = desk.x + (deskWidthPx.value - totalW) / 2
      const monY = desk.y - d.heightPx - (10 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: false,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: startX, y: monY, color: colors[0], swivelAngle: 30, depthOffsetPx: 10 * CM_TO_PX
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: true,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: startX + projW, y: monY, color: colors[1], swivelAngle: 0, depthOffsetPx: 10 * CM_TO_PX
        },
        {
          id: 3, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: false,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: startX + projW + d.widthPx, y: monY, color: colors[2], swivelAngle: -30, depthOffsetPx: 10 * CM_TO_PX
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '104'
      accessories.keyboard.yDepthPx = 0
      accessories.mouse.enabled = true
      accessories.mouse.yDepthPx = 0
      accessories.speakers.enabled = true
      accessories.speakers.leftYDepthPx = 0
      accessories.speakers.rightYDepthPx = 0
      accessories.laptop.enabled = false

      const kbW = 44.5 * CM_TO_PX
      accessories.keyboard.xPx = Math.round((deskWidthPx.value - kbW) / 2)
      accessories.mouse.xPx = accessories.keyboard.xPx + Math.round(kbW) + 24
      accessories.speakers.leftXPx = 20
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 20
      idCounter = 4
    }
  },
  {
    id: 'ultrawide-plus-vertical',
    icon: '👨‍💻',
    nameZh: '34" 带鱼屏 + 27" 竖屏 (双机械臂)',
    nameEn: '34" Ultrawide + 27" Vertical (Dual Arms)',
    descZh: '160×75cm 程序员最爱：主屏沉浸编程，副屏竖看代码文档',
    descEn: '160×75cm Developer favorite: wide code editor + vertical documentation viewer',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 160
      deskDepthCm.value = 75
      const main = calculateDimensions(34, 21, 9, false)
      const sub = calculateDimensions(27, 16, 9, true)
      const totalW = main.widthPx + sub.widthPx
      const startX = desk.x + (deskWidthPx.value - totalW) / 2
      const monY = desk.y - main.heightPx - (10 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 34, ratioString: '21:9', ratioX: 21, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: true,
          widthPx: main.widthPx, heightPx: main.heightPx, widthCm: main.widthCm, heightCm: main.heightCm,
          x: startX, y: monY, color: colors[0], swivelAngle: 0, depthOffsetPx: 10 * CM_TO_PX
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: true,
          standType: 'arm', hasLightBar: false,
          widthPx: sub.widthPx, heightPx: sub.heightPx, widthCm: sub.widthCm, heightCm: sub.heightCm,
          x: startX + main.widthPx, y: desk.y - sub.heightPx - (6 * CM_TO_PX), color: colors[1], swivelAngle: 0, depthOffsetPx: 12 * CM_TO_PX
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '87'
      accessories.keyboard.yDepthPx = 0
      accessories.mouse.enabled = true
      accessories.mouse.yDepthPx = 0
      accessories.speakers.enabled = true
      accessories.speakers.leftYDepthPx = 0
      accessories.speakers.rightYDepthPx = 0
      accessories.laptop.enabled = false
      accessories.laptop.yDepthPx = 0

      const kbW = 36 * CM_TO_PX
      accessories.keyboard.xPx = Math.round((deskWidthPx.value - kbW) / 2)
      accessories.mouse.xPx = accessories.keyboard.xPx + Math.round(kbW) + 24
      accessories.speakers.leftXPx = 15
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 15
      idCounter = 3
    }
  },
  {
    id: 'super-ultrawide-49',
    icon: '🎮',
    nameZh: '49" 巨无霸 32:9 (带挂灯与监听箱)',
    nameEn: '49" Super Ultrawide 32:9 (Gaming & Sim)',
    descZh: '160×80cm 桌面：两台 27" 无缝拼接，沉浸电竞、剪辑与金融看盘',
    descEn: '160×80cm desk: Seamless dual 27" display for gaming, trading, and timeline workflows',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 160
      deskDepthCm.value = 80
      const d = calculateDimensions(49, 32, 9, false)
      monitors.value = [
        {
          id: 1, diagonal: 49, ratioString: '32:9', ratioX: 32, ratioY: 9, isPortrait: false,
          standType: 'stand', hasLightBar: true, swivelAngle: 0,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: desk.x + (deskWidthPx.value - d.widthPx) / 2, 
          y: desk.y - d.heightPx - (10 * CM_TO_PX), 
          depthOffsetPx: 10 * CM_TO_PX,
          color: colors[4]
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '104'
      accessories.keyboard.yDepthPx = 0
      accessories.mouse.enabled = true
      accessories.mouse.yDepthPx = 0
      accessories.speakers.enabled = true
      accessories.speakers.leftYDepthPx = 0
      accessories.speakers.rightYDepthPx = 0
      accessories.laptop.enabled = false

      const kbW = 44.5 * CM_TO_PX
      accessories.keyboard.xPx = Math.round((deskWidthPx.value - kbW) / 2)
      accessories.mouse.xPx = accessories.keyboard.xPx + Math.round(kbW) + 24
      accessories.speakers.leftXPx = 15
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 15
      idCounter = 2
    }
  }
]

// --- 预设方案管理与状态追踪 ---
const activePresetId = ref('dual-flagship-27v-32h')
const toastMessage = ref('')
let toastTimer = null

const showToast = (msg) => {
  toastMessage.value = msg
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => {
    toastMessage.value = ''
  }, 2600)
}

const currentPresetObj = computed(() => {
  return PRESETS.find(p => p.id === activePresetId.value) || PRESETS[0]
})

const applyPreset = (preset) => {
  activePresetId.value = preset.id
  preset.action()
  showPresetsModal.value = false
  clearSelection()
  saveToLocalStorage()
  setTimeout(() => fitToScreen(), 60)
  showToast(lang.value === 'zh' ? `已应用方案：${preset.nameZh}` : `Applied preset: ${preset.nameEn}`)
}

const restorePreset = (preset) => {
  activePresetId.value = preset.id
  preset.action()
  clearSelection()
  saveToLocalStorage()
  setTimeout(() => fitToScreen(), 60)
  showToast(lang.value === 'zh' ? `已恢复【${preset.nameZh}】出厂默认布局！` : `Restored [${preset.nameEn}] to default layout!`)
}

const restoreCurrentPreset = () => {
  restorePreset(currentPresetObj.value)
}

// --- LocalStorage 持久化保存 ---
const STORAGE_KEY = 'deskcraft_monitor_layout_v15'

const saveToLocalStorage = () => {
  try {
    const data = {
      activePresetId: activePresetId.value,
      deskWidth: desk.widthCm,
      deskDepth: deskDepthCm.value,
      monitors: monitors.value,
      accessories: accessories
    }
    localStorage.setItem(STORAGE_KEY, JSON.stringify(data))
  } catch (err) {
    console.error('Save failed', err)
  }
}

const loadFromLocalStorage = () => {
  try {
    const savedLang = localStorage.getItem('deskcraft_lang')
    if (savedLang === 'en' || savedLang === 'zh') {
      lang.value = savedLang
    }

    const raw = localStorage.getItem(STORAGE_KEY)
    if (raw) {
      const data = JSON.parse(raw)
      if (data.activePresetId) activePresetId.value = data.activePresetId
      if (data.deskWidth) desk.widthCm = data.deskWidth
      if (data.deskDepth) deskDepthCm.value = data.deskDepth
      if (Array.isArray(data.monitors) && data.monitors.length > 0) {
        monitors.value = data.monitors
        if (data.activePresetId === 'triple-27' && monitors.value.length === 3) {
          // 自动修复旧缓存中中间屏向后错位 3cm (9cm vs 12cm) 的问题，平齐三屏铰接深度
          if (monitors.value.some(m => m.depthOffsetPx && Math.round(m.depthOffsetPx) !== Math.round(10 * CM_TO_PX))) {
            monitors.value.forEach(m => { m.depthOffsetPx = 10 * CM_TO_PX })
          }
        }
        idCounter = Math.max(...monitors.value.map(m => m.id), 0) + 1
      }
      if (data.accessories) {
        if (data.accessories.keyboard) Object.assign(accessories.keyboard, data.accessories.keyboard)
        if (data.accessories.mouse) Object.assign(accessories.mouse, data.accessories.mouse)
        if (data.accessories.speakers) Object.assign(accessories.speakers, data.accessories.speakers)
        if (data.accessories.laptop) Object.assign(accessories.laptop, data.accessories.laptop)
      }
      return true
    }
  } catch (err) {
    console.error('Load failed', err)
  }
  return false
}

// 辅助函数：HEX 转 RGBA
const hexToRgba = (hex, alpha) => {
  let c = hex.replace('#', '')
  if (c.length === 3) c = c.split('').map(x => x + x).join('')
  const num = parseInt(c, 16)
  return `rgba(${(num >> 16) & 255}, ${(num >> 8) & 255}, ${num & 255}, ${alpha})`
}

// --- 生命周期 ---
let frontResizeObserver
let topResizeObserver

onMounted(() => {
  checkMobile()
  window.addEventListener('resize', checkMobile)

  if (frontCanvasRef.value) {
    frontResizeObserver = new ResizeObserver(() => {
      fitFrontToScreen()
    })
    frontResizeObserver.observe(frontCanvasRef.value)
  }

  if (topCanvasRef.value) {
    topResizeObserver = new ResizeObserver(() => {
      fitTopToScreen()
    })
    topResizeObserver.observe(topCanvasRef.value)
  }

  const loaded = loadFromLocalStorage()
  if (!loaded) {
    setDefaultSetup()
  }

  setTimeout(() => {
    fitToScreen()
  }, 100)
})

onUnmounted(() => {
  window.removeEventListener('resize', checkMobile)
  frontResizeObserver?.disconnect()
  topResizeObserver?.disconnect()
})
</script>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
  height: 100dvh;
  background-color: #0b101b;
  color: #f8fafc;
  overflow: hidden;
  position: relative;
}

/* 顶部导航栏 */
.app-header {
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 16px;
  background-color: rgba(15, 23, 42, 0.85);
  backdrop-filter: blur(12px);
  border-bottom: 1px solid rgba(51, 65, 85, 0.6);
  z-index: 40;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.brand-icon {
  font-size: 1.5rem;
}

.brand-text {
  display: flex;
  flex-direction: column;
}

.brand-title {
  font-size: 1.05rem;
  font-weight: 700;
  letter-spacing: -0.3px;
  background: linear-gradient(135deg, #38bdf8, #818cf8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.brand-subtitle {
  font-size: 0.72rem;
  color: #94a3b8;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.lang-btn {
  border-color: #3b82f6 !important;
  color: #38bdf8 !important;
  background: rgba(59, 130, 246, 0.12) !important;
}

.lang-btn:hover {
  background: rgba(59, 130, 246, 0.25) !important;
}

.font-bold {
  font-weight: 700;
}

.quick-chip {
  display: flex;
  align-items: center;
  gap: 6px;
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid rgba(71, 85, 105, 0.5);
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.82rem;
  cursor: pointer;
  transition: all 0.2s;
}

.quick-chip:hover {
  background: rgba(51, 65, 85, 0.8);
  border-color: #38bdf8;
}

.icon-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  background: rgba(30, 41, 59, 0.7);
  border: 1px solid rgba(71, 85, 105, 0.5);
  color: #f1f5f9;
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 0.84rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.icon-btn:hover {
  background: #334155;
  border-color: #64748b;
}

/* --- 视角模式切换组件 (View Mode Pill) --- */
.view-mode-pill {
  display: flex;
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 20px;
  padding: 2px;
  gap: 2px;
}

.view-mode-btn {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 12px;
  border-radius: 16px;
  border: none;
  background: transparent;
  color: #94a3b8;
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.view-mode-btn.active {
  background: #38bdf8;
  color: #0f172a;
  box-shadow: 0 0 10px rgba(56, 189, 248, 0.4);
}

.view-mode-btn:hover:not(.active) {
  color: #f8fafc;
  background: rgba(255, 255, 255, 0.05);
}

/* ================= 主工作区布局 (Canvas on Left + Dedicated Sidebar on Right) ================= */
.app-main-layout {
  display: flex;
  flex-direction: row;
  flex: 1;
  min-height: 0;
  width: 100%;
  overflow: hidden;
  position: relative;
}

/* ================= 画布双视口系统 (Dual Viewports Workspace) ================= */
.canvas-workspace {
  flex: 1;
  position: relative;
  overflow: hidden;
  background-color: #080d1a;
  user-select: none;
  touch-action: none;
}

.canvas-workspace.view-dual {
  display: grid;
  grid-template-columns: minmax(0, 1.7fr) 4px minmax(0, 1fr);
  width: 100%;
  height: 100%;
}

.canvas-workspace.view-front {
  display: grid;
  grid-template-columns: 1fr;
  width: 100%;
  height: 100%;
}

.canvas-workspace.view-top {
  display: grid;
  grid-template-columns: 1fr;
  width: 100%;
  height: 100%;
}

.canvas-pane {
  position: relative;
  overflow: hidden;
  background-color: #0b1120;
  cursor: grab;
}

.canvas-pane:active {
  cursor: grabbing;
}

.pane-divider {
  background: #1e293b;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 30;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.divider-line {
  width: 2px;
  height: 60px;
  background: #475569;
  border-radius: 1px;
}

.pane-header-tag {
  position: absolute;
  top: 14px;
  left: 16px;
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(15, 23, 42, 0.85);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(51, 65, 85, 0.8);
  padding: 5px 12px;
  border-radius: 8px;
  z-index: 30;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

.pane-tag-title {
  display: flex;
  align-items: center;
  gap: 6px;
}

.pane-icon {
  font-size: 1rem;
}

.pane-name {
  font-size: 0.82rem;
  font-weight: 700;
  color: #f8fafc;
}

.pane-badge {
  font-size: 0.68rem;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.15);
  padding: 1px 6px;
  border-radius: 4px;
  font-weight: 600;
}

.pane-tag-actions {
  display: flex;
  align-items: center;
  gap: 8px;
  border-left: 1px solid #334155;
  padding-left: 8px;
}

.pane-zoom-text {
  font-size: 0.72rem;
  color: #94a3b8;
  font-family: monospace;
}

.pane-mini-btn {
  background: #1e293b;
  border: 1px solid #475569;
  color: #cbd5e1;
  border-radius: 4px;
  padding: 2px 6px;
  font-size: 0.75rem;
  cursor: pointer;
  transition: all 0.2s;
}

.pane-mini-btn:hover {
  background: #334155;
  color: #38bdf8;
  border-color: #38bdf8;
}

.pane-hud {
  position: absolute;
  bottom: 20px;
  right: 20px;
  display: flex;
  align-items: center;
  gap: 6px;
  background: rgba(15, 23, 42, 0.85);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(51, 65, 85, 0.8);
  padding: 4px 8px;
  border-radius: 8px;
  z-index: 30;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

.hud-btn {
  width: 28px;
  height: 28px;
  background: #1e293b;
  border: 1px solid #475569;
  color: white;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.82rem;
  transition: all 0.15s;
}

.hud-btn:hover {
  background: #334155;
  border-color: #38bdf8;
}

.hud-zoom-label {
  font-size: 0.76rem;
  font-weight: 600;
  color: #cbd5e1;
  min-width: 44px;
  text-align: center;
  cursor: pointer;
  font-family: monospace;
}

.grid-overlay {
  position: absolute;
  inset: 0;
  background-image: 
    radial-gradient(rgba(148, 163, 184, 0.12) 1.2px, transparent 1.2px),
    linear-gradient(to right, rgba(51, 65, 85, 0.08) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(51, 65, 85, 0.08) 1px, transparent 1px);
  background-size: 24px 24px, 120px 120px, 120px 120px;
  pointer-events: none;
}

.world-layer {
  position: absolute;
  top: 0;
  left: 0;
  will-change: transform;
}

/* 支架与机械臂 SVG 图层 */
.stands-svg-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 5000px;
  height: 3000px;
  pointer-events: none;
  z-index: 5;
}

/* 桌面样式 */
.desk-group {
  position: absolute;
  pointer-events: none;
}

.desk-surface {
  position: absolute;
  background: linear-gradient(180deg, #785135 0%, #563821 100%);
  border-top: 3px solid #a3724c;
  border-radius: 3px;
  box-shadow: 0 16px 36px rgba(0, 0, 0, 0.55), inset 0 1px 0 rgba(255, 255, 255, 0.18);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 8;
  transition: width 0.2s ease;
}

.desk-ruler {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 90%;
  color: #f1d7bf;
  font-size: 0.76rem;
  font-weight: 600;
  text-shadow: 0 1px 2px rgba(0,0,0,0.6);
  opacity: 0.9;
}

.ruler-line {
  flex: 1;
  height: 1px;
  background: rgba(241, 215, 191, 0.4);
}

.ruler-text {
  padding: 0 8px;
  white-space: nowrap;
}

.desk-leg {
  position: absolute;
  width: 14px;
  height: 280px;
  background: linear-gradient(180deg, #1e293b 0%, #0f172a 100%);
  border-left: 1px solid #334155;
  border-right: 1px solid #0f172a;
  z-index: 6;
}

/* ================= 显示器样式 (严格 projectedWidth 宽，四条边近大远小/近粗远细) ================= */
.monitor-card {
  position: absolute;
  cursor: grab;
  user-select: none;
  touch-action: none;
  overflow: visible;
  transition: filter 0.2s;
}

.monitor-card.is-dragging {
  cursor: grabbing;
}

.monitor-frame-svg {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
  overflow: visible;
  filter: drop-shadow(0 10px 24px rgba(0, 0, 0, 0.55));
  transition: filter 0.2s;
}

.monitor-card.is-selected .monitor-frame-svg {
  filter: drop-shadow(0 0 14px rgba(56, 189, 248, 0.75)) drop-shadow(0 10px 24px rgba(0, 0, 0, 0.65));
}

.screen-info-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 4px;
  pointer-events: none;
  z-index: 10;
}

.size-pill {
  display: flex;
  align-items: baseline;
  gap: 6px;
  background: rgba(0, 0, 0, 0.55);
  padding: 4px 10px;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(4px);
}

.diagonal-num {
  font-size: 1.35rem;
  font-weight: 800;
  color: #fff;
  line-height: 1;
}

.ratio-text {
  font-size: 0.78rem;
  color: #94a3b8;
  font-weight: 500;
}

.swivel-num-pill {
  background: rgba(56, 189, 248, 0.2);
  color: #38bdf8;
  padding: 1px 6px;
  border-radius: 4px;
  font-size: 0.65rem;
  margin-left: 4px;
  font-weight: 700;
}

.physical-dim {
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.8);
  background: rgba(0, 0, 0, 0.35);
  padding: 2px 6px;
  border-radius: 4px;
  text-align: center;
}

.proj-dim-text {
  color: #38bdf8;
  font-weight: 600;
  margin-top: 1px;
}

.screen-chin {
  position: absolute;
  bottom: 4px;
  width: 100%;
  display: flex;
  justify-content: center;
  pointer-events: none;
  z-index: 10;
}

.chin-dot {
  width: 5px;
  height: 5px;
  background: rgba(255, 255, 255, 0.25);
  border-radius: 50%;
}

/* 屏幕挂灯 */
.screen-light-bar {
  position: absolute;
  top: -12px;
  left: 20%;
  right: 20%;
  display: flex;
  flex-direction: column;
  align-items: center;
  pointer-events: none;
  z-index: 105;
}

.light-fixture {
  width: 100%;
  height: 6px;
  background: linear-gradient(90deg, #1e293b, #475569 50%, #1e293b);
  border-radius: 3px;
  border: 1px solid #64748b;
  box-shadow: 0 -2px 6px rgba(0, 0, 0, 0.4);
}

.light-beam {
  width: 130%;
  height: 180px;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.18) 0%, rgba(255, 240, 200, 0.08) 50%, transparent 100%);
  clip-path: polygon(15% 0%, 85% 0%, 100% 100%, 0% 100%);
  pointer-events: none;
}

/* 快捷操作浮标 */
.monitor-quick-actions {
  position: absolute;
  top: -38px;
  right: 0;
  display: flex;
  gap: 6px;
  background: rgba(15, 23, 42, 0.9);
  padding: 3px 6px;
  border-radius: 8px;
  border: 1px solid #334155;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
  z-index: 110;
}

.quick-btn {
  width: 28px;
  height: 28px;
  border: none;
  background: #1e293b;
  border-radius: 6px;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.15s;
}

.quick-btn:hover {
  background: #334155;
  transform: translateY(-1px);
}

.active-light {
  background: #f59e0b !important;
}

.delete-btn:hover {
  background: #ef4444;
}

/* ================= 桌面外设与配件样式 (Front View) ================= */
.table-accessory {
  position: absolute;
  user-select: none;
  touch-action: none;
  z-index: 25;
}

.table-accessory.keyboard-item,
.table-accessory.mouse-item {
  z-index: 35;
}

.table-accessory.speaker-box {
  z-index: 8;
}

.table-accessory-draggable {
  cursor: grab;
  transition: box-shadow 0.2s, border-color 0.2s;
}

.table-accessory-draggable:active {
  cursor: grabbing;
}

.table-accessory-draggable:hover {
  outline: 1.5px dashed #38bdf8;
  outline-offset: 2px;
}

.is-selected-accessory {
  outline: 2px solid #38bdf8 !important;
  outline-offset: 3px !important;
  box-shadow: 0 0 16px rgba(56, 189, 248, 0.5) !important;
}

/* 机械键盘 */
.mech-keyboard-unit {
  width: 100%;
  height: 100%;
  background: #1e293b;
  border: 1.5px solid #475569;
  border-radius: 6px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.6);
  position: relative;
  display: flex;
  flex-direction: column;
  padding: 3px;
}

.keyboard-layout-tag {
  position: absolute;
  top: -16px;
  right: 4px;
  font-size: 0.62rem;
  color: #38bdf8;
  font-weight: 700;
  background: rgba(15, 23, 42, 0.85);
  padding: 1px 5px;
  border-radius: 4px;
  border: 1px solid #334155;
}

.keyboard-chassis {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
  justify-content: space-between;
}

.key-strip {
  display: flex;
  gap: 2px;
  height: 22%;
}

.cap {
  flex: 1;
  background: #334155;
  border-radius: 2px;
  box-shadow: 0 1px 2px rgba(0,0,0,0.4);
}

.cap-esc {
  background: #ef4444;
}

.cap-accent {
  background: #38bdf8;
}

.cap-enter {
  flex: 1.8;
  background: #f59e0b;
}

.cap-space {
  flex: 5;
  background: #475569;
}

.cap-ctrl {
  flex: 1.4;
}

.cap-arrow {
  background: #64748b;
}

.cap-numpad {
  background: #3b4252;
}

.cap-numpad-zero {
  flex: 2;
  background: #3b4252;
}

.keyboard-rgb-glow {
  position: absolute;
  bottom: -2px;
  left: 10%;
  right: 10%;
  height: 3px;
  background: linear-gradient(90deg, #f43f5e, #8b5cf6, #06b6d4, #10b981);
  border-radius: 2px;
  filter: blur(1.5px);
  opacity: 0.85;
}

/* 独立鼠标 */
.wireless-mouse-unit {
  width: 100%;
  height: 100%;
  position: relative;
}

.mouse-shell {
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, #334155 0%, #1e293b 100%);
  border: 1.5px solid #64748b;
  border-radius: 14px 14px 8px 8px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.5);
  position: relative;
}

.mouse-buttons-split {
  position: absolute;
  top: 0;
  left: 50%;
  width: 1px;
  height: 40%;
  background: #475569;
}

.mouse-wheel {
  position: absolute;
  top: 15%;
  left: 50%;
  transform: translateX(-50%);
  width: 4px;
  height: 9px;
  background: #38bdf8;
  border-radius: 2px;
}

.mouse-thumb-rest {
  position: absolute;
  left: -2px;
  top: 45%;
  width: 3px;
  height: 12px;
  background: #475569;
  border-radius: 2px;
}

/* 桌面音箱 */
.speaker-box {
  background: #111827;
  border: 2px solid #334155;
  border-radius: 6px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.65);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 8px 4px;
}

.speaker-box.is-selected-accessory {
  z-index: 90 !important;
}

.speaker-tweeter {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: radial-gradient(circle, #f59e0b 35%, #1f2937 70%);
  border: 1px solid #475569;
}

.speaker-woofer {
  width: 34px;
  height: 34px;
  border-radius: 50%;
  background: radial-gradient(circle, #334155 25%, #0f172a 75%);
  border: 1.5px solid #475569;
}

.speaker-label {
  font-size: 0.65rem;
  font-weight: 700;
  color: #64748b;
}

/* 笔记本 */
.laptop-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.laptop-screen-lid {
  width: 100%;
  height: 82%;
  background: #0f172a;
  border: 2px solid #64748b;
  border-radius: 5px 5px 0 0;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.laptop-camera-notch {
  position: absolute;
  top: 0;
  width: 18px;
  height: 4px;
  background: #000;
  border-bottom-left-radius: 3px;
  border-bottom-right-radius: 3px;
}

.laptop-screen-display {
  width: 92%;
  height: 88%;
  background: radial-gradient(ellipse at center, #1e293b 0%, #0b1120 100%);
  border-radius: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.laptop-tag {
  font-size: 0.65rem;
  color: #94a3b8;
  font-weight: 600;
}

.laptop-base {
  width: 108%;
  height: 18%;
  background: linear-gradient(180deg, #475569 0%, #1e293b 100%);
  border-radius: 0 0 5px 5px;
  position: relative;
  display: flex;
  justify-content: center;
}

.laptop-notch-cutout {
  width: 24px;
  height: 3px;
  background: #0f172a;
  border-radius: 0 0 2px 2px;
}

/* ================= 俯视图容器与外设样式 (Top View) ================= */
.top-view-container {
  position: relative;
  width: 100%;
  height: 100%;
}

.top-desk-surface {
  position: absolute;
  background: linear-gradient(180deg, #1e293b 0%, #0f172a 100%);
  border: 2px solid #475569;
  border-radius: 10px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.7), inset 0 2px 4px rgba(255, 255, 255, 0.1);
  overflow: hidden;
  z-index: 10;
}

.top-desk-grain {
  position: absolute;
  inset: 0;
  background-image: repeating-linear-gradient(90deg, rgba(255,255,255,0.015) 0px, rgba(255,255,255,0.015) 1px, transparent 1px, transparent 40px);
  pointer-events: none;
}

.desk-back-trench {
  position: absolute;
  top: 0;
  left: 20%;
  right: 20%;
  height: 8px;
  background: #090d16;
  border-bottom-left-radius: 4px;
  border-bottom-right-radius: 4px;
  border: 1px solid #334155;
  border-top: none;
}

.top-desk-ruler-x {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(15, 23, 42, 0.85);
  border: 1px dashed #64748b;
  border-radius: 4px;
  padding: 2px 10px;
  font-size: 0.75rem;
  color: #cbd5e1;
  pointer-events: none;
}

.top-desk-ruler-y {
  position: absolute;
  top: 50%;
  left: 10px;
  transform: translateY(-50%) rotate(-90deg);
  background: rgba(15, 23, 42, 0.85);
  border: 1px dashed #64748b;
  border-radius: 4px;
  padding: 2px 8px;
  font-size: 0.7rem;
  color: #cbd5e1;
  pointer-events: none;
}

/* 俯视图键盘与鼠标 */
.top-keyboard {
  position: absolute;
  border-radius: 6px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.65);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 12;
  cursor: grab;
  padding: 0;
  overflow: visible;
  transition: box-shadow 0.2s;
}

.top-keyboard.is-selected-accessory {
  box-shadow: 0 0 14px rgba(56, 189, 248, 0.8), 0 6px 16px rgba(0, 0, 0, 0.65);
}

.top-keyboard-svg {
  width: 100%;
  height: 100%;
  display: block;
  filter: drop-shadow(0 2px 5px rgba(0, 0, 0, 0.45));
}

.top-kb-badge {
  position: absolute;
  top: -8px;
  right: 6px;
  font-size: 0.6rem;
  font-weight: 800;
  color: #38bdf8;
  background: rgba(15, 23, 42, 0.92);
  padding: 1px 5px;
  border-radius: 4px;
  border: 1px solid rgba(56, 189, 248, 0.4);
  pointer-events: none;
  box-shadow: 0 2px 6px rgba(0,0,0,0.6);
}

.top-mouse {
  position: absolute;
  background: #1e293b;
  border: 1.5px solid #475569;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0,0,0,0.5);
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding-top: 5px;
  z-index: 12;
}

.top-mouse-wheel {
  width: 4px;
  height: 8px;
  background: #38bdf8;
  border-radius: 2px;
}

/* 俯视图监听音箱 */
.top-speaker {
  position: absolute;
  background: #111827;
  border: 2px solid #334155;
  border-radius: 6px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  box-shadow: 0 6px 14px rgba(0,0,0,0.6);
  z-index: 12;
}

.left-top-speaker,
.right-top-speaker {
  transform: none;
}

.top-speaker-driver {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: radial-gradient(circle, #f59e0b 35%, #1f2937 70%);
  border: 1.5px solid #475569;
}

.top-spk-tag {
  font-size: 0.65rem;
  font-weight: 700;
  color: #94a3b8;
}

/* 俯视图笔记本 (Top Laptop) */
.top-laptop {
  position: absolute;
  background: #242938;
  border: 1.5px solid #64748b;
  border-radius: 6px;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 12;
}

.top-laptop-lid {
  width: 90%;
  height: 88%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 4px 0;
  position: relative;
}

.top-laptop-kb-well {
  width: 82%;
  height: 48%;
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 3px;
}

.top-laptop-trackpad {
  width: 32%;
  height: 32%;
  border: 1px solid #475569;
  border-radius: 2px;
  background: rgba(255, 255, 255, 0.03);
}

.top-laptop-tag {
  font-size: 0.6rem;
  color: #94a3b8;
  position: absolute;
  top: 2px;
}

/* 俯视图显示器条 (Top Monitor Bar) */
.top-view-svg-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: visible;
  pointer-events: none;
  z-index: 15;
}

.top-monitor-bar {
  position: absolute;
  height: 20px;
  background: #0f172a;
  border: 2px solid #475569;
  border-radius: 4px;
  cursor: grab;
  z-index: 25;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
  flex-shrink: 0;
  transition: border-color 0.15s, box-shadow 0.15s;
}

.top-monitor-bar:active {
  cursor: grabbing;
}

.top-monitor-bar:hover {
  border-color: #38bdf8 !important;
}

.top-monitor-bar.is-selected {
  border-color: #38bdf8 !important;
  box-shadow: 0 0 16px rgba(56, 189, 248, 0.6) !important;
}

.top-screen-glass {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 3px;
  border-radius: 2px;
}

.top-screen-body {
  display: flex;
  align-items: center;
  gap: 5px;
}

.top-screen-tag {
  font-size: 0.65rem;
  font-weight: 700;
  color: #f1f5f9;
  white-space: nowrap;
}

.top-angle-pill {
  color: #38bdf8;
  margin-left: 2px;
}

.top-lightbar-badge {
  font-size: 0.7rem;
}

/* 视距中心与座椅 */
.user-sweet-spot {
  position: absolute;
  transform: translateX(-50%);
  pointer-events: none;
  z-index: 20;
  white-space: nowrap;
}

.sweet-spot-label {
  display: inline-flex;
  flex-direction: row;
  align-items: center;
  gap: 7px;
  background: rgba(15, 23, 42, 0.92);
  border: 1px solid rgba(56, 189, 248, 0.45);
  box-shadow: 0 4px 14px rgba(0, 0, 0, 0.45);
  padding: 5px 14px;
  border-radius: 9999px;
  white-space: nowrap;
}

.chair-icon-top {
  font-size: 1.15rem;
  line-height: 1;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.5));
}

.spot-title {
  font-size: 0.74rem;
  font-weight: 700;
  color: #38bdf8;
  letter-spacing: 0.02em;
}

.spot-divider {
  color: #64748b;
  font-weight: 700;
  font-size: 0.72rem;
}

.spot-sub {
  font-size: 0.7rem;
  font-weight: 600;
  color: #cbd5e1;
}

/* 吸附辅助对齐虚线 */
.snap-guide-line {
  position: absolute;
  pointer-events: none;
  z-index: 80;
}

.snap-guide-line.horizontal {
  height: 2px;
  background: #38bdf8;
  box-shadow: 0 0 8px #38bdf8;
}

.snap-guide-line.vertical {
  width: 2px;
  background: #38bdf8;
  box-shadow: 0 0 8px #38bdf8;
}

.canvas-empty-hint {
  position: absolute;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  background: rgba(15, 23, 42, 0.85);
  backdrop-filter: blur(10px);
  border: 1px solid #334155;
  padding: 24px 32px;
  border-radius: 12px;
  text-align: center;
}

.empty-icon {
  font-size: 2.2rem;
}

.empty-title {
  font-size: 0.95rem;
  color: #94a3b8;
}

/* ================= 桌面端右侧独立工具面板 (Dedicated Right Sidebar Panel) ================= */
.desktop-sidebar-panel {
  width: 340px;
  flex-shrink: 0;
  height: 100%;
  background: rgba(15, 23, 42, 0.95);
  backdrop-filter: blur(16px);
  border-left: 1px solid rgba(51, 65, 85, 0.7);
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  padding: 16px;
  gap: 12px;
  z-index: 30;
  box-shadow: -6px 0 24px rgba(0, 0, 0, 0.45);
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 4px 2px 8px 2px;
  border-bottom: 1px solid rgba(51, 65, 85, 0.4);
}

.sidebar-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.95rem;
  font-weight: 700;
  color: #f1f5f9;
}

.sidebar-icon {
  font-size: 1.15rem;
}

.sidebar-mon-count {
  font-size: 0.75rem;
  font-weight: 600;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.12);
  padding: 3px 9px;
  border-radius: 12px;
  border: 1px solid rgba(56, 189, 248, 0.3);
}

.icon-mini-add-btn {
  background: rgba(56, 189, 248, 0.15);
  border: 1px solid rgba(56, 189, 248, 0.4);
  color: #38bdf8;
  font-size: 0.75rem;
  padding: 2px 8px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.icon-mini-add-btn:hover {
  background: #38bdf8;
  color: #0f172a;
}

.monitor-chips-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  max-height: 190px;
  overflow-y: auto;
}

.monitor-chip-row {
  display: flex;
  align-items: center;
  gap: 10px;
  background: rgba(30, 41, 59, 0.5);
  border: 1px solid rgba(51, 65, 85, 0.6);
  border-radius: 8px;
  padding: 7px 10px;
  cursor: pointer;
  transition: all 0.15s;
}

.monitor-chip-row:hover {
  background: rgba(51, 65, 85, 0.65);
  border-color: #38bdf8;
}

.monitor-chip-row.active {
  background: rgba(56, 189, 248, 0.15);
  border-color: #38bdf8;
}

.chip-color-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  flex-shrink: 0;
  box-shadow: 0 0 6px currentColor;
}

.chip-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.chip-name {
  font-size: 0.8rem;
  font-weight: 600;
  color: #f1f5f9;
}

.chip-sub {
  font-size: 0.68rem;
  color: #94a3b8;
}

.chip-del-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  font-size: 0.82rem;
  cursor: pointer;
  padding: 3px;
  border-radius: 4px;
  transition: all 0.15s;
}

.chip-del-btn:hover {
  background: rgba(239, 68, 68, 0.2);
  color: #f87171;
}

.inspector-card {
  background: rgba(15, 23, 42, 0.88);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(51, 65, 85, 0.8);
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
}

.card-title {
  font-size: 0.88rem;
  font-weight: 700;
  color: #f1f5f9;
  margin-bottom: 10px;
}

.card-header-flex {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.card-header-flex .card-title {
  margin-bottom: 0;
}

.close-x {
  background: transparent;
  border: none;
  color: #94a3b8;
  font-size: 1rem;
  cursor: pointer;
  padding: 2px 6px;
  border-radius: 4px;
}

.close-x:hover {
  color: #fff;
  background: rgba(255, 255, 255, 0.1);
}

.field-sub-label {
  font-size: 0.75rem;
  color: #94a3b8;
  margin-bottom: 4px;
}

.field-group {
  margin-bottom: 12px;
}

.field-group label {
  display: block;
  font-size: 0.78rem;
  color: #94a3b8;
  margin-bottom: 6px;
}

.field-header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 6px;
}

.field-value-badge {
  font-size: 0.72rem;
  font-weight: 700;
  color: #38bdf8;
  background: rgba(56, 189, 248, 0.15);
  padding: 2px 6px;
  border-radius: 4px;
}

.input-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.range-slider {
  flex: 1;
  accent-color: #38bdf8;
  height: 6px;
  background: #334155;
  border-radius: 3px;
  cursor: pointer;
}

.number-input-wrap {
  display: flex;
  align-items: center;
  background: #1e293b;
  border: 1px solid #475569;
  border-radius: 6px;
  padding: 3px 6px;
  width: 70px;
}

.number-input-wrap input {
  width: 100%;
  background: transparent;
  border: none;
  color: #f8fafc;
  font-size: 0.82rem;
  outline: none;
  text-align: right;
}

.number-input-wrap .unit {
  font-size: 0.72rem;
  color: #94a3b8;
  margin-left: 3px;
}

.quick-chips, .ratio-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 6px;
}

.chip-button {
  background: #1e293b;
  border: 1px solid #475569;
  color: #cbd5e1;
  border-radius: 6px;
  padding: 4px 10px;
  font-size: 0.76rem;
  cursor: pointer;
  transition: all 0.15s;
}

.chip-button:hover {
  background: #334155;
  border-color: #64748b;
}

.chip-button.active {
  background: rgba(56, 189, 248, 0.2);
  border-color: #38bdf8;
  color: #38bdf8;
  font-weight: 600;
}

.chip-button.sm {
  padding: 2px 8px;
  font-size: 0.72rem;
}

.custom-ratio-row {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-top: 8px;
}

.custom-ratio-row input {
  width: 60px;
  background: #1e293b;
  border: 1px solid #475569;
  border-radius: 6px;
  color: white;
  padding: 4px;
  text-align: center;
  font-size: 0.82rem;
}

.angle-chips {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 6px;
}

.slider-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.65rem;
  color: #64748b;
  margin-top: 2px;
}

.help-hint {
  font-size: 0.72rem;
  color: #94a3b8;
  margin-top: 5px;
  line-height: 1.35;
}

.metric-badge {
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid #334155;
  padding: 6px 10px;
  border-radius: 6px;
  font-size: 0.78rem;
  color: #cbd5e1;
}

.card-footer-buttons {
  display: flex;
  gap: 8px;
  margin-top: 14px;
}

.toggle-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
}

.toggle-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.8rem;
  color: #cbd5e1;
  cursor: pointer;
}

.toggle-label input[type="checkbox"] {
  accent-color: #38bdf8;
  width: 15px;
  height: 15px;
  cursor: pointer;
}

.accessories-toggle-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.accessory-block {
  border-bottom: 1px solid rgba(51, 65, 85, 0.4);
  padding-bottom: 8px;
}

.picker-label {
  font-size: 0.72rem;
  color: #94a3b8;
  margin-bottom: 4px;
  display: block;
}

.sub-chips {
  display: flex;
  gap: 6px;
}

/* 按钮通用 */
.primary-btn {
  background: linear-gradient(135deg, #0284c7, #2563eb);
  color: white;
  border: none;
  padding: 7px 16px;
  border-radius: 8px;
  font-size: 0.84rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 12px rgba(2, 132, 199, 0.3);
}

.primary-btn:hover {
  background: linear-gradient(135deg, #0369a1, #1d4ed8);
  box-shadow: 0 4px 16px rgba(2, 132, 199, 0.5);
  transform: translateY(-1px);
}

.secondary-btn {
  flex: 1;
  background: #334155;
  border: 1px solid #475569;
  color: #f1f5f9;
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.secondary-btn:hover {
  background: #475569;
}

.danger-btn {
  background: rgba(239, 68, 68, 0.15);
  border: 1px solid rgba(239, 68, 68, 0.4);
  color: #f87171;
  padding: 6px 12px;
  border-radius: 8px;
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.danger-btn:hover {
  background: #ef4444;
  color: white;
}

.full-btn {
  width: 100%;
}

.mt-1 { margin-top: 4px; }
.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 12px; }
.mb-3 { margin-bottom: 12px; }
.text-cyan { color: #38bdf8; }

/* ================= 移动端抽屉与浮动按钮 ================= */
.mobile-floating-actions {
  position: absolute;
  bottom: 24px;
  left: 16px;
  right: 16px;
  display: flex;
  gap: 8px;
  z-index: 50;
}

.floating-add-btn, .floating-acc-btn, .floating-edit-btn {
  flex: 1;
  background: #1e293b;
  border: 1px solid #475569;
  color: white;
  padding: 10px 14px;
  border-radius: 12px;
  font-size: 0.84rem;
  font-weight: 600;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.5);
  cursor: pointer;
}

.floating-add-btn {
  background: linear-gradient(135deg, #0284c7, #2563eb);
  border: none;
}

.mobile-drawer-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(4px);
  z-index: 100;
  display: flex;
  align-items: flex-end;
}

.mobile-drawer-sheet {
  width: 100%;
  max-height: 80vh;
  background: #0f172a;
  border-top: 1px solid #334155;
  border-radius: 20px 20px 0 0;
  padding: 20px;
  overflow-y: auto;
  box-shadow: 0 -10px 30px rgba(0, 0, 0, 0.8);
}

.drawer-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.drawer-header h3 {
  font-size: 1rem;
  font-weight: 700;
  margin: 0;
}

.drawer-btn-row {
  display: flex;
  gap: 10px;
}

/* ================= 布局预设弹窗 (Presets Modal) ================= */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(8px);
  z-index: 200;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}

.modal-card {
  width: 100%;
  max-width: 680px;
  max-height: 85vh;
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 16px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid #1e293b;
}

.modal-title {
  font-size: 1rem;
  font-weight: 700;
  color: #f8fafc;
}

.presets-grid {
  padding: 16px;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

/* 预设侧边栏卡片 */
.preset-sidebar-card {
  border-left: 3px solid #38bdf8;
  background: linear-gradient(180deg, rgba(30, 41, 59, 0.7) 0%, rgba(15, 23, 42, 0.9) 100%);
}

.preset-summary-box {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(51, 65, 85, 0.6);
  border-radius: 8px;
  padding: 10px 12px;
  margin-top: 6px;
}

.preset-summary-icon {
  font-size: 1.6rem;
  flex-shrink: 0;
}

.preset-summary-content {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
}

.preset-summary-name {
  font-size: 0.88rem;
  font-weight: 700;
  color: #f8fafc;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.preset-summary-desc {
  font-size: 0.72rem;
  color: #94a3b8;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.restore-highlight-btn {
  background: linear-gradient(135deg, #0284c7, #2563eb);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  font-weight: 700;
}

/* 预设弹窗卡片重构 */
.preset-item-card {
  display: flex;
  flex-direction: column;
  gap: 10px;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 12px;
  padding: 14px 16px;
  transition: all 0.2s;
}

.preset-item-card:hover {
  border-color: #38bdf8;
  background: #24344d;
}

.preset-item-card.is-active-preset {
  border-color: #38bdf8;
  background: linear-gradient(90deg, rgba(2, 132, 199, 0.15) 0%, rgba(30, 41, 59, 0.8) 100%);
  box-shadow: 0 0 16px rgba(56, 189, 248, 0.2);
}

.preset-card-top {
  display: flex;
  align-items: flex-start;
  gap: 14px;
}

.preset-icon-badge {
  font-size: 1.8rem;
  line-height: 1;
}

.preset-info-col {
  flex: 1;
  min-width: 0;
}

.preset-title-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  margin-bottom: 4px;
}

.preset-name {
  font-size: 0.92rem;
  font-weight: 700;
  color: #f1f5f9;
}

.preset-active-badge {
  font-size: 0.72rem;
  font-weight: 700;
  color: #34d399;
  background: rgba(16, 185, 129, 0.15);
  border: 1px solid rgba(16, 185, 129, 0.35);
  padding: 2px 8px;
  border-radius: 9999px;
  white-space: nowrap;
}

.preset-desc {
  font-size: 0.75rem;
  color: #94a3b8;
  line-height: 1.4;
}

.preset-card-actions {
  display: flex;
  gap: 10px;
  margin-top: 4px;
  padding-top: 8px;
  border-top: 1px solid rgba(51, 65, 85, 0.5);
}

.preset-card-btn {
  flex: 1;
  padding: 8px 14px;
  border-radius: 8px;
  font-size: 0.82rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.preset-card-btn.apply-btn {
  background: #334155;
  color: #f8fafc;
  border: 1px solid #475569;
}

.preset-card-btn.apply-btn:hover {
  background: #38bdf8;
  color: #0f172a;
  border-color: #38bdf8;
}

.preset-card-btn.restore-btn {
  background: rgba(2, 132, 199, 0.18);
  color: #38bdf8;
  border: 1px solid rgba(56, 189, 248, 0.4);
}

.preset-card-btn.restore-btn:hover {
  background: #0284c7;
  color: #ffffff;
  border-color: #0284c7;
}

/* Toast 提示 */
.toast-notification {
  position: fixed;
  bottom: 28px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(15, 23, 42, 0.95);
  border: 1px solid #38bdf8;
  color: #f1f5f9;
  padding: 8px 22px;
  border-radius: 9999px;
  font-size: 0.84rem;
  font-weight: 600;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.8), 0 0 16px rgba(56, 189, 248, 0.4);
  z-index: 9999;
  pointer-events: none;
  white-space: nowrap;
}

.toast-fade-enter-active,
.toast-fade-leave-active {
  transition: opacity 0.25s, transform 0.25s;
}

.toast-fade-enter-from,
.toast-fade-leave-to {
  opacity: 0;
  transform: translate(-50%, 12px);
}

/* ================= 移动端自适应 (Mobile Media Queries) ================= */
@media (max-width: 768px) {
  .app-main-layout {
    flex-direction: column;
  }
  .canvas-workspace.view-dual {
    display: flex;
    flex-direction: column;
  }
  .canvas-workspace.view-dual .front-pane {
    flex: 1.2;
  }
  .canvas-workspace.view-dual .top-pane {
    flex: 1;
  }
  .pane-divider {
    height: 4px;
    width: 100%;
  }
  .divider-line {
    width: 40px;
    height: 2px;
  }
}
</style>
