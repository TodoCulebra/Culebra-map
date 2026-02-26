<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Culebra Map</title>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.css" crossorigin="anonymous"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet/1.9.4/leaflet.min.js" crossorigin="anonymous"></script>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet-routing-machine/3.2.12/leaflet-routing-machine.min.css" crossorigin="anonymous"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet-routing-machine/3.2.12/leaflet-routing-machine.min.js" crossorigin="anonymous"></script>
<!-- MarkerCluster — collapses dense pins at low zoom, expands on zoom-in -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.5.3/MarkerCluster.css" crossorigin="anonymous"/>
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.5.3/MarkerCluster.Default.css" crossorigin="anonymous"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.5.3/leaflet.markercluster.js" crossorigin="anonymous"></script>

<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600;700&family=DM+Serif+Display:ital@0;1&display=swap" rel="stylesheet"/>
<style>
  :root {
    --c-ocean: #006994;
    --c-aqua:  #00a0b0;
    --c-navy:  #0b2240;
    --c-sand:  #f5f0e8;
    --c-white: #ffffff;
    --c-text:  #1a1a2e;
    --c-muted: #6b7280;
    --c-green: #10b981;
    --c-amber: #f59e0b;
    --c-red:   #ef4444;
    --radius:  14px;
    --shadow:  0 4px 24px rgba(0,0,0,0.14);
    --font-ui: 'DM Sans', system-ui, -apple-system, sans-serif;
  }
  *, *::before, *::after { box-sizing: border-box; }
  html, body { height: 100%; margin: 0; font-family: var(--font-ui); }
  body { display: -webkit-box; display: -ms-flexbox; display: flex; -webkit-box-orient: vertical; -webkit-box-direction: normal; -ms-flex-direction: column; flex-direction: column; background: #e8f4f8; }
  #map { -webkit-box-flex: 1; -ms-flex: 1; flex: 1; min-height: 0; width: 100%; }

  /* ── BOTTOM SHEET / PANEL BASE ── */
  .sheet {
    position: absolute; bottom: 0; left: 0; right: 0; z-index: 1000;
    background: var(--c-white); border-radius: 20px 20px 0 0;
    box-shadow: 0 -6px 32px rgba(0,0,0,0.16);
    -webkit-transform: translateY(100%); transform: translateY(100%);
    -webkit-transition: -webkit-transform 0.34s cubic-bezier(0.4,0,0.2,1);
    transition: transform 0.34s cubic-bezier(0.4,0,0.2,1);
    max-height: 70vh; display: flex; flex-direction: column;
  }
  .sheet.open { -webkit-transform: translateY(0); transform: translateY(0); }
  .sheet-handle { width: 40px; height: 4px; background: #e0e0e0; border-radius: 2px; margin: 10px auto 0; flex-shrink: 0; }

  /* ── CATEGORY BAR (bottom) ── */
  #cat-bar {
    position: absolute; bottom: 0; left: 0; right: 0; z-index: 999;
    background: rgba(255,255,255,0.97);
    -webkit-backdrop-filter: blur(16px); backdrop-filter: blur(16px);
    border-top: 1px solid rgba(0,0,0,0.08);
    padding: 10px 12px max(12px, env(safe-area-inset-bottom));
    display: flex; gap: 8px; overflow-x: auto; -webkit-overflow-scrolling: touch;
    scrollbar-width: none;
    transition: transform 0.34s cubic-bezier(0.4,0,0.2,1);
  }
  #cat-bar::-webkit-scrollbar { display: none; }
  #cat-bar.hidden { transform: translateY(100%); }
  .cat-pill {
    display: flex; align-items: center; gap: 6px; white-space: nowrap;
    padding: 8px 14px; border-radius: 999px; border: 1.5px solid rgba(0,0,0,0.12);
    background: var(--c-white); color: var(--c-text);
    font-family: var(--font-ui); font-size: 13px; font-weight: 600;
    cursor: pointer; flex-shrink: 0;
    transition: all 0.18s ease; -webkit-tap-highlight-color: transparent;
    box-shadow: 0 1px 4px rgba(0,0,0,0.08);
  }
  .cat-pill:hover  { border-color: var(--c-ocean); color: var(--c-ocean); }
  .cat-pill.active { background: var(--c-ocean); border-color: var(--c-ocean); color: #fff; box-shadow: 0 2px 10px rgba(0,105,148,0.4); }
  .cat-pill .pill-icon { font-size: 15px; }

  /* ── TOP SEARCH BAR — upper left ── */
  #topbar {
    position: absolute; top: 12px; left: 12px; z-index: 1002;
    width: min(280px, calc(100vw - 24px));
    pointer-events: none;
  }
  #search-outer {
    display: flex; align-items: center; gap: 0; pointer-events: all;
  }
  #search-card {
    flex: 1; display: flex; align-items: center; gap: 8px;
    background: rgba(255,255,255,0.97);
    -webkit-backdrop-filter: blur(16px); backdrop-filter: blur(16px);
    border-radius: 14px; padding: 0 14px;
    box-shadow: 0 2px 16px rgba(0,0,0,0.18);
    border: 1px solid rgba(255,255,255,0.6);
    height: 44px; transition: box-shadow 0.2s;
  }
  #search-card:focus-within { box-shadow: 0 2px 20px rgba(0,105,148,0.28); }
  #search-icon { color: var(--c-muted); font-size: 15px; flex-shrink: 0; }
  #search-input {
    border: none; outline: none; background: transparent;
    font-family: var(--font-ui); font-size: 14px; font-weight: 400;
    color: var(--c-text); flex: 1; min-width: 0;
  }
  #search-input::placeholder { color: #aaa; }
  #search-clear {
    display: none; background: #e0e0e0; border: none; border-radius: 50%;
    width: 20px; height: 20px; font-size: 11px; cursor: pointer; color: #555;
    align-items: center; justify-content: center; flex-shrink: 0;
    font-weight: 700; line-height: 1;
  }
  #search-clear.visible { display: flex; }
  #topbar-btn { display: none; }
  #search-results {
    position: absolute; top: calc(100% + 4px); left: 0; right: 0;
    background: var(--c-white); border-radius: 14px;
    box-shadow: var(--shadow); overflow: hidden; display: none;
    pointer-events: all; max-height: 60vh; overflow-y: auto;
  }
  .search-result-item {
    display: flex; align-items: center; gap: 10px;
    padding: 11px 16px; font-size: 14px; color: var(--c-text);
    cursor: pointer; border-bottom: 1px solid #f3f3f3; font-family: var(--font-ui);
  }
  .search-result-item:last-child { border-bottom: none; }
  .search-result-item:hover { background: #f0f8ff; }
  .sri-icon { font-size: 16px; flex-shrink: 0; }
  .sri-text { flex: 1; min-width: 0; }
  .sri-name { font-weight: 600; font-size: 14px; }
  .sri-type { font-size: 11px; color: var(--c-muted); text-transform: uppercase; letter-spacing: 0.05em; }

  /* ── LEFT SIDE BUTTON STACK ── */
  #left-stack {
    position: absolute; left: 12px; z-index: 1001;
    display: flex; flex-direction: column; gap: 8px;
    top: 68px; /* below search bar */
  }
  .stack-btn {
    width: 44px; height: 44px; border-radius: 12px; border: none;
    background: var(--c-navy); color: #fff;
    box-shadow: 0 2px 12px rgba(0,0,0,0.35);
    font-size: 20px; cursor: pointer; display: flex; align-items: center; justify-content: center;
    transition: background 0.15s; flex-shrink: 0;
  }
  .stack-btn:hover { background: #1a4a7a; }

  /* ── LEGEND DROPDOWN ── */
  #legend-wrap { position: relative; }
  #legend-drop {
    position: absolute; top: calc(100% + 6px); left: 0;
    background: var(--c-navy); border: 1.5px solid #1a4a7a;
    border-radius: 14px; box-shadow: 0 4px 20px rgba(0,0,0,0.45);
    min-width: 200px; overflow: hidden;
    transform: scale(0.92) translateY(-8px);
    transform-origin: top left;
    opacity: 0; pointer-events: none;
    transition: opacity 0.2s ease, transform 0.2s ease;
  }
  #legend-drop.open { opacity: 1; transform: scale(1) translateY(0); pointer-events: all; }
  #legend-drop ul { list-style: none; margin: 0; padding: 8px 0; }
  #legend-drop li {
    display: flex; align-items: center; gap: 10px;
    padding: 9px 16px; color: #e8f4f8; font-size: 13px; font-weight: 500;
    cursor: pointer; transition: background 0.15s; white-space: nowrap;
  }
  #legend-drop li:hover { background: rgba(255,255,255,0.08); }
  #legend-drop li.active { background: rgba(0,160,176,0.25); color: #fff; }
  #legend-drop li .ld-dot {
    width: 12px; height: 12px; border-radius: 50%; flex-shrink: 0;
  }
  #legend-drop-title {
    padding: 10px 16px 6px; font-size: 10px; font-weight: 700; color: #7ecfde;
    text-transform: uppercase; letter-spacing: 0.1em; border-bottom: 1px solid rgba(255,255,255,0.1);
    margin-bottom: 2px;
  }

  /* ── LOCATE ME ── */
  #btn-locate {
    width: 44px; height: 44px; border-radius: 12px; border: none;
    background: var(--c-navy); color: #fff;
    box-shadow: 0 2px 12px rgba(0,0,0,0.35);
    font-size: 20px; cursor: pointer; display: flex; align-items: center; justify-content: center;
    transition: background 0.15s;
  }
  #btn-locate:hover  { background: #1a4a7a; }
  #btn-locate.locating { animation: locatePulse 1.2s infinite; }
  @keyframes locatePulse { 0%,100% { opacity:1; } 50% { opacity:0.5; } }

  /* ── DISTANCE BADGE in popups ── */
  .dist-badge {
    display: inline-flex; align-items: center; gap: 4px;
    font-size: 11px; font-weight: 600; color: var(--c-ocean);
    background: rgba(0,105,148,0.1); border-radius: 999px;
    padding: 3px 8px; margin-bottom: 8px;
  }

  /* ── SHARE BUTTON ── */
  .btn-share {
    display: flex; align-items: center; justify-content: center; gap: 6px;
    width: 100%; padding: 9px 0;
    background: transparent; color: var(--c-ocean);
    border: 1.5px solid var(--c-ocean); border-radius: 8px;
    font-family: var(--font-ui); font-size: 13px; font-weight: 600;
    cursor: pointer; transition: all 0.15s; margin-top: 6px;
  }
  .btn-share:hover { background: rgba(0,105,148,0.06); }
  .btn-share.copied { background: var(--c-green); border-color: var(--c-green); color: #fff; }

  /* ── POPUP REFINEMENTS ── */
  .leaflet-popup-content-wrapper {
    background: var(--c-white); border-radius: 18px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.18); padding: 0; overflow: hidden; border: none;
  }
  .leaflet-popup-tip { background: var(--c-white); }
  .leaflet-popup-content { margin: 0; font-family: var(--font-ui); font-size: 13px; line-height: 1.5; min-width: 260px; max-width: 310px; }
  .popup-img { width: 100%; height: 135px; object-fit: cover; display: block; }
  .popup-body { padding: 13px 15px 15px; }
  .popup-title { font-size: 16px; font-weight: 700; color: var(--c-text); margin-bottom: 4px; letter-spacing: -0.01em; }
  .popup-type-row { display: flex; align-items: center; gap: 6px; margin-bottom: 8px; }
  .popup-type-tag {
    font-size: 10px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.08em;
    padding: 2px 7px; border-radius: 999px;
  }
  .popup-road-tag {
    font-size: 10px; font-weight: 600; color: #b45309;
    background: #fef3c7; padding: 2px 7px; border-radius: 999px;
  }
  .popup-desc { font-size: 12.5px; color: #555; margin-bottom: 10px; line-height: 1.55; }
  .popup-desc a { color: var(--c-ocean); }
  .popup-hours { font-size: 11.5px; color: var(--c-muted); margin-bottom: 10px; display: flex; align-items: center; gap: 5px; }
  .popup-hours .open-dot  { color: var(--c-green); font-size: 10px; }
  .popup-hours .closed-dot{ color: var(--c-red);   font-size: 10px; }

  /* ── ROAD CONDITION BADGE ── */
  .road-4x4  { background: #fef3c7; color: #b45309; }
  .road-sandy { background: #fef3c7; color: #b45309; }
  .road-paved { background: #d1fae5; color: #065f46; }
  .road-boat  { background: #dbeafe; color: #1e40af; }

  /* ── TOAST NOTIFICATION ── */
  #toast {
    position: absolute; top: 78px; left: 50%; transform: translateX(-50%);
    z-index: 2000; background: var(--c-text); color: #fff;
    padding: 10px 18px; border-radius: 999px; font-size: 13px; font-weight: 600;
    box-shadow: 0 4px 16px rgba(0,0,0,0.3); white-space: nowrap;
    opacity: 0; transition: opacity 0.3s ease; pointer-events: none;
  }
  #toast.show { opacity: 1; }

  /* ── INFO PANEL (side sheet on desktop, bottom sheet on mobile) ── */
  @media (min-width: 768px) {
    #turn-panel, #info-panel {
      left: auto !important; right: 16px !important;
      bottom: 16px !important;
      width: 340px;
      border-radius: 18px !important;
      max-height: calc(100vh - 80px) !important;
      transform: translateX(120%) !important;
    }
    #turn-panel.open, #info-panel.open {
      transform: translateX(0) !important;
    }
  }

  /* ── LEAFLET OVERRIDES ── */
  .leaflet-control-zoom { border-radius: 12px !important; overflow: hidden; border: none !important; box-shadow: 0 2px 12px rgba(0,0,0,0.18) !important; }
  .leaflet-control-zoom a { font-size: 18px !important; width: 36px !important; height: 36px !important; line-height: 36px !important; color: var(--c-text) !important; background: rgba(255,255,255,0.97) !important; border: none !important; font-weight: 300 !important; }
  .leaflet-control-zoom a:hover { background: #f0f8ff !important; }
  .leaflet-routing-container { display: none !important; }
  .leaflet-control-attribution { font-size: 10px !important; background: rgba(255,255,255,0.75) !important; backdrop-filter: blur(4px) !important; border-radius: 6px 0 0 0 !important; }

  /* ── MARKER CLUSTER ── */
  .marker-cluster { background: transparent !important; }
  .marker-cluster div { background: transparent !important; }

  /* ── FULLSCREEN ── */
  #btn-fullscreen {
    position: absolute; top: 70px; right: 10px; z-index: 1001;
    background: rgba(255,255,255,0.97); border: 1px solid rgba(0,0,0,0.15);
    border-radius: 10px; width: 36px; height: 36px; cursor: pointer; font-size: 16px;
    display: flex; align-items: center; justify-content: center;
    box-shadow: 0 2px 8px rgba(0,0,0,0.15); transition: background 0.15s; color: var(--c-text);
  }
  #btn-fullscreen:hover { background: #f0f8ff; }

  /* ── USER DOT ── */
  .user-dot { width: 16px; height: 16px; border-radius: 50%; background: #2f80ff; border: 3px solid #fff; box-shadow: 0 0 0 4px rgba(47,128,255,0.3), 0 0 0 8px rgba(47,128,255,0.1); animation: pulse 2s infinite; }
  @keyframes pulse {
    0%,100% { box-shadow: 0 0 0 4px rgba(47,128,255,0.3), 0 0 0 8px rgba(47,128,255,0.1); }
    50%      { box-shadow: 0 0 0 6px rgba(47,128,255,0.4), 0 0 0 14px rgba(47,128,255,0.15); }
  }

  /* ── MAP CONTROLS ── */
  .leaflet-control-custom button { background: rgba(255,255,255,0.97); border: 1px solid rgba(0,0,0,0.15); border-radius: 10px; width: 36px; height: 36px; cursor: pointer; font-size: 18px; display: flex; align-items: center; justify-content: center; box-shadow: 0 2px 8px rgba(0,0,0,0.15); margin: 2px; }
  .leaflet-control-custom button:hover { background: #f0f8ff; }

  /* ── DIRECTIONS BUTTON ── */
  .btn-directions {
    display: block; width: 100%; padding: 10px 0;
    background: linear-gradient(135deg, var(--c-ocean), var(--c-aqua)); color: #fff;
    border: none; border-radius: 10px; font-family: var(--font-ui); font-size: 13px; font-weight: 700;
    cursor: pointer; text-align: center; transition: opacity 0.15s;
    margin-bottom: 6px; letter-spacing: 0.01em;
  }
  .btn-directions:hover { opacity: 0.88; }
  .btn-directions:disabled { background: #ccc; cursor: not-allowed; opacity: 1; }

  /* ── FALLBACK BUTTONS ── */
  .fallback-row { display: none; gap: 6px; margin-top: 6px; }
  .fallback-row.visible { display: flex; }
  .btn-fallback { flex: 1; padding: 8px 0; border: none; border-radius: 9px; font-family: var(--font-ui); font-size: 12px; font-weight: 600; cursor: pointer; text-align: center; text-decoration: none; color: #fff; transition: opacity 0.15s; display: flex; align-items: center; justify-content: center; gap: 4px; }
  .btn-fallback:hover { opacity: 0.85; }
  .btn-apple  { background: #1c1c1e; }
  .btn-google { background: #4285F4; }

  /* ── ROUTE STATUS ── */
  .route-status { font-size: 11px; color: var(--c-muted); text-align: center; margin-top: 4px; min-height: 16px; }
  .route-status.error { color: var(--c-red); }

  /* ── TURN PANEL ── */
  #turn-panel {
    position: absolute; bottom: 0; left: 0; right: 0; z-index: 1000;
    background: var(--c-white); border-radius: 20px 20px 0 0;
    box-shadow: 0 -4px 24px rgba(0,0,0,0.14);
    padding-bottom: 16px;
    -webkit-transform: translateY(100%); transform: translateY(100%);
    -webkit-transition: -webkit-transform 0.32s cubic-bezier(0.4,0,0.2,1);
    transition: transform 0.32s cubic-bezier(0.4,0,0.2,1);
    max-height: 52vh; display: flex; flex-direction: column;
  }
  #turn-panel.open { -webkit-transform: translateY(0); transform: translateY(0); }
  #turn-handle { width: 36px; height: 4px; background: #e0e0e0; border-radius: 2px; margin: 10px auto 0; flex-shrink: 0; }
  #turn-header { display: flex; align-items: center; justify-content: space-between; padding: 10px 16px 8px; flex-shrink: 0; border-bottom: 1px solid #f0f0f0; }
  #turn-title { font-size: 15px; font-weight: 700; color: var(--c-text); }
  #turn-meta  { font-size: 12px; color: var(--c-muted); margin-top: 2px; }
  #turn-close { background: #f0f0f0; border: none; border-radius: 50%; width: 28px; height: 28px; font-size: 15px; cursor: pointer; display: flex; align-items: center; justify-content: center; color: #555; flex-shrink: 0; }
  #turn-steps { overflow-y: auto; padding: 8px 0 20px; flex: 1; }
  .turn-step { display: flex; align-items: flex-start; gap: 10px; padding: 8px 16px; border-bottom: 1px solid #f5f5f5; font-size: 13px; color: var(--c-text); line-height: 1.4; }
  .turn-step:last-child { border-bottom: none; }
  .turn-icon { font-size: 17px; flex-shrink: 0; width: 22px; text-align: center; margin-top: 1px; }
  .turn-dist { font-size: 11px; color: var(--c-muted); margin-top: 2px; }

  /* ── EASTER EGG OVERLAY ── */
  #ee-overlay {
    position: fixed; inset: 0; z-index: 9999;
    background: radial-gradient(ellipse at 50% 100%, #001a3a 0%, #000d1f 100%);
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    opacity: 0; pointer-events: none; transition: opacity 0.6s;
  }
  #ee-overlay.show { opacity: 1; pointer-events: all; }
  #ee-overlay canvas { position: absolute; inset: 0; }
  #ee-msg {
    position: relative; z-index: 2; text-align: center;
    font-family: 'DM Serif Display', serif; color: #fff;
    text-shadow: 0 2px 12px rgba(0,200,255,0.5);
  }
  #ee-msg h2 { font-size: clamp(24px,6vw,48px); margin: 0 0 8px; letter-spacing: -0.02em; }
  #ee-msg p  { font-size: clamp(13px,3vw,18px); opacity: 0.75; margin: 0 0 24px; font-family: var(--font-ui); }
  #ee-close  {
    position: relative; z-index: 2;
    padding: 12px 32px; border-radius: 999px; border: 2px solid rgba(255,255,255,0.4);
    background: transparent; color: #fff; font-family: var(--font-ui); font-size: 14px; font-weight: 600;
    cursor: pointer; transition: background 0.2s; margin-top: 8px;
  }
  #ee-close:hover { background: rgba(255,255,255,0.15); }

  /* ── MOBILE TAP RIPPLE ── */
  .tap-ripple {
    position: fixed; border-radius: 50%; pointer-events: none; z-index: 9998;
    background: radial-gradient(circle, rgba(0,160,176,0.5) 0%, transparent 70%);
    transform: scale(0); animation: rippleOut 0.6s ease-out forwards;
  }
  @keyframes rippleOut { to { transform: scale(4); opacity: 0; } }

  /* ── WELCOME BANNER — top center, persistent ── */
  .welcome-badge {
    position: absolute; top: 12px; left: 50%;
    -webkit-transform: translateX(-50%); transform: translateX(-50%);
    z-index: 1001; background: linear-gradient(135deg, var(--c-navy), #1a4a7a); color: #fff;
    padding: 7px 20px; font-size: 13px; font-weight: 600; border-radius: 999px;
    box-shadow: 0 2px 12px rgba(0,0,0,0.35); white-space: nowrap; pointer-events: none;
    letter-spacing: 0.02em; font-family: var(--font-ui);
  }

  /* ── HIDE OLD ELEMENTS ── */
  #legend, #search-wrap, #search-btn { display: none !important; }
  #cat-bar { display: none !important; }

  /* ── MOBILE SAFE AREA ── */
  @media (max-width: 767px) {
    #btn-fullscreen { display: none; }
    .leaflet-control-zoom { bottom: 80px !important; }
  }


</style>
</head>
<body>

<!-- WELCOME BANNER -->
<div class="welcome-badge">¡Bienvenidos a Culebra! 🌴 Tap any pin for info &amp; directions</div>

<!-- TOP SEARCH BAR — upper left -->
<div id="topbar">
  <div id="search-outer">
    <div id="search-card">
      <span id="search-icon">🔍</span>
      <input id="search-input" type="text" placeholder="Search Culebra…" autocomplete="off"
        oninput="liveSearch(this.value)" onkeydown="if(event.key==='Enter')doSearch()"/>
      <button id="search-clear" onclick="clearSearch()" aria-label="Clear">✕</button>
    </div>
  </div>
  <div id="search-results"></div>
</div>

<!-- MAP -->
<div id="map"></div>

<!-- FULLSCREEN (alias kept for JS compat) -->
<button id="btn-fullscreen" title="Toggle fullscreen" onclick="toggleFullscreen()" style="display:none">⛶</button>

<!-- LEFT BUTTON STACK: Legend → Home → Locate -->
<div id="left-stack">
  <!-- LEGEND -->
  <div id="legend-wrap">
    <button class="stack-btn" id="legend-toggle-btn" onclick="toggleLegendDrop()" title="Map Legend">🗺️</button>
    <div id="legend-drop">
      <div id="legend-drop-title">Filter by Category</div>
      <ul id="legend-list"></ul>
    </div>
  </div>
  <!-- HOME -->
  <button class="stack-btn" onclick="map.flyTo([18.310, -65.302], 13)" title="Home">🏠</button>
  <!-- LOCATE ME -->
  <button id="btn-locate" title="My Location" onclick="locateMe()">📍</button>
</div>

<!-- CATEGORY PILL BAR -->
<div id="cat-bar">
  <button class="cat-pill active" id="pill-all"        onclick="filterMarkers(null)">          <span class="pill-icon">🗺️</span><span class="pill-label">All</span></button>
  <button class="cat-pill"        id="pill-beach"       onclick="filterMarkers('beach')">       <span class="pill-icon">🏖️</span><span class="pill-label">Beaches</span></button>
  <button class="cat-pill"        id="pill-food"        onclick="filterMarkers('food')">        <span class="pill-icon">🍽️</span><span class="pill-label">Food</span></button>
  <button class="cat-pill"        id="pill-ferry"       onclick="filterMarkers('ferry')">       <span class="pill-icon">⛴️</span><span class="pill-label">Ferry</span></button>
  <button class="cat-pill"        id="pill-gas"         onclick="filterMarkers('gas')">         <span class="pill-icon">⛽</span><span class="pill-label">Gas</span></button>
  <button class="cat-pill"        id="pill-atm"         onclick="filterMarkers('atm')">         <span class="pill-icon">🏧</span><span class="pill-label">ATM</span></button>
  <button class="cat-pill"        id="pill-parking"     onclick="filterMarkers('parking')">     <span class="pill-icon">🅿️</span><span class="pill-label">Parking</span></button>
  <button class="cat-pill"        id="pill-restroom"    onclick="filterMarkers('restroom')">    <span class="pill-icon">🚻</span><span class="pill-label">Restrooms</span></button>
  <button class="cat-pill"        id="pill-rental"      onclick="filterMarkers('rental')">      <span class="pill-icon">🚤</span><span class="pill-label">Rentals</span></button>
  <button class="cat-pill"        id="pill-anchor"      onclick="filterMarkers('anchor')">      <span class="pill-icon">⚓</span><span class="pill-label">Anchorage</span></button>
  <button class="cat-pill"        id="pill-hike"        onclick="filterMarkers('hike')">        <span class="pill-icon">🥾</span><span class="pill-label">Hiking</span></button>
  <button class="cat-pill"        id="pill-grocery"     onclick="filterMarkers('grocery')">     <span class="pill-icon">🛒</span><span class="pill-label">Grocery</span></button>
  <button class="cat-pill"        id="pill-watertaxi"   onclick="filterMarkers('watertaxi')">   <span class="pill-icon">🚢</span><span class="pill-label">Water Taxi</span></button>
  <button class="cat-pill"        id="pill-hector"      onclick="filterMarkers('hector')">      <span class="pill-icon">🧌</span><span class="pill-label">Hector</span></button>
</div>

<!-- TURN-BY-TURN DRAWER -->
<div id="turn-panel">
  <div id="turn-handle"></div>
  <div id="turn-header">
    <div>
      <div id="turn-title">Directions</div>
      <div id="turn-meta"></div>
    </div>
    <button id="turn-close" onclick="clearRoute()">✕</button>
  </div>
  <div id="turn-steps"></div>
</div>

<!-- TOAST -->
<div id="toast"></div>

<!-- EASTER EGG OCEAN OVERLAY -->
<div id="ee-overlay">
  <canvas id="ee-canvas"></canvas>
  <div id="ee-msg">
    <h2>🌊 You Found the Deep!</h2>
    <p>Welcome to the secret ocean depths of Culebra…</p>
  </div>
  <button id="ee-close" onclick="closeOceanEE()">Return to Surface 🤿</button>
</div>

<script>
/* ══════════════════════════════════════════════════════════════════
   INVISIBLE GRID — snap GPS to stable ~16m x ~8m point pre-routing
══════════════════════════════════════════════════════════════════ */
var BOUNDS   = { west: -65.365, east: -65.215, south: 18.270, north: 18.345 };
var CELLS    = 1000;
var CELL_LNG = (BOUNDS.east  - BOUNDS.west)  / CELLS;
var CELL_LAT = (BOUNDS.north - BOUNDS.south) / CELLS;

function snapToGrid(lat, lng) {
  var col = Math.round((lng - BOUNDS.west)  / CELL_LNG);
  var row = Math.round((BOUNDS.north - lat) / CELL_LAT);
  col = Math.max(0, Math.min(CELLS, col));
  row = Math.max(0, Math.min(CELLS, row));
  return { lat: BOUNDS.north - row * CELL_LAT, lng: BOUNDS.west + col * CELL_LNG };
}

/* ══════════════════════════════════════════════════════════════════
   LOCATIONS
══════════════════════════════════════════════════════════════════ */
var locations = [
  { name: 'Culebra Ferry Terminal', lat: 18.301451, lng: -65.302530, type: 'ferry', img: null,
    desc: 'Our one and only ferry terminal &#8212; arrive 1 hour before departure to avoid delays!' },
  { name: 'Ceiba Ferry Terminal', lat: 18.228669, lng: -65.619310, type: 'ferry', img: null,
    desc: 'Mainland ferry terminal in Ceiba &#8212; your gateway to Culebra. <a href="https://www.puertoricoferry.com/en/" target="_blank" style="color:#006994;">Check schedules &amp; book tickets</a> before you go!' },
  { name: 'Gas Station', lat: 18.301129, lng: -65.300764, type: 'gas', img: null,
    desc: 'The only gas station on the island.' },
  { name: 'Isla Culebrita', lat: 18.318654, lng: -65.227238, type: 'beach',
    img: 'https://upload.wikimedia.org/wikipedia/commons/3/3f/Culebrita_Island%2C_Culebra_Island%2C_Puerto_Rico.jpg',
    desc: 'The Caribbean\'s last true virgin island &#8212; lighthouse, natural jacuzzis, turtles &amp; rays. 15-min boat ride. Book with <a href="https://www.gammyswatertours.com" target="_blank" style="color:#006994;">Gammy\'s Water Tours</a>!' },
  { name: 'Punta Soldado', lat: 18.281876, lng: -65.286575, type: 'beach', img: './soldado.jpg',
    desc: 'Southernmost beach. Stunning sunsets &amp; views of Vieques. 4x4 or E-Bike only &#8212; golf carts strictly prohibited.' },
  { name: 'Playa Datiles', lat: 18.296597, lng: -65.300179, type: 'beach', img: null,
    desc: 'Most romantic beach in the Caribbean! Views of Cayo Luis Pe&ntilde;a, Vieques &amp; El Yunque. 4x4 or E-Bike required.' },
  { name: 'Playa Melones', lat: 18.303469, lng: -65.310197, type: 'beach', img: null,
    desc: 'Marine reserve reef snorkeling &#8212; fish everywhere, turtles &amp; rays. <a href="https://www.snorkelwithlocals.com" target="_blank" style="color:#006994;">snorkelwithlocals.com</a>' },
  { name: 'Playa Zoni', lat: 18.319636, lng: -65.254862, type: 'beach', img: null,
    desc: 'Beautiful 15-min drive from town. Stunning views of Culebrita, Cayo Norte &amp; Saint Thomas.' },
  { name: 'Playa Flamenco', lat: 18.328677, lng: -65.316936, type: 'beach', img: null,
    road: 'paved',
    desc: 'Consistently voted one of the world\'s best beaches. WW2 tanks, white sands, crystal-clear water.' },
  { name: 'Playa Tamarindo', lat: 18.318033, lng: -65.317453, type: 'beach', img: null,
    desc: 'One of the most popular snorkeling beaches in the entire Caribbean. Swim with Turtles, Rays, and even some friendly nurse sharks.' },
  { name: 'Hector El Protector', lat: 18.300682, lng: -65.302170, type: 'hector',
    img: 'https://www.discoverpuertorico.com/sites/default/files/listing_images/profile/8059/Hector-El-Protector_69BCB335-5056-A36A-080F2CD5E9C0A166-69bcb26d5056a36_69bcb61d-5056-a36a-0867dfd14ace9c20.jpg',
    desc: 'Our beloved guardian troll, rebuilt by locals after Hurricane Maria. By Thomas Dambo (2014).' },

  /* ── FOOD & DRINK ── */
  { name: 'El Batey', lat: 18.301800, lng: -65.302100, type: 'food', img: null,
    hours: '11am–10pm', road: 'paved',
    desc: 'Local favorite for cold Medallas and classic Puerto Rican dishes. Casual open-air vibes right in town.' },
  { name: 'Susie\'s Restaurant', lat: 18.300950, lng: -65.302800, type: 'food', img: null,
    hours: '8am–3pm', road: 'paved',
    desc: 'Beloved breakfast and lunch spot. Famous for their fresh fish plates and friendly service.' },
  { name: 'Dinghy Dock Restaurant', lat: 18.302200, lng: -65.302500, type: 'food', img: null,
    hours: '11am–9pm', road: 'paved',
    desc: 'Right on the water with tarpon swimming below your feet. Cold drinks, fresh seafood, and unbeatable sunsets.' },
  { name: 'Heather\'s Pizza', lat: 18.301200, lng: -65.302200, type: 'food', img: null,
    hours: '5pm–10pm', road: 'paved',
    desc: 'Best pizza on the island — wood-fired with fresh toppings. Popular with locals and visitors alike.' },
  { name: 'Mamacita\'s Restaurant & Bar', lat: 18.301600, lng: -65.302700, type: 'food', img: null,
    hours: '8am–10pm', road: 'paved',
    desc: 'Iconic Culebra institution on the canal. Known for the frozen painkiller cocktail and grilled fish.' },
  { name: 'El Caobo Food Truck', lat: 18.300700, lng: -65.301900, type: 'food', img: null,
    hours: '11am–7pm', road: 'paved',
    desc: 'Local food truck with rotating daily specials. Mofongo, empanadillas, and fresh fruit drinks.' },

  /* ── GROCERY & ESSENTIALS ── */
  { name: 'Milka\'s Supermarket', lat: 18.301000, lng: -65.301600, type: 'grocery', img: null,
    hours: '7am–9pm', road: 'paved',
    desc: 'Main grocery store on the island. Stock up on snacks, drinks, sunscreen, and basic supplies.' },
  { name: 'Colmado El Pescador', lat: 18.300800, lng: -65.302300, type: 'grocery', img: null,
    hours: '6am–8pm', road: 'paved',
    desc: 'Small neighborhood colmado — great for cold drinks, quick snacks, and forgotten essentials.' },

  /* ── ATMs ── */
  { name: 'Banco Popular ATM', lat: 18.301300, lng: -65.301800, type: 'atm', img: null,
    hours: '24/7', road: 'paved',
    desc: 'One of the very few ATMs on the island. Stock up on cash — many places are cash only!' },
  { name: 'FirstBank ATM', lat: 18.301100, lng: -65.302000, type: 'atm', img: null,
    hours: '24/7', road: 'paved',
    desc: 'Secondary ATM option in town. Can run out on busy weekends — grab cash early in your trip.' },

  /* ── PARKING ── */
  { name: 'Flamenco Beach Parking', lat: 18.328490, lng: -65.319426, type: 'parking', img: null,
    hours: 'Sunrise–Sunset', road: 'paved',
    desc: 'Main parking area for Playa Flamenco. Can fill up fast on weekends — arrive early! Paid parking can get expensive. E-Bike parking is free! Rent your E-Bike from <a href="https://www.todoculebra.com" target="_blank" style="color:#006994;">www.todoculebra.com</a> today!' },
  { name: 'Town Center Parking', lat: 18.301500, lng: -65.301200, type: 'parking', img: null,
    hours: 'Always open', road: 'paved',
    desc: 'Free street parking near the main plaza. Great starting point for walking the town.' },
  { name: 'Zoni Beach Parking', lat: 18.319800, lng: -65.254200, type: 'parking', img: null,
    hours: 'Sunrise–Sunset', road: '4x4',
    desc: 'Small gravel lot at the end of the road to Zoni. Limited spots — 4x4 or E-bike recommended.' },

  /* ── RESTROOMS ── */
  { name: 'Flamenco Beach Facilities', lat: 18.329470, lng: -65.320673, type: 'restroom', img: null,
    hours: '7am–6pm (inside facility)', road: null,
    desc: 'Public restrooms, cold showers, and changing areas. The best beach facilities on the island.' },

  { name: 'Flamenco Beach Kioskos', lat: 18.328980, lng: -65.320050, type: 'food', img: null,
    hours: '9am–6pm', road: null,
    desc: 'Outdoor food and drink kiosks right at Flamenco Beach. Grab pastelillos, empanadas, burgers, steaks and more. Cold drinks and plenty of bars to keep you refreshed all day. Umbrella and beach chair rentals available — everything you need without ever leaving the sand!' },

  { name: 'Culebra Beach Villas', lat: 18.327853, lng: -65.315115, type: 'villa', img: null,
    hours: 'Open · Closes 5:30 AM', road: 'unpaved',
    desc: "Culebra's Best Beach Villa, located on Flamenco Beach, you'll wake up to the sounds of the crashing waves and walk out to one of the worlds most beautiful beaches every morning.<br><br>Enhance your stay with grocery or food delivery from <a href='https://www.shopculebra.com' target='_blank' style='color:#006994;'>www.shopculebra.com</a>!<br><br>📍 Flamenco, Culebra 00775, Puerto Rico<br>📞 <a href='tel:+17877546236' style='color:#006994;'>+1 787-754-6236</a>" },
  { name: 'Town Plaza Restrooms', lat: 18.300900, lng: -65.302100, type: 'restroom', img: null,
    hours: '8am–5pm', road: 'paved',
    desc: 'Public restrooms near the main plaza in Dewey. Convenient for visitors exploring town.' },
  { name: 'Ferry Terminal Restrooms', lat: 18.301600, lng: -65.302600, type: 'restroom', img: null,
    hours: 'Ferry hours', road: 'paved',
    desc: 'Restrooms inside the ferry terminal building. Available to passengers and visitors.' },

  /* ── RENTALS ── */
  { name: 'Jerry\'s Rentals', lat: 18.301700, lng: -65.302400, type: 'rental', img: null,
    hours: '8am–5pm', road: 'paved',
    desc: 'Golf carts, E-bikes, and snorkel gear. Popular spot — reserve ahead during peak season.' },
  { name: 'Culebra Bike Shop', lat: 18.300600, lng: -65.302000, type: 'rental', img: null,
    hours: '8am–5pm', road: 'paved',
    desc: 'E-bikes and regular bikes for rent. Great way to explore the island sustainably — and reach beaches that ban golf carts.' },
  { name: 'Carlos Jeep Rental', lat: 18.301400, lng: -65.302300, type: 'rental', img: null,
    hours: '8am–4pm', road: 'paved',
    desc: '4WD jeeps and trucks for rent. Required for reaching Punta Soldado, Datiles, and other remote spots.' },
  { name: 'Culebra Divers', lat: 18.302100, lng: -65.302600, type: 'rental', img: null,
    hours: '8am–4pm', road: 'paved',
    desc: 'PADI dive shop offering tank rentals, scuba certification, and guided dive trips to the best reefs.' },
  { name: 'Paddle Culebra Kayaks', lat: 18.303200, lng: -65.311000, type: 'rental', img: null,
    hours: '8am–5pm', road: 'paved',
    desc: 'Kayak and paddleboard rentals near Melones. Perfect for paddling out to snorkel spots and small cays.' },

  /* ── WATER TAXIS ── */
  { name: 'Water Taxi Dock (Dewey)', lat: 18.302400, lng: -65.302200, type: 'watertaxi', img: null,
    hours: '7am–5pm', road: 'paved',
    desc: 'Main water taxi pickup spot in Dewey. Boats run to Culebrita, Luis Peña, and nearby snorkel spots. Cash only.' },
  /* ── ANCHORAGE ── */
  { name: 'Dakity Bay Anchorage', lat: 18.291000, lng: -65.267000, type: 'anchor', img: null,
    road: 'boat',
    desc: 'Protected anchorage on the SE side of Culebra. Calm, clear water with good holding. Popular with cruisers.' },
  { name: 'Ensenada Honda (Main Harbor)', lat: 18.302000, lng: -65.293000, type: 'anchor', img: null,
    road: 'boat',
    desc: 'Culebra\'s main harbor — well-protected from most weather. Many mooring balls available. Check in at customs if arriving from abroad.' },
  { name: 'Luis Peña Channel Anchorage', lat: 18.308000, lng: -65.314000, type: 'anchor', img: null,
    road: 'boat',
    desc: 'Scenic anchorage between Culebra and Cayo Luis Peña. Marine reserve — no anchoring on coral. Use sand patches only.' },
  { name: 'Culebrita Anchorage', lat: 18.317000, lng: -65.232000, type: 'anchor', img: null,
    road: 'boat',
    desc: 'Stunning anchorage off Culebrita island. Exposed to NE swell — best in calm conditions. Worth every wave.' },

  /* ── HIKING ── */
  { name: 'Flamenco Tank Trail', lat: 18.332886, lng: -65.322174, type: 'hike', img: null,
    road: 'sandy',
    desc: 'Short trail directly behind Flamenco beach with WWII tanks situated on the left side of the trail, and another in the water! A stark reminder of the history of our beach. Visit today! Pro tip: Bring beach shoes, as the sand is VERY soft and gets everywhere!!!' },
  { name: 'Monte Resaca Trail', lat: 18.313000, lng: -65.289000, type: 'hike', img: null,
    road: '4x4',
    desc: 'Culebra\'s highest peak at 650ft. Dry tropical forest with iguanas, frigatebirds, and panoramic views. Bring water — no shade.' },
  { name: 'Punta Molino Coastal Path', lat: 18.285000, lng: -65.272000, type: 'hike', img: null,
    road: '4x4',
    desc: 'Remote coastal walk along the SE peninsula. Rugged terrain with incredible views and total solitude. E-bike or 4x4 to trailhead.' }
];

/* ══════════════════════════════════════════════════════════════════
   MAP  — centered above Culebra at zoom 13
══════════════════════════════════════════════════════════════════ */
var map = L.map('map', { zoomControl: false }).setView([18.310, -65.302], 13);

L.tileLayer(
  'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
  { maxZoom: 19, attribution: '&copy; Esri' }
).addTo(map);

L.control.zoom({ position: 'bottomright' }).addTo(map);

/* MARKER CLUSTER GROUP */
var clusterGroup = L.markerClusterGroup({
  disableClusteringAtZoom: 14,
  maxClusterRadius: 60,
  spiderfyOnMaxZoom: true,
  showCoverageOnHover: false,
  iconCreateFunction: function (cluster) {
    var count = cluster.getChildCount();
    return L.divIcon({
      html: '<div style="' +
        'width:38px;height:38px;border-radius:50%;' +
        'background:linear-gradient(135deg,#006994,#00a0b0);' +
        'border:3px solid #fff;' +
        'box-shadow:0 2px 8px rgba(0,0,0,0.45);' +
        'display:flex;align-items:center;justify-content:center;' +
        'color:#fff;font-size:14px;font-weight:700;' +
        '">' + count + '</div>',
      className: '',
      iconSize: [38, 38],
      iconAnchor: [19, 19]
    });
  }
});
map.addLayer(clusterGroup);

/* HOME BUTTON */
var HomeBtn = L.Control.extend({
  options: { position: 'topleft' },
  onAdd: function () {
    var c = L.DomUtil.create('div', 'leaflet-bar leaflet-control leaflet-control-custom');
    c.innerHTML = '<button title="Re-center map">&#127968;</button>';
    L.DomEvent.on(c, 'click', function () { map.setView([18.310, -65.302], 13); });
    L.DomEvent.disableClickPropagation(c);
    return c;
  }
});
map.addControl(new HomeBtn());

/* Build legend list */
document.addEventListener('DOMContentLoaded', function(){ buildLegend(); });
if (document.readyState !== 'loading') { buildLegend(); }

/* ══════════════════════════════════════════════════════════════════
   FULLSCREEN
══════════════════════════════════════════════════════════════════ */
function toggleFullscreen() {
  var btn = document.getElementById('btn-fullscreen');
  if (!document.fullscreenElement && !document.webkitFullscreenElement && !document.mozFullScreenElement) {
    var el = document.documentElement;
    if (el.requestFullscreen)       { el.requestFullscreen(); }
    else if (el.webkitRequestFullscreen) { el.webkitRequestFullscreen(); }
    else if (el.mozRequestFullScreen)    { el.mozRequestFullScreen(); }
    btn.innerHTML = '&#x2715;';
    btn.title = 'Exit fullscreen';
  } else {
    if (document.exitFullscreen)         { document.exitFullscreen(); }
    else if (document.webkitExitFullscreen) { document.webkitExitFullscreen(); }
    else if (document.mozCancelFullScreen)  { document.mozCancelFullScreen(); }
    btn.innerHTML = '&#x26F6;';
    btn.title = 'Toggle fullscreen';
  }
  setTimeout(function () { map.invalidateSize(); }, 200);
}

document.addEventListener('fullscreenchange',       syncFullscreenBtn);
document.addEventListener('webkitfullscreenchange', syncFullscreenBtn);
document.addEventListener('mozfullscreenchange',    syncFullscreenBtn);
function syncFullscreenBtn() {
  var btn = document.getElementById('btn-fullscreen');
  var inFS = !!(document.fullscreenElement || document.webkitFullscreenElement || document.mozFullScreenElement);
  btn.innerHTML = inFS ? '&#x2715;' : '&#x26F6;';
  btn.title     = inFS ? 'Exit fullscreen' : 'Toggle fullscreen';
}

/* ══════════════════════════════════════════════════════════════════
   LEGEND COLLAPSE
══════════════════════════════════════════════════════════════════ */
function toggleLegend() { /* legacy stub */ }

/* ── LEGEND DROPDOWN ── */
var legendCategories = [
  { type: null,        label: 'Show All',      color: '#006994' },
  { type: 'beach',     label: 'Beaches',       color: '#00b4d8' },
  { type: 'food',      label: 'Food & Drink',  color: '#f59e0b' },
  { type: 'ferry',     label: 'Ferry',         color: '#2563eb' },
  { type: 'gas',       label: 'Gas Station',   color: '#ea580c' },
  { type: 'atm',       label: 'ATM',           color: '#6366f1' },
  { type: 'parking',   label: 'Parking',       color: '#64748b' },
  { type: 'restroom',  label: 'Restrooms',     color: '#0891b2' },
  { type: 'rental',    label: 'Rentals',       color: '#d97706' },
  { type: 'villa',     label: 'Villa Rental',  color: '#0891b2' },
  { type: 'watertaxi', label: 'Water Taxi',    color: '#1d4ed8' },
  { type: 'anchor',    label: 'Anchorage',     color: '#475569' },
  { type: 'hike',      label: 'Hiking',        color: '#16a34a' },
  { type: 'grocery',   label: 'Grocery',       color: '#10b981' },
  { type: 'hector',    label: 'Hector',        color: '#7c3aed' }
];

function buildLegend() {
  var ul = document.getElementById('legend-list');
  if (!ul) return;
  ul.innerHTML = '';
  legendCategories.forEach(function(cat) {
    var li = document.createElement('li');
    li.dataset.type = cat.type || '';
    li.innerHTML = '<span class="ld-dot" style="background:' + cat.color + '"></span>' + cat.label;
    li.onclick = function() {
      filterMarkers(cat.type);
      /* Update active state */
      ul.querySelectorAll('li').forEach(function(el){ el.classList.remove('active'); });
      li.classList.add('active');
      /* Close dropdown after selection */
      document.getElementById('legend-drop').classList.remove('open');
    };
    ul.appendChild(li);
  });
  /* Set "Show All" as default active */
  ul.querySelector('li').classList.add('active');
}

function toggleLegendDrop() {
  var drop = document.getElementById('legend-drop');
  drop.classList.toggle('open');
  /* Close when clicking outside */
}

/* Close legend dropdown on map click */
document.addEventListener('click', function(e) {
  var wrap = document.getElementById('legend-wrap');
  var drop = document.getElementById('legend-drop');
  if (drop && wrap && !wrap.contains(e.target)) {
    drop.classList.remove('open');
  }
});

/* ══════════════════════════════════════════════════════════════════
   PIN SVG BUILDER
══════════════════════════════════════════════════════════════════ */
var pinColors = {
  beach:'#00b4d8', ferry:'#2563eb', gas:'#ea580c', hector:'#7c3aed',
  food:'#f59e0b', grocery:'#10b981', atm:'#6366f1', parking:'#64748b',
  restroom:'#0891b2', rental:'#d97706', watertaxi:'#1d4ed8',
  anchor:'#475569', hike:'#16a34a', villa:'#0891b2'
};
var pinEmoji = {
  beach:'&#127958;', ferry:'&#9972;', gas:'&#9981;', hector:'&#129548;',
  food:'&#127860;', grocery:'&#128722;', atm:'&#127967;', parking:'&#128663;',
  restroom:'&#128701;', rental:'&#128676;', watertaxi:'&#128674;',
  anchor:'&#9875;', hike:'&#129510;', villa:'&#127968;'
};
var roadLabels  = { 'paved':'✅ Paved road', '4x4':'🚙 4x4 / E-Bike only', 'boat':'⛵ Boat access only', 'sandy':'⚠️ Sandy trail', 'unpaved':'⚠️ Unpaved road' };
var roadClasses = { 'paved':'road-paved', '4x4':'road-4x4', 'boat':'road-boat', 'sandy':'road-sandy', 'unpaved':'road-sandy' };
var typeNames   = {
  beach:'Beach', ferry:'Ferry Terminal', gas:'Gas Station', hector:'Landmark',
  food:'Restaurant', grocery:'Grocery', atm:'ATM', parking:'Parking',
  restroom:'Restrooms', rental:'Rentals', watertaxi:'Water Taxi',
  anchor:'Anchorage', hike:'Hiking', villa:'Villa Rental'
};

function makePinIcon(type) {
  var color = pinColors[type] || '#555';
  var emoji = pinEmoji[type]  || '&#128205;';
  var svg =
    '<svg xmlns="http://www.w3.org/2000/svg" width="38" height="50" viewBox="0 0 38 50">' +
      '<defs>' +
        '<filter id="sh' + type + '" x="-40%" y="-20%" width="180%" height="160%">' +
          '<feDropShadow dx="0" dy="3" stdDeviation="3" flood-color="rgba(0,0,0,0.55)"/>' +
        '</filter>' +
      '</defs>' +
      '<path d="M19 1C10.16 1 3 8.16 3 17c0 11.5 16 32 16 32S35 28.5 35 17C35 8.16 27.84 1 19 1z"' +
        ' fill="' + color + '" stroke="#ffffff" stroke-width="2.5" filter="url(#sh' + type + ')"/>' +
      '<circle cx="19" cy="17" r="9" fill="rgba(255,255,255,0.22)"/>' +
      '<text x="19" y="22" text-anchor="middle" font-size="13" font-family="serif">' + emoji + '</text>' +
    '</svg>';
  return L.divIcon({
    html: svg, className: '',
    iconSize: [38, 50], iconAnchor: [19, 50], popupAnchor: [0, -52]
  });
}

/* ══════════════════════════════════════════════════════════════════
   STATE
══════════════════════════════════════════════════════════════════ */
var userLat       = null;
var userLng       = null;
var userMarker    = null;
var userAccCircle = null;
var activeRoute   = null;
var watchId       = null;
var activeFilter  = null;
var markerObjects = [];

/* Ferry route polyline layers — built once, added/removed on demand */
var ferryRouteLines = [];

function clearRoute() {
  if (activeRoute) {
    try { map.removeControl(activeRoute); } catch (e) {}
    activeRoute = null;
  }
  document.getElementById('turn-panel').classList.remove('open');
}

/* ══════════════════════════════════════════════════════════════════
   FERRY ROUTE LINE HELPERS
   Lines are shown when a ferry terminal popup opens,
   hidden when that popup closes.
══════════════════════════════════════════════════════════════════ */
(function () {
  var ceiba   = [18.228669, -65.619310];
  var apex    = [18.205370, -65.615147];
  var culebra = [18.301451, -65.302530];

  var ctrlLat = (ceiba[0] + apex[0]) / 2 - 0.004;
  var ctrlLng = (ceiba[1] + apex[1]) / 2 - 0.006;
  var ctrl    = [ctrlLat, ctrlLng];

  function bezier2(p0, p1, p2, steps) {
    var pts = [];
    for (var i = 0; i <= steps; i++) {
      var t  = i / steps;
      var u  = 1 - t;
      pts.push([
        u*u*p0[0] + 2*u*t*p1[0] + t*t*p2[0],
        u*u*p0[1] + 2*u*t*p1[1] + t*t*p2[1]
      ]);
    }
    return pts;
  }

  var seg1 = bezier2(ceiba, ctrl, apex, 50);
  var seg2 = [apex, culebra];

  [seg1, seg2].forEach(function (pts) {
    ferryRouteLines.push(
      L.polyline(pts, { color: '#ffffff', weight: 6, opacity: 0.4, smoothFactor: 1, interactive: false }),
      L.polyline(pts, { color: '#2563eb', weight: 3, opacity: 0.95, dashArray: '10, 8', smoothFactor: 1, interactive: false })
    );
  });
})();

function showFerryRoute() {
  ferryRouteLines.forEach(function (line) {
    if (!map.hasLayer(line)) { line.addTo(map); }
  });
}

function hideFerryRoute() {
  ferryRouteLines.forEach(function (line) {
    if (map.hasLayer(line)) { map.removeLayer(line); }
  });
}

/* ══════════════════════════════════════════════════════════════════
   FILTER LOGIC
══════════════════════════════════════════════════════════════════ */
function filterMarkers(type) {
  /* null = show all */
  if (activeFilter === type || type === null) { resetFilter(); return; }
  activeFilter = type;

  /* Update pill bar */
  document.querySelectorAll('.cat-pill').forEach(function(b){ b.classList.remove('active'); });
  var pill = document.getElementById('pill-' + (type || 'all'));
  if (pill) { pill.classList.add('active'); }

  for (var mi = 0; mi < markerObjects.length; mi++) {
    if (markerObjects[mi].type === type) {
      if (!clusterGroup.hasLayer(markerObjects[mi].marker)) { clusterGroup.addLayer(markerObjects[mi].marker); }
    } else {
      if (clusterGroup.hasLayer(markerObjects[mi].marker)) { clusterGroup.removeLayer(markerObjects[mi].marker); }
    }
  }
  map.closePopup();
}

function resetFilter() {
  activeFilter = null;
  document.querySelectorAll('.cat-pill').forEach(function(b){ b.classList.remove('active'); });
  var allPill = document.getElementById('pill-all');
  if (allPill) { allPill.classList.add('active'); }
  for (var mi = 0; mi < markerObjects.length; mi++) {
    if (!clusterGroup.hasLayer(markerObjects[mi].marker)) { clusterGroup.addLayer(markerObjects[mi].marker); }
  }
}

map.on('click', function () { /* intentionally not resetting filter on map click */ });

/* ══════════════════════════════════════════════════════════════════
   HELPERS
══════════════════════════════════════════════════════════════════ */
function turnIcon(type, modifier) {
  var t = ((type || '') + ' ' + (modifier || '')).toLowerCase();
  if (t.indexOf('depart')      !== -1) return '\uD83D\uDE80';
  if (t.indexOf('arrive')      !== -1) return '\uD83D\uDCCD';
  if (t.indexOf('roundabout')  !== -1) return '\uD83D\uDD04';
  if (t.indexOf('sharp left')  !== -1) return '\u21B0';
  if (t.indexOf('left')        !== -1) return '\u2B05\uFE0F';
  if (t.indexOf('sharp right') !== -1) return '\u21B1';
  if (t.indexOf('right')       !== -1) return '\u27A1\uFE0F';
  if (t.indexOf('straight')    !== -1) return '\u2B06\uFE0F';
  if (t.indexOf('continue')    !== -1) return '\u2B06\uFE0F';
  if (t.indexOf('uturn')       !== -1) return '\u21A9\uFE0F';
  return '\u2B06\uFE0F';
}
function fmtDist(m) {
  if (m == null) return '';
  return m >= 1000 ? (m / 1000).toFixed(1) + ' km' : Math.round(m) + ' m';
}
function fmtTime(s) {
  if (s == null) return '';
  var m = Math.round(s / 60);
  return m < 60 ? m + ' min' : Math.floor(m / 60) + ' h ' + (m % 60) + ' min';
}

/* ══════════════════════════════════════════════════════════════════
   ROUTING
══════════════════════════════════════════════════════════════════ */
function requestRoute(destName, destLat, destLng, statusEl, fallbackEl, btnEl) {
  if (userLat === null) {
    statusEl.textContent = 'Location not available \u2014 please allow location access.';
    statusEl.className = 'route-status error';
    return;
  }

  var snapped = snapToGrid(userLat, userLng);
  clearRoute();

  btnEl.disabled = true;
  btnEl.textContent = 'Finding route\u2026';
  statusEl.textContent = '';
  statusEl.className = 'route-status';
  fallbackEl.classList.remove('visible');

  activeRoute = L.Routing.control({
    waypoints: [
      L.latLng(snapped.lat, snapped.lng),
      L.latLng(destLat, destLng)
    ],
    routeWhileDragging: false,
    show: false,
    addWaypoints: false,
    draggableWaypoints: false,
    createMarker: function () { return null; },
    lineOptions: {
      styles: [
        { color: '#ffffff', weight: 9, opacity: 0.35 },
        { color: '#007AFF', weight: 5, opacity: 1.0 }
      ],
      extendToWaypoints: true,
      missingRouteTolerance: 0
    },
    router: L.Routing.osrmv1({
      serviceUrl: 'https://router.project-osrm.org/route/v1',
      profile: 'driving',
      timeout: 12000
    })
  }).addTo(map);

  activeRoute.on('routesfound', function (e) {
    var route = e.routes[0];
    var dist  = route.summary.totalDistance;
    var time  = route.summary.totalTime;
    var steps = route.instructions || [];

    btnEl.disabled = false;
    btnEl.textContent = '\u2191 View turn-by-turn';
    statusEl.textContent = fmtDist(dist) + ' \u00B7 ' + fmtTime(time);
    statusEl.className = 'route-status';

    document.getElementById('turn-title').textContent = '\u2192 ' + destName;
    document.getElementById('turn-meta').textContent  = fmtDist(dist) + ' \u00B7 ' + fmtTime(time);

    var stepsEl = document.getElementById('turn-steps');
    stepsEl.innerHTML = '';

    for (var si = 0; si < steps.length; si++) {
      var step = steps[si];
      var row  = document.createElement('div');
      row.className = 'turn-step';
      var ico = document.createElement('div');
      ico.className = 'turn-icon';
      ico.textContent = turnIcon(step.type, step.modifier);
      var txt = document.createElement('div');
      txt.innerHTML = '<div>' + (step.text || '') + '</div>' +
        (step.distance > 0 ? '<div class="turn-dist">' + fmtDist(step.distance) + '</div>' : '');
      row.appendChild(ico);
      row.appendChild(txt);
      stepsEl.appendChild(row);
    }

    document.getElementById('turn-panel').classList.add('open');
    map.closePopup();
    map.fitBounds(route.bounds, { padding: [30, 30] });
  });

  activeRoute.on('routingerror', function () {
    try { map.removeControl(activeRoute); } catch (e) {}
    activeRoute = null;
    btnEl.disabled = false;
    btnEl.textContent = 'Get Directions';
    statusEl.textContent = '\u26A0\uFE0F No in-map route found. Try external maps:';
    statusEl.className = 'route-status error';
    fallbackEl.querySelector('.btn-apple').href =
      'maps://maps.apple.com/?saddr=' + userLat + ',' + userLng +
      '&daddr=' + destLat + ',' + destLng + '&dirflg=d';
    fallbackEl.querySelector('.btn-google').href =
      'https://www.google.com/maps/dir/?api=1&origin=' + userLat + ',' + userLng +
      '&destination=' + destLat + ',' + destLng + '&travelmode=driving';
    fallbackEl.classList.add('visible');
  });
}

/* ══════════════════════════════════════════════════════════════════
   MARKERS
══════════════════════════════════════════════════════════════════ */
for (var mi = 0; mi < locations.length; mi++) {
  (function (p) {
    var icon = makePinIcon(p.type);

    var imgHtml = p.img
      ? '<img class="popup-img" src="' + p.img + '" onerror="this.style.display=\'none\'" alt="">'
      : '';

    var typeColor = pinColors[p.type] || '#555';
    var typeName  = typeNames[p.type]  || p.type;
    var typeTag   = '<span class="popup-type-tag" style="background:' + typeColor + '22;color:' + typeColor + '">' + typeName + '</span>';
    var roadTag   = p.road ? '<span class="popup-road-tag ' + (roadClasses[p.road]||'') + '">' + (roadLabels[p.road]||p.road) + '</span>' : '';
    var hoursHtml = p.hours ? '<div class="popup-hours"><span class="open-dot">●</span> ' + p.hours + '</div>' : '';

    var popupHtml =
      imgHtml +
      '<div class="popup-body">' +
        '<div class="popup-title">' + p.name + '</div>' +
        '<div class="popup-type-row">' + typeTag + roadTag + '</div>' +
        hoursHtml +
        '<div class="dist-badge" id="dist-' + mi + '">📍 calculating…</div>' +
        '<div class="popup-desc">'  + p.desc  + '</div>' +
        '<button class="btn-directions">Get Directions</button>' +
        '<div class="route-status"></div>' +
        '<div class="fallback-row">' +
          '<a class="btn-fallback btn-apple"  href="#" target="_blank">&#127822; Apple Maps</a>' +
          '<a class="btn-fallback btn-google" href="#" target="_blank">&#128506;&#65039; Google Maps</a>' +
        '</div>' +
        '<button class="btn-share" data-lat="' + p.lat + '" data-lng="' + p.lng + '" data-name="' + p.name.replace(/"/g, '&quot;') + '">🔗 Share Location</button>' +
      '</div>';

    var marker = L.marker([p.lat, p.lng], { icon: icon })
      .bindPopup(popupHtml, { maxWidth: 310, minWidth: 250 });

    markerObjects.push({ marker: marker, type: p.type });

    marker.on('popupopen', function (e) {
      /* Show ferry route line when either ferry terminal is tapped */
      if (p.type === 'ferry') { showFerryRoute(); }

      /* Hide cat bar while popup is open */
      document.getElementById('cat-bar').classList.add('hidden');

      var el         = e.popup.getElement();
      var btn        = el.querySelector('.btn-directions');
      var statusEl   = el.querySelector('.route-status');
      var fallbackEl = el.querySelector('.fallback-row');
      var distEl     = el.querySelector('.dist-badge');
      var shareBtn   = el.querySelector('.btn-share');

      statusEl.textContent = '';
      statusEl.className = 'route-status';
      fallbackEl.classList.remove('visible');

      /* Distance from user */
      if (distEl) {
        if (userLat !== null) {
          var d = haversine(userLat, userLng, p.lat, p.lng);
          distEl.textContent = '📍 ' + fmtDist(d) + ' away';
        } else {
          distEl.textContent = '📍 Enable location for distance';
        }
      }

      /* Share button */
      if (shareBtn) {
        shareBtn.onclick = function() { shareLocation(p.name, p.lat, p.lng, shareBtn); };
      }

      var freshBtn = btn.cloneNode(true);
      freshBtn.disabled = false;
      freshBtn.textContent = 'Get Directions';
      btn.parentNode.replaceChild(freshBtn, btn);

      freshBtn.addEventListener('click', function () {
        requestRoute(p.name, p.lat, p.lng, statusEl, fallbackEl, freshBtn);
      });
    });

    marker.on('popupclose', function () {
      /* Hide ferry route line when the ferry terminal popup is closed */
      if (p.type === 'ferry') { hideFerryRoute(); }
      /* Show cat bar again */
      document.getElementById('cat-bar').classList.remove('hidden');
    });

    clusterGroup.addLayer(marker);
  })(locations[mi]);
}

/* ══════════════════════════════════════════════════════════════════
   SEARCH (upgraded)
══════════════════════════════════════════════════════════════════ */
function liveSearch(val) {
  var resultsEl = document.getElementById('search-results');
  var clearBtn  = document.getElementById('search-clear');
  val = val.trim().toLowerCase();
  if (clearBtn) { clearBtn.classList.toggle('visible', val.length > 0); }
  if (!val) { resultsEl.style.display = 'none'; resultsEl.innerHTML = ''; return; }

  var matches = [];
  for (var si = 0; si < markerObjects.length; si++) {
    var loc = locations[si];
    if (loc.name.toLowerCase().indexOf(val) !== -1 ||
        (loc.type && loc.type.indexOf(val) !== -1) ||
        ((typeNames[loc.type]||'').toLowerCase().indexOf(val) !== -1)) {
      matches.push({ idx: si, name: loc.name, lat: loc.lat, lng: loc.lng, type: loc.type });
    }
  }

  if (matches.length === 0) {
    resultsEl.innerHTML = '<div class="search-result-item" style="color:#999;font-size:13px;padding:14px 16px;">No results found</div>';
  } else {
    resultsEl.innerHTML = '';
    var limit = Math.min(matches.length, 8);
    for (var mi = 0; mi < limit; mi++) {
      (function(m) {
        var div = document.createElement('div');
        div.className = 'search-result-item';
        var emoji = pinEmoji[m.type] ? m.type : 'beach';
        var emojiChar = {'beach':'🏖️','ferry':'⛴️','gas':'⛽','hector':'🧌','food':'🍽️',
          'grocery':'🛒','atm':'🏧','parking':'🅿️','restroom':'🚻','rental':'🚤',
          'watertaxi':'🚢','anchor':'⚓','hike':'🥾'}[m.type] || '📍';
        div.innerHTML = '<span class="sri-icon">' + emojiChar + '</span>' +
          '<span class="sri-text"><div class="sri-name">' + m.name + '</div>' +
          '<div class="sri-type">' + (typeNames[m.type]||m.type) + '</div></span>';
        div.onclick = function () {
          map.setView([m.lat, m.lng], 15);
          /* Make sure marker is visible even if filtered */
          if (!clusterGroup.hasLayer(markerObjects[m.idx].marker)) {
            clusterGroup.addLayer(markerObjects[m.idx].marker);
          }
          markerObjects[m.idx].marker.openPopup();
          document.getElementById('search-results').style.display = 'none';
          document.getElementById('search-input').value = m.name;
        };
        resultsEl.appendChild(div);
      })(matches[mi]);
    }
  }
  resultsEl.style.display = 'block';
}

function doSearch() {
  var val = document.getElementById('search-input').value.trim().toLowerCase();
  if (!val) return;
  for (var si = 0; si < locations.length; si++) {
    if (locations[si].name.toLowerCase().indexOf(val) !== -1) {
      map.setView([locations[si].lat, locations[si].lng], 15);
      if (!clusterGroup.hasLayer(markerObjects[si].marker)) { clusterGroup.addLayer(markerObjects[si].marker); }
      markerObjects[si].marker.openPopup();
      document.getElementById('search-results').style.display = 'none';
      return;
    }
  }
}

function clearSearch() {
  document.getElementById('search-input').value = '';
  document.getElementById('search-results').style.display = 'none';
  document.getElementById('search-results').innerHTML = '';
  var cb = document.getElementById('search-clear');
  if (cb) { cb.classList.remove('visible'); }
}

/* Close search when clicking outside */
document.addEventListener('click', function(e) {
  var tb = document.getElementById('topbar');
  if (tb && !tb.contains(e.target)) {
    document.getElementById('search-results').style.display = 'none';
  }
});

/* ══════════════════════════════════════════════════════════════════
   LOCATE ME (fly to user position)
══════════════════════════════════════════════════════════════════ */
function locateMe() {
  var btn = document.getElementById('btn-locate');
  if (userLat !== null) {
    map.flyTo([userLat, userLng], 16, { animate: true, duration: 1.2 });
    btn.classList.add('located');
    btn.textContent = '📍';
    return;
  }
  btn.classList.add('locating');
  btn.textContent = '⏳';
  navigator.geolocation && navigator.geolocation.getCurrentPosition(function(pos) {
    userLat = pos.coords.latitude;
    userLng = pos.coords.longitude;
    map.flyTo([userLat, userLng], 16, { animate: true, duration: 1.2 });
    btn.classList.remove('locating');
    btn.classList.add('located');
    btn.textContent = '📍';
  }, function() {
    btn.classList.remove('locating');
    btn.textContent = '📍';
    showToast('Location access denied');
  });
}

/* ══════════════════════════════════════════════════════════════════
   HAVERSINE DISTANCE
══════════════════════════════════════════════════════════════════ */
function haversine(lat1, lng1, lat2, lng2) {
  var R = 6371000;
  var dLat = (lat2 - lat1) * Math.PI / 180;
  var dLng = (lng2 - lng1) * Math.PI / 180;
  var a = Math.sin(dLat/2) * Math.sin(dLat/2) +
          Math.cos(lat1 * Math.PI/180) * Math.cos(lat2 * Math.PI/180) *
          Math.sin(dLng/2) * Math.sin(dLng/2);
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
}

/* ══════════════════════════════════════════════════════════════════
   SHARE LOCATION
══════════════════════════════════════════════════════════════════ */
function shareLocation(name, lat, lng, btn) {
  var url = 'https://www.google.com/maps/search/?api=1&query=' + lat + ',' + lng;
  var text = name + ' — ' + url;
  if (navigator.share) {
    navigator.share({ title: name, text: 'Check out ' + name + ' on Culebra!', url: url })
      .catch(function(){});
  } else if (navigator.clipboard) {
    navigator.clipboard.writeText(text).then(function() {
      btn.textContent = '✅ Copied!';
      btn.classList.add('copied');
      setTimeout(function(){ btn.textContent = '🔗 Share Location'; btn.classList.remove('copied'); }, 2000);
    });
  }
}

/* ══════════════════════════════════════════════════════════════════
   TOAST NOTIFICATION
══════════════════════════════════════════════════════════════════ */
function showToast(msg, duration) {
  var t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(function(){ t.classList.remove('show'); }, duration || 2500);
}

/* ══════════════════════════════════════════════════════════════════
   GEOLOCATION
══════════════════════════════════════════════════════════════════ */
if ('geolocation' in navigator) {
  watchId = navigator.geolocation.watchPosition(
    function (pos) {
      userLat = pos.coords.latitude;
      userLng = pos.coords.longitude;
      var ll  = [userLat, userLng];

      if (!userMarker) {
        userMarker = L.marker(ll, {
          icon: L.divIcon({ className: 'user-dot', iconSize: [14, 14], iconAnchor: [7, 7] }),
          zIndexOffset: 1000, interactive: false
        }).addTo(map);
      } else {
        userMarker.setLatLng(ll);
      }

      var acc = pos.coords.accuracy;
      if (!userAccCircle) {
        userAccCircle = L.circle(ll, {
          radius: acc, color: '#2f80ff', fillColor: '#2f80ff',
          fillOpacity: 0.07, weight: 1, opacity: 0.35, interactive: false
        }).addTo(map);
      } else {
        userAccCircle.setLatLng(ll).setRadius(acc);
      }
    },
    function (err) { console.warn('Geolocation:', err.message); },
    { enableHighAccuracy: true, maximumAge: 4000, timeout: 10000 }
  );
}

/* RESIZE */
if (typeof ResizeObserver !== 'undefined') {
  new ResizeObserver(function () { map.invalidateSize(); }).observe(document.getElementById('map'));
} else {
  setTimeout(function () { map.invalidateSize(); }, 300);
}

/* ================================================================
   EASTER EGG 1: SHARK CURSOR (existing — desktop only)
   Ctrl+Shift+H then Ctrl+Shift+L = shark cursor.
   Backspace to deactivate.
================================================================ */
(function () {
  var sharkSVG =
    '<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48">' +
    '<ellipse cx="22" cy="26" rx="18" ry="9" fill="#4a9aba" opacity="0.95"/>' +
    '<polygon points="4,26 0,16 8,26 0,36" fill="#357a96"/>' +
    '<polygon points="20,17 26,8 30,17" fill="#357a96"/>' +
    '<ellipse cx="24" cy="29" rx="12" ry="5" fill="#d0eaf5" opacity="0.8"/>' +
    '<circle cx="35" cy="24" r="2.2" fill="#1a1a2e"/>' +
    '<circle cx="35.8" cy="23.2" r="0.7" fill="#fff"/>' +
    '<path d="M37,27 Q40,30 37,31" stroke="#1a1a2e" stroke-width="1.2" fill="none" stroke-linecap="round"/>' +
    '<line x1="37.5" y1="27.5" x2="37.5" y2="29.5" stroke="#fff" stroke-width="1"/>' +
    '<line x1="39" y1="28.5" x2="38.5" y2="30.5" stroke="#fff" stroke-width="1"/>' +
    '</svg>';

  var sharkCursor = 'url("data:image/svg+xml,' + encodeURIComponent(sharkSVG) + '") 8 24, auto';
  var sharkActive = false;
  var hPressed    = false;

  function activateShark() {
    sharkActive = true;
    document.body.style.cursor = sharkCursor;
    var mapEl = document.getElementById('map');
    if (mapEl) { mapEl.style.cursor = sharkCursor; }
  }

  function deactivateShark() {
    sharkActive = false;
    document.body.style.cursor = '';
    var mapEl = document.getElementById('map');
    if (mapEl) { mapEl.style.cursor = ''; }
  }

  document.addEventListener('keydown', function (e) {
    var ctrl  = e.ctrlKey;
    var shift = e.shiftKey;

    if (e.key === 'Backspace' && sharkActive) {
      deactivateShark();
      return;
    }

    if (ctrl && shift && (e.key === 'H' || e.key === 'h')) {
      hPressed = true;
      return;
    }

    if (ctrl && shift && (e.key === 'L' || e.key === 'l') && hPressed) {
      hPressed = false;
      activateShark();
      return;
    }

    if (!ctrl || !shift) { hPressed = false; }
  });

  document.addEventListener('keyup', function (e) {
    if (e.key === 'Control' || e.key === 'Shift') { hPressed = false; }
  });
})();

/* ================================================================
   EASTER EGG 2: OCEAN DEPTHS OVERLAY
   Mobile: Tap screen 5 times fast (within 2 seconds)
   Desktop: Type the word "ocean" anywhere (no input focused)
   Shows animated bioluminescent fish swimming in the deep.
   Close with the "Return to Surface" button.
================================================================ */
(function () {
  var tapCount  = 0;
  var tapTimer  = null;
  var typedKeys = '';

  /* ── Canvas fish animation ── */
  function startOceanAnimation() {
    var canvas = document.getElementById('ee-canvas');
    var ctx    = canvas.getContext('2d');
    var fishes = [];
    var bubbles= [];
    var raf;

    function resize() {
      canvas.width  = window.innerWidth;
      canvas.height = window.innerHeight;
    }
    resize();
    window.addEventListener('resize', resize);

    /* Spawn fish */
    for (var i = 0; i < 18; i++) {
      fishes.push({
        x:     Math.random() * canvas.width,
        y:     Math.random() * canvas.height,
        vx:    (Math.random() - 0.5) * 1.4,
        vy:    (Math.random() - 0.5) * 0.6,
        size:  8 + Math.random() * 20,
        hue:   160 + Math.random() * 80,
        glow:  0.3 + Math.random() * 0.7,
        phase: Math.random() * Math.PI * 2
      });
    }
    /* Spawn bubbles */
    for (var j = 0; j < 30; j++) {
      bubbles.push({
        x:    Math.random() * canvas.width,
        y:    canvas.height + Math.random() * 200,
        r:    2 + Math.random() * 6,
        vy:   0.4 + Math.random() * 0.8,
        alpha:0.2 + Math.random() * 0.5
      });
    }

    function drawFish(f, t) {
      ctx.save();
      ctx.translate(f.x, f.y);
      if (f.vx < 0) { ctx.scale(-1, 1); }
      var wobble = Math.sin(t * 0.002 + f.phase) * 3;
      ctx.rotate(wobble * 0.05);
      var g = Math.sin(t * 0.003 + f.phase) * 0.4 + 0.6;
      ctx.shadowColor = 'hsl(' + f.hue + ',100%,60%)';
      ctx.shadowBlur  = 18;
      ctx.fillStyle   = 'hsla(' + f.hue + ',90%,65%,' + (f.glow * g) + ')';
      /* Body */
      ctx.beginPath();
      ctx.ellipse(0, 0, f.size, f.size * 0.45, 0, 0, Math.PI * 2);
      ctx.fill();
      /* Tail */
      ctx.beginPath();
      ctx.moveTo(-f.size, 0);
      ctx.lineTo(-f.size - f.size * 0.7, -f.size * 0.4);
      ctx.lineTo(-f.size - f.size * 0.7,  f.size * 0.4);
      ctx.closePath();
      ctx.fill();
      /* Eye */
      ctx.fillStyle = 'rgba(255,255,255,0.9)';
      ctx.beginPath();
      ctx.arc(f.size * 0.55, -f.size * 0.1, f.size * 0.12, 0, Math.PI * 2);
      ctx.fill();
      ctx.restore();
    }

    function animate(t) {
      raf = requestAnimationFrame(animate);
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      /* Stars/plankton background */
      ctx.fillStyle = 'rgba(0,180,255,0.03)';
      for (var si = 0; si < 80; si++) {
        var sx = (si * 137.5) % canvas.width;
        var sy = (si * 89.3 + t * 0.01) % canvas.height;
        ctx.beginPath();
        ctx.arc(sx, sy, 1, 0, Math.PI * 2);
        ctx.fill();
      }

      /* Fish */
      for (var fi = 0; fi < fishes.length; fi++) {
        var f = fishes[fi];
        f.x += f.vx;
        f.y += f.vy + Math.sin(t * 0.001 + f.phase) * 0.3;
        if (f.x > canvas.width  + 60) { f.x = -60; }
        if (f.x < -60)                { f.x = canvas.width + 60; }
        if (f.y > canvas.height + 40) { f.y = -40; }
        if (f.y < -40)                { f.y = canvas.height + 40; }
        drawFish(f, t);
      }

      /* Bubbles */
      for (var bi = 0; bi < bubbles.length; bi++) {
        var b = bubbles[bi];
        b.y -= b.vy;
        b.x += Math.sin(t * 0.002 + bi) * 0.4;
        if (b.y < -20) { b.y = canvas.height + 20; b.x = Math.random() * canvas.width; }
        ctx.strokeStyle = 'rgba(100,220,255,' + b.alpha + ')';
        ctx.lineWidth   = 1.2;
        ctx.beginPath();
        ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2);
        ctx.stroke();
      }
    }
    raf = requestAnimationFrame(animate);
    canvas._stopAnim = function() { cancelAnimationFrame(raf); window.removeEventListener('resize', resize); };
  }

  window.openOceanEE = function() {
    var overlay = document.getElementById('ee-overlay');
    overlay.classList.add('show');
    startOceanAnimation();
    showToast('🌊 You found the ocean depths!');
  };

  window.closeOceanEE = function() {
    var overlay = document.getElementById('ee-overlay');
    overlay.classList.remove('show');
    var canvas = document.getElementById('ee-canvas');
    if (canvas && canvas._stopAnim) { canvas._stopAnim(); }
  };

  /* Mobile: 5 fast taps */
  document.addEventListener('touchstart', function(e) {
    /* Don't count taps on interactive elements */
    if (e.target.closest('button,a,input,.cat-pill,.leaflet-container')) return;
    tapCount++;
    if (tapTimer) { clearTimeout(tapTimer); }
    tapTimer = setTimeout(function() { tapCount = 0; }, 2000);
    if (tapCount >= 5) {
      tapCount = 0;
      clearTimeout(tapTimer);
      openOceanEE();
      /* Ripple effect at tap location */
      var touch = e.touches[0];
      spawnRipple(touch.clientX, touch.clientY);
    }
  }, { passive: true });

  /* Desktop: type "ocean" (no input focused) */
  document.addEventListener('keypress', function(e) {
    if (document.activeElement && (document.activeElement.tagName === 'INPUT' || document.activeElement.tagName === 'TEXTAREA')) return;
    typedKeys = (typedKeys + e.key).slice(-5).toLowerCase();
    if (typedKeys === 'ocean') { openOceanEE(); typedKeys = ''; }
  });
})();

/* ================================================================
   EASTER EGG 3: TURTLE PARADE (mobile + desktop)
   Mobile: Long-press (1.5s) on the map background
   Desktop: Type "turtle"
   Spawns animated CSS turtles swimming across the screen.
================================================================ */
(function () {
  var ltTimer   = null;
  var turtleTyped = '';
  var turtlesActive = false;
  var turtleEls = [];

  function spawnTurtleParade() {
    if (turtlesActive) { clearTurtles(); return; }
    turtlesActive = true;
    showToast('🐢 Turtle parade! Tap anywhere to stop.');

    var count = 8;
    for (var i = 0; i < count; i++) {
      (function(idx) {
        setTimeout(function() {
          var el = document.createElement('div');
          el.style.cssText = [
            'position:fixed',
            'z-index:9990',
            'font-size:' + (28 + Math.random() * 20) + 'px',
            'top:' + (10 + Math.random() * 80) + 'vh',
            'left:-60px',
            'pointer-events:none',
            'transition:left 6s linear',
            'filter:drop-shadow(0 2px 6px rgba(0,0,0,0.3))'
          ].join(';');
          el.textContent = '🐢';
          document.body.appendChild(el);
          turtleEls.push(el);
          setTimeout(function() { el.style.left = (window.innerWidth + 80) + 'px'; }, 50);
          setTimeout(function() {
            if (el.parentNode) { el.parentNode.removeChild(el); }
            turtleEls = turtleEls.filter(function(t){ return t !== el; });
            if (turtleEls.length === 0) { turtlesActive = false; }
          }, 6500);
        }, idx * 600);
      })(i);
    }
  }

  function clearTurtles() {
    turtleEls.forEach(function(el) { if (el.parentNode) el.parentNode.removeChild(el); });
    turtleEls = [];
    turtlesActive = false;
  }

  /* Mobile long-press on map */
  var mapEl = document.getElementById('map');
  mapEl.addEventListener('touchstart', function(e) {
    ltTimer = setTimeout(function() { spawnTurtleParade(); }, 1500);
  }, { passive: true });
  mapEl.addEventListener('touchend',   function() { clearTimeout(ltTimer); }, { passive: true });
  mapEl.addEventListener('touchmove',  function() { clearTimeout(ltTimer); }, { passive: true });

  /* Desktop: type "turtle" */
  document.addEventListener('keypress', function(e) {
    if (document.activeElement && (document.activeElement.tagName === 'INPUT' || document.activeElement.tagName === 'TEXTAREA')) return;
    turtleTyped = (turtleTyped + e.key).slice(-6).toLowerCase();
    if (turtleTyped === 'turtle') { spawnTurtleParade(); turtleTyped = ''; }
  });

  /* Tap anywhere to dismiss */
  document.addEventListener('click', function(e) {
    if (turtlesActive && !e.target.closest('button,a,input,.leaflet-control')) {
      clearTurtles();
    }
  });
})();

/* ================================================================
   EASTER EGG 4: PARTY MODE (mobile + desktop)
   Mobile: Shake the device 3 times (uses devicemotion)
   Desktop: Type "fiesta"
   Launches a confetti-style emoji rain celebration.
================================================================ */
(function () {
  var partyTyped    = '';
  var partyActive   = false;
  var lastShake     = 0;
  var shakeCount    = 0;
  var lastAcc       = null;

  var partyEmojis = ['🎉','🌊','🐠','🦀','🌴','☀️','🎶','🍹','🦈','🐢','⚓','🐚','🦭'];

  function launchParty() {
    if (partyActive) return;
    partyActive = true;
    showToast('🎉 ¡Fiesta en Culebra!', 3000);

    var launched = 0;
    var interval = setInterval(function() {
      if (launched >= 60) { clearInterval(interval); setTimeout(function(){ partyActive = false; }, 500); return; }
      var el = document.createElement('div');
      var emoji = partyEmojis[Math.floor(Math.random() * partyEmojis.length)];
      var startX = Math.random() * window.innerWidth;
      var size   = 20 + Math.random() * 24;
      var dur    = 2.5 + Math.random() * 2;
      el.style.cssText = [
        'position:fixed',
        'z-index:9995',
        'left:' + startX + 'px',
        'top:-40px',
        'font-size:' + size + 'px',
        'pointer-events:none',
        'animation:confettiFall ' + dur + 's ease-in forwards'
      ].join(';');
      el.textContent = emoji;
      document.body.appendChild(el);
      setTimeout(function(){ if (el.parentNode) el.parentNode.removeChild(el); }, dur * 1000 + 100);
      launched++;
    }, 80);
  }

  /* Inject confetti keyframe once */
  var styleEl = document.createElement('style');
  styleEl.textContent = '@keyframes confettiFall { to { top:110vh; transform:rotate(' + (Math.random()*720-360) + 'deg) translateX(' + (Math.random()*200-100) + 'px); opacity:0; } }';
  document.head.appendChild(styleEl);

  /* Device shake detection */
  if (window.DeviceMotionEvent) {
    window.addEventListener('devicemotion', function(e) {
      var acc = e.accelerationIncludingGravity;
      if (!acc) return;
      if (lastAcc) {
        var delta = Math.abs(acc.x - lastAcc.x) + Math.abs(acc.y - lastAcc.y) + Math.abs(acc.z - lastAcc.z);
        var now = Date.now();
        if (delta > 25 && now - lastShake > 400) {
          lastShake = now;
          shakeCount++;
          if (shakeCount >= 3) { shakeCount = 0; launchParty(); }
        }
        if (now - lastShake > 2000) { shakeCount = 0; }
      }
      lastAcc = { x: acc.x, y: acc.y, z: acc.z };
    }, { passive: true });
  }

  /* Desktop: type "fiesta" */
  document.addEventListener('keypress', function(e) {
    if (document.activeElement && (document.activeElement.tagName === 'INPUT' || document.activeElement.tagName === 'TEXTAREA')) return;
    partyTyped = (partyTyped + e.key).slice(-6).toLowerCase();
    if (partyTyped === 'fiesta') { launchParty(); partyTyped = ''; }
  });
})();

/* ================================================================
   TAP RIPPLE (mobile UX delight — not an easter egg, just polish)
================================================================ */
function spawnRipple(x, y) {
  var el = document.createElement('div');
  el.className = 'tap-ripple';
  el.style.cssText = 'left:' + (x - 30) + 'px;top:' + (y - 30) + 'px;width:60px;height:60px;';
  document.body.appendChild(el);
  setTimeout(function(){ if (el.parentNode) el.parentNode.removeChild(el); }, 700);
}
document.addEventListener('touchstart', function(e) {
  if (e.touches.length === 1) {
    spawnRipple(e.touches[0].clientX, e.touches[0].clientY);
  }
}, { passive: true });
</script>
</body>
</html>
