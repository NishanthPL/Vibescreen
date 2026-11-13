[index.html](https://github.com/user-attachments/files/34816292/index.html)
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <title>VibeScreen — Interactive Ambient Music Visualizer & Live Wallpaper</title>
  <meta name="description" content="Transform any screen into a reactive music visualizer and ambient live wallpaper. Reacts in real time to your room microphone, browser tab audio, local music files, or built-in synth." />
  <meta name="keywords" content="music visualizer, live wallpaper, audio reactive, ambient screen, web audio visualizer, synthwave wallpaper" />
  <meta name="theme-color" content="#050508" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
  <meta name="apple-mobile-web-app-title" content="VibeScreen" />

  <!-- Open Graph / Social Share Preview -->
  <meta property="og:type" content="website" />
  <meta property="og:title" content="VibeScreen — Interactive Ambient Music Visualizer" />
  <meta property="og:description" content="A live wallpaper and music visualizer that shifts colors and motion to the beat of your background music." />
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="VibeScreen — Interactive Ambient Music Visualizer" />
  <meta name="twitter:description" content="Turn your screen into a reactive music wallpaper with 8 visual modes and dynamic color engines." />

  <!-- Inline SVG App Favicon -->
  <link rel="icon" type="image/svg+xml" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cdefs%3E%3ClinearGradient id='g' x1='0' y1='0' x2='1' y2='1'%3E%3Cstop offset='0%25' stop-color='%2322d3ee'/%3E%3Cstop offset='50%25' stop-color='%23d946ef'/%3E%3Cstop offset='100%25' stop-color='%23f59e0b'/%3E%3C/linearGradient%3E%3C/defs%3E%3Crect width='64' height='64' rx='18' fill='%23090a10'/%3E%3Ccircle cx='32' cy='32' r='18' fill='none' stroke='url(%23g)' stroke-width='5'/%3E%3Ccircle cx='32' cy='32' r='7' fill='%23ffffff'/%3E%3C/svg%3E" />

  <!-- Tailwind CSS -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Google Fonts: Inter & JetBrains Mono -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
  <!-- Lucide Icons -->
  <script src="https://unpkg.com/lucide@latest"></script>

  <style>
    :root {
      --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
      --font-mono: 'JetBrains Mono', monospace;
    }

    * {
      box-sizing: border-box;
      user-select: none;
      -webkit-user-select: none;
    }

    body, html {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: #050508;
      font-family: var(--font-sans);
      color: #ffffff;
    }

    /* Fullscreen Visualizer Canvas */
    #visualizer-canvas {
      position: fixed;
      inset: 0;
      width: 100%;
      height: 100%;
      display: block;
      z-index: 1;
    }

    /* Subtle Film Grain & Vignette Overlay for Wallpaper Depth */
    .ambient-overlay {
      position: fixed;
      inset: 0;
      pointer-events: none;
      z-index: 2;
      background: radial-gradient(circle at center, transparent 35%, rgba(0, 0, 0, 0.55) 100%);
    }

    /* Glassmorphism Utilities */
    .glass-panel {
      background: rgba(14, 16, 24, 0.52);
      backdrop-filter: blur(24px) saturate(160%);
      -webkit-backdrop-filter: blur(24px) saturate(160%);
      border: 1px solid rgba(255, 255, 255, 0.12);
      box-shadow: 0 20px 50px rgba(0, 0, 0, 0.45), inset 0 1px 0 rgba(255, 255, 255, 0.1);
    }

    .glass-pill {
      background: rgba(255, 255, 255, 0.07);
      border: 1px solid rgba(255, 255, 255, 0.1);
      transition: all 0.22s cubic-bezier(0.22, 1, 0.36, 1);
    }

    .glass-pill:hover {
      background: rgba(255, 255, 255, 0.14);
      border-color: rgba(255, 255, 255, 0.24);
      transform: translateY(-1px);
    }

    .glass-pill.active {
      background: rgba(255, 255, 255, 0.2);
      border-color: rgba(255, 255, 255, 0.45);
      box-shadow: 0 0 20px rgba(255, 255, 255, 0.16);
    }

    /* Hide scrollbar for horizontal interface bar */
    .no-scrollbar::-webkit-scrollbar {
      display: none;
    }
    .no-scrollbar {
      -ms-overflow-style: none;
      scrollbar-width: none;
    }

    /* HUD Auto-Hide Transitions */
    .hud-layer {
      transition: opacity 0.6s cubic-bezier(0.22, 1, 0.36, 1), transform 0.6s cubic-bezier(0.22, 1, 0.36, 1);
    }

    body.hud-hidden .hud-layer {
      opacity: 0;
      pointer-events: none;
    }

    body.hud-hidden .hud-top {
      transform: translateY(-16px);
    }

    body.hud-hidden .hud-bottom {
      transform: translateY(20px);
    }

    body.hud-hidden {
      cursor: none;
    }

    /* Custom Range Sliders */
    input[type="range"] {
      -webkit-appearance: none;
      appearance: none;
      background: rgba(255, 255, 255, 0.16);
      height: 5px;
      border-radius: 999px;
      outline: none;
      cursor: pointer;
    }

    input[type="range"]::-webkit-slider-thumb {
      -webkit-appearance: none;
      appearance: none;
      width: 14px;
      height: 14px;
      border-radius: 50%;
      background: #ffffff;
      box-shadow: 0 0 10px rgba(255, 255, 255, 0.7);
      cursor: pointer;
      transition: transform 0.15s ease;
    }

    input[type="range"]::-webkit-slider-thumb:hover {
      transform: scale(1.25);
    }

    /* Equalizer Bars Animation */
    @keyframes eqBounce {
      0%, 100% { transform: scaleY(0.25); }
      50% { transform: scaleY(1); }
    }

    .eq-bar {
      transform-origin: bottom;
      animation: eqBounce 0.8s ease-in-out infinite;
    }

    .eq-paused .eq-bar {
      animation-play-state: paused;
      transform: scaleY(0.25);
    }

    /* Slow Spin for Album Disc */
    @keyframes slowSpin {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }

    .spin-disc {
      animation: slowSpin 18s linear infinite;
    }

    .spin-paused {
      animation-play-state: paused;
    }

    /* Drag & Drop Overlay */
    #drop-overlay {
      transition: opacity 0.25s ease;
    }
  </style>
</head>
<body class="antialiased selection:bg-white/20">

  <!-- Interactive Visualizer Canvas -->
  <canvas id="visualizer-canvas"></canvas>
  <div class="ambient-overlay"></div>

  <!-- Hidden File Input for Local Audio -->
  <input type="file" id="audio-file-input" accept="audio/*" class="hidden" />

  <!-- Drag & Drop Overlay -->
  <div id="drop-overlay" class="fixed inset-0 z-50 hidden items-center justify-center bg-black/75 backdrop-blur-xl pointer-events-none">
    <div class="glass-panel rounded-3xl p-10 text-center max-w-md mx-4 border-2 border-dashed border-white/40">
      <div class="w-16 h-16 rounded-2xl bg-white/10 flex items-center justify-center mx-auto mb-4">
        <i data-lucide="music" class="w-8 h-8 text-white"></i>
      </div>
      <h2 class="text-2xl font-bold tracking-tight mb-2">Drop Audio File to Vibe</h2>
      <p class="text-sm text-white/70">Supports MP3, WAV, OGG, FLAC, and M4A</p>
    </div>
  </div>

  <!-- Notification Toast (Replaces alert) -->
  <div id="toast-container" class="fixed top-20 left-1/2 -translate-x-1/2 z-50 flex flex-col items-center gap-2 pointer-events-none w-full max-w-md px-4"></div>

  <!-- TOP BAR HUD -->
  <header class="hud-layer hud-top fixed top-0 left-0 right-0 z-30 p-3 sm:p-5 flex flex-wrap items-center justify-between gap-2.5 pointer-events-none">
    <!-- Brand & Live Vibe Status -->
    <div class="glass-panel rounded-full px-3.5 py-2 flex items-center gap-2.5 pointer-events-auto">
      <div id="status-dot" class="w-2.5 h-2.5 rounded-full bg-emerald-400 shadow-[0_0_12px_#34d399] transition-colors duration-500"></div>
      <span class="font-bold tracking-tight text-xs sm:text-sm">VibeScreen</span>
      <span class="text-white/25 hidden sm:inline">|</span>
      <span id="vibe-mood-badge" class="hidden sm:inline text-[11px] font-medium text-white/80 tracking-wide uppercase">Standby</span>
    </div>

    <!-- Center: Expanded 8-Interface Switcher -->
    <nav class="glass-panel rounded-full p-1 flex items-center gap-1 pointer-events-auto overflow-x-auto no-scrollbar max-w-full order-3 lg:order-2">
      <button data-visual="aura" class="visual-btn glass-pill active rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="sparkles" class="w-3.5 h-3.5"></i>
        <span>Ambient Aura</span>
      </button>
      <button data-visual="orb" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="disc" class="w-3.5 h-3.5"></i>
        <span>Pulse Orb</span>
      </button>
      <button data-visual="starfield" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="orbit" class="w-3.5 h-3.5"></i>
        <span>Starfield Warp</span>
      </button>
      <button data-visual="horizon" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="waves" class="w-3.5 h-3.5"></i>
        <span>Waveform</span>
      </button>
      <button data-visual="aurora" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="wind" class="w-3.5 h-3.5"></i>
        <span>Aurora Borealis</span>
      </button>
      <button data-visual="cybergrid" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="grid" class="w-3.5 h-3.5"></i>
        <span>Cyber Grid</span>
      </button>
      <button data-visual="dna" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="dna" class="w-3.5 h-3.5"></i>
        <span>DNA Helix</span>
      </button>
      <button data-visual="kaleidoscope" class="visual-btn glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5 whitespace-nowrap">
        <i data-lucide="aperture" class="w-3.5 h-3.5"></i>
        <span>Kaleidoscope</span>
      </button>
    </nav>

    <!-- Right: Color Palette, Quick Settings, Clock & Fullscreen -->
    <div class="glass-panel rounded-full p-1.5 flex items-center gap-1.5 pointer-events-auto order-2 lg:order-3">
      <!-- Expanded Palette Selector -->
      <div class="relative">
        <button id="palette-btn" title="Color Palette & Auto-Vibe Engine (C)" class="glass-pill rounded-full px-3 py-1.5 text-xs font-medium flex items-center gap-1.5">
          <span id="palette-swatch" class="w-3 h-3 rounded-full bg-gradient-to-tr from-cyan-400 via-fuchsia-500 to-amber-400"></span>
          <span id="palette-label" class="hidden sm:inline">Auto-Vibe Spectral</span>
        </button>
        <!-- Palette Dropdown Grid -->
        <div id="palette-menu" class="hidden absolute right-0 mt-3 w-72 sm:w-80 glass-panel rounded-2xl p-2.5 z-40 shadow-2xl max-h-[75vh] overflow-y-auto">
          <div class="text-[10px] font-bold uppercase tracking-wider text-white/50 px-2.5 py-1">Dynamic Auto-Vibe Engines</div>
          <div class="grid grid-cols-1 gap-1 mb-2">
            <button data-palette="auto" data-grad="from-cyan-400 via-fuchsia-500 to-amber-400" class="palette-option w-full text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between bg-white/10">
              <span class="flex items-center gap-2">
                <span class="w-3.5 h-3.5 rounded-full bg-gradient-to-r from-cyan-400 via-fuchsia-500 to-amber-400"></span>
                Auto-Vibe (Spectral)
              </span>
              <i data-lucide="check" class="w-3.5 h-3.5 text-white"></i>
            </button>
            <button data-palette="moodshift" data-grad="from-orange-500 via-rose-500 to-indigo-500" class="palette-option w-full text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2">
                <span class="w-3.5 h-3.5 rounded-full bg-gradient-to-r from-orange-500 via-rose-500 to-indigo-500"></span>
                Auto-Vibe (Mood Shift)
              </span>
            </button>
          </div>

          <div class="text-[10px] font-bold uppercase tracking-wider text-white/50 px-2.5 py-1 border-t border-white/10 pt-2">Curated Color Themes</div>
          <div class="grid grid-cols-2 gap-1 mt-1">
            <button data-palette="midnight" data-grad="from-indigo-500 via-purple-500 to-pink-500" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500"></span>
                <span class="truncate">Midnight Neon</span>
              </span>
            </button>
            <button data-palette="sunset" data-grad="from-rose-500 via-orange-500 to-amber-400" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-rose-500 via-orange-500 to-amber-400"></span>
                <span class="truncate">Sunset Vapor</span>
              </span>
            </button>
            <button data-palette="aurora" data-grad="from-emerald-400 via-teal-500 to-cyan-500" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-emerald-400 via-teal-500 to-cyan-500"></span>
                <span class="truncate">Emerald Aurora</span>
              </span>
            </button>
            <button data-palette="crimson" data-grad="from-red-600 via-rose-600 to-violet-700" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-red-600 via-rose-600 to-violet-700"></span>
                <span class="truncate">Crimson Pulse</span>
              </span>
            </button>
            <button data-palette="cybergold" data-grad="from-amber-400 via-yellow-500 to-orange-600" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-amber-400 via-yellow-500 to-orange-600"></span>
                <span class="truncate">Cyber Gold</span>
              </span>
            </button>
            <button data-palette="arctic" data-grad="from-sky-300 via-blue-400 to-indigo-400" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-sky-300 via-blue-400 to-indigo-400"></span>
                <span class="truncate">Arctic Ice</span>
              </span>
            </button>
            <button data-palette="amethyst" data-grad="from-purple-600 via-fuchsia-500 to-indigo-600" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-purple-600 via-fuchsia-500 to-indigo-600"></span>
                <span class="truncate">Royal Amethyst</span>
              </span>
            </button>
            <button data-palette="solar" data-grad="from-yellow-400 via-orange-500 to-red-600" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-yellow-400 via-orange-500 to-red-600"></span>
                <span class="truncate">Solar Flare</span>
              </span>
            </button>
            <button data-palette="monochrome" data-grad="from-zinc-200 via-slate-400 to-zinc-600" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-zinc-200 via-slate-400 to-zinc-600"></span>
                <span class="truncate">Monochrome Zen</span>
              </span>
            </button>
            <button data-palette="sakura" data-grad="from-pink-300 via-rose-400 to-fuchsia-400" class="palette-option text-left px-2.5 py-2 rounded-xl text-xs font-medium hover:bg-white/10 flex items-center justify-between">
              <span class="flex items-center gap-2 truncate">
                <span class="w-3 h-3 rounded-full flex-shrink-0 bg-gradient-to-r from-pink-300 via-rose-400 to-fuchsia-400"></span>
                <span class="truncate">Sakura Dream</span>
              </span>
            </button>
          </div>
        </div>
      </div>

      <!-- Quick Customization Popover Button -->
      <div class="relative">
        <button id="settings-btn" title="Visual & Wallpaper Settings" class="glass-pill rounded-full p-2 text-white/90">
          <i data-lucide="sliders-horizontal" class="w-4 h-4"></i>
        </button>
        <div id="settings-menu" class="hidden absolute right-0 mt-3 w-72 glass-panel rounded-2xl p-4 z-40 shadow-2xl space-y-3.5 text-left">
          <div class="flex items-center justify-between border-b border-white/10 pb-2">
            <span class="text-xs font-bold uppercase tracking-wider text-white/80">Visual & HUD Tuning</span>
            <span class="text-[10px] font-mono text-white/40">V / C / F</span>
          </div>

          <!-- Glow / Bloom Intensity -->
          <div>
            <div class="flex justify-between text-xs mb-1">
              <span class="text-white/75">Glow & Bloom Intensity</span>
              <span id="glow-val-text" class="font-mono text-white/60">100%</span>
            </div>
            <input id="glow-slider" type="range" min="0.2" max="2.0" step="0.05" value="1.0" class="w-full" />
          </div>

          <!-- Color Shift Reactivity -->
          <div>
            <div class="flex justify-between text-xs mb-1">
              <span class="text-white/75">Color Shift Reactivity</span>
              <span id="color-speed-text" class="font-mono text-white/60">1.0x</span>
            </div>
            <input id="color-speed-slider" type="range" min="0.2" max="2.5" step="0.1" value="1.0" class="w-full" />
          </div>

          <!-- Clock & Overlay Toggles -->
          <div class="pt-1 border-t border-white/10 space-y-2">
            <div class="flex items-center justify-between">
              <span class="text-xs text-white/80">Clock Format</span>
              <div class="flex rounded-lg bg-white/10 p-0.5 text-[11px]">
                <button id="fmt-12h" class="px-2 py-0.5 rounded-md bg-white/20 font-medium">12H</button>
                <button id="fmt-24h" class="px-2 py-0.5 rounded-md text-white/60 hover:text-white">24H</button>
              </div>
            </div>

            <div class="flex items-center justify-between">
              <span class="text-xs text-white/80">Clock Typography</span>
              <div class="flex rounded-lg bg-white/10 p-0.5 text-[11px]">
                <button id="clock-style-minimal" class="px-2 py-0.5 rounded-md bg-white/20 font-medium">Minimal</button>
                <button id="clock-style-glass" class="px-2 py-0.5 rounded-md text-white/60 hover:text-white">Glass</button>
              </div>
            </div>

            <div class="flex items-center justify-between">
              <span class="text-xs text-white/80">Now Playing Pill</span>
              <button id="toggle-nowplaying-btn" class="px-2.5 py-0.5 rounded-lg bg-emerald-500/25 border border-emerald-400/40 text-emerald-200 text-[11px] font-medium">Visible</button>
            </div>
          </div>
        </div>
      </div>

      <!-- Toggle Clock/Wallpaper Info -->
      <button id="toggle-clock-btn" title="Toggle Wallpaper Clock & Date" class="glass-pill active rounded-full p-2 text-white/90">
        <i data-lucide="clock" class="w-4 h-4"></i>
      </button>

      <!-- Toggle Fullscreen -->
      <button id="fullscreen-btn" title="Fullscreen Wallpaper Mode (F)" class="glass-pill rounded-full p-2 text-white/90">
        <i id="fullscreen-icon" data-lucide="maximize" class="w-4 h-4"></i>
      </button>

      <!-- Info, Shortcuts & Share Modal Trigger -->
      <button id="about-btn" title="How to Use, Shortcuts & Privacy" class="glass-pill rounded-full p-2 text-white/90">
        <i data-lucide="help-circle" class="w-4 h-4"></i>
      </button>
    </div>
  </header>

  <!-- CREATOR PROFILE BADGE (Bottom-Right Corner, Auto-Hides with HUD) -->
  <div id="creator-profile-badge" class="hud-layer hud-bottom fixed bottom-4 right-4 sm:bottom-6 sm:right-6 z-30 pointer-events-auto">
    <button id="open-profile-modal-btn" title="Customize Creator Profile" class="glass-panel rounded-full pl-2 pr-3.5 py-1.5 flex items-center gap-2.5 hover:border-white/30 transition-all group">
      <div id="profile-avatar-circle" class="w-7 h-7 rounded-full bg-gradient-to-tr from-cyan-400 via-fuchsia-500 to-amber-400 flex items-center justify-center text-xs font-bold text-white shadow-md">
        Y
      </div>
      <div class="text-left leading-tight">
        <div class="text-[9px] uppercase tracking-widest text-white/45 font-semibold">Created by</div>
        <div id="profile-display-name" class="text-xs font-semibold text-white/95 group-hover:text-white flex items-center gap-1">
          <span>Your Name</span>
          <i data-lucide="pencil" class="w-2.5 h-2.5 text-white/40 group-hover:text-white/80"></i>
        </div>
      </div>
    </button>
  </div>

  <!-- PROFILE CUSTOMIZATION MODAL -->
  <div id="profile-modal" class="fixed inset-0 z-50 hidden items-center justify-center bg-black/75 backdrop-blur-xl p-4">
    <div class="glass-panel rounded-3xl p-6 sm:p-7 max-w-sm w-full text-left relative shadow-2xl border border-white/15">
      <button id="close-profile-btn" class="absolute top-5 right-5 glass-pill rounded-full p-2 text-white/80 hover:text-white">
        <i data-lucide="x" class="w-4 h-4"></i>
      </button>

      <div class="flex items-center gap-3 mb-5">
        <div id="modal-avatar-preview" class="w-11 h-11 rounded-full bg-gradient-to-tr from-cyan-400 via-fuchsia-500 to-amber-400 flex items-center justify-center text-base font-bold text-white shadow-lg">
          Y
        </div>
        <div>
          <h2 class="text-base font-bold tracking-tight">Creator Profile</h2>
          <p class="text-xs text-white/60">Shown on your public VibeScreen</p>
        </div>
      </div>

      <div class="space-y-3.5 text-xs">
        <div>
          <label class="block text-white/70 font-medium mb-1">Your Name / Handle</label>
          <input id="profile-name-input" type="text" maxlength="28" placeholder="Enter your name..." class="w-full rounded-xl bg-white/10 border border-white/15 px-3.5 py-2.5 text-white placeholder-white/40 outline-none focus:border-white/45" />
        </div>
        <div>
          <label class="block text-white/70 font-medium mb-1">Role / Bio Tagline</label>
          <input id="profile-bio-input" type="text" maxlength="48" placeholder="e.g. Music Lover & Creator" class="w-full rounded-xl bg-white/10 border border-white/15 px-3.5 py-2.5 text-white placeholder-white/40 outline-none focus:border-white/45" />
        </div>
        <div>
          <label class="block text-white/70 font-medium mb-1">Social / Website Link (Optional)</label>
          <input id="profile-link-input" type="url" placeholder="https://instagram.com/yourhandle" class="w-full rounded-xl bg-white/10 border border-white/15 px-3.5 py-2.5 text-white placeholder-white/40 outline-none focus:border-white/45" />
        </div>
      </div>

      <div class="flex items-center justify-end gap-2.5 mt-6">
        <button id="save-profile-btn" class="w-full rounded-xl px-5 py-2.5 text-xs font-semibold bg-white text-black hover:bg-white/90 transition-colors">
          Save Profile
        </button>
      </div>
    </div>
  </div>

  <!-- PUBLIC INFO, SHORTCUTS & PRIVACY MODAL -->
  <div id="about-modal" class="fixed inset-0 z-50 hidden items-center justify-center bg-black/75 backdrop-blur-xl p-4">
    <div class="glass-panel rounded-3xl p-6 sm:p-8 max-w-lg w-full text-left relative shadow-2xl border border-white/15">
      <button id="close-about-btn" class="absolute top-5 right-5 glass-pill rounded-full p-2 text-white/80 hover:text-white">
        <i data-lucide="x" class="w-4 h-4"></i>
      </button>

      <div class="flex items-center gap-3 mb-4">
        <div class="w-10 h-10 rounded-2xl bg-gradient-to-tr from-cyan-400 via-fuchsia-500 to-amber-400 flex items-center justify-center shadow-lg">
          <i data-lucide="sparkles" class="w-5 h-5 text-white"></i>
        </div>
        <div>
          <h2 class="text-lg font-bold tracking-tight">Welcome to VibeScreen</h2>
          <p class="text-xs text-white/60">Ambient Music Wallpaper & Real-Time Visualizer</p>
        </div>
      </div>

      <div class="space-y-3 text-xs text-white/80 leading-relaxed mb-5">
        <p><strong class="text-white">1. Choose an Audio Source:</strong> Use <em>Demo Synth</em> for instant chill beats, <em>Room / Mic</em> to react to external speakers, <em>Tab / System Audio</em> to capture Spotify/YouTube tabs, or drag &amp; drop any audio file.</p>
        <p><strong class="text-white">2. Live Wallpaper Mode:</strong> Press <kbd class="px-1.5 py-0.5 rounded bg-white/15 font-mono text-[11px]">F</kbd> for fullscreen and keep your cursor still—all menus automatically fade away into a clean ambient display.</p>
        <p class="p-3 rounded-2xl bg-emerald-500/10 border border-emerald-400/20 text-emerald-200 flex items-start gap-2.5">
          <i data-lucide="shield-check" class="w-4 h-4 flex-shrink-0 mt-0.5 text-emerald-400"></i>
          <span><strong>100% Private &amp; Local:</strong> All microphone and audio analysis runs strictly inside your browser. Zero audio is ever recorded or uploaded.</span>
        </p>
      </div>

      <!-- Creator Credit Strip inside About Modal -->
      <div class="p-3 rounded-2xl bg-white/5 border border-white/10 flex items-center justify-between gap-3 mb-5">
        <div class="flex items-center gap-3 min-w-0">
          <div id="about-avatar-circle" class="w-9 h-9 rounded-full bg-gradient-to-tr from-cyan-400 via-fuchsia-500 to-amber-400 flex items-center justify-center text-sm font-bold text-white flex-shrink-0">
            Y
          </div>
          <div class="min-w-0">
            <div id="about-profile-name" class="text-xs font-bold text-white truncate">Your Name</div>
            <div id="about-profile-bio" class="text-[11px] text-white/60 truncate">Music Lover & Creator</div>
          </div>
        </div>
        <div class="flex items-center gap-1.5 flex-shrink-0">
          <a id="about-profile-link" href="#" target="_blank" rel="noopener noreferrer" class="hidden glass-pill rounded-xl px-2.5 py-1.5 text-[11px] font-medium text-white items-center gap-1">
            <span>Visit</span>
            <i data-lucide="external-link" class="w-3 h-3"></i>
          </a>
          <button id="about-edit-profile-btn" class="glass-pill rounded-xl px-2.5 py-1.5 text-[11px] font-medium text-white/85">
            Edit
          </button>
        </div>
      </div>

      <!-- Keyboard Shortcuts Grid -->
      <div class="border-t border-white/10 pt-4 mb-5">
        <div class="text-[10px] font-bold uppercase tracking-wider text-white/50 mb-2.5">Keyboard Shortcuts</div>
        <div class="grid grid-cols-2 gap-2 text-xs">
          <div class="flex items-center justify-between bg-white/5 rounded-xl px-3 py-1.5">
            <span class="text-white/70">Next Visual</span>
            <kbd class="font-mono text-[11px] bg-white/15 px-1.5 py-0.5 rounded">V / →</kbd>
          </div>
          <div class="flex items-center justify-between bg-white/5 rounded-xl px-3 py-1.5">
            <span class="text-white/70">Cycle Colors</span>
            <kbd class="font-mono text-[11px] bg-white/15 px-1.5 py-0.5 rounded">C</kbd>
          </div>
          <div class="flex items-center justify-between bg-white/5 rounded-xl px-3 py-1.5">
            <span class="text-white/70">Fullscreen</span>
            <kbd class="font-mono text-[11px] bg-white/15 px-1.5 py-0.5 rounded">F</kbd>
          </div>
          <div class="flex items-center justify-between bg-white/5 rounded-xl px-3 py-1.5">
            <span class="text-white/70">Play / Pause</span>
            <kbd class="font-mono text-[11px] bg-white/15 px-1.5 py-0.5 rounded">Space</kbd>
          </div>
        </div>
      </div>

      <div class="flex items-center justify-between gap-3">
        <button id="share-link-btn" class="glass-pill rounded-xl px-4 py-2.5 text-xs font-medium flex items-center gap-2 text-white">
          <i data-lucide="share-2" class="w-3.5 h-3.5"></i>
          <span>Copy Page Link</span>
        </button>
        <button id="start-vibing-btn" class="rounded-xl px-5 py-2.5 text-xs font-semibold bg-white text-black hover:bg-white/90 transition-colors">
          Got It, Let's Vibe
        </button>
      </div>
    </div>
  </div>

  <!-- CENTER AMBIENT WALLPAPER OVERLAY (Always subtle, stays visible when HUD hides!) -->
  <main id="wallpaper-center" class="fixed inset-0 z-10 pointer-events-none flex flex-col items-center justify-center p-6 text-center transition-opacity duration-500">
    <!-- Ambient Clock & Date -->
    <div id="clock-widget" class="mb-6 transition-all duration-500 rounded-3xl">
      <div id="live-time" class="text-6xl sm:text-8xl md:text-9xl font-extralight tracking-tighter text-white/90 drop-shadow-[0_8px_32px_rgba(0,0,0,0.45)]" style="font-variant-numeric: tabular-nums;">
        12:00
      </div>
      <div id="live-date" class="text-xs sm:text-sm font-medium uppercase tracking-[0.28em] text-white/60 mt-1">
        FRIDAY, NOVEMBER 13
      </div>
    </div>

    <!-- Now Playing Floating Glass Pill (Apple Music / Dynamic Island Wallpaper vibe) -->
    <div id="now-playing-card" class="glass-panel rounded-3xl p-3.5 sm:p-4 pr-6 flex items-center gap-4 max-w-md w-full shadow-2xl transition-all duration-300">
      <!-- Dynamic Glowing Album Disc / Badge -->
      <div id="album-art-badge" class="relative w-14 h-14 sm:w-16 sm:h-16 rounded-2xl flex-shrink-0 flex items-center justify-center overflow-hidden shadow-lg transition-transform duration-150" style="background: linear-gradient(135deg, #6366f1, #ec4899);">
        <div id="album-disc-inner" class="w-10 h-10 rounded-full border-2 border-white/30 flex items-center justify-center spin-disc spin-paused">
          <div class="w-3 h-3 rounded-full bg-white/90"></div>
        </div>
      </div>

      <!-- Track & Source Info -->
      <div class="flex-1 min-w-0 text-left">
        <div class="flex items-center gap-2 mb-0.5">
          <span id="source-tag" class="text-[10px] font-bold uppercase tracking-wider px-2 py-0.5 rounded-full bg-white/15 text-white/90">
            Ready
          </span>
          <span id="bpm-tag" class="text-[11px] font-mono text-white/50 hidden">-- BPM</span>
        </div>
        <h1 id="track-title" class="text-sm sm:text-base font-semibold text-white truncate">
          Choose an Audio Source Below
        </h1>
        <p id="track-subtitle" class="text-xs text-white/60 truncate">
          Mic, Tab Audio, Local Music, or Built-in Synth
        </p>
      </div>

      <!-- Mini Animated Equalizer -->
      <div id="mini-eq" class="eq-paused flex items-end gap-1 h-6 px-1">
        <span class="eq-bar w-1 h-full bg-white/80 rounded-full" style="animation-delay: 0.0s;"></span>
        <span class="eq-bar w-1 h-full bg-white/80 rounded-full" style="animation-delay: 0.2s;"></span>
        <span class="eq-bar w-1 h-full bg-white/80 rounded-full" style="animation-delay: 0.4s;"></span>
        <span class="eq-bar w-1 h-full bg-white/80 rounded-full" style="animation-delay: 0.15s;"></span>
      </div>
    </div>
  </main>

  <!-- BOTTOM AUDIO SOURCE & PLAYBACK DOCK (Auto-hides on inactivity) -->
  <footer class="hud-layer hud-bottom fixed bottom-0 left-0 right-0 z-30 p-4 sm:p-6 flex flex-col items-center pointer-events-none">
    <div class="glass-panel rounded-3xl p-3 sm:p-4 w-full max-w-3xl pointer-events-auto flex flex-col gap-3">
      
      <!-- Local Audio Player Controls (Shown when a file is loaded) -->
      <div id="local-player-bar" class="hidden flex-col gap-2 pb-2 border-b border-white/10">
        <div class="flex items-center gap-3">
          <button id="play-pause-btn" class="w-9 h-9 rounded-full bg-white text-black flex items-center justify-center hover:scale-105 transition-transform flex-shrink-0">
            <i id="play-pause-icon" data-lucide="play" class="w-4 h-4 fill-current ml-0.5"></i>
          </button>
          <span id="current-time-text" class="text-xs font-mono text-white/70 w-10 text-right">0:00</span>
          <input id="seek-slider" type="range" min="0" max="100" value="0" step="0.1" class="flex-1" />
          <span id="duration-text" class="text-xs font-mono text-white/70 w-10">0:00</span>
          <div class="hidden sm:flex items-center gap-2 ml-2">
            <i data-lucide="volume-2" class="w-4 h-4 text-white/70"></i>
            <input id="volume-slider" type="range" min="0" max="1" step="0.01" value="0.85" class="w-20" />
          </div>
        </div>
      </div>

      <!-- Audio Input Mode Selector Row -->
      <div class="flex flex-wrap items-center justify-between gap-2">
        <div class="flex flex-wrap items-center gap-1.5 sm:gap-2">
          <!-- 1. Ambient Demo Synth + Preset Selector -->
          <div class="flex items-center rounded-2xl glass-pill overflow-hidden" id="synth-pill-group">
            <button id="btn-source-synth" class="source-btn px-3.5 py-2 text-xs font-medium flex items-center gap-2 hover:bg-white/10 transition-colors">
              <i data-lucide="radio" class="w-4 h-4 text-amber-300"></i>
              <span>Demo Synth</span>
            </button>
            <select id="synth-preset-select" title="Choose Demo Synth Vibe" class="bg-transparent text-[11px] font-medium text-white/80 pr-2.5 pl-1 py-2 border-l border-white/15 outline-none cursor-pointer">
              <option value="chillwave" class="bg-zinc-900 text-white">Chillwave</option>
              <option value="lofi" class="bg-zinc-900 text-white">Lo-Fi Rain</option>
              <option value="deepbass" class="bg-zinc-900 text-white">Deep Bass</option>
              <option value="cosmic" class="bg-zinc-900 text-white">Cosmic Drift</option>
            </select>
          </div>

          <!-- 2. Microphone / Room Audio -->
          <button id="btn-source-mic" class="source-btn glass-pill rounded-2xl px-3.5 py-2 text-xs font-medium flex items-center gap-2">
            <i data-lucide="mic" class="w-4 h-4 text-rose-400"></i>
            <span>Room / Mic</span>
          </button>

          <!-- 3. Tab / System Audio Capture -->
          <button id="btn-source-tab" class="source-btn glass-pill rounded-2xl px-3.5 py-2 text-xs font-medium flex items-center gap-2">
            <i data-lucide="monitor-speaker" class="w-4 h-4 text-cyan-400"></i>
            <span>Tab / System Audio</span>
          </button>

          <!-- 4. Upload Local File -->
          <button id="btn-source-file" class="source-btn glass-pill rounded-2xl px-3.5 py-2 text-xs font-medium flex items-center gap-2">
            <i data-lucide="folder-open" class="w-4 h-4 text-emerald-400"></i>
            <span>Open Audio File</span>
          </button>
        </div>

        <!-- Sensitivity / Reactivity Slider -->
        <div class="flex items-center gap-2.5 px-2 ml-auto">
          <span class="text-[11px] font-medium text-white/60 uppercase tracking-wider">Vibe Gain</span>
          <input id="sensitivity-slider" type="range" min="0.5" max="2.5" step="0.05" value="1.25" class="w-20 sm:w-24" title="Adjust visualizer reactivity" />
        </div>
      </div>
    </div>
  </footer>

  <!-- Hidden HTML5 Audio Element for Local File Playback -->
  <audio id="local-audio-element" crossorigin="anonymous"></audio>

  <script>
    /**
     * VibeScreen — Interactive Music Visualizer & Ambient Wallpaper Engine
     * Built with Web Audio API + HTML5 2D Canvas + Tailwind CSS
     */
    (function () {
      // Initialize Lucide Icons
      lucide.createIcons();

      // ==========================================
      // 1. STATE & CONFIGURATION
      // ==========================================
      const state = {
        mode: 'idle', // 'idle' | 'synth' | 'mic' | 'tab' | 'file'
        synthPreset: 'chillwave', // 'chillwave' | 'lofi' | 'deepbass' | 'cosmic'
        visual: 'aura', // 'aura' | 'orb' | 'starfield' | 'horizon' | 'aurora' | 'cybergrid' | 'dna' | 'kaleidoscope'
        palette: 'auto', // 'auto' | 'moodshift' | 10 curated themes
        sensitivity: 1.25,
        glowIntensity: 1.0,
        colorSpeed: 1.0,
        showClock: true,
        is24Hour: false,
        clockStyle: 'minimal', // 'minimal' | 'glass'
        showNowPlaying: true,
        isPlaying: false,
        trackTitle: 'Choose an Audio Source Below',
        trackSubtitle: 'Mic, Tab Audio, Local Music, or Built-in Synth',
        // Real-time audio metrics (0.0 to 1.0)
        bass: 0,
        mid: 0,
        treble: 0,
        energy: 0,
        beatPulse: 0,
        bpm: 0,
        // Smooth interpolated colors (HSL)
        hue1: 250,
        hue2: 310,
        hue3: 195,
        saturation: 78,
        lightness: 52
      };

      // Expanded Curated Color Palettes (Hues & Saturation)
      const PALETTES = {
        midnight:   { h1: 245, h2: 295, h3: 330, sat: 84 },
        sunset:     { h1: 12,  h2: 35,  h3: 335, sat: 90 },
        aurora:     { h1: 152, h2: 188, h3: 265, sat: 82 },
        crimson:    { h1: 348, h2: 280, h3: 15,  sat: 88 },
        cybergold:  { h1: 42,  h2: 24,  h3: 55,  sat: 92 },
        arctic:     { h1: 198, h2: 218, h3: 175, sat: 76 },
        amethyst:   { h1: 274, h2: 308, h3: 238, sat: 85 },
        solar:      { h1: 28,  h2: 48,  h3: 4,   sat: 94 },
        monochrome: { h1: 220, h2: 210, h3: 230, sat: 8  },
        sakura:     { h1: 334, h2: 352, h3: 295, sat: 78 }
      };

      const PALETTE_NAMES = {
        auto: 'Auto-Vibe Spectral',
        moodshift: 'Auto-Vibe Mood Shift',
        midnight: 'Midnight Neon',
        sunset: 'Sunset Vapor',
        aurora: 'Emerald Aurora',
        crimson: 'Crimson Pulse',
        cybergold: 'Cyber Gold',
        arctic: 'Arctic Ice',
        amethyst: 'Royal Amethyst',
        solar: 'Solar Flare',
        monochrome: 'Monochrome Zen',
        sakura: 'Sakura Dream'
      };

      // ==========================================
      // 2. DOM REFERENCES
      // ==========================================
      const canvas = document.getElementById('visualizer-canvas');
      const ctx = canvas.getContext('2d');
      const audioEl = document.getElementById('local-audio-element');
      const fileInput = document.getElementById('audio-file-input');
      const dropOverlay = document.getElementById('drop-overlay');

      // UI Elements
      const statusDot = document.getElementById('status-dot');
      const vibeMoodBadge = document.getElementById('vibe-mood-badge');
      const sourceTag = document.getElementById('source-tag');
      const bpmTag = document.getElementById('bpm-tag');
      const trackTitleEl = document.getElementById('track-title');
      const trackSubtitleEl = document.getElementById('track-subtitle');
      const albumArtBadge = document.getElementById('album-art-badge');
      const albumDiscInner = document.getElementById('album-disc-inner');
      const miniEq = document.getElementById('mini-eq');
      const nowPlayingCard = document.getElementById('now-playing-card');

      // Clock & Wallpaper Controls
      const clockWidget = document.getElementById('clock-widget');
      const liveTimeEl = document.getElementById('live-time');
      const liveDateEl = document.getElementById('live-date');
      const toggleClockBtn = document.getElementById('toggle-clock-btn');
      const fullscreenBtn = document.getElementById('fullscreen-btn');
      const fullscreenIcon = document.getElementById('fullscreen-icon');
      const aboutBtn = document.getElementById('about-btn');
      const aboutModal = document.getElementById('about-modal');
      const closeAboutBtn = document.getElementById('close-about-btn');
      const startVibingBtn = document.getElementById('start-vibing-btn');
      const shareLinkBtn = document.getElementById('share-link-btn');

      // Creator Profile Controls
      const openProfileModalBtn = document.getElementById('open-profile-modal-btn');
      const profileModal = document.getElementById('profile-modal');
      const closeProfileBtn = document.getElementById('close-profile-btn');
      const saveProfileBtn = document.getElementById('save-profile-btn');
      const profileNameInput = document.getElementById('profile-name-input');
      const profileBioInput = document.getElementById('profile-bio-input');
      const profileLinkInput = document.getElementById('profile-link-input');
      const profileDisplayName = document.getElementById('profile-display-name');
      const profileAvatarCircle = document.getElementById('profile-avatar-circle');
      const modalAvatarPreview = document.getElementById('modal-avatar-preview');
      const aboutAvatarCircle = document.getElementById('about-avatar-circle');
      const aboutProfileName = document.getElementById('about-profile-name');
      const aboutProfileBio = document.getElementById('about-profile-bio');
      const aboutProfileLink = document.getElementById('about-profile-link');
      const aboutEditProfileBtn = document.getElementById('about-edit-profile-btn');

      // Palette Controls
      const paletteBtn = document.getElementById('palette-btn');
      const paletteMenu = document.getElementById('palette-menu');
      const paletteLabel = document.getElementById('palette-label');
      const paletteSwatch = document.getElementById('palette-swatch');

      // Quick Settings Controls
      const settingsBtn = document.getElementById('settings-btn');
      const settingsMenu = document.getElementById('settings-menu');
      const glowSlider = document.getElementById('glow-slider');
      const glowValText = document.getElementById('glow-val-text');
      const colorSpeedSlider = document.getElementById('color-speed-slider');
      const colorSpeedText = document.getElementById('color-speed-text');
      const fmt12hBtn = document.getElementById('fmt-12h');
      const fmt24hBtn = document.getElementById('fmt-24h');
      const clockStyleMinimalBtn = document.getElementById('clock-style-minimal');
      const clockStyleGlassBtn = document.getElementById('clock-style-glass');
      const toggleNowPlayingBtn = document.getElementById('toggle-nowplaying-btn');

      // Local File Player Controls
      const localPlayerBar = document.getElementById('local-player-bar');
      const playPauseBtn = document.getElementById('play-pause-btn');
      const playPauseIcon = document.getElementById('play-pause-icon');
      const seekSlider = document.getElementById('seek-slider');
      const volumeSlider = document.getElementById('volume-slider');
      const currentTimeText = document.getElementById('current-time-text');
      const durationText = document.getElementById('duration-text');
      const sensitivitySlider = document.getElementById('sensitivity-slider');

      // Source Buttons
      const btnSynth = document.getElementById('btn-source-synth');
      const synthPillGroup = document.getElementById('synth-pill-group');
      const synthPresetSelect = document.getElementById('synth-preset-select');
      const btnMic = document.getElementById('btn-source-mic');
      const btnTab = document.getElementById('btn-source-tab');
      const btnFile = document.getElementById('btn-source-file');

      // ==========================================
      // 3. WEB AUDIO API & SYNTH ENGINE
      // ==========================================
      let audioCtx = null;
      let analyser = null;
      let frequencyData = null;
      let timeDomainData = null;
      let currentStream = null;
      let mediaSourceNode = null;
      let fileSourceNode = null;
      let synthInterval = null;
      let synthMasterGain = null;

      function ensureAudioContext() {
        if (!audioCtx) {
          const AudioContextClass = window.AudioContext || window.webkitAudioContext;
          audioCtx = new AudioContextClass();
          analyser = audioCtx.createAnalyser();
          analyser.fftSize = 1024;
          analyser.smoothingTimeConstant = 0.82;
          frequencyData = new Uint8Array(analyser.frequencyBinCount);
          timeDomainData = new Uint8Array(analyser.fftSize);
        }
        if (audioCtx.state === 'suspended') {
          audioCtx.resume();
        }
      }

      function stopActiveSources() {
        // Stop MediaStream (Mic or Tab Capture)
        if (currentStream) {
          currentStream.getTracks().forEach(track => track.stop());
          currentStream = null;
        }
        if (mediaSourceNode) {
          try { mediaSourceNode.disconnect(); } catch (e) {}
          mediaSourceNode = null;
        }

        // Stop Built-in Synth
        if (synthInterval) {
          clearInterval(synthInterval);
          synthInterval = null;
        }
        if (synthMasterGain) {
          try {
            synthMasterGain.gain.setTargetAtTime(0, audioCtx.currentTime, 0.05);
            setTimeout(() => {
              try { synthMasterGain.disconnect(); } catch (e) {}
            }, 150);
          } catch (e) {}
          synthMasterGain = null;
        }

        // Pause Local Audio File if switching away
        if (!audioEl.paused) {
          audioEl.pause();
        }
        localPlayerBar.classList.add('hidden');
        localPlayerBar.classList.remove('flex');

        // Reset active button states
        document.querySelectorAll('.source-btn').forEach(btn => btn.classList.remove('active'));
        synthPillGroup.classList.remove('active');
      }

      // --- SOURCE 1: BUILT-IN AMBIENT DEMO SYNTH (4 VIBE PRESETS) ---
      const SYNTH_PRESETS = {
        chillwave: {
          title: 'Midnight Chillwave (Live Synth)',
          subtitle: '110 BPM • Warm Analog Pads & Dream Arps',
          stepMs: 136,
          wave: 'triangle',
          lpfFreq: 950,
          kickFreq: 125,
          chords: [
            [174.61, 207.65, 261.63, 311.13, 392.00],
            [138.59, 174.61, 207.65, 261.63, 311.13],
            [103.83, 155.56, 207.65, 261.63, 311.13],
            [155.56, 196.00, 233.08, 311.13, 349.23]
          ],
          arp: [523.25, 622.25, 783.99, 932.33, 783.99, 622.25, 523.25, 466.16]
        },
        lofi: {
          title: 'Tokyo Lo-Fi Rain (Live Synth)',
          subtitle: '84 BPM • Dusty Rhodes Chords & Vinyl Crackle',
          stepMs: 178,
          wave: 'sine',
          lpfFreq: 680,
          kickFreq: 98,
          chords: [
            [146.83, 174.61, 220.00, 261.63, 329.63], // Dm9
            [196.00, 246.94, 293.66, 349.23, 440.00], // G13
            [130.81, 164.81, 196.00, 246.94, 293.66], // Cmaj9
            [110.00, 164.81, 196.00, 246.94, 329.63]  // Am11
          ],
          arp: [440.00, 523.25, 659.25, 587.33, 493.88, 392.00, 329.63, 293.66]
        },
        deepbass: {
          title: 'Subterranean Pulse (Live Synth)',
          subtitle: '126 BPM • Heavy Sub Bass & Cyberpunk Leads',
          stepMs: 119,
          wave: 'sawtooth',
          lpfFreq: 620,
          kickFreq: 155,
          chords: [
            [82.41, 164.81, 246.94, 329.63], // Em
            [87.31, 174.61, 261.63, 349.23], // F
            [98.00, 196.00, 293.66, 392.00], // G
            [73.42, 146.83, 220.00, 293.66]  // Dm
          ],
          arp: [329.63, 329.63, 493.88, 329.63, 523.25, 329.63, 587.33, 493.88]
        },
        cosmic: {
          title: 'Nebula Drift (Live Synth)',
          subtitle: '92 BPM • Ethereal Space Drones & Shimmer',
          stepMs: 163,
          wave: 'sine',
          lpfFreq: 1200,
          kickFreq: 85,
          chords: [
            [130.81, 196.00, 293.66, 392.00, 493.88],
            [146.83, 220.00, 329.63, 440.00, 523.25],
            [164.81, 246.94, 369.99, 493.88, 587.33],
            [146.83, 220.00, 293.66, 392.00, 440.00]
          ],
          arp: [587.33, 739.99, 880.00, 1174.66, 880.00, 739.99, 659.25, 587.33]
        }
      };

      function startDemoSynth() {
        ensureAudioContext();
        stopActiveSources();

        const preset = SYNTH_PRESETS[state.synthPreset] || SYNTH_PRESETS.chillwave;
        state.mode = 'synth';
        state.isPlaying = true;
        btnSynth.classList.add('active');
        synthPillGroup.classList.add('active');
        updateTrackUI('Synth', preset.title, preset.subtitle, true);

        synthMasterGain = audioCtx.createGain();
        synthMasterGain.gain.value = 0.28;
        synthMasterGain.connect(analyser);
        synthMasterGain.connect(audioCtx.destination);

        let step = 0;

        const playStep = () => {
          if (state.mode !== 'synth' || !audioCtx) return;
          const now = audioCtx.currentTime;

          // Warm Kick on beats 0, 4, 8, 12
          if (step % 4 === 0) {
            const kickOsc = audioCtx.createOscillator();
            const kickGain = audioCtx.createGain();
            kickOsc.type = 'sine';
            kickOsc.frequency.setValueAtTime(preset.kickFreq, now);
            kickOsc.frequency.exponentialRampToValueAtTime(36, now + 0.17);
            kickGain.gain.setValueAtTime(state.synthPreset === 'deepbass' ? 1.0 : 0.85, now);
            kickGain.gain.exponentialRampToValueAtTime(0.001, now + 0.4);
            kickOsc.connect(kickGain);
            kickGain.connect(synthMasterGain);
            kickOsc.start(now);
            kickOsc.stop(now + 0.42);
          }

          // Soft Hi-Hat / Shimmer on off-beats
          if (step % 2 === 1) {
            const bufferSize = audioCtx.sampleRate * 0.05;
            const noiseBuffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
            const data = noiseBuffer.getChannelData(0);
            for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;

            const noise = audioCtx.createBufferSource();
            noise.buffer = noiseBuffer;
            const filter = audioCtx.createBiquadFilter();
            filter.type = 'highpass';
            filter.frequency.value = state.synthPreset === 'lofi' ? 5200 : 7500;

            const hatGain = audioCtx.createGain();
            hatGain.gain.setValueAtTime(0.13, now);
            hatGain.gain.exponentialRampToValueAtTime(0.001, now + 0.05);

            noise.connect(filter);
            filter.connect(hatGain);
            hatGain.connect(synthMasterGain);
            noise.start(now);
          }

          // Lush Pad Chord every 16 steps (1 bar)
          if (step % 16 === 0) {
            const chord = preset.chords[Math.floor(step / 16) % preset.chords.length];
            chord.forEach((freq) => {
              const osc = audioCtx.createOscillator();
              const padGain = audioCtx.createGain();
              const lpf = audioCtx.createBiquadFilter();

              osc.type = preset.wave;
              osc.frequency.setValueAtTime(freq, now);
              lpf.type = 'lowpass';
              lpf.frequency.setValueAtTime(preset.lpfFreq, now);

              padGain.gain.setValueAtTime(0.001, now);
              padGain.gain.linearRampToValueAtTime(0.1, now + 0.45);
              padGain.gain.exponentialRampToValueAtTime(0.001, now + 2.3);

              osc.connect(lpf);
              lpf.connect(padGain);
              padGain.connect(synthMasterGain);
              osc.start(now);
              osc.stop(now + 2.35);
            });
          }

          // Dreamy Bell / Lead Arp
          if (step % 2 === 0) {
            const noteFreq = preset.arp[(step / 2) % preset.arp.length];
            const arpOsc = audioCtx.createOscillator();
            const arpGain = audioCtx.createGain();
            arpOsc.type = state.synthPreset === 'deepbass' ? 'triangle' : 'sine';
            arpOsc.frequency.setValueAtTime(noteFreq, now);
            arpGain.gain.setValueAtTime(0.085, now);
            arpGain.gain.exponentialRampToValueAtTime(0.001, now + 0.36);
            arpOsc.connect(arpGain);
            arpGain.connect(synthMasterGain);
            arpOsc.start(now);
            arpOsc.stop(now + 0.4);
          }

          step++;
        };

        playStep();
        synthInterval = setInterval(playStep, preset.stepMs);
        showToast(`Playing ${preset.title.replace(' (Live Synth)', '')}`);
      }

      // --- SOURCE 2: MICROPHONE / ROOM AUDIO ---
      async function startMicAudio() {
        try {
          ensureAudioContext();
          stopActiveSources();

          const stream = await navigator.mediaDevices.getUserMedia({
            audio: {
              echoCancellation: false,
              noiseSuppression: false,
              autoGainControl: false
            }
          });

          currentStream = stream;
          mediaSourceNode = audioCtx.createMediaStreamSource(stream);
          mediaSourceNode.connect(analyser);

          state.mode = 'mic';
          state.isPlaying = true;
          btnMic.classList.add('active');
          updateTrackUI('Live Mic', 'Listening to Room Audio', 'Reacting to background speakers & ambient sound', true);
          showToast('Microphone active — play music in your room!');
        } catch (err) {
          console.error('Microphone error:', err);
          showToast('Microphone permission denied or unavailable.');
        }
      }

      // --- SOURCE 3: SYSTEM / TAB AUDIO CAPTURE ---
      async function startTabAudio() {
        try {
          if (!navigator.mediaDevices || !navigator.mediaDevices.getDisplayMedia) {
            showToast('System/Tab audio capture is not supported in this browser.');
            return;
          }
          ensureAudioContext();
          stopActiveSources();

          const stream = await navigator.mediaDevices.getDisplayMedia({
            video: true, // Required by browser API to prompt tab/screen picker
            audio: {
              echoCancellation: false,
              noiseSuppression: false,
              autoGainControl: false
            }
          });

          const audioTracks = stream.getAudioTracks();
          if (audioTracks.length === 0) {
            stream.getTracks().forEach(t => t.stop());
            showToast('No audio shared! Make sure to check "Share tab audio" in the popup.');
            return;
          }

          // Stop video track immediately since we only need the audio stream
          stream.getVideoTracks().forEach(t => t.stop());

          currentStream = stream;
          mediaSourceNode = audioCtx.createMediaStreamSource(stream);
          mediaSourceNode.connect(analyser);

          const trackLabel = audioTracks[0].label || 'Browser Tab / System Audio';
          state.mode = 'tab';
          state.isPlaying = true;
          btnTab.classList.add('active');
          updateTrackUI('System Audio', trackLabel, 'Direct digital audio feed from browser/system', true);
          showToast('Connected to Tab/System audio!');

          audioTracks[0].onended = () => {
            if (state.mode === 'tab') {
              resetToIdle();
              showToast('Tab audio sharing ended.');
            }
          };
        } catch (err) {
          console.error('Tab capture error:', err);
          if (err.name !== 'NotAllowedError') {
            showToast('Could not capture tab audio.');
          }
        }
      }

      // --- SOURCE 4: LOCAL AUDIO FILE PLAYER ---
      function loadAudioFile(file) {
        if (!file) return;
        ensureAudioContext();
        stopActiveSources();

        const fileURL = URL.createObjectURL(file);
        audioEl.src = fileURL;
        audioEl.volume = parseFloat(volumeSlider.value);

        if (!fileSourceNode) {
          fileSourceNode = audioCtx.createMediaElementSource(audioEl);
          fileSourceNode.connect(analyser);
          analyser.connect(audioCtx.destination);
        }

        const cleanName = file.name.replace(/\.[^/.]+$/, '');
        state.mode = 'file';
        btnFile.classList.add('active');
        localPlayerBar.classList.remove('hidden');
        localPlayerBar.classList.add('flex');

        updateTrackUI('Local Track', cleanName, 'Playing from local device', true);
        audioEl.play().then(() => {
          state.isPlaying = true;
          updatePlayPauseUI(true);
        }).catch(err => {
          console.error('Playback error:', err);
        });
      }

      function resetToIdle() {
        stopActiveSources();
        state.mode = 'idle';
        state.isPlaying = false;
        updateTrackUI('Standby', 'Choose an Audio Source Below', 'Mic, Tab Audio, Local Music, or Built-in Synth', false);
      }

      // ==========================================
      // 4. UI & PLAYER HELPERS
      // ==========================================
      function updateTrackUI(tag, title, subtitle, active) {
        sourceTag.textContent = tag;
        trackTitleEl.textContent = title;
        trackSubtitleEl.textContent = subtitle;

        if (active) {
          miniEq.classList.remove('eq-paused');
          albumDiscInner.classList.remove('spin-paused');
          statusDot.className = 'w-2.5 h-2.5 rounded-full bg-emerald-400 shadow-[0_0_12px_#34d399]';
        } else {
          miniEq.classList.add('eq-paused');
          albumDiscInner.classList.add('spin-paused');
          statusDot.className = 'w-2.5 h-2.5 rounded-full bg-white/40';
          vibeMoodBadge.textContent = 'Standby';
          bpmTag.classList.add('hidden');
        }
      }

      function updatePlayPauseUI(playing) {
        state.isPlaying = playing;
        playPauseIcon.setAttribute('data-lucide', playing ? 'pause' : 'play');
        lucide.createIcons();
        if (playing) {
          miniEq.classList.remove('eq-paused');
          albumDiscInner.classList.remove('spin-paused');
        } else {
          miniEq.classList.add('eq-paused');
          albumDiscInner.classList.add('spin-paused');
        }
      }

      function formatTime(secs) {
        if (!isFinite(secs) || secs < 0) return '0:00';
        const m = Math.floor(secs / 60);
        const s = Math.floor(secs % 60);
        return `${m}:${s < 10 ? '0' : ''}${s}`;
      }

      function showToast(message) {
        const container = document.getElementById('toast-container');
        const toast = document.createElement('div');
        toast.className = 'glass-panel rounded-full px-4 py-2 text-xs font-medium text-white shadow-xl transition-all duration-300 opacity-0 translate-y-2';
        toast.textContent = message;
        container.appendChild(toast);

        requestAnimationFrame(() => {
          toast.classList.remove('opacity-0', 'translate-y-2');
        });

        setTimeout(() => {
          toast.classList.add('opacity-0', '-translate-y-2');
          setTimeout(() => toast.remove(), 300);
        }, 3200);
      }

      // ==========================================
      // 5. EVENT LISTENERS
      // ==========================================
      btnSynth.addEventListener('click', () => {
        if (state.mode === 'synth') {
          resetToIdle();
        } else {
          startDemoSynth();
        }
      });

      synthPresetSelect.addEventListener('change', (e) => {
        state.synthPreset = e.target.value;
        startDemoSynth();
      });

      btnMic.addEventListener('click', () => {
        if (state.mode === 'mic') {
          resetToIdle();
        } else {
          startMicAudio();
        }
      });

      btnTab.addEventListener('click', () => {
        if (state.mode === 'tab') {
          resetToIdle();
        } else {
          startTabAudio();
        }
      });

      btnFile.addEventListener('click', () => fileInput.click());
      fileInput.addEventListener('change', (e) => {
        if (e.target.files && e.target.files[0]) {
          loadAudioFile(e.target.files[0]);
        }
      });

      // Drag and Drop Audio Support
      let dragCounter = 0;
      window.addEventListener('dragenter', (e) => {
        e.preventDefault();
        dragCounter++;
        dropOverlay.classList.remove('hidden');
        dropOverlay.classList.add('flex');
      });

      window.addEventListener('dragleave', (e) => {
        e.preventDefault();
        dragCounter--;
        if (dragCounter <= 0) {
          dragCounter = 0;
          dropOverlay.classList.add('hidden');
          dropOverlay.classList.remove('flex');
        }
      });

      window.addEventListener('dragover', (e) => e.preventDefault());

      window.addEventListener('drop', (e) => {
        e.preventDefault();
        dragCounter = 0;
        dropOverlay.classList.add('hidden');
        dropOverlay.classList.remove('flex');
        if (e.dataTransfer.files && e.dataTransfer.files[0]) {
          const file = e.dataTransfer.files[0];
          if (file.type.startsWith('audio/') || /\.(mp3|wav|ogg|flac|m4a)$/i.test(file.name)) {
            loadAudioFile(file);
          } else {
            showToast('Please drop a valid audio file (MP3, WAV, OGG, FLAC).');
          }
        }
      });

      // Local Audio Controls
      playPauseBtn.addEventListener('click', () => {
        if (audioEl.paused) {
          ensureAudioContext();
          audioEl.play();
          updatePlayPauseUI(true);
        } else {
          audioEl.pause();
          updatePlayPauseUI(false);
        }
      });

      audioEl.addEventListener('timeupdate', () => {
        if (audioEl.duration) {
          seekSlider.value = (audioEl.currentTime / audioEl.duration) * 100;
          currentTimeText.textContent = formatTime(audioEl.currentTime);
          durationText.textContent = formatTime(audioEl.duration);
        }
      });

      audioEl.addEventListener('ended', () => {
        updatePlayPauseUI(false);
      });

      seekSlider.addEventListener('input', () => {
        if (audioEl.duration) {
          audioEl.currentTime = (seekSlider.value / 100) * audioEl.duration;
        }
      });

      volumeSlider.addEventListener('input', () => {
        audioEl.volume = parseFloat(volumeSlider.value);
      });

      sensitivitySlider.addEventListener('input', () => {
        state.sensitivity = parseFloat(sensitivitySlider.value);
      });

      // Visual Mode Buttons
      function selectVisualMode(visualKey) {
        state.visual = visualKey;
        document.querySelectorAll('.visual-btn').forEach(b => {
          b.classList.toggle('active', b.dataset.visual === visualKey);
        });
      }

      document.querySelectorAll('.visual-btn').forEach(btn => {
        btn.addEventListener('click', () => {
          selectVisualMode(btn.dataset.visual);
        });
      });

      // Palette Dropdown
      paletteBtn.addEventListener('click', (e) => {
        e.stopPropagation();
        settingsMenu.classList.add('hidden');
        paletteMenu.classList.toggle('hidden');
      });

      paletteMenu.addEventListener('click', (e) => e.stopPropagation());

      // Quick Settings Dropdown
      settingsBtn.addEventListener('click', (e) => {
        e.stopPropagation();
        paletteMenu.classList.add('hidden');
        settingsMenu.classList.toggle('hidden');
      });

      settingsMenu.addEventListener('click', (e) => e.stopPropagation());

      document.addEventListener('click', () => {
        paletteMenu.classList.add('hidden');
        settingsMenu.classList.add('hidden');
      });

      function selectPalette(paletteKey) {
        state.palette = paletteKey;
        document.querySelectorAll('.palette-option').forEach(o => {
          const isMatch = o.dataset.palette === paletteKey;
          o.classList.toggle('bg-white/10', isMatch);
          const chk = o.querySelector('[data-lucide="check"]');
          if (chk) chk.remove();
          if (isMatch) {
            o.insertAdjacentHTML('beforeend', '<i data-lucide="check" class="w-3.5 h-3.5 text-white flex-shrink-0"></i>');
            if (o.dataset.grad) {
              paletteSwatch.className = `w-3 h-3 rounded-full bg-gradient-to-tr ${o.dataset.grad}`;
            }
          }
        });
        lucide.createIcons();
        paletteLabel.textContent = PALETTE_NAMES[paletteKey] || 'Auto-Vibe Spectral';
      }

      document.querySelectorAll('.palette-option').forEach(opt => {
        opt.addEventListener('click', () => {
          selectPalette(opt.dataset.palette);
        });
      });

      // Quick Settings Sliders & Toggles
      glowSlider.addEventListener('input', () => {
        state.glowIntensity = parseFloat(glowSlider.value);
        glowValText.textContent = `${Math.round(state.glowIntensity * 100)}%`;
      });

      colorSpeedSlider.addEventListener('input', () => {
        state.colorSpeed = parseFloat(colorSpeedSlider.value);
        colorSpeedText.textContent = `${state.colorSpeed.toFixed(1)}x`;
      });

      fmt12hBtn.addEventListener('click', () => {
        state.is24Hour = false;
        fmt12hBtn.className = 'px-2 py-0.5 rounded-md bg-white/20 font-medium';
        fmt24hBtn.className = 'px-2 py-0.5 rounded-md text-white/60 hover:text-white';
        updateClock();
      });

      fmt24hBtn.addEventListener('click', () => {
        state.is24Hour = true;
        fmt24hBtn.className = 'px-2 py-0.5 rounded-md bg-white/20 font-medium';
        fmt12hBtn.className = 'px-2 py-0.5 rounded-md text-white/60 hover:text-white';
        updateClock();
      });

      clockStyleMinimalBtn.addEventListener('click', () => {
        state.clockStyle = 'minimal';
        clockStyleMinimalBtn.className = 'px-2 py-0.5 rounded-md bg-white/20 font-medium';
        clockStyleGlassBtn.className = 'px-2 py-0.5 rounded-md text-white/60 hover:text-white';
        clockWidget.classList.remove('glass-panel', 'px-8', 'py-5');
      });

      clockStyleGlassBtn.addEventListener('click', () => {
        state.clockStyle = 'glass';
        clockStyleGlassBtn.className = 'px-2 py-0.5 rounded-md bg-white/20 font-medium';
        clockStyleMinimalBtn.className = 'px-2 py-0.5 rounded-md text-white/60 hover:text-white';
        clockWidget.classList.add('glass-panel', 'px-8', 'py-5');
      });

      toggleNowPlayingBtn.addEventListener('click', () => {
        state.showNowPlaying = !state.showNowPlaying;
        nowPlayingCard.style.display = state.showNowPlaying ? 'flex' : 'none';
        toggleNowPlayingBtn.textContent = state.showNowPlaying ? 'Visible' : 'Hidden';
        toggleNowPlayingBtn.className = state.showNowPlaying
          ? 'px-2.5 py-0.5 rounded-lg bg-emerald-500/25 border border-emerald-400/40 text-emerald-200 text-[11px] font-medium'
          : 'px-2.5 py-0.5 rounded-lg bg-white/10 border border-white/15 text-white/60 text-[11px] font-medium';
      });

      // Clock Toggle
      toggleClockBtn.addEventListener('click', () => {
        state.showClock = !state.showClock;
        toggleClockBtn.classList.toggle('active', state.showClock);
        clockWidget.style.display = state.showClock ? 'block' : 'none';
      });

      // Fullscreen Toggle
      function toggleFullscreen() {
        if (!document.fullscreenElement) {
          document.documentElement.requestFullscreen().catch(() => {
            showToast('Fullscreen is blocked by the browser.');
          });
        } else {
          document.exitFullscreen().catch(() => {});
        }
      }

      fullscreenBtn.addEventListener('click', toggleFullscreen);

      document.addEventListener('fullscreenchange', () => {
        const isFull = !!document.fullscreenElement;
        fullscreenIcon.setAttribute('data-lucide', isFull ? 'minimize' : 'maximize');
        lucide.createIcons();
      });

      // About / Public Info Modal Controls
      function openAboutModal() {
        aboutModal.classList.remove('hidden');
        aboutModal.classList.add('flex');
      }

      function closeAboutModal() {
        aboutModal.classList.add('hidden');
        aboutModal.classList.remove('flex');
      }

      aboutBtn.addEventListener('click', openAboutModal);
      closeAboutBtn.addEventListener('click', closeAboutModal);
      startVibingBtn.addEventListener('click', closeAboutModal);
      aboutModal.addEventListener('click', (e) => {
        if (e.target === aboutModal) closeAboutModal();
      });

      shareLinkBtn.addEventListener('click', () => {
        const tempInput = document.createElement('input');
        tempInput.value = window.location.href;
        document.body.appendChild(tempInput);
        tempInput.select();
        document.execCommand('copy');
        document.body.removeChild(tempInput);
        showToast('Link copied to clipboard!');
      });

      // ==========================================
      // CREATOR PROFILE MANAGEMENT (LOCALSTORAGE)
      // ==========================================
      const DEFAULT_PROFILE = {
        name: 'Your Name',
        bio: 'Music Lover & Creator',
        link: ''
      };

      function loadCreatorProfile() {
        try {
          const saved = JSON.parse(localStorage.getItem('vibescreen_creator_profile'));
          return saved && saved.name ? { ...DEFAULT_PROFILE, ...saved } : DEFAULT_PROFILE;
        } catch (e) {
          return DEFAULT_PROFILE;
        }
      }

      function applyCreatorProfile(profile) {
        const displayName = (profile.name || 'Your Name').trim();
        const displayBio = (profile.bio || 'Music Lover & Creator').trim();
        const initial = displayName.charAt(0).toUpperCase() || 'Y';

        profileDisplayName.querySelector('span').textContent = displayName;
        profileAvatarCircle.textContent = initial;
        modalAvatarPreview.textContent = initial;
        aboutAvatarCircle.textContent = initial;
        aboutProfileName.textContent = displayName;
        aboutProfileBio.textContent = displayBio;

        if (profile.link && profile.link.trim()) {
          aboutProfileLink.href = profile.link.trim();
          aboutProfileLink.classList.remove('hidden');
          aboutProfileLink.classList.add('inline-flex');
        } else {
          aboutProfileLink.classList.add('hidden');
          aboutProfileLink.classList.remove('inline-flex');
        }
      }

      function openProfileModal() {
        const current = loadCreatorProfile();
        profileNameInput.value = current.name === 'Your Name' ? '' : current.name;
        profileBioInput.value = current.bio;
        profileLinkInput.value = current.link || '';
        modalAvatarPreview.textContent = (current.name.charAt(0) || 'Y').toUpperCase();
        profileModal.classList.remove('hidden');
        profileModal.classList.add('flex');
        setTimeout(() => profileNameInput.focus(), 50);
      }

      function closeProfileModal() {
        profileModal.classList.add('hidden');
        profileModal.classList.remove('flex');
      }

      profileNameInput.addEventListener('input', () => {
        const val = profileNameInput.value.trim();
        modalAvatarPreview.textContent = (val.charAt(0) || 'Y').toUpperCase();
      });

      openProfileModalBtn.addEventListener('click', openProfileModal);
      closeProfileBtn.addEventListener('click', closeProfileModal);
      aboutEditProfileBtn.addEventListener('click', () => {
        closeAboutModal();
        openProfileModal();
      });

      profileModal.addEventListener('click', (e) => {
        if (e.target === profileModal) closeProfileModal();
      });

      saveProfileBtn.addEventListener('click', () => {
        const updated = {
          name: profileNameInput.value.trim() || 'Your Name',
          bio: profileBioInput.value.trim() || 'Music Lover & Creator',
          link: profileLinkInput.value.trim()
        };
        try {
          localStorage.setItem('vibescreen_creator_profile', JSON.stringify(updated));
        } catch (e) {}
        applyCreatorProfile(updated);
        closeProfileModal();
        showToast(`Profile updated for ${updated.name}!`);
      });

      // Initialize saved profile on load
      applyCreatorProfile(loadCreatorProfile());

      // Keyboard Shortcuts (Space, F, V, C, Escape, Left/Right Arrows)
      const VISUAL_ORDER = ['aura', 'orb', 'starfield', 'horizon', 'aurora', 'cybergrid', 'dna', 'kaleidoscope'];
      const PALETTE_ORDER = Object.keys(PALETTE_NAMES);

      window.addEventListener('keydown', (e) => {
        if (e.key === 'Escape') {
          if (!profileModal.classList.contains('hidden')) {
            closeProfileModal();
            return;
          }
          if (!aboutModal.classList.contains('hidden')) {
            closeAboutModal();
            return;
          }
        }
        if (e.target.tagName === 'SELECT' || e.target.tagName === 'INPUT') return;
        if (e.code === 'Space' && state.mode === 'file') {
          e.preventDefault();
          playPauseBtn.click();
        } else if (e.key === 'f' || e.key === 'F') {
          toggleFullscreen();
        } else if (e.key === 'v' || e.key === 'V' || e.key === 'ArrowRight') {
          const nextIdx = (VISUAL_ORDER.indexOf(state.visual) + 1) % VISUAL_ORDER.length;
          selectVisualMode(VISUAL_ORDER[nextIdx]);
          showToast(`Interface: ${document.querySelector(`[data-visual="${VISUAL_ORDER[nextIdx]}"] span`).textContent}`);
        } else if (e.key === 'ArrowLeft') {
          const prevIdx = (VISUAL_ORDER.indexOf(state.visual) - 1 + VISUAL_ORDER.length) % VISUAL_ORDER.length;
          selectVisualMode(VISUAL_ORDER[prevIdx]);
          showToast(`Interface: ${document.querySelector(`[data-visual="${VISUAL_ORDER[prevIdx]}"] span`).textContent}`);
        } else if (e.key === 'c' || e.key === 'C') {
          const nextPalIdx = (PALETTE_ORDER.indexOf(state.palette) + 1) % PALETTE_ORDER.length;
          const nextPal = PALETTE_ORDER[nextPalIdx];
          selectPalette(nextPal);
          showToast(`Theme: ${PALETTE_NAMES[nextPal]}`);
        }
      });

      // ==========================================
      // 6. AUTO-HIDE HUD ON INACTIVITY (WALLPAPER MODE)
      // ==========================================
      let inactivityTimer = null;
      function wakeHUD() {
        document.body.classList.remove('hud-hidden');
        clearTimeout(inactivityTimer);
        inactivityTimer = setTimeout(() => {
          if (
            paletteMenu.classList.contains('hidden') &&
            settingsMenu.classList.contains('hidden') &&
            aboutModal.classList.contains('hidden') &&
            profileModal.classList.contains('hidden')
          ) {
            document.body.classList.add('hud-hidden');
          }
        }, 3500);
      }

      window.addEventListener('mousemove', wakeHUD);
      window.addEventListener('mousedown', wakeHUD);
      window.addEventListener('touchstart', wakeHUD);
      wakeHUD();

      // ==========================================
      // 7. LIVE CLOCK UPDATER
      // ==========================================
      function updateClock() {
        const now = new Date();
        const hours = now.getHours();
        const minutes = now.getMinutes();
        const displayHours = state.is24Hour ? (hours < 10 ? '0' + hours : hours) : (hours % 12 || 12);
        const formattedMinutes = minutes < 10 ? '0' + minutes : minutes;
        liveTimeEl.textContent = `${displayHours}:${formattedMinutes}`;

        const options = { weekday: 'long', month: 'long', day: 'numeric' };
        liveDateEl.textContent = now.toLocaleDateString(undefined, options).toUpperCase();
      }
      updateClock();
      setInterval(updateClock, 1000);

      // ==========================================
      // 8. FREQUENCY ANALYSIS & COLOR MOOD ENGINE
      // ==========================================
      let lastBeatTime = 0;
      let beatIntervals = [];
      let prevBass = 0;

      function analyzeAudio(time) {
        if (analyser && state.mode !== 'idle' && state.isPlaying) {
          analyser.getByteFrequencyData(frequencyData);
          analyser.getByteTimeDomainData(timeDomainData);

          const len = frequencyData.length;
          // Bass: ~20Hz - 250Hz
          let bassSum = 0;
          const bassEnd = Math.floor(len * 0.08);
          for (let i = 1; i < bassEnd; i++) bassSum += frequencyData[i];
          const rawBass = (bassSum / (bassEnd * 255)) * state.sensitivity;

          // Mids: ~250Hz - 2500Hz
          let midSum = 0;
          const midEnd = Math.floor(len * 0.38);
          for (let i = bassEnd; i < midEnd; i++) midSum += frequencyData[i];
          const rawMid = (midSum / ((midEnd - bassEnd) * 255)) * state.sensitivity;

          // Treble: ~2500Hz - 16kHz
          let trebleSum = 0;
          const trebleEnd = Math.floor(len * 0.85);
          for (let i = midEnd; i < trebleEnd; i++) trebleSum += frequencyData[i];
          const rawTreble = (trebleSum / ((trebleEnd - midEnd) * 255)) * state.sensitivity;

          // Smooth attack & decay
          state.bass += (Math.min(1, rawBass) - state.bass) * 0.28;
          state.mid += (Math.min(1, rawMid) - state.mid) * 0.22;
          state.treble += (Math.min(1, rawTreble) - state.treble) * 0.25;
          state.energy = state.bass * 0.5 + state.mid * 0.35 + state.treble * 0.15;

          // Simple Beat / Kick Detection
          if (state.bass > 0.48 && state.bass - prevBass > 0.08 && time - lastBeatTime > 260) {
            state.beatPulse = 1.0;
            if (lastBeatTime > 0) {
              const interval = time - lastBeatTime;
              if (interval > 280 && interval < 1500) {
                beatIntervals.push(interval);
                if (beatIntervals.length > 8) beatIntervals.shift();
                const avgInterval = beatIntervals.reduce((a, b) => a + b, 0) / beatIntervals.length;
                const estBpm = Math.round(60000 / avgInterval);
                if (estBpm >= 60 && estBpm <= 190) {
                  state.bpm = estBpm;
                  bpmTag.textContent = `${estBpm} BPM`;
                  bpmTag.classList.remove('hidden');
                }
              }
            }
            lastBeatTime = time;
          }
          prevBass = state.bass;
        } else {
          // Gentle breathing idle motion when no audio is playing
          const breath = (Math.sin(time * 0.0012) + 1) * 0.5;
          state.bass += (0.08 + breath * 0.06 - state.bass) * 0.05;
          state.mid += (0.06 + (1 - breath) * 0.05 - state.mid) * 0.05;
          state.treble += (0.04 - state.treble) * 0.05;
          state.energy = 0.08;
        }

        state.beatPulse *= 0.91;

        // Dynamic Color Mood Shifting
        let targetH1, targetH2, targetH3, targetSat;
        const speedFactor = state.colorSpeed;

        if (state.palette === 'auto') {
          if (state.mode === 'idle' || !state.isPlaying) {
            targetH1 = (time * 0.008 * speedFactor) % 360;
            targetH2 = (targetH1 + 55) % 360;
            targetH3 = (targetH1 + 120) % 360;
            targetSat = 65;
          } else {
            const total = state.bass + state.mid + state.treble + 0.001;
            const bassWeight = state.bass / total;
            const midWeight = state.mid / total;
            const trebleWeight = state.treble / total;

            const slowDrift = Math.sin(time * 0.0005 * speedFactor) * 32;
            targetH1 = (bassWeight * 275 + midWeight * 24 + trebleWeight * 185 + slowDrift + 360) % 360;
            targetH2 = (targetH1 + 52 + state.treble * 45) % 360;
            targetH3 = (targetH1 - 58 - state.bass * 35 + 360) % 360;
            targetSat = Math.min(96, 68 + state.energy * 32);

            if (bassWeight > 0.46) vibeMoodBadge.textContent = 'Deep Bass Vibe';
            else if (trebleWeight > 0.36) vibeMoodBadge.textContent = 'Electric Highs';
            else if (midWeight > 0.42) vibeMoodBadge.textContent = 'Warm Acoustic';
            else vibeMoodBadge.textContent = 'Harmonic Flow';
          }
        } else if (state.palette === 'moodshift') {
          // Transitions smoothly between Cool Calm (Cyan/Indigo) and Warm High-Energy (Sunset/Magenta)
          const intensity = Math.min(1, state.energy * 1.45);
          const sway = Math.sin(time * 0.0007 * speedFactor) * 18;
          targetH1 = (210 + intensity * 145 + sway) % 360;
          targetH2 = (175 + intensity * 195 + sway) % 360;
          targetH3 = (260 + intensity * 115 + sway) % 360;
          targetSat = 74 + intensity * 22;
          if (state.isPlaying) {
            vibeMoodBadge.textContent = intensity > 0.55 ? 'High Energy Warm' : 'Chill Cool Flow';
          }
        } else {
          const p = PALETTES[state.palette];
          const sway = Math.sin(time * 0.0008 * speedFactor) * 16;
          targetH1 = (p.h1 + sway + 360) % 360;
          targetH2 = (p.h2 + sway + 360) % 360;
          targetH3 = (p.h3 + sway + 360) % 360;
          targetSat = p.sat;
          if (state.isPlaying) vibeMoodBadge.textContent = PALETTE_NAMES[state.palette];
        }

        // Smooth hue interpolation
        const interpRate = Math.min(0.18, 0.04 * speedFactor);
        state.hue1 = lerpHue(state.hue1, targetH1, interpRate);
        state.hue2 = lerpHue(state.hue2, targetH2, interpRate);
        state.hue3 = lerpHue(state.hue3, targetH3, interpRate);
        state.saturation += (targetSat - state.saturation) * 0.05;

        // Sync Now-Playing Badge Gradient & Scale with Music
        const scale = 1 + state.beatPulse * 0.08 + state.bass * 0.05;
        albumArtBadge.style.transform = `scale(${scale.toFixed(3)})`;
        albumArtBadge.style.background = `linear-gradient(135deg, hsl(${state.hue1}, ${state.saturation}%, 52%), hsl(${state.hue2}, ${state.saturation}%, 48%))`;
        albumArtBadge.style.boxShadow = `0 8px 28px hsla(${state.hue1}, ${state.saturation}%, 50%, ${(0.35 + state.bass * 0.4) * state.glowIntensity})`;
      }

      function lerpHue(a, b, t) {
        let diff = ((b - a + 540) % 360) - 180;
        return (a + diff * t + 360) % 360;
      }

      // ==========================================
      // 9. CANVAS VISUALIZER MODES
      // ==========================================
      let width = 0;
      let height = 0;

      function resizeCanvas() {
        const dpr = Math.min(window.devicePixelRatio || 1, 2);
        width = window.innerWidth;
        height = window.innerHeight;
        canvas.width = width * dpr;
        canvas.height = height * dpr;
        ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
      }
      window.addEventListener('resize', resizeCanvas);
      resizeCanvas();

      // Pre-create Starfield Particles
      const STAR_COUNT = 260;
      const stars = Array.from({ length: STAR_COUNT }, () => ({
        x: (Math.random() - 0.5) * 2000,
        y: (Math.random() - 0.5) * 2000,
        z: Math.random() * 1500 + 100,
        size: Math.random() * 1.8 + 0.6
      }));

      // Floating Ambient Orbs for Aura Mode
      const auraBlobs = [
        { angle: 0.0, speed: 0.00045, radiusFactor: 0.42, hueOffset: 0 },
        { angle: 2.1, speed: -0.00038, radiusFactor: 0.38, hueOffset: 1 },
        { angle: 4.2, speed: 0.00052, radiusFactor: 0.45, hueOffset: 2 },
        { angle: 1.1, speed: -0.00031, radiusFactor: 0.35, hueOffset: 0 }
      ];

      function drawAmbientBackground(time) {
        // Deep reactive base fill
        const bgLight = 6 + state.bass * 7;
        ctx.fillStyle = `hsl(${state.hue1}, ${state.saturation * 0.65}%, ${bgLight}%)`;
        ctx.fillRect(0, 0, width, height);

        const cx = width * 0.5;
        const cy = height * 0.5;
        const maxDim = Math.max(width, height);
        const hues = [state.hue1, state.hue2, state.hue3];

        ctx.save();
        ctx.globalCompositeOperation = 'screen';

        auraBlobs.forEach((blob, idx) => {
          const t = time * blob.speed + blob.angle;
          const orbitX = Math.cos(t) * width * 0.26;
          const orbitY = Math.sin(t * 1.3) * height * 0.24;
          const bx = cx + orbitX;
          const by = cy + orbitY;

          const pulseScale = 1 + (idx % 2 === 0 ? state.bass * 0.42 : state.mid * 0.35) + state.beatPulse * 0.14;
          const r = maxDim * blob.radiusFactor * pulseScale;
          const h = hues[blob.hueOffset];
          const alpha = Math.min(0.85, (0.32 + state.energy * 0.25) * state.glowIntensity);

          const grad = ctx.createRadialGradient(bx, by, 0, bx, by, r);
          grad.addColorStop(0, `hsla(${h}, ${state.saturation}%, 54%, ${alpha})`);
          grad.addColorStop(0.55, `hsla(${(h + 25) % 360}, ${state.saturation}%, 42%, ${alpha * 0.45})`);
          grad.addColorStop(1, `hsla(${h}, ${state.saturation}%, 20%, 0)`);

          ctx.fillStyle = grad;
          ctx.beginPath();
          ctx.arc(bx, by, r, 0, Math.PI * 2);
          ctx.fill();
        });

        ctx.restore();
      }

      // VISUAL 1: AMBIENT AURA (Apple Music / Fluid Wallpaper)
      function renderAuraMode(time) {
        drawAmbientBackground(time);

        // Subtle flowing silk ribbons across screen
        const cx = width * 0.5;
        const cy = height * 0.5;
        ctx.save();
        ctx.globalCompositeOperation = 'lighter';

        for (let ring = 0; ring < 3; ring++) {
          ctx.beginPath();
          const baseRadius = Math.min(width, height) * (0.24 + ring * 0.09) + state.bass * 45;
          const segments = 90;
          for (let i = 0; i <= segments; i++) {
            const theta = (i / segments) * Math.PI * 2;
            const freqIdx = Math.floor((i / segments) * 120) + ring * 20;
            const amp = frequencyData && state.isPlaying ? (frequencyData[freqIdx] / 255) * 48 * state.sensitivity : Math.sin(theta * 3 + time * 0.002) * 8;
            const wave = Math.sin(theta * (3 + ring) + time * 0.0015 * (ring % 2 === 0 ? 1 : -1)) * (14 + state.mid * 35);
            const r = baseRadius + amp + wave;
            const x = cx + Math.cos(theta) * r;
            const y = cy + Math.sin(theta) * r;
            if (i === 0) ctx.moveTo(x, y);
            else ctx.lineTo(x, y);
          }
          ctx.closePath();
          const h = ring === 0 ? state.hue1 : ring === 1 ? state.hue2 : state.hue3;
          ctx.strokeStyle = `hsla(${h}, ${state.saturation}%, 72%, ${0.08 + state.energy * 0.14})`;
          ctx.lineWidth = 1.5 + state.bass * 2;
          ctx.stroke();
        }
        ctx.restore();
      }

      // VISUAL 2: PULSE ORB & RADIAL FREQUENCY RING
      function renderPulseOrbMode(time) {
        drawAmbientBackground(time);

        const cx = width * 0.5;
        const cy = height * 0.5;
        const baseR = Math.min(width, height) * 0.19 + state.bass * 42 + state.beatPulse * 18;

        ctx.save();
        ctx.globalCompositeOperation = 'lighter';

        // Outer Glowing Halo
        const haloGrad = ctx.createRadialGradient(cx, cy, baseR * 0.4, cx, cy, baseR * 1.9);
        haloGrad.addColorStop(0, `hsla(${state.hue1}, ${state.saturation}%, 65%, ${0.28 + state.bass * 0.3})`);
        haloGrad.addColorStop(0.6, `hsla(${state.hue2}, ${state.saturation}%, 50%, 0.1)`);
        haloGrad.addColorStop(1, 'transparent');
        ctx.fillStyle = haloGrad;
        ctx.beginPath();
        ctx.arc(cx, cy, baseR * 1.9, 0, Math.PI * 2);
        ctx.fill();

        // Radial Frequency Bars
        const bars = 96;
        for (let i = 0; i < bars; i++) {
          const angle = (i / bars) * Math.PI * 2 - Math.PI / 2;
          // Mirror spectrum symmetrically for balanced aesthetic
          const specIndex = i < bars / 2 ? i * 2 : (bars - i) * 2;
          const val = frequencyData && state.isPlaying
            ? (frequencyData[specIndex] / 255) * state.sensitivity
            : (Math.sin(i * 0.25 + time * 0.003) + 1) * 0.08;

          const barLen = 8 + Math.pow(val, 1.3) * (Math.min(width, height) * 0.22);
          const x1 = cx + Math.cos(angle) * (baseR + 6);
          const y1 = cy + Math.sin(angle) * (baseR + 6);
          const x2 = cx + Math.cos(angle) * (baseR + 6 + barLen);
          const y2 = cy + Math.sin(angle) * (baseR + 6 + barLen);

          const barHue = (state.hue1 + (i / bars) * 90) % 360;
          ctx.strokeStyle = `hsla(${barHue}, ${state.saturation}%, 72%, ${0.45 + val * 0.55})`;
          ctx.lineWidth = Math.max(2, (Math.PI * 2 * baseR) / bars * 0.48);
          ctx.lineCap = 'round';
          ctx.beginPath();
          ctx.moveTo(x1, y1);
          ctx.lineTo(x2, y2);
          ctx.stroke();
        }

        // Inner Energy Ring
        ctx.beginPath();
        ctx.arc(cx, cy, baseR, 0, Math.PI * 2);
        ctx.strokeStyle = `hsla(${state.hue2}, 90%, 82%, ${0.4 + state.energy * 0.5})`;
        ctx.lineWidth = 2.5;
        ctx.stroke();

        ctx.restore();
      }

      // VISUAL 3: STARFIELD WARP
      function renderStarfieldMode(time) {
        // Slightly darker backdrop so stars shine brilliantly
        ctx.fillStyle = `hsla(${state.hue1}, ${state.saturation * 0.7}%, 5%, 0.42)`;
        ctx.fillRect(0, 0, width, height);

        const cx = width * 0.5;
        const cy = height * 0.5;

        // Ambient center nebula glow
        const nebGrad = ctx.createRadialGradient(cx, cy, 0, cx, cy, Math.max(width, height) * 0.55);
        nebGrad.addColorStop(0, `hsla(${state.hue2}, ${state.saturation}%, 45%, ${0.16 + state.bass * 0.22})`);
        nebGrad.addColorStop(1, 'transparent');
        ctx.fillStyle = nebGrad;
        ctx.fillRect(0, 0, width, height);

        const speed = 2.5 + state.bass * 26 + state.beatPulse * 18;

        ctx.save();
        ctx.globalCompositeOperation = 'lighter';

        for (let i = 0; i < STAR_COUNT; i++) {
          const s = stars[i];
          const prevZ = s.z;
          s.z -= speed;

          if (s.z <= 10) {
            s.x = (Math.random() - 0.5) * 2000;
            s.y = (Math.random() - 0.5) * 2000;
            s.z = 1500;
            continue;
          }

          const k = 420 / s.z;
          const px = cx + s.x * k;
          const py = cy + s.y * k;

          const prevK = 420 / prevZ;
          const prevPx = cx + s.x * prevK;
          const prevPy = cy + s.y * prevK;

          if (px < -50 || px > width + 50 || py < -50 || py > height + 50) continue;

          const depthRatio = 1 - s.z / 1500;
          const starHue = i % 3 === 0 ? state.hue1 : i % 3 === 1 ? state.hue2 : state.hue3;
          const alpha = Math.min(1, depthRatio * 1.3);

          ctx.strokeStyle = `hsla(${starHue}, ${state.saturation}%, 78%, ${alpha})`;
          ctx.lineWidth = s.size * (0.6 + depthRatio * 2.2);
          ctx.lineCap = 'round';
          ctx.beginPath();
          ctx.moveTo(prevPx, prevPy);
          ctx.lineTo(px, py);
          ctx.stroke();
        }
        ctx.restore();
      }

      // VISUAL 4: WAVEFORM HORIZON
      function renderWaveformHorizonMode(time) {
        drawAmbientBackground(time);

        const layers = 4;
        const points = 120;

        ctx.save();
        ctx.globalCompositeOperation = 'lighter';

        for (let l = 0; l < layers; l++) {
          const baseY = height * (0.52 + l * 0.07);
          const layerHue = l === 0 ? state.hue1 : l === 1 ? state.hue2 : state.hue3;

          ctx.beginPath();
          ctx.moveTo(0, height);

          for (let i = 0; i <= points; i++) {
            const pct = i / points;
            const x = pct * width;

            // Envelope tapers edges smoothly
            const envelope = Math.sin(pct * Math.PI);
            let audioOffset = 0;

            if (timeDomainData && state.isPlaying) {
              const tdIdx = Math.floor(pct * (timeDomainData.length - 1));
              const normalizedWave = (timeDomainData[tdIdx] - 128) / 128;
              const freqIdx = Math.floor(pct * 160);
              const freqBoost = (frequencyData[freqIdx] / 255) * 90;
              audioOffset = (normalizedWave * 110 * state.sensitivity + Math.sin(pct * 10 + time * 0.004 + l) * freqBoost) * envelope;
            } else {
              audioOffset = Math.sin(pct * 8 + time * 0.002 + l * 1.4) * 24 * envelope;
            }

            const y = baseY + audioOffset;
            ctx.lineTo(x, y);
          }

          ctx.lineTo(width, height);
          ctx.closePath();

          const grad = ctx.createLinearGradient(0, baseY - 120, 0, height);
          grad.addColorStop(0, `hsla(${layerHue}, ${state.saturation}%, 65%, ${0.24 - l * 0.03})`);
          grad.addColorStop(1, `hsla(${layerHue}, ${state.saturation}%, 30%, 0.02)`);

          ctx.fillStyle = grad;
          ctx.fill();

          ctx.strokeStyle = `hsla(${layerHue}, 90%, 78%, ${0.45 - l * 0.08})`;
          ctx.lineWidth = 2;
          ctx.stroke();
        }

        ctx.restore();
      }

      // VISUAL 5: AURORA BOREALIS (Undulating Northern Lights Curtains + Frequency Rays)
      function renderAuroraMode(time) {
        // Deep starry night sky gradient
        const skyGrad = ctx.createLinearGradient(0, 0, 0, height);
        skyGrad.addColorStop(0, `hsl(${state.hue1}, ${state.saturation * 0.5}%, 4%)`);
        skyGrad.addColorStop(0.65, `hsl(${state.hue2}, ${state.saturation * 0.6}%, 8%)`);
        skyGrad.addColorStop(1, `hsl(${state.hue3}, ${state.saturation * 0.7}%, 12%)`);
        ctx.fillStyle = skyGrad;
        ctx.fillRect(0, 0, width, height);

        // Twinkling background stars
        ctx.save();
        for (let i = 0; i < 90; i++) {
          const sx = ((i * 197.3) % width);
          const sy = ((i * 89.7) % (height * 0.72));
          const twinkle = 0.25 + 0.75 * Math.abs(Math.sin(time * 0.0015 + i));
          ctx.fillStyle = `rgba(255, 255, 255, ${twinkle * 0.6})`;
          ctx.beginPath();
          ctx.arc(sx, sy, (i % 2 === 0 ? 1.3 : 0.8), 0, Math.PI * 2);
          ctx.fill();
        }
        ctx.restore();

        // Flowing Aurora Curtains
        ctx.save();
        ctx.globalCompositeOperation = 'screen';
        const curtains = 3;
        const rayCount = 85;

        for (let c = 0; c < curtains; c++) {
          const baseY = height * (0.34 + c * 0.11);
          const curtainHue = c === 0 ? state.hue1 : c === 1 ? state.hue2 : state.hue3;

          for (let i = 0; i < rayCount; i++) {
            const pct = i / rayCount;
            const x = pct * width;
            const freqIdx = Math.floor(pct * 150) + c * 15;
            const freqVal = frequencyData && state.isPlaying
              ? (frequencyData[freqIdx] / 255) * state.sensitivity
              : (Math.sin(pct * 6 + time * 0.002 + c) + 1) * 0.16;

            const waveY = Math.sin(pct * 5.5 + time * 0.0009 * (c % 2 === 0 ? 1 : -1) + c * 2) * (45 + state.mid * 55)
                        + Math.cos(pct * 11 - time * 0.0014) * 20;
            const yTop = baseY + waveY - (70 + freqVal * 190 * state.glowIntensity);
            const yBot = baseY + waveY + 45;

            const rayGrad = ctx.createLinearGradient(x, yTop, x, yBot);
            const alpha = Math.min(0.65, (0.14 + freqVal * 0.36) * state.glowIntensity);
            rayGrad.addColorStop(0, `hsla(${(curtainHue + pct * 40) % 360}, ${state.saturation}%, 65%, 0)`);
            rayGrad.addColorStop(0.55, `hsla(${(curtainHue + pct * 40) % 360}, ${state.saturation}%, 58%, ${alpha})`);
            rayGrad.addColorStop(1, `hsla(${curtainHue}, ${state.saturation}%, 45%, 0)`);

            ctx.strokeStyle = rayGrad;
            ctx.lineWidth = (width / rayCount) * 1.45;
            ctx.beginPath();
            ctx.moveTo(x, yTop);
            ctx.lineTo(x, yBot);
            ctx.stroke();
          }
        }
        ctx.restore();
      }

      // VISUAL 6: CYBER GRID (Retro-Futuristic 3D Synthwave Horizon & Sun)
      function renderCyberGridMode(time) {
        // Sky Gradient
        const skyGrad = ctx.createLinearGradient(0, 0, 0, height * 0.56);
        skyGrad.addColorStop(0, `hsl(${state.hue3}, ${state.saturation * 0.65}%, 6%)`);
        skyGrad.addColorStop(0.65, `hsl(${state.hue1}, ${state.saturation * 0.8}%, 16%)`);
        skyGrad.addColorStop(1, `hsl(${state.hue2}, ${state.saturation}%, 28%)`);
        ctx.fillStyle = skyGrad;
        ctx.fillRect(0, 0, width, height);

        const cx = width * 0.5;
        const horizonY = height * 0.54;
        const sunRadius = Math.min(width, height) * (0.17 + state.bass * 0.045 + state.beatPulse * 0.02);

        // Synthwave Retro Sun with Horizontal Scanline Cutouts
        ctx.save();
        const sunGrad = ctx.createLinearGradient(cx, horizonY - sunRadius * 1.6, cx, horizonY + sunRadius * 0.3);
        sunGrad.addColorStop(0, `hsl(${state.hue2}, 95%, 68%)`);
        sunGrad.addColorStop(0.6, `hsl(${state.hue1}, 92%, 55%)`);
        sunGrad.addColorStop(1, `hsl(${state.hue3}, 90%, 42%)`);

        ctx.shadowColor = `hsla(${state.hue2}, 95%, 60%, ${0.65 * state.glowIntensity})`;
        ctx.shadowBlur = 45 * state.glowIntensity;
        ctx.fillStyle = sunGrad;
        ctx.beginPath();
        ctx.arc(cx, horizonY - sunRadius * 0.35, sunRadius, 0, Math.PI * 2);
        ctx.fill();
        ctx.shadowBlur = 0;

        // Horizontal sun stripes
        ctx.fillStyle = `hsl(${state.hue1}, ${state.saturation * 0.75}%, 14%)`;
        for (let s = 0; s < 6; s++) {
          const stripeY = horizonY - sunRadius * 0.25 + s * (sunRadius * 0.14);
          const stripeH = 1.5 + s * 1.1;
          ctx.fillRect(cx - sunRadius, stripeY, sunRadius * 2, stripeH);
        }
        ctx.restore();

        // Reactive Mountain Silhouettes on Horizon
        ctx.save();
        ctx.fillStyle = `hsl(${state.hue3}, ${state.saturation * 0.5}%, 7%)`;
        ctx.beginPath();
        ctx.moveTo(0, horizonY);
        const peaks = 48;
        for (let i = 0; i <= peaks; i++) {
          const pct = i / peaks;
          const x = pct * width;
          // Higher mountains on left & right, valley in center to frame the sun
          const valleyMask = Math.pow(Math.abs(pct - 0.5) * 2, 1.35);
          const freqIdx = Math.floor(pct * 120);
          const freqBoost = frequencyData && state.isPlaying ? (frequencyData[freqIdx] / 255) * 65 * state.sensitivity : 12;
          const ridge = (Math.sin(i * 1.7) * 28 + Math.cos(i * 3.1) * 18 + freqBoost) * valleyMask;
          ctx.lineTo(x, horizonY - Math.max(4, ridge));
        }
        ctx.lineTo(width, horizonY);
        ctx.closePath();
        ctx.fill();
        ctx.restore();

        // Ground Plane & 3D Perspective Moving Grid
        ctx.fillStyle = `hsl(${state.hue3}, ${state.saturation * 0.6}%, 5%)`;
        ctx.fillRect(0, horizonY, width, height - horizonY);

        ctx.save();
        ctx.globalCompositeOperation = 'lighter';
        ctx.strokeStyle = `hsla(${state.hue1}, ${state.saturation}%, 65%, ${Math.min(0.95, (0.45 + state.bass * 0.45) * state.glowIntensity)})`;
        ctx.lineWidth = 1.5;

        // Perspective radiating vertical lines
        const vLines = 24;
        for (let i = -vLines; i <= vLines; i++) {
          const topX = cx + i * (width * 0.025);
          const botX = cx + i * (width * 0.14);
          ctx.beginPath();
          ctx.moveTo(topX, horizonY);
          ctx.lineTo(botX, height);
          ctx.stroke();
        }

        // Animated scrolling horizontal grid lines
        const speed = (time * (0.0008 + state.bass * 0.0025)) % 1;
        const hLines = 14;
        for (let j = 0; j < hLines; j++) {
          const progress = (j + speed) / hLines;
          const y = horizonY + Math.pow(progress, 2.3) * (height - horizonY);
          ctx.beginPath();
          ctx.moveTo(0, y);
          ctx.lineTo(width, y);
          ctx.stroke();
        }

        // Glowing Horizon Laser Line
        ctx.strokeStyle = `hsla(${state.hue2}, 95%, 78%, 0.9)`;
        ctx.lineWidth = 2.5;
        ctx.beginPath();
        ctx.moveTo(0, horizonY);
        ctx.lineTo(width, horizonY);
        ctx.stroke();
        ctx.restore();
      }

      // VISUAL 7: DNA HELIX (3D Rotating Double-Helix Particle Strand)
      function renderDnaHelixMode(time) {
        drawAmbientBackground(time);

        const cx = width * 0.5;
        const cy = height * 0.5;
        const isLandscape = width >= height;
        const span = isLandscape ? width * 0.78 : height * 0.72;
        const amplitude = Math.min(width, height) * (0.15 + state.bass * 0.08 + state.beatPulse * 0.03);
        const nodes = 44;
        const rotSpeed = time * 0.0022;

        ctx.save();
        ctx.globalCompositeOperation = 'lighter';

        for (let i = 0; i < nodes; i++) {
          const pct = i / (nodes - 1);
          const posAlong = (pct - 0.5) * span;
          const freqIdx = Math.floor(pct * 140);
          const freqVal = frequencyData && state.isPlaying
            ? (frequencyData[freqIdx] / 255) * state.sensitivity
            : 0.2;

          const angle1 = pct * Math.PI * 3.5 + rotSpeed;
          const angle2 = angle1 + Math.PI;

          const localAmp = amplitude * (0.85 + freqVal * 0.55);
          const offset1 = Math.sin(angle1) * localAmp;
          const depth1 = Math.cos(angle1); // -1 to 1

          const offset2 = Math.sin(angle2) * localAmp;
          const depth2 = Math.cos(angle2);

          const x1 = isLandscape ? cx + posAlong : cx + offset1;
          const y1 = isLandscape ? cy + offset1 : cy + posAlong;
          const x2 = isLandscape ? cx + posAlong : cx + offset2;
          const y2 = isLandscape ? cy + offset2 : cy + posAlong;

          // Connecting Base-Pair Rung
          const rungHue = (state.hue3 + pct * 70) % 360;
          ctx.strokeStyle = `hsla(${rungHue}, ${state.saturation}%, 68%, ${(0.18 + freqVal * 0.35) * state.glowIntensity})`;
          ctx.lineWidth = 1.5 + freqVal * 2;
          ctx.beginPath();
          ctx.moveTo(x1, y1);
          ctx.lineTo(x2, y2);
          ctx.stroke();

          // Strand A Node
          const size1 = Math.max(2.5, (5 + depth1 * 3.2 + freqVal * 5) * (0.8 + state.glowIntensity * 0.2));
          ctx.fillStyle = `hsla(${state.hue1}, ${state.saturation}%, ${58 + depth1 * 18}%, ${0.45 + (depth1 + 1) * 0.27})`;
          ctx.beginPath();
          ctx.arc(x1, y1, size1, 0, Math.PI * 2);
          ctx.fill();

          // Strand B Node
          const size2 = Math.max(2.5, (5 + depth2 * 3.2 + freqVal * 5) * (0.8 + state.glowIntensity * 0.2));
          ctx.fillStyle = `hsla(${state.hue2}, ${state.saturation}%, ${58 + depth2 * 18}%, ${0.45 + (depth2 + 1) * 0.27})`;
          ctx.beginPath();
          ctx.arc(x2, y2, size2, 0, Math.PI * 2);
          ctx.fill();
        }

        ctx.restore();
      }

      // VISUAL 8: KALEIDOSCOPE (Symmetrical Geometric Mandala Fractals)
      function renderKaleidoscopeMode(time) {
        drawAmbientBackground(time);

        const cx = width * 0.5;
        const cy = height * 0.5;
        const maxR = Math.min(width, height) * 0.44;
        const slices = 12;
        const sliceAngle = (Math.PI * 2) / slices;

        ctx.save();
        ctx.translate(cx, cy);
        ctx.rotate(time * 0.0004 + state.bass * 0.15);
        ctx.globalCompositeOperation = 'lighter';

        const rings = 5;
        for (let r = 1; r <= rings; r++) {
          const ringPct = r / rings;
          const radius = maxR * ringPct * (0.85 + state.bass * 0.28 + state.beatPulse * 0.08);
          const freqIdx = r * 24;
          const freqVal = frequencyData && state.isPlaying
            ? (frequencyData[freqIdx] / 255) * state.sensitivity
            : 0.25;

          const ringHue = r % 3 === 0 ? state.hue1 : r % 3 === 1 ? state.hue2 : state.hue3;

          for (let s = 0; s < slices; s++) {
            ctx.save();
            ctx.rotate(s * sliceAngle);
            if (s % 2 === 1) ctx.scale(1, -1); // Mirror symmetry

            const petalLen = (maxR / rings) * (0.8 + freqVal * 1.1);
            const spread = sliceAngle * (0.45 + state.mid * 0.4);

            ctx.strokeStyle = `hsla(${ringHue}, ${state.saturation}%, 72%, ${Math.min(0.9, (0.28 + freqVal * 0.5) * state.glowIntensity)})`;
            ctx.fillStyle = `hsla(${ringHue}, ${state.saturation}%, 55%, ${Math.min(0.35, (0.06 + freqVal * 0.16) * state.glowIntensity)})`;
            ctx.lineWidth = 1.6 + state.bass * 1.5;

            ctx.beginPath();
            ctx.moveTo(radius * 0.65, 0);
            ctx.quadraticCurveTo(
              radius + petalLen * 0.5,
              Math.sin(spread) * radius,
              radius + petalLen,
              0
            );
            ctx.quadraticCurveTo(
              radius + petalLen * 0.5,
              -Math.sin(spread) * radius,
              radius * 0.65,
              0
            );
            ctx.closePath();
            ctx.fill();
            ctx.stroke();

            ctx.restore();
          }
        }

        ctx.restore();
      }

      // ==========================================
      // 10. MAIN RENDER LOOP
      // ==========================================
      function renderLoop(time) {
        analyzeAudio(time);

        if (state.visual === 'aura') {
          renderAuraMode(time);
        } else if (state.visual === 'orb') {
          renderPulseOrbMode(time);
        } else if (state.visual === 'starfield') {
          renderStarfieldMode(time);
        } else if (state.visual === 'horizon') {
          renderWaveformHorizonMode(time);
        } else if (state.visual === 'aurora') {
          renderAuroraMode(time);
        } else if (state.visual === 'cybergrid') {
          renderCyberGridMode(time);
        } else if (state.visual === 'dna') {
          renderDnaHelixMode(time);
        } else if (state.visual === 'kaleidoscope') {
          renderKaleidoscopeMode(time);
        }

        requestAnimationFrame(renderLoop);
      }

      window.onload = function () {
        requestAnimationFrame(renderLoop);
      };
    })();
  </script>
</body>
</html>
