<template>
  <div class="app-container" :class="{ 'is-mobile': isMobile }">
    <!-- Top Navigation Bar -->
    <header class="app-header">
      <div class="brand">
        <span class="brand-icon">🖥️</span>
        <div class="brand-text">
          <span class="brand-title">DeskCraft Pro</span>
          <span class="brand-subtitle">{{ t('brandSubtitle') }}</span>
        </div>
      </div>

      <div class="header-actions">
        <!-- Language Switcher Button -->
        <button class="icon-btn lang-btn" @click="toggleLanguage" :title="lang === 'zh' ? 'Switch to English' : '切换为中文'">
          <span class="btn-icon">🌐</span>
          <span class="btn-text font-bold">{{ lang === 'zh' ? 'EN' : '中文' }}</span>
        </button>

        <!-- Quick Desk Width -->
        <div class="quick-chip desk-chip" @click="openDeskSettings">
          <span class="chip-icon">🪑</span>
          <span class="chip-label">{{ t('deskWidth') }}: {{ desk.widthCm }} cm</span>
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

    <!-- Canvas Area -->
    <div 
      class="canvas-viewport" 
      ref="canvasRef"
      @wheel.prevent="onWheel"
      @pointerdown="onCanvasPointerDown"
      @pointermove="onCanvasPointerMove"
      @pointerup="onCanvasPointerUp"
      @pointercancel="onCanvasPointerUp"
    >
      <!-- Background Grid -->
      <div class="grid-overlay"></div>

      <!-- World Transform Layer (Zoom & Pan) -->
      <div 
        class="world-layer"
        :style="{
          transform: `translate(${pan.x}px, ${pan.y}px) scale(${zoom})`,
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
                :x="m.x + m.widthPx / 2 - 9" 
                :y="m.y + m.heightPx * 0.45" 
                width="18" 
                :height="Math.max(12, desk.y - (m.y + m.heightPx * 0.45))" 
                fill="url(#poleMetalGrad)"
                rx="2"
              />
              <!-- Desk Base Plate on desk tabletop -->
              <path 
                :d="`M ${m.x + m.widthPx / 2 - 48} ${desk.y} 
                     L ${m.x + m.widthPx / 2 - 38} ${desk.y - 7} 
                     L ${m.x + m.widthPx / 2 + 38} ${desk.y - 7} 
                     L ${m.x + m.widthPx / 2 + 48} ${desk.y} Z`" 
                fill="#1e293b" 
                stroke="#475569" 
                stroke-width="1.5"
              />
            </template>
          </g>
        </svg>

        <!-- Desk Visualization -->
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
              <span class="ruler-text">◄ {{ effectiveDeskWidthCm }} {{ t('deskRulerText') }} ►</span>
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

        <!-- ================= Table Accessories ================= -->

        <!-- 1. Keyboard (Separate & Draggable) -->
        <div 
          v-if="accessories.keyboard.enabled"
          class="table-accessory keyboard-item"
          :class="{ 'is-selected-accessory': selectedAccessory === 'keyboard' }"
          :style="{
            left: (desk.x + accessories.keyboard.xPx) + 'px',
            top: (desk.y - keyboardHeightPx) + 'px',
            width: keyboardWidthPx + 'px',
            height: keyboardHeightPx + 'px'
          }"
          @pointerdown.stop="startAccessoryDrag('keyboard', $event)"
          :title="`${t('keyboard')} (${currentKeyboardLayout.label}) · ${t('dragHint')}`"
        >
          <div class="mech-keyboard-unit" :class="`layout-${accessories.keyboard.layout}`">
            <!-- Layout Indicator Badge -->
            <div class="keyboard-layout-tag">{{ accessories.keyboard.layout }}</div>
            
            <div class="keyboard-chassis">
              <!-- Keycaps Rows based on layout -->
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
                <!-- Arrow Keys for 87/98/104 -->
                <template v-if="accessories.keyboard.layout !== '61'">
                  <span class="cap cap-arrow" v-for="a in 3" :key="'ar' + a"></span>
                </template>
                <!-- Numpad 0/Enter for 98/104 -->
                <template v-if="accessories.keyboard.layout === '104' || accessories.keyboard.layout === '98'">
                  <span class="cap cap-numpad-zero"></span>
                  <span class="cap cap-enter"></span>
                </template>
              </div>
            </div>
            <!-- Subtle RGB underglow strip -->
            <div class="keyboard-rgb-glow"></div>
          </div>
        </div>

        <!-- 2. Mouse (Separate & Draggable) -->
        <div 
          v-if="accessories.mouse.enabled"
          class="table-accessory mouse-item"
          :class="{ 'is-selected-accessory': selectedAccessory === 'mouse' }"
          :style="{
            left: (desk.x + accessories.mouse.xPx) + 'px',
            top: (desk.y - mouseHeightPx) + 'px',
            width: mouseWidthPx + 'px',
            height: mouseHeightPx + 'px'
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
            class="table-accessory speaker-box speaker-left"
            :style="{
              left: (desk.x + accessories.speakers.leftXPx) + 'px',
              top: (desk.y - speakerHeightPx) + 'px',
              width: speakerWidthPx + 'px',
              height: speakerHeightPx + 'px'
            }"
            @pointerdown.stop="startSpeakerDrag('left', $event)"
            :title="`${t('speakerLeft')} (${t('dragHint')})`"
          >
            <div class="speaker-tweeter"></div>
            <div class="speaker-woofer"></div>
            <div class="speaker-label">L</div>
          </div>

          <!-- Right Speaker -->
          <div 
            class="table-accessory speaker-box speaker-right"
            :style="{
              left: (desk.x + accessories.speakers.rightXPx) + 'px',
              top: (desk.y - speakerHeightPx) + 'px',
              width: speakerWidthPx + 'px',
              height: speakerHeightPx + 'px'
            }"
            @pointerdown.stop="startSpeakerDrag('right', $event)"
            :title="`${t('speakerRight')} (${t('dragHint')})`"
          >
            <div class="speaker-tweeter"></div>
            <div class="speaker-woofer"></div>
            <div class="speaker-label">R</div>
          </div>
        </template>

        <!-- 4. Laptop / MacBook -->
        <div 
          v-if="accessories.laptop.enabled"
          class="table-accessory laptop-item"
          :style="{
            left: (desk.x + accessories.laptop.xPx) + 'px',
            top: (desk.y - laptopHeightPx) + 'px',
            width: laptopWidthPx + 'px',
            height: laptopHeightPx + 'px'
          }"
          @pointerdown.stop="startAccessoryDrag('laptop', $event)"
          :title="`${t('laptop')} (${t('dragHint')})`"
        >
          <!-- Laptop Screen Lid -->
          <div class="laptop-screen-lid">
            <div class="laptop-camera-notch"></div>
            <div class="laptop-screen-display">
              <span class="laptop-tag">MacBook {{ accessories.laptop.size }}"</span>
            </div>
          </div>
          <!-- Laptop Base & Keyboard Deck -->
          <div class="laptop-base">
            <div class="laptop-notch-cutout"></div>
          </div>
        </div>

        <!-- Snapping Active Guide Line -->
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

        <!-- ================= Monitors ================= -->
        <div 
          v-for="m in monitors" 
          :key="m.id"
          class="monitor-card"
          :class="{ 
            'is-selected': selectedMonitor?.id === m.id,
            'is-dragging': draggingId === m.id,
            'is-portrait': m.isPortrait
          }"
          :style="{
            width: m.widthPx + 'px',
            height: m.heightPx + 'px',
            left: m.x + 'px',
            top: m.y + 'px',
            borderColor: selectedMonitor?.id === m.id ? '#38bdf8' : m.color,
            zIndex: (draggingId === m.id || selectedMonitor?.id === m.id) ? 100 : 10
          }"
          @pointerdown.stop="startMonitorDrag(m.id, $event)"
        >
          <!-- Screen Hanging Light Bar -->
          <div v-if="m.hasLightBar" class="screen-light-bar">
            <div class="light-fixture"></div>
            <div class="light-beam"></div>
          </div>

          <!-- Screen Outer Glass / Bezel -->
          <div 
            class="screen-inner"
            :style="{
              background: `radial-gradient(ellipse at top right, ${hexToRgba(m.color, 0.45)}, #090d16 85%)`
            }"
          >
            <!-- Screen Content Badges -->
            <div class="screen-info">
              <div class="size-pill">
                <span class="diagonal-num">{{ m.diagonal }}"</span>
                <span class="ratio-text">{{ m.isPortrait ? `${m.ratioY}:${m.ratioX}` : `${m.ratioX}:${m.ratioY}` }}</span>
              </div>
              <div class="physical-dim">
                {{ m.widthCm }} × {{ m.heightCm }} cm
              </div>
            </div>

            <!-- Bottom Chin / Logo Pip -->
            <div class="screen-chin">
              <span class="chin-dot"></span>
            </div>
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

      <!-- Floating Canvas HUD (Zoom & Recenter controls) -->
      <div class="canvas-hud">
        <button class="hud-btn" @click.stop="zoomIn" :title="t('zoomIn')">➕</button>
        <span class="hud-zoom-label" @click.stop="resetView">{{ Math.round(zoom * 100) }}%</span>
        <button class="hud-btn" @click.stop="zoomOut" :title="t('zoomOut')">➖</button>
        <button class="hud-btn fit-btn" @click.stop="fitToScreen" :title="t('fitScreenTitle')">⤢</button>
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

    <!-- Desktop Floating Sidebar / Inspector (on PC screen) -->
    <aside v-if="!isMobile" class="desktop-inspector">
      <!-- Desk Controls -->
      <div class="inspector-card">
        <div class="card-title">{{ t('deskSize') }}</div>
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
            @click="desk.widthCm = w"
          >
            {{ w }}cm
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

        <div class="field-group">
          <label>{{ t('physicalDimensions') }}</label>
          <div class="metric-badge">
            {{ selectedMonitor.widthCm }} cm (W) × {{ selectedMonitor.heightCm }} cm (H)
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
          <div v-if="addForm.ratioString === 'custom'" class="custom-ratio-row">
            <input type="number" v-model.number="addForm.customX" min="1" placeholder="X" />
            <span>:</span>
            <input type="number" v-model.number="addForm.customY" min="1" placeholder="Y" />
          </div>
        </div>

        <div class="field-group">
          <label>{{ t('mountType') }}</label>
          <div class="quick-chips">
            <button 
              class="chip-button" 
              :class="{ active: addForm.standType === 'stand' }"
              @click="addForm.standType = 'stand'"
            >{{ t('standOriginal') }}</button>
            <button 
              class="chip-button" 
              :class="{ active: addForm.standType === 'arm' }"
              @click="addForm.standType = 'arm'"
            >{{ t('standArm') }}</button>
          </div>
        </div>

        <button class="primary-btn full-btn" @click="addMonitor">
          ➕ {{ t('putOnDesk') }}
        </button>
      </div>
    </aside>

    <!-- Mobile Bottom Sheet / Modal Drawer -->
    <div 
      v-if="isMobile && activeDrawer" 
      class="mobile-drawer-overlay"
      @click.self="activeDrawer = null"
    >
      <div class="mobile-drawer-content">
        <div class="drawer-drag-bar" @click="activeDrawer = null"></div>

        <!-- Add Monitor Drawer -->
        <template v-if="activeDrawer === 'add'">
          <div class="drawer-header">
            <h3>➕ {{ t('addMonitor') }}</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>

          <div class="drawer-body">
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
                >{{ t('standOriginal') }}</button>
                <button 
                  class="chip-button" 
                  :class="{ active: addForm.standType === 'arm' }"
                  @click="addForm.standType = 'arm'"
                >{{ t('standArm') }}</button>
              </div>
            </div>

            <button class="primary-btn full-btn" @click="addMonitor">
              ➕ {{ t('putOnDesk') }}
            </button>
          </div>
        </template>

        <!-- Edit Selected Monitor Drawer -->
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

            <div class="field-group">
              <label class="toggle-label">
                <input type="checkbox" v-model="selectedMonitor.hasLightBar" @change="saveToLocalStorage" />
                <span>{{ t('lightBar') }}</span>
              </label>
            </div>

            <div class="metric-badge">
              {{ t('physicalDimensions') }} {{ selectedMonitor.widthCm }} × {{ selectedMonitor.heightCm }} cm
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
        </template>

        <!-- Accessories Drawer for Mobile -->
        <template v-else-if="activeDrawer === 'accessories'">
          <div class="drawer-header">
            <h3>🖲️ {{ t('accessoriesTitle') }}</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>

          <div class="drawer-body">
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

              <!-- Speakers Option -->
              <div class="toggle-row">
                <label class="toggle-label">
                  <input type="checkbox" v-model="accessories.speakers.enabled" @change="saveToLocalStorage" />
                  <span>{{ t('speakers') }}</span>
                </label>
              </div>

              <!-- Laptop Option -->
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

            <button class="primary-btn full-btn mt-3" @click="activeDrawer = null">
              ✓ {{ t('done') }}
            </button>
          </div>
        </template>

        <!-- Desk Settings Drawer -->
        <template v-else-if="activeDrawer === 'desk'">
          <div class="drawer-header">
            <h3>🪑 {{ t('deskSettings') }}</h3>
            <button class="close-x" @click="activeDrawer = null">✕</button>
          </div>

          <div class="drawer-body">
            <div class="field-group">
              <label>{{ t('deskSize') }}</label>
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

            <button class="primary-btn full-btn" @click="activeDrawer = null; fitToScreen()">
              ✓ {{ t('doneAndCenter') }}
            </button>
          </div>
        </template>
      </div>
    </div>

    <!-- Presets Modal Dialog -->
    <div 
      v-if="showPresetsModal" 
      class="modal-backdrop"
      @click.self="showPresetsModal = false"
    >
      <div class="modal-card">
        <div class="modal-header">
          <div class="modal-title">{{ t('presetsTitle') }}</div>
          <button class="close-x" @click="showPresetsModal = false">✕</button>
        </div>

        <div class="presets-grid">
          <div 
            v-for="p in PRESETS" 
            :key="p.id" 
            class="preset-item"
            @click="applyPreset(p)"
          >
            <div class="preset-icon">{{ p.icon }}</div>
            <div class="preset-name">{{ lang === 'zh' ? p.nameZh : p.nameEn }}</div>
            <div class="preset-desc">{{ lang === 'zh' ? p.descZh : p.descEn }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'

// --- 多语言国际化 (i18n) ---
const lang = ref('zh')

const TRANSLATIONS = {
  zh: {
    brandSubtitle: '桌面显示器与外设布局搭配',
    deskWidth: '桌宽',
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
    toLandscape: '切为横屏',
    toPortrait: '切为竖屏',
    delete: '删除',
    putOnDesk: '放入桌面',
    emptyTitle: '当前桌面没有显示器',
    emptyBtn: '点击添加显示器',
    dragHint: '可左右拖移',
    edit: '编辑',
    done: '完成配置',
    doneAndCenter: '完成并居中适应',
    zoomIn: '放大',
    zoomOut: '缩小'
  },
  en: {
    brandSubtitle: 'Desktop Monitor & Battlestation Planner',
    deskWidth: 'Desk',
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
    toLandscape: 'To Landscape',
    toPortrait: 'To Portrait',
    delete: 'Delete',
    putOnDesk: 'Place on Desk',
    emptyTitle: 'No displays on the desk',
    emptyBtn: 'Click to Add Monitor',
    dragHint: 'Draggable',
    edit: 'Edit',
    done: 'Save Configuration',
    doneAndCenter: 'Done & Center',
    zoomIn: 'Zoom In',
    zoomOut: 'Zoom Out'
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

// 固定世界坐标参考原点，彻底防止显示器漂移脱离桌面！
const FIXED_DESK_X = 100 

// --- 响应式移动端检测 ---
const isMobile = ref(false)
const checkMobile = () => {
  isMobile.value = window.innerWidth <= 768
}

// --- 桌面环境配置 ---
const canvasRef = ref(null)

const desk = reactive({
  widthCm: 140, 
  heightCm: 4,  
  x: FIXED_DESK_X,         
  y: 520        
})

const effectiveDeskWidthCm = computed(() => {
  const val = desk.widthCm
  if (val === '' || val === null || val <= 0) return 120
  return Number(val)
})

const deskWidthPx = computed(() => effectiveDeskWidthCm.value * CM_TO_PX)
const deskHeightPx = computed(() => desk.heightCm * CM_TO_PX)

// --- 画布相机系统 (Zoom & Pan) ---
const zoom = ref(1.0)
const pan = reactive({ x: 0, y: 0 })

const zoomIn = () => {
  zoom.value = Math.min(2.5, Number((zoom.value * 1.15).toFixed(2)))
}

const zoomOut = () => {
  zoom.value = Math.max(0.25, Number((zoom.value * 0.85).toFixed(2)))
}

const resetView = () => {
  fitToScreen()
}

// 自动居中并适配视口 (自适应屏幕大小，保持桌子和屏幕始终居中！)
const fitToScreen = () => {
  if (!canvasRef.value) return
  const rect = canvasRef.value.getBoundingClientRect()
  const cw = rect.width
  const ch = rect.height
  if (cw <= 0 || ch <= 0) return

  let minX = desk.x
  let maxX = desk.x + deskWidthPx.value
  let minY = desk.y - 140
  let maxY = desk.y + deskHeightPx.value + 60

  monitors.value.forEach(m => {
    minX = Math.min(minX, m.x)
    maxX = Math.max(maxX, m.x + m.widthPx)
    minY = Math.min(minY, m.y)
    maxY = Math.max(maxY, m.y + m.heightPx)
  })

  if (accessories.speakers.enabled) {
    minX = Math.min(minX, desk.x + accessories.speakers.leftXPx)
    maxX = Math.max(maxX, desk.x + accessories.speakers.rightXPx + speakerWidthPx.value)
  }

  const contentW = maxX - minX
  const contentH = maxY - minY
  const padding = isMobile.value ? 24 : 70

  const scaleX = (cw - padding * 2) / contentW
  const scaleY = (ch - padding * 2) / contentH
  const targetZoom = Math.min(scaleX, scaleY, 1.1)

  zoom.value = Math.max(0.25, Math.min(1.5, Number(targetZoom.toFixed(2))))
  pan.x = (cw - contentW * zoom.value) / 2 - minX * zoom.value
  pan.y = (ch - contentH * zoom.value) / 2 - minY * zoom.value
}

// 将屏幕 client 坐标转为画布世界坐标
const screenToWorld = (clientX, clientY) => {
  if (!canvasRef.value) return { x: 0, y: 0 }
  const rect = canvasRef.value.getBoundingClientRect()
  return {
    x: (clientX - rect.left - pan.x) / zoom.value,
    y: (clientY - rect.top - pan.y) / zoom.value
  }
}

// 滚轮缩放画布
const onWheel = (e) => {
  if (!canvasRef.value) return
  const rect = canvasRef.value.getBoundingClientRect()
  const mouseCanvasX = e.clientX - rect.left
  const mouseCanvasY = e.clientY - rect.top

  const factor = e.deltaY < 0 ? 1.1 : 0.9
  const oldZoom = zoom.value
  const newZoom = Math.max(0.25, Math.min(2.5, Number((oldZoom * factor).toFixed(2))))
  if (newZoom === oldZoom) return

  pan.x = mouseCanvasX - (mouseCanvasX - pan.x) * (newZoom / oldZoom)
  pan.y = mouseCanvasY - (mouseCanvasY - pan.y) * (newZoom / oldZoom)
  zoom.value = newZoom
}

// --- 画布多指平移/缩放状态 ---
const activePointers = new Map()
let isPanningCanvas = false
let startPanMouse = { x: 0, y: 0 }
let startPanOffset = { x: 0, y: 0 }
let pinchStartDist = 0
let pinchStartZoom = 1.0

const onCanvasPointerDown = (e) => {
  activePointers.set(e.pointerId, { x: e.clientX, y: e.clientY })
  clearSelection()

  if (activePointers.size === 1) {
    isPanningCanvas = true
    startPanMouse = { x: e.clientX, y: e.clientY }
    startPanOffset = { x: pan.x, y: pan.y }
  } else if (activePointers.size === 2) {
    isPanningCanvas = false
    const points = Array.from(activePointers.values())
    pinchStartDist = Math.hypot(points[0].x - points[1].x, points[0].y - points[1].y)
    pinchStartZoom = zoom.value
  }
}

const onCanvasPointerMove = (e) => {
  if (!activePointers.has(e.pointerId)) return
  activePointers.set(e.pointerId, { x: e.clientX, y: e.clientY })

  if (activePointers.size === 2 && pinchStartDist > 0) {
    const points = Array.from(activePointers.values())
    const currentDist = Math.hypot(points[0].x - points[1].x, points[0].y - points[1].y)
    const factor = currentDist / pinchStartDist
    zoom.value = Math.max(0.25, Math.min(2.5, Number((pinchStartZoom * factor).toFixed(2))))
    return
  }

  if (isPanningCanvas && activePointers.size === 1) {
    pan.x = startPanOffset.x + (e.clientX - startPanMouse.x)
    pan.y = startPanOffset.y + (e.clientY - startPanMouse.y)
  }
}

const onCanvasPointerUp = (e) => {
  activePointers.delete(e.pointerId)
  if (activePointers.size === 0) {
    isPanningCanvas = false
    pinchStartDist = 0
  } else if (activePointers.size === 1) {
    const remaining = Array.from(activePointers.values())[0]
    isPanningCanvas = true
    startPanMouse = { x: remaining.x, y: remaining.y }
    startPanOffset = { x: pan.x, y: pan.y }
  }
}

// --- 显示器配置与状态 ---
const RATIO_OPTIONS = [
  { label: '16:9', value: '16:9' },
  { label: '16:10', value: '16:10' },
  { label: '21:9', value: '21:9' },
  { label: '32:9', value: '32:9' },
  { label: '4:3', value: '4:3' },
  { label: '自定义', value: 'custom' }
]

const addForm = reactive({ 
  diagonal: 32, 
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

// 计算显示器像素尺寸及物理 cm 尺寸
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
    standType: addForm.standType || 'stand',
    hasLightBar: false,
    widthPx,
    heightPx,
    widthCm,
    heightCm,
    // 居中放置在桌面上方，保持 12cm 舒适悬空高度
    x: desk.x + (deskWidthPx.value / 2) - (widthPx / 2) + ((monitors.value.length % 3) * 20),
    y: desk.y - heightPx - (12 * CM_TO_PX),
    color: colors[(newId - 1) % colors.length]
  }

  monitors.value.push(newMonitor)
  selectedId.value = newId
  activeDrawer.value = null
  saveToLocalStorage()
}

const updateSelected = () => {
  if (!selectedMonitor.value) return
  const m = selectedMonitor.value
  if (m.ratioString !== 'custom') {
    m.ratioX = Number(m.ratioString.split(':')[0])
    m.ratioY = Number(m.ratioString.split(':')[1])
  }
  const { widthPx, heightPx, widthCm, heightCm } = calculateDimensions(m.diagonal, m.ratioX, m.ratioY, m.isPortrait)
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

  if (m.y + m.heightPx > desk.y) {
    m.y = desk.y - m.heightPx
  }
  saveToLocalStorage()
}

const removeMonitor = (id) => {
  monitors.value = monitors.value.filter(m => m.id !== id)
  if (selectedId.value === id) clearSelection()
  if (activeDrawer.value === 'edit') activeDrawer.value = null
  saveToLocalStorage()
}

// --- 支架与机械臂路径计算 ---
const getArmJoints = (m) => {
  const vesaX = m.x + m.widthPx / 2
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

// --- 机械键盘配列规格 ---
const KEYBOARD_LAYOUTS = [
  { value: '61', labelZh: '61 键 (60%)', labelEn: '61-key (60%)', widthCm: 29.5 },
  { value: '87', labelZh: '87 键 (TKL)', labelEn: '87-key (TKL)', widthCm: 36 },
  { value: '98', labelZh: '98 键 (980)', labelEn: '98-key (98%)', widthCm: 39 },
  { value: '104', labelZh: '104 键 (全尺寸)', labelEn: '104-key (Full)', widthCm: 44.5 }
]

// --- 桌面外设与配件状态 (键盘与鼠标完全独立) ---
const accessories = reactive({
  keyboard: {
    enabled: true,
    layout: '104',
    xPx: 250
  },
  mouse: {
    enabled: true,
    xPx: 530
  },
  speakers: {
    enabled: true,
    leftXPx: 25,
    rightXPx: 740
  },
  laptop: {
    enabled: false,
    size: 14,
    xPx: 120
  }
})

// 键盘动态尺寸换算
const currentKeyboardLayout = computed(() => {
  return KEYBOARD_LAYOUTS.find(l => l.value === accessories.keyboard.layout) || KEYBOARD_LAYOUTS[3]
})
const keyboardWidthPx = computed(() => currentKeyboardLayout.value.widthCm * CM_TO_PX)
const keyboardHeightPx = computed(() => 3.6 * CM_TO_PX)

// 鼠标动态尺寸换算 (约 7.5cm × 3.4cm)
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

// 外设独立拖动逻辑
const selectedAccessory = ref(null)
let isDraggingAccessory = false
let activeDragTarget = null // 'keyboard' | 'mouse' | 'laptop' | 'speaker-left' | 'speaker-right'
let startAccessoryMouseX = 0
let startAccessoryInitialX = 0

const startAccessoryDrag = (key, event) => {
  event.stopPropagation()
  isDraggingAccessory = true
  activeDragTarget = key
  selectedAccessory.value = key
  clearSelection()

  const worldPos = screenToWorld(event.clientX, event.clientY)
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
  clearSelection()

  const worldPos = screenToWorld(event.clientX, event.clientY)
  startAccessoryMouseX = worldPos.x
  startAccessoryInitialX = side === 'left' ? accessories.speakers.leftXPx : accessories.speakers.rightXPx

  window.addEventListener('pointermove', onAccessoryPointerMove)
  window.addEventListener('pointerup', onAccessoryPointerUp)
  window.addEventListener('pointercancel', onAccessoryPointerUp)
}

const onAccessoryPointerMove = (event) => {
  if (!isDraggingAccessory) return
  const worldPos = screenToWorld(event.clientX, event.clientY)
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

// --- 物理拖拽、吸附与碰撞检测 ---
const draggingId = ref(null)
let dragPointerId = null
let dragStartOffset = { x: 0, y: 0 }
const activeSnapGuide = ref(null)

const startMonitorDrag = (id, event) => {
  event.stopPropagation()
  draggingId.value = id
  selectedId.value = id
  selectedAccessory.value = null
  dragPointerId = event.pointerId
  event.target.setPointerCapture?.(event.pointerId)

  const m = monitors.value.find(item => item.id === id)
  if (!m) return

  const worldPos = screenToWorld(event.clientX, event.clientY)
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

  const worldPos = screenToWorld(event.clientX, event.clientY)
  let targetX = worldPos.x - dragStartOffset.x
  let targetY = worldPos.y - dragStartOffset.y

  activeSnapGuide.value = null

  // 1. 桌面顶部吸附
  if (
    targetX + current.widthPx > desk.x - SNAP_THRESHOLD &&
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

  // 2. 显示器之间吸附
  monitors.value.forEach(m => {
    if (m.id === current.id) return

    const isYAligned = targetY < m.y + m.heightPx + SNAP_THRESHOLD && targetY + current.heightPx > m.y - SNAP_THRESHOLD
    const isXAligned = targetX < m.x + m.widthPx + SNAP_THRESHOLD && targetX + current.widthPx > m.x - SNAP_THRESHOLD

    if (isYAligned) {
      if (Math.abs(targetX - m.x) < SNAP_THRESHOLD) targetX = m.x
      if (Math.abs(targetX - (m.x + m.widthPx)) < SNAP_THRESHOLD) targetX = m.x + m.widthPx
      if (Math.abs((targetX + current.widthPx) - m.x) < SNAP_THRESHOLD) targetX = m.x - current.widthPx
      if (Math.abs((targetX + current.widthPx) - (m.x + m.widthPx)) < SNAP_THRESHOLD) targetX = m.x + m.widthPx - current.widthPx
    }

    if (isXAligned) {
      if (Math.abs(targetY - m.y) < SNAP_THRESHOLD) targetY = m.y
      if (Math.abs(targetY - (m.y + m.heightPx)) < SNAP_THRESHOLD) targetY = m.y + m.heightPx
      if (Math.abs((targetY + current.heightPx) - m.y) < SNAP_THRESHOLD) targetY = m.y - current.heightPx
      if (Math.abs((targetY + current.heightPx) - (m.y + m.heightPx)) < SNAP_THRESHOLD) targetY = m.y + m.heightPx - current.heightPx
    }
  })

  // 3. 严格防重叠与防沉桌
  for (let iter = 0; iter < 2; iter++) {
    monitors.value.forEach(m => {
      if (m.id === current.id) return
      const isOverlapping = targetX < m.x + m.widthPx && targetX + current.widthPx > m.x && targetY < m.y + m.heightPx && targetY + current.heightPx > m.y
      if (isOverlapping) {
        const overlapLeft = (targetX + current.widthPx) - m.x
        const overlapRight = (m.x + m.widthPx) - targetX
        const overlapTop = (targetY + current.heightPx) - m.y
        const overlapBottom = (m.y + m.heightPx) - targetY
        const minOverlap = Math.min(overlapLeft, overlapRight, overlapTop, overlapBottom)

        if (minOverlap === overlapLeft) targetX = m.x - current.widthPx
        else if (minOverlap === overlapRight) targetX = m.x + m.widthPx
        else if (minOverlap === overlapTop) targetY = m.y - current.heightPx
        else if (minOverlap === overlapBottom) targetY = m.y + m.heightPx
      }
    })

    const isOverlappingDesk = targetX < desk.x + deskWidthPx.value && targetX + current.widthPx > desk.x && targetY < desk.y + deskHeightPx.value && targetY + current.heightPx > desk.y
    if (isOverlappingDesk) {
      const overlapLeft = (targetX + current.widthPx) - desk.x
      const overlapRight = (desk.x + deskWidthPx.value) - targetX
      const overlapTop = (targetY + current.heightPx) - desk.y
      const overlapBottom = (desk.y + deskHeightPx.value) - targetY
      const minOverlap = Math.min(overlapLeft, overlapRight, overlapTop, overlapBottom)

      if (minOverlap === overlapLeft) targetX = desk.x - current.widthPx
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

// --- 移动端底部抽屉与预设弹窗 ---
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

// 设置默认桌面布局 (单台 32" 16:9 悬空舒适高度 + 104 键键盘 + 鼠标 + 对箱音箱)
const setDefaultSetup = () => {
  desk.x = FIXED_DESK_X
  desk.widthCm = 140
  const d = calculateDimensions(32, 16, 9, false)
  // 严格居中在桌面上方！
  const monX = desk.x + (deskWidthPx.value - d.widthPx) / 2
  // 保持约 12cm 的悬空高度，原装支架立柱延伸至桌面，键盘置于下方
  const monY = desk.y - d.heightPx - (12 * CM_TO_PX)

  monitors.value = [
    {
      id: 1,
      diagonal: 32,
      ratioString: '16:9',
      ratioX: 16,
      ratioY: 9,
      isPortrait: false,
      standType: 'stand',
      hasLightBar: true,
      widthPx: d.widthPx,
      heightPx: d.heightPx,
      widthCm: d.widthCm,
      heightCm: d.heightCm,
      x: monX,
      y: monY,
      color: colors[0]
    }
  ]
  idCounter = 2

  accessories.keyboard.enabled = true
  accessories.keyboard.layout = '104'
  accessories.mouse.enabled = true
  accessories.speakers.enabled = true
  accessories.laptop.enabled = false

  const kbW = 44.5 * CM_TO_PX
  const mPulse = 7.5 * CM_TO_PX
  const totalCenter = (deskWidthPx.value - (kbW + 24 + mPulse)) / 2
  accessories.keyboard.xPx = Math.max(10, totalCenter)
  accessories.mouse.xPx = Math.min(deskWidthPx.value - mPulse - 10, totalCenter + kbW + 24)

  accessories.speakers.leftXPx = 35
  accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 35
}

// --- 常用布局预设模版 ---
const PRESETS = [
  {
    id: 'single-32-default',
    icon: '🌟',
    nameZh: '经典单屏 (32" 悬空立架 + 104键 + 鼠标 + 音箱)',
    nameEn: 'Classic Single (32" Stand + 104-Key + Mouse + Audio)',
    descZh: '单台 32 寸大屏悬空黄金高度 + 104 键全尺寸键盘 + 独立鼠标 + 监听音箱',
    descEn: '32" screen at ergonomic elevated height with 104-key keyboard, mouse & studio monitors',
    action: () => {
      setDefaultSetup()
    }
  },
  {
    id: 'dual-27',
    icon: '💼',
    nameZh: '双 27" 横向办公',
    nameEn: 'Dual 27" Productivity',
    descZh: '经典双屏并排，办公分屏高效神器',
    descEn: 'Classic side-by-side dual screen for maximum multitasking efficiency',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 140
      const d1 = calculateDimensions(27, 16, 9, false)
      const d2 = calculateDimensions(27, 16, 9, false)
      const totalW = d1.widthPx + d2.widthPx
      const startX = desk.x + (deskWidthPx.value - totalW) / 2
      const monY = desk.y - d1.heightPx - (10 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'stand', hasLightBar: true,
          widthPx: d1.widthPx, heightPx: d1.heightPx, widthCm: d1.widthCm, heightCm: d1.heightCm,
          x: startX, y: monY, color: colors[0]
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'stand', hasLightBar: false,
          widthPx: d2.widthPx, heightPx: d2.heightPx, widthCm: d2.widthCm, heightCm: d2.heightCm,
          x: startX + d1.widthPx, y: monY, color: colors[1]
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '104'
      accessories.mouse.enabled = true
      accessories.speakers.enabled = true
      accessories.laptop.enabled = false

      accessories.keyboard.xPx = (deskWidthPx.value - (44.5 * CM_TO_PX + 20 + 7.5 * CM_TO_PX)) / 2
      accessories.mouse.xPx = accessories.keyboard.xPx + 44.5 * CM_TO_PX + 20
      accessories.speakers.leftXPx = 30
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 30
      idCounter = 3
    }
  },
  {
    id: 'ultrawide-plus-vertical',
    icon: '👨‍💻',
    nameZh: '34" 带鱼屏 + 27" 竖屏 (双机械臂)',
    nameEn: '34" Ultrawide + 27" Vertical (Dual Arms)',
    descZh: '程序员最爱：主屏沉浸编程，副屏竖看代码文档',
    descEn: 'Developer favorite: wide code editor + vertical documentation viewer',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 160
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
          x: startX, y: monY, color: colors[0]
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: true,
          standType: 'arm', hasLightBar: false,
          widthPx: sub.widthPx, heightPx: sub.heightPx, widthCm: sub.widthCm, heightCm: sub.heightCm,
          x: startX + main.widthPx, y: desk.y - sub.heightPx - (6 * CM_TO_PX), color: colors[1]
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '87'
      accessories.mouse.enabled = true
      accessories.speakers.enabled = true
      accessories.laptop.enabled = true
      accessories.laptop.xPx = 25

      accessories.keyboard.xPx = 200
      accessories.mouse.xPx = 450
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
    descZh: '两台 27" 无缝拼接，沉浸电竞、剪辑与金融看盘',
    descEn: 'Seamless dual 27" display for gaming, trading, and timeline workflows',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 160
      const d = calculateDimensions(49, 32, 9, false)
      monitors.value = [
        {
          id: 1, diagonal: 49, ratioString: '32:9', ratioX: 32, ratioY: 9, isPortrait: false,
          standType: 'stand', hasLightBar: true,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: desk.x + (deskWidthPx.value - d.widthPx) / 2, 
          y: desk.y - d.heightPx - (10 * CM_TO_PX), 
          color: colors[4]
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '104'
      accessories.mouse.enabled = true
      accessories.speakers.enabled = true
      accessories.laptop.enabled = false

      accessories.keyboard.xPx = (deskWidthPx.value - (44.5 * CM_TO_PX + 20 + 7.5 * CM_TO_PX)) / 2
      accessories.mouse.xPx = accessories.keyboard.xPx + 44.5 * CM_TO_PX + 20
      accessories.speakers.leftXPx = 15
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 15
      idCounter = 2
    }
  },
  {
    id: 'stack-dual-27',
    icon: '🚀',
    nameZh: '上下双屏 (27" + 27")',
    nameEn: 'Vertical Stack (27" + 27")',
    descZh: '省桌面横向空间，视线抬头即见辅屏',
    descEn: 'Saves horizontal footprint with convenient vertical stacked displays',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 120
      const d1 = calculateDimensions(27, 16, 9, false)
      const d2 = calculateDimensions(27, 16, 9, false)
      const x = desk.x + (deskWidthPx.value - d1.widthPx) / 2
      const gapY = desk.y - d1.heightPx - (8 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'stand', hasLightBar: true,
          widthPx: d1.widthPx, heightPx: d1.heightPx, widthCm: d1.widthCm, heightCm: d1.heightCm,
          x: x, y: gapY, color: colors[0]
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: false,
          widthPx: d2.widthPx, heightPx: d2.heightPx, widthCm: d2.widthCm, heightCm: d2.heightCm,
          x: x, y: gapY - d2.heightPx, color: colors[2]
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '87'
      accessories.mouse.enabled = true
      accessories.speakers.enabled = true
      accessories.laptop.enabled = false

      accessories.keyboard.xPx = (deskWidthPx.value - (36 * CM_TO_PX + 20 + 7.5 * CM_TO_PX)) / 2
      accessories.mouse.xPx = accessories.keyboard.xPx + 36 * CM_TO_PX + 20
      accessories.speakers.leftXPx = 10
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 10
      idCounter = 3
    }
  },
  {
    id: 'triple-27',
    icon: '🏎️',
    nameZh: '三屏横向环绕 (27" × 3)',
    nameEn: 'Triple 27" Panoramic Surround',
    descZh: '赛车模拟与极致多任务工作台',
    descEn: 'Triple-monitor panoramic setup for sim-racing and ultimate productivity',
    action: () => {
      desk.x = FIXED_DESK_X
      desk.widthCm = 200
      const d = calculateDimensions(27, 16, 9, false)
      const totalW = d.widthPx * 3
      const startX = desk.x + (deskWidthPx.value - totalW) / 2
      const monY = desk.y - d.heightPx - (10 * CM_TO_PX)
      monitors.value = [
        {
          id: 1, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: false,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: startX, y: monY, color: colors[0]
        },
        {
          id: 2, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: true,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: startX + d.widthPx, y: monY, color: colors[1]
        },
        {
          id: 3, diagonal: 27, ratioString: '16:9', ratioX: 16, ratioY: 9, isPortrait: false,
          standType: 'arm', hasLightBar: false,
          widthPx: d.widthPx, heightPx: d.heightPx, widthCm: d.widthCm, heightCm: d.heightCm,
          x: startX + d.widthPx * 2, y: monY, color: colors[2]
        }
      ]
      accessories.keyboard.enabled = true
      accessories.keyboard.layout = '104'
      accessories.mouse.enabled = true
      accessories.speakers.enabled = true
      accessories.laptop.enabled = false

      accessories.keyboard.xPx = (deskWidthPx.value - (44.5 * CM_TO_PX + 20 + 7.5 * CM_TO_PX)) / 2
      accessories.mouse.xPx = accessories.keyboard.xPx + 44.5 * CM_TO_PX + 20
      accessories.speakers.leftXPx = 10
      accessories.speakers.rightXPx = deskWidthPx.value - speakerWidthPx.value - 10
      idCounter = 4
    }
  }
]

const applyPreset = (preset) => {
  preset.action()
  showPresetsModal.value = false
  clearSelection()
  saveToLocalStorage()
  setTimeout(() => fitToScreen(), 60)
}

// --- LocalStorage 持久化保存 ---
const STORAGE_KEY = 'deskcraft_monitor_layout_v7'

const saveToLocalStorage = () => {
  try {
    const data = {
      deskWidth: desk.widthCm,
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
      if (data.deskWidth) desk.widthCm = data.deskWidth
      if (Array.isArray(data.monitors) && data.monitors.length > 0) {
        monitors.value = data.monitors
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
let resizeObserver
onMounted(() => {
  checkMobile()
  window.addEventListener('resize', checkMobile)

  if (canvasRef.value) {
    resizeObserver = new ResizeObserver(() => {
      // 保持画布居中，但不改变桌子与显示器的相对世界坐标
      fitToScreen()
    })
    resizeObserver.observe(canvasRef.value)
  }

  const loaded = loadFromLocalStorage()
  if (!loaded) {
    // 首次进入加载完美的 32" 16:9 + 104 键盘 + 鼠标 + 左右监听音箱
    setDefaultSetup()
  }

  setTimeout(() => {
    fitToScreen()
  }, 80)
})

onUnmounted(() => {
  window.removeEventListener('resize', checkMobile)
  if (resizeObserver) resizeObserver.disconnect()
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

/* 画布主体与网格 */
.canvas-viewport {
  flex: 1;
  position: relative;
  overflow: hidden;
  background-color: #0b1120;
  cursor: grab;
  user-select: none;
  touch-action: none;
}

.canvas-viewport:active {
  cursor: grabbing;
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
  background: linear-gradient(90deg, #1e293b, #334155, #0f172a);
  border-radius: 0 0 2px 2px;
  box-shadow: 2px 4px 10px rgba(0,0,0,0.4);
  z-index: 4;
}

/* ================= 桌面外设组件样式 ================= */
.table-accessory {
  position: absolute;
  z-index: 15;
  cursor: grab;
  user-select: none;
  touch-action: none;
  transition: filter 0.15s ease;
}

.table-accessory:hover {
  filter: brightness(1.15);
}

.table-accessory:active {
  cursor: grabbing;
}

/* 机械键盘 */
.keyboard-item {
  display: flex;
  flex-direction: column;
}

.mech-keyboard-unit {
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, #1e293b 0%, #0f172a 100%);
  border: 1.5px solid #475569;
  border-radius: 4px;
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.55), inset 0 1px 0 rgba(255, 255, 255, 0.12);
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 2.5px 3.5px;
  position: relative;
}

.keyboard-layout-tag {
  position: absolute;
  top: -16px;
  right: 2px;
  font-size: 0.65rem;
  font-weight: 700;
  color: #94a3b8;
  background: rgba(15, 23, 42, 0.8);
  border: 1px solid #334155;
  padding: 0 4px;
  border-radius: 3px;
  pointer-events: none;
}

.keyboard-chassis {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
}

.key-strip {
  display: flex;
  gap: 1.5px;
  height: 18%;
  align-items: center;
}

.cap {
  flex: 1;
  height: 100%;
  background: #334155;
  border-radius: 1px;
  box-shadow: 0 1px 0 rgba(0, 0, 0, 0.4);
}

.cap-esc {
  background: #ef4444;
  flex: 1.3;
}

.cap-accent {
  background: #f59e0b;
}

.cap-enter {
  background: #38bdf8;
  flex: 1.6;
}

.cap-ctrl {
  background: #475569;
  flex: 1.4;
}

.cap-space {
  background: #64748b;
  flex: 4.8;
}

.cap-arrow {
  background: #38bdf8;
  flex: 0.9;
}

.cap-numpad {
  background: #2563eb;
  flex: 0.95;
}

.cap-num-f {
  background: #475569;
  flex: 0.95;
}

.cap-numpad-zero {
  background: #1e3a8a;
  flex: 1.8;
}

.keyboard-rgb-glow {
  position: absolute;
  bottom: -1px;
  left: 10%;
  width: 80%;
  height: 2px;
  background: linear-gradient(90deg, #38bdf8, #818cf8, #f472b6);
  opacity: 0.75;
  filter: blur(1.5px);
  pointer-events: none;
}

/* 独立人体工学无线鼠标 */
.mouse-item {
  display: flex;
  flex-direction: column;
}

.wireless-mouse-unit {
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, #1e293b, #090d16);
  border: 1.5px solid #475569;
  border-radius: 10px 10px 12px 12px;
  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  overflow: hidden;
}

.mouse-shell {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
}

.mouse-buttons-split {
  position: absolute;
  top: 0;
  width: 1px;
  height: 40%;
  background: rgba(255, 255, 255, 0.15);
}

.mouse-wheel {
  position: absolute;
  top: 4px;
  width: 4px;
  height: 7px;
  background: #38bdf8;
  border-radius: 2px;
  box-shadow: 0 0 5px #38bdf8;
}

.mouse-thumb-rest {
  position: absolute;
  left: 0;
  bottom: 2px;
  width: 4px;
  height: 50%;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 0 3px 3px 0;
}

/* 监听音箱 */
.speaker-box {
  background: linear-gradient(180deg, #1a2234, #0b1120);
  border: 2px solid #334155;
  border-radius: 4px;
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.6);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-evenly;
  padding: 8px 0;
}

.speaker-tweeter {
  width: 22px;
  height: 22px;
  background: radial-gradient(circle, #0f172a 40%, #475569 80%, #000 100%);
  border-radius: 50%;
  border: 1.5px solid #64748b;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.8);
}

.speaker-woofer {
  width: 48px;
  height: 48px;
  background: radial-gradient(circle, #f59e0b 35%, #b45309 65%, #1e293b 90%);
  border-radius: 50%;
  border: 2px solid #334155;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.7), inset 0 0 8px rgba(0, 0, 0, 0.9);
}

.speaker-label {
  font-size: 0.68rem;
  font-weight: 700;
  color: #64748b;
}

/* 笔记本电脑 */
.laptop-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.laptop-screen-lid {
  width: 100%;
  height: 84%;
  background: #090d16;
  border: 3px solid #334155;
  border-radius: 6px 6px 0 0;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  overflow: hidden;
}

.laptop-camera-notch {
  width: 16px;
  height: 4px;
  background: #1e293b;
  border-radius: 0 0 3px 3px;
}

.laptop-screen-display {
  flex: 1;
  width: 100%;
  background: radial-gradient(ellipse at center, rgba(56, 189, 248, 0.3), #090d16 75%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.laptop-tag {
  font-size: 0.75rem;
  color: #e2e8f0;
  font-weight: 600;
  background: rgba(0, 0, 0, 0.4);
  padding: 2px 8px;
  border-radius: 4px;
}

.laptop-base {
  width: 106%;
  height: 16%;
  background: linear-gradient(180deg, #475569, #1e293b);
  border-radius: 0 0 5px 5px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.6);
  position: relative;
}

.laptop-notch-cutout {
  width: 28px;
  height: 4px;
  background: #0f172a;
  border-radius: 0 0 3px 3px;
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
}

/* 屏幕挂灯 */
.screen-light-bar {
  position: absolute;
  top: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 70%;
  max-width: 220px;
  height: 10px;
  z-index: 120;
  pointer-events: none;
}

.light-fixture {
  width: 100%;
  height: 6px;
  background: linear-gradient(90deg, #1e293b, #475569, #1e293b);
  border: 1px solid #64748b;
  border-radius: 3px;
  box-shadow: 0 0 8px rgba(245, 158, 11, 0.4);
}

.light-beam {
  position: absolute;
  top: 6px;
  left: -15%;
  width: 130%;
  height: 65px;
  background: linear-gradient(180deg, rgba(254, 243, 199, 0.25) 0%, rgba(245, 158, 11, 0.05) 50%, transparent 100%);
  clip-path: polygon(15% 0%, 85% 0%, 100% 100%, 0% 100%);
  pointer-events: none;
}

/* 显示器矩形样式 */
.monitor-card {
  position: absolute;
  border: 4px solid #1e293b;
  border-radius: 8px;
  background-color: #090d16;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.45);
  cursor: grab;
  user-select: none;
  touch-action: none;
  display: flex;
  flex-direction: column;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.monitor-card.is-dragging {
  cursor: grabbing;
  box-shadow: 0 18px 40px rgba(0, 0, 0, 0.7);
  opacity: 0.95;
}

.monitor-card.is-selected {
  outline: 2px solid #38bdf8;
  outline-offset: 2px;
  box-shadow: 0 0 24px rgba(56, 189, 248, 0.35);
}

.screen-inner {
  flex: 1;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.screen-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  pointer-events: none;
}

.size-pill {
  display: flex;
  align-items: baseline;
  gap: 6px;
  background: rgba(0, 0, 0, 0.45);
  padding: 4px 10px;
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.1);
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

.physical-dim {
  font-size: 0.72rem;
  color: rgba(255, 255, 255, 0.75);
  background: rgba(0, 0, 0, 0.25);
  padding: 2px 6px;
  border-radius: 4px;
}

.screen-chin {
  position: absolute;
  bottom: 3px;
  width: 100%;
  display: flex;
  justify-content: center;
  pointer-events: none;
}

.chin-dot {
  width: 5px;
  height: 5px;
  background: rgba(255, 255, 255, 0.25);
  border-radius: 50%;
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
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85rem;
  transition: all 0.15s;
}

.quick-btn:hover {
  background: #334155;
}

.quick-btn.active-light {
  background: #f59e0b;
  color: #000;
}

.quick-btn.delete-btn:hover {
  background: #ef4444;
}

/* 吸附参考红线 */
.snap-guide-line {
  position: absolute;
  background: #38bdf8;
  box-shadow: 0 0 8px #38bdf8;
  pointer-events: none;
  z-index: 30;
}

.snap-guide-line.horizontal {
  height: 2px;
}

.snap-guide-line.vertical {
  width: 2px;
}

/* 空白提示 */
.canvas-empty-hint {
  position: absolute;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  text-align: center;
  pointer-events: auto;
  z-index: 20;
}

.empty-icon {
  font-size: 3rem;
  opacity: 0.8;
}

.empty-title {
  color: #94a3b8;
  font-size: 1rem;
}

/* 悬浮 HUD 控件 (放大/缩小/适配) */
.canvas-hud {
  position: absolute;
  bottom: 24px;
  left: 20px;
  display: flex;
  align-items: center;
  background: rgba(15, 23, 42, 0.85);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(51, 65, 85, 0.6);
  border-radius: 10px;
  padding: 4px;
  gap: 4px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.35);
  z-index: 30;
}

.hud-btn {
  width: 34px;
  height: 34px;
  border: none;
  background: transparent;
  color: #f1f5f9;
  font-size: 0.9rem;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s;
}

.hud-btn:hover {
  background: rgba(51, 65, 85, 0.7);
}

.hud-zoom-label {
  font-size: 0.78rem;
  color: #94a3b8;
  font-family: monospace;
  padding: 0 6px;
  min-width: 44px;
  text-align: center;
  cursor: pointer;
}

.hud-zoom-label:hover {
  color: #38bdf8;
}

/* 桌面端右侧检查器/侧边栏 */
.desktop-inspector {
  position: absolute;
  top: 70px;
  right: 20px;
  width: 320px;
  max-height: calc(100vh - 90px);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  z-index: 35;
  pointer-events: none;
}

.desktop-inspector > * {
  pointer-events: auto;
}

.inspector-card {
  background: rgba(15, 23, 42, 0.88);
  backdrop-filter: blur(16px);
  border: 1px solid rgba(51, 65, 85, 0.7);
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.35);
}

.card-title {
  font-size: 0.92rem;
  font-weight: 600;
  margin-bottom: 10px;
  color: #e2e8f0;
}

.card-header-flex {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.card-header-flex .card-title {
  margin-bottom: 0;
}

.close-x {
  background: transparent;
  border: none;
  color: #94a3b8;
  font-size: 1.1rem;
  cursor: pointer;
  padding: 4px;
}

.close-x:hover {
  color: white;
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

.quick-chips, .ratio-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}

.chip-button {
  background: #1e293b;
  border: 1px solid #334155;
  color: #cbd5e1;
  padding: 5px 10px;
  border-radius: 6px;
  font-size: 0.78rem;
  cursor: pointer;
  transition: all 0.15s;
}

.chip-button.sm {
  padding: 2.5px 8px;
  font-size: 0.74rem;
}

.chip-button:hover {
  background: #334155;
}

.chip-button.active {
  background: #2563eb;
  border-color: #3b82f6;
  color: white;
  font-weight: 600;
}

.accessories-toggle-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.accessory-block {
  background: rgba(30, 41, 59, 0.6);
  padding: 8px 10px;
  border-radius: 6px;
  border: 1px solid #334155;
}

.sub-layout-picker {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.picker-label {
  font-size: 0.74rem;
  color: #94a3b8;
}

.toggle-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(30, 41, 59, 0.6);
  padding: 6px 10px;
  border-radius: 6px;
  border: 1px solid #334155;
}

.accessory-block .toggle-row {
  background: transparent;
  padding: 0;
  border: none;
}

.toggle-label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.8rem;
  color: #e2e8f0;
  cursor: pointer;
  user-select: none;
}

.sub-chips {
  display: flex;
  gap: 4px;
}

.input-row {
  display: flex;
  align-items: center;
  gap: 10px;
}

.range-slider {
  flex: 1;
  accent-color: #38bdf8;
  cursor: pointer;
}

.number-input-wrap {
  display: flex;
  align-items: center;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 4px 8px;
  width: 80px;
}

.number-input-wrap input {
  width: 100%;
  background: transparent;
  border: none;
  color: white;
  font-size: 0.88rem;
  text-align: right;
  outline: none;
}

.number-input-wrap .unit {
  font-size: 0.75rem;
  color: #94a3b8;
  margin-left: 4px;
}

.custom-ratio-row {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: 8px;
}

.custom-ratio-row input {
  width: 60px;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 6px;
  color: white;
  padding: 4px 8px;
  font-size: 0.85rem;
}

.metric-badge {
  background: rgba(30, 41, 59, 0.7);
  border: 1px solid #334155;
  border-radius: 6px;
  padding: 6px 10px;
  font-size: 0.78rem;
  color: #38bdf8;
  text-align: center;
}

.card-footer-buttons {
  display: flex;
  gap: 8px;
  margin-top: 12px;
}

.card-footer-buttons button {
  flex: 1;
}

/* 按钮样式 */
.primary-btn {
  background: linear-gradient(135deg, #2563eb, #3b82f6);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  font-size: 0.86rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.35);
}

.primary-btn:hover {
  background: linear-gradient(135deg, #1d4ed8, #2563eb);
}

.primary-btn.full-btn {
  width: 100%;
  padding: 10px;
}

.secondary-btn {
  background: #334155;
  color: white;
  border: 1px solid #475569;
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 0.82rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s;
}

.secondary-btn:hover {
  background: #475569;
}

.danger-btn {
  background: rgba(239, 68, 68, 0.18);
  color: #f87171;
  border: 1px solid rgba(239, 68, 68, 0.4);
  padding: 8px 12px;
  border-radius: 8px;
  font-size: 0.82rem;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.15s;
}

.danger-btn:hover {
  background: #ef4444;
  color: white;
}

.mt-2 { margin-top: 8px; }
.mt-3 { margin-top: 12px; }

/* 移动端悬浮动作栏 */
.mobile-floating-actions {
  position: absolute;
  bottom: 24px;
  right: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  z-index: 30;
}

.floating-add-btn {
  background: linear-gradient(135deg, #2563eb, #38bdf8);
  color: white;
  border: none;
  padding: 12px 18px;
  border-radius: 24px;
  font-size: 0.9rem;
  font-weight: 600;
  box-shadow: 0 8px 24px rgba(37, 99, 235, 0.45);
  cursor: pointer;
}

.floating-acc-btn {
  background: rgba(30, 41, 59, 0.95);
  color: #f1f5f9;
  border: 1px solid #475569;
  padding: 10px 16px;
  border-radius: 24px;
  font-size: 0.84rem;
  font-weight: 600;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.35);
  cursor: pointer;
}

.floating-edit-btn {
  background: rgba(15, 23, 42, 0.92);
  color: #38bdf8;
  border: 1px solid #38bdf8;
  padding: 10px 16px;
  border-radius: 24px;
  font-size: 0.84rem;
  font-weight: 600;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  cursor: pointer;
}

/* 移动端底部抽屉 (Bottom Sheet) */
.mobile-drawer-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.65);
  backdrop-filter: blur(4px);
  z-index: 100;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
}

.mobile-drawer-content {
  background: #0f172a;
  border-top: 1px solid #334155;
  border-radius: 20px 20px 0 0;
  padding: 12px 20px 32px 20px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 -10px 30px rgba(0, 0, 0, 0.5);
}

.drawer-drag-bar {
  width: 40px;
  height: 4px;
  background: #475569;
  border-radius: 2px;
  margin: 0 auto 14px auto;
  cursor: pointer;
}

.drawer-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.drawer-header h3 {
  font-size: 1.1rem;
  font-weight: 600;
}

.drawer-body .field-group {
  margin-bottom: 18px;
}

/* 预设搭配弹窗 */
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(6px);
  z-index: 120;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}

.modal-card {
  background: #0f172a;
  border: 1px solid #334155;
  border-radius: 16px;
  width: 100%;
  max-width: 520px;
  padding: 20px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.6);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.modal-title {
  font-size: 1.15rem;
  font-weight: 700;
  color: #f8fafc;
}

.presets-grid {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-height: 60vh;
  overflow-y: auto;
}

.preset-item {
  display: flex;
  flex-direction: column;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 10px;
  padding: 12px 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.preset-item:hover {
  background: #273549;
  border-color: #38bdf8;
  transform: translateY(-1px);
}

.preset-item .preset-icon {
  font-size: 1.5rem;
  margin-bottom: 4px;
}

.preset-item .preset-name {
  font-size: 0.95rem;
  font-weight: 600;
  color: #f1f5f9;
}

.preset-item .preset-desc {
  font-size: 0.8rem;
  color: #94a3b8;
  margin-top: 2px;
}
</style>