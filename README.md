<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, minimum-scale=1.0, viewport-fit=cover">
    <title>Open Cape Pack 3D 智能堆叠优化器 | 工业级引擎</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; user-select: none; }
        body {
            font-family: 'Inter', 'Segoe UI', 'Roboto', system-ui, sans-serif;
            background: #0a0c15; color: #eef2ff; overflow: hidden; height: 100vh;
            -webkit-tap-highlight-color: transparent;
        }
        .app { display: flex; width: 100%; height: 100%; }
        .controls {
            width: 480px; min-width: 480px; background: linear-gradient(145deg, #10141f 0%, #0b0f18 100%);
            border-right: 1px solid rgba(59,130,246,0.25); padding: 20px 18px; overflow-y: auto; z-index: 20;
            box-shadow: 4px 0 24px rgba(0,0,0,0.5); transition: all 0.2s; -webkit-overflow-scrolling: touch;
        }
        .controls::-webkit-scrollbar { width: 5px; }
        .controls::-webkit-scrollbar-track { background: #1e293b; border-radius: 8px; }
        .controls::-webkit-scrollbar-thumb { background: #3b82f6; border-radius: 8px; }
        .viewer { flex: 1; position: relative; background: #f8fafc; min-width: 0; }
        #canvas3d { width: 100%; height: 100%; display: block; outline: none; }
        .info-panel {
            position: absolute; top: 20px; right: 20px; background: rgba(255,255,255,0.96); backdrop-filter: blur(10px);
            border-radius: 24px; padding: 12px 18px; border-left: 4px solid #3b82f6; font-size: 0.8rem;
            font-weight: 500; z-index: 25; pointer-events: none; box-shadow: 0 12px 28px rgba(0,0,0,0.12);
            color: #0f172a; font-family: monospace; transition: all 0.3s;
        }
        .info-panel div { margin: 4px 0; }
        .info-panel .label { color: #475569; font-size: 0.65rem; font-weight: 600; text-transform: uppercase; }
        .info-panel .value { font-size: 1rem; font-weight: 800; color: #2563eb; }
        .info-panel.container-mode { border-left-color: #f59e0b; }
        .info-panel.container-mode .value { color: #d97706; }
        .export-toolbar {
            position: absolute; bottom: 16px; right: 16px; display: flex; gap: 4px;
            background: rgba(15,23,42,0.88); backdrop-filter: blur(12px); padding: 3px 5px;
            border-radius: 20px; border: 1px solid rgba(59,130,246,0.4); z-index: 30;
            box-shadow: 0 3px 8px rgba(0,0,0,0.2); flex-wrap: wrap; justify-content: center;
        }
        .export-btn {
            background: rgba(255,255,255,0.12); border: none; padding: 2px 5px; border-radius: 16px;
            color: #eef2ff; font-size: 0.55rem; font-weight: 600; cursor: pointer; transition: all 0.2s;
            display: flex; align-items: center; gap: 2px; backdrop-filter: blur(4px); font-family: inherit; white-space: nowrap;
        }
        .export-btn:hover { background: #3b82f6; transform: translateY(-1px); box-shadow: 0 2px 6px rgba(59,130,246,0.4); }
        .export-btn:active { transform: translateY(0); }
        .view-switch-btn {
            background: rgba(255,255,255,0.08); border: 1px solid rgba(255,255,255,0.2); padding: 2px 5px;
            border-radius: 16px; color: #eef2ff; font-size: 0.55rem; font-weight: 600; cursor: pointer;
            transition: all 0.2s; font-family: inherit; display: flex; align-items: center; gap: 2px;
            backdrop-filter: blur(4px);
        }
        .view-switch-btn:hover { background: rgba(245,158,11,0.7); border-color: #fbbf24; color: #fff; }
        .view-switch-btn.pallet-active { background: #2563eb; border-color: #3b82f6; color: #fff; }
        .view-switch-btn.container-active { background: #d97706; border-color: #f59e0b; color: #fff; }

        @media (max-width: 780px) {
            .controls { width: 360px; min-width: 360px; padding: 14px; }
            .export-toolbar { padding: 2px 4px; gap: 3px; right: 6px; bottom: 6px; border-radius: 16px; }
            .export-btn { padding: 2px 4px; font-size: 0.5rem; gap: 1px; border-radius: 12px; }
            .view-switch-btn { padding: 2px 4px; font-size: 0.5rem; gap: 1px; border-radius: 12px; }
            .info-panel { top: 12px; right: 12px; padding: 10px 14px; font-size: 0.7rem; }
        }
        @media (max-width: 600px) {
            body { overflow: hidden; }
            .app { flex-direction: column; }
            .controls { width: 100%; min-width: 100%; height: 44vh; padding: 8px 10px; border-right: none; border-bottom: 1px solid rgba(59,130,246,0.25); box-shadow: none; overflow-y: auto; }
            .viewer { flex: 1; min-height: 0; min-width: 0; }
            .info-panel { top: 6px; right: 6px; left: 6px; padding: 6px 10px; font-size: 0.65rem; border-radius: 16px; }
            .export-toolbar { left: 4px; right: 4px; bottom: 10px; justify-content: space-around; padding: 2px 3px; gap: 2px; border-radius: 14px; }
            .export-btn { flex: 1; justify-content: center; padding: 3px 2px; font-size: 0.5rem; border-radius: 10px; }
            .view-switch-btn { flex: 1; justify-content: center; padding: 3px 2px; font-size: 0.5rem; border-radius: 10px; }
        }

        .card {
            background: rgba(18,25,45,0.55); backdrop-filter: blur(8px); border-radius: 24px; padding: 15px;
            margin-bottom: 16px; border: 1px solid rgba(59,130,246,0.2); transition: all 0.2s;
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        }
        .card:hover { border-color: rgba(59,130,246,0.4); background: rgba(18,25,45,0.7); }
        .card.card-container { border-color: rgba(245,158,11,0.35); }
        .card.card-container:hover { border-color: rgba(245,158,11,0.6); }
        .card-title {
            font-size: 0.95rem; font-weight: 600; margin-bottom: 12px; display: flex; align-items: center;
            gap: 8px; border-left: 3px solid #3b82f6; padding-left: 10px; color: #f1f5f9; letter-spacing: 0.3px;
        }
        .card-container .card-title { border-left-color: #f59e0b; }
        .input-row { display: flex; gap: 10px; flex-wrap: wrap; margin-bottom: 10px; }
        .input-field { flex: 1; min-width: 60px; }
        label { font-size: 0.68rem; font-weight: 600; display: block; margin-bottom: 5px; color: #9ca3af; letter-spacing: 0.3px; }
        input, select {
            width: 100%; padding: 9px 12px; background: #0a0f1a; border: 1px solid #2a364c;
            border-radius: 18px; color: #f1f5f9; font-size: 0.82rem; font-weight: 500; transition: 0.2s;
        }
        input:focus, select:focus { border-color: #3b82f6; box-shadow: 0 0 0 2px rgba(59,130,246,0.2); outline: none; background: #0f1622; }
        .preset-btns { display: flex; flex-wrap: wrap; gap: 6px; margin-bottom: 10px; }
        .preset-btn {
            flex: 1; min-width: 60px; padding: 8px 6px; border-radius: 20px; border: 1px solid #4a5568;
            background: #1a2235; color: #cbd5e1; font-size: 0.7rem; font-weight: 600; cursor: pointer;
            transition: all 0.2s; font-family: inherit; text-align: center; white-space: nowrap;
        }
        .preset-btn:hover { background: #2d3a5e; border-color: #f59e0b; color: #fbbf24; }
        .preset-btn.active-preset { background: #d97706; border-color: #f59e0b; color: #fff; box-shadow: 0 0 10px rgba(245,158,11,0.4); }
        .radio-group { display: flex; flex-wrap: nowrap; gap: 6px; margin: 10px 0 6px; justify-content: space-between; }
        .radio-item {
            display: flex; align-items: center; gap: 3px; font-size: 0.68rem; color: #cbd5e1;
            background: #0a0f1a; padding: 4px 7px; border-radius: 28px; border: 1px solid #2a364c;
            transition: 0.2s; cursor: pointer; flex: 1; justify-content: center; white-space: nowrap;
        }
        .radio-item:hover { background: #1e2a3a; border-color: #3b82f6; }
        input[type="radio"] { width: auto; margin-right: 3px; accent-color: #3b82f6; }
        button {
            background: linear-gradient(105deg, #2563eb, #3b82f6); color: white; border: none; padding: 11px;
            width: 100%; border-radius: 36px; font-weight: 700; font-size: 0.9rem; cursor: pointer;
            transition: all 0.2s ease; box-shadow: 0 4px 12px rgba(37,99,235,0.3); margin-top: 6px;
        }
        button:hover { transform: translateY(-2px); filter: brightness(1.05); box-shadow: 0 8px 20px rgba(37,99,235,0.4); }
        .stats {
            background: #0f172a; border-radius: 20px; padding: 12px; margin-top: 14px;
            display: grid; grid-template-columns: 1fr 1fr; gap: 10px; border: 1px solid #2d3a5e;
        }
        .stats.container-stats { border-color: #92400e; background: #1a1208; }
        .stat-item { text-align: center; background: #1e293b; padding: 9px 5px; border-radius: 16px; }
        .container-stats .stat-item { background: #2d1f0c; }
        .stat-value { font-size: 1.35rem; font-weight: 800; color: #fbbf24; line-height: 1.2; }
        .container-stats .stat-value { color: #f59e0b; }
        .stat-label { font-size: 0.65rem; color: #94a3b8; font-weight: 600; }
        .warning {
            background: rgba(250,204,21,0.18); border-left: 3px solid #facc15; padding: 8px 10px;
            border-radius: 16px; font-size: 0.7rem; margin-top: 10px; text-align: center; color: #fde047;
        }
        .section-divider { border: none; border-top: 1px solid rgba(255,255,255,0.08); margin: 4px 0 8px; }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
</head>
<body>
<div class="app">
    <div class="controls">
        <div class="card">
            <div class="card-title">📦 托盘参数</div>
            <div class="input-row"><div class="input-field"><label>长 L (mm)</label><input type="number" id="trayL" value="1200" step="10"></div>
            <div class="input-field"><label>宽 W (mm)</label><input type="number" id="trayW" value="1000" step="10"></div>
            <div class="input-field"><label>高 H (mm)</label><input type="number" id="trayH" value="126" step="5"></div></div>
            <div><label>最大载重 (kg)</label><input type="number" id="trayLoad" value="1500" step="50"></div>
        </div>
        <div class="card"><div class="card-title">🎁 纸箱规格</div>
            <div class="input-row"><div class="input-field"><label>长 L (mm)</label><input type="number" id="boxL" value="400" step="10"></div>
            <div class="input-field"><label>宽 W (mm)</label><input type="number" id="boxW" value="300" step="10"></div>
            <div class="input-field"><label>高 H (mm)</label><input type="number" id="boxH" value="250" step="10"></div></div>
            <div><label>单箱重量 (kg)</label><input type="number" id="boxWeight" value="15" step="1"></div>
            <div style="margin-top:10px;"><label>📦 纸箱内产品数 (个/箱)</label><input type="number" id="productsPerBox" value="1" step="1" min="1"></div>
        </div>
        <div class="card"><div class="card-title">⚙️ 堆叠策略 & 限制</div>
            <div class="radio-group"><label class="radio-item"><input type="radio" name="pattern" value="grid" checked> 📐 标准行列式</label>
            <label class="radio-item"><input type="radio" name="pattern" value="staggered"> 🔄 交错堆叠</label>
            <label class="radio-item"><input type="radio" name="pattern" value="mixed"> 🧠 混合朝向（四块回字算法）</label></div>
            <div class="radio-group"><label class="radio-item"><input type="radio" name="limitMode" value="height" checked> ⬆️ 高度限制</label>
            <label class="radio-item"><input type="radio" name="limitMode" value="weight"> ⚖️ 重量限制</label>
            <label class="radio-item"><input type="radio" name="limitMode" value="both"> 🔒 两者取严</label></div>
            <div><label>📏 自定义堆叠限高 (mm) (0=使用托盘自身高度)</label><input type="number" id="customHeight" value="2200" step="50"></div>
        </div>
        <button id="calcBtn">🚀 智能计算 & 更新3D视图</button>
        <div id="statsPanel" class="stats"></div>
        <div id="warningMsg" class="warning" style="display: none;"></div>
        <hr class="section-divider">
        <div class="card card-container" id="containerCard">
            <div class="card-title">🚢 集装箱装载 (尺柜计算)</div>
            <div class="preset-btns"><button class="preset-btn active-preset" data-preset="20ft">📐 20尺柜</button>
            <button class="preset-btn" data-preset="40ft">📏 40尺柜</button><button class="preset-btn" data-preset="40hc">📏 40尺高柜</button>
            <button class="preset-btn" data-preset="custom">✏️ 自定义</button></div>
            <div class="input-row"><div class="input-field"><label>柜内长 (mm)</label><input type="number" id="contL" value="5800" step="10"></div>
            <div class="input-field"><label>柜内宽 (mm)</label><input type="number" id="contW" value="2350" step="10"></div>
            <div class="input-field"><label>柜内高 (mm)</label><input type="number" id="contH" value="2380" step="10"></div></div>
            <div><label>🔧 栈板间隙 (mm)</label><input type="number" id="contGap" value="20" step="5" min="0"></div>
            <div class="radio-group" style="margin-top: 10px;"><label class="radio-item"><input type="radio" name="containerPattern" value="staggered" checked> 🔄 交错堆叠</label>
            <label class="radio-item"><input type="radio" name="containerPattern" value="mixed"> 🧠 混合朝向（智能择优）</label>
            <label class="radio-item"><input type="radio" name="containerPattern" value="ring"> 🔲 回字形堆叠</label></div>
            <button id="calcContainerBtn" style="background:linear-gradient(105deg,#d97706,#f59e0b);">🚢 计算集装箱最大装载</button>
            <div id="containerStatsPanel" class="stats container-stats"></div>
            <div id="containerWarningMsg" class="warning" style="display:none;"></div>
        </div>
    </div>
    <div class="viewer">
        <canvas id="canvas3d"></canvas>
        <div id="infoPanel" class="info-panel"><div class="label">📐 托盘尺寸</div><div><span class="label">长:</span> <span id="infoTrayL" class="value">--</span> mm &nbsp; <span class="label">宽:</span> <span id="infoTrayW" class="value">--</span> mm</div>
        <div class="label" style="margin-top:4px;">📏 实际满载总高 (含托盘)</div><div><span id="infoTotalHeight" class="value">--</span> mm</div>
        <div id="infoContainerExtra" style="display:none;margin-top:4px;"></div></div>
        <div class="export-toolbar" id="exportToolbar">
            <button class="view-switch-btn pallet-active" id="viewPalletBtn">📦 栈板</button>
            <button class="view-switch-btn" id="viewContainerBtn">🚢 集装箱</button>
            <button class="export-btn" id="exportPngBtn"><span>📸</span> PNG</button>
            <button class="export-btn" id="exportJpgBtn"><span>🖼️</span> JPG</button>
            <button class="export-btn" id="exportPdfBtn"><span>📄</span> PDF</button>
            <button class="export-btn" id="nextStrategyBtn" style="background:rgba(245,158,11,0.3); border-color: #f59e0b;">⏭ 下一页</button>
        </div>
    </div>
</div>

<script type="importmap">
    { 
        "imports": { 
            "three": "https://unpkg.com/three@0.128.0/build/three.module.js",
            "three/addons/": "https://unpkg.com/three@0.128.0/examples/jsm/"
        } 
    }
</script>
<script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

    const dom = {
        trayL: document.getElementById('trayL'), trayW: document.getElementById('trayW'), trayH: document.getElementById('trayH'),
        trayLoad: document.getElementById('trayLoad'), boxL: document.getElementById('boxL'), boxW: document.getElementById('boxW'),
        boxH: document.getElementById('boxH'), boxWeight: document.getElementById('boxWeight'), productsPerBox: document.getElementById('productsPerBox'),
        customHeight: document.getElementById('customHeight'), calcBtn: document.getElementById('calcBtn'),
        statsPanel: document.getElementById('statsPanel'), warningMsg: document.getElementById('warningMsg'),
        infoTrayL: document.getElementById('infoTrayL'), infoTrayW: document.getElementById('infoTrayW'),
        infoTotalHeight: document.getElementById('infoTotalHeight'), infoContainerExtra: document.getElementById('infoContainerExtra'),
        infoPanel: document.getElementById('infoPanel'), contL: document.getElementById('contL'), contW: document.getElementById('contW'),
        contH: document.getElementById('contH'), contGap: document.getElementById('contGap'),
        calcContainerBtn: document.getElementById('calcContainerBtn'), containerStatsPanel: document.getElementById('containerStatsPanel'),
        containerWarningMsg: document.getElementById('containerWarningMsg'),
        viewPalletBtn: document.getElementById('viewPalletBtn'), viewContainerBtn: document.getElementById('viewContainerBtn'),
    };
    let currentResult = null, currentParams = null, currentContainerResult = null, currentViewMode = 'pallet';
    
    const canvas = document.getElementById('canvas3d');
    const scene = new THREE.Scene(); scene.background = new THREE.Color(0xeef2f8);
    const camera = new THREE.PerspectiveCamera(42, 1, 10, 50000);
    camera.position.set(3200, 3000, 3800);
    const renderer = new THREE.WebGLRenderer({ canvas, antialias: true, preserveDrawingBuffer: true });
    renderer.shadowMap.enabled = true; renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true; controls.dampingFactor = 0.06; controls.target.set(0, 500, 0);
    
    scene.add(new THREE.AmbientLight(0xffffff, 0.52));
    scene.add(new THREE.HemisphereLight(0xffffff, 0xb9c7db, 0.38));
    const mainLight = new THREE.DirectionalLight(0xfff0e0, 0.82); mainLight.position.set(2.2, 3.2, 1.8); mainLight.castShadow = true; scene.add(mainLight);
    scene.add(new THREE.DirectionalLight(0xaaccff, 0.38).position.set(-1.5, 2.0, -2.2));
    let currentGroup = null;

    // ========== 原有的基础布局函数 ==========
    function gridPlacement(trayW, trayD, boxW, boxD) {
        let cntX = Math.floor(trayW / boxW), cntY = Math.floor(trayD / boxD);
        if (cntX <= 0 || cntY <= 0) return { count: 0, positions: [], gap: Infinity };
        let positions = [];
        for (let i = 0; i < cntX; i++) {
            for (let j = 0; j < cntY; j++) {
                positions.push({ x: i * boxW, y: j * boxD, w: boxW, h: boxD });
            }
        }
        const usedW = cntX * boxW;
        const usedD = cntY * boxD;
        const gap = (trayW - usedW) + (trayD - usedD);
        return { count: positions.length, positions, gap };
    }

    // 原有的混合朝向算法（保留用于 mixed 策略的降级）
    function optimalMixedLayout(trayL, trayW, boxLen, boxWid) {
        const HORIZ = { w: boxLen, d: boxWid };
        const VERT  = { w: boxWid, d: boxLen };

        let bestCount = 0;
        let bestPositions = [];

        let try1 = [];
        const vertW = VERT.w;
        const vertD = VERT.d;
        const horizW = HORIZ.w;
        const horizD = HORIZ.d;

        const leftWidth = vertW * 2; 
        if (leftWidth <= trayL) {
            const leftRows = Math.floor(trayW / vertD);
            for (let i = 0; i < 2; i++) {
                for (let j = 0; j < leftRows; j++) {
                    try1.push({
                        x: i * vertW,
                        y: j * vertD,
                        w: vertW,
                        h: vertD
                    });
                }
            }
            const rightStartX = leftWidth;
            if (rightStartX + horizW <= trayL) {
                const rightRows = Math.floor(trayW / horizD);
                for (let j = 0; j < rightRows; j++) {
                    try1.push({
                        x: rightStartX,
                        y: j * horizD,
                        w: horizW,
                        h: horizD
                    });
                }
            }
        }

        let try2 = [];
        const rightWidth = vertW * 2;
        const rightStartX2 = trayL - rightWidth;
        if (rightStartX2 >= 0) {
            const rightRows2 = Math.floor(trayW / vertD);
            for (let i = 0; i < 2; i++) {
                for (let j = 0; j < rightRows2; j++) {
                    try2.push({
                        x: rightStartX2 + i * vertW,
                        y: j * vertD,
                        w: vertW,
                        h: vertD
                    });
                }
            }
            const leftWidth2 = trayL - rightWidth;
            if (leftWidth2 >= horizW) {
                const leftRows2 = Math.floor(trayW / horizD);
                for (let j = 0; j < leftRows2; j++) {
                    try2.push({
                        x: 0,
                        y: j * horizD,
                        w: horizW,
                        h: horizD
                    });
                }
            }
        }

        let g1 = gridPlacement(trayL, trayW, boxLen, boxWid);
        let g2 = gridPlacement(trayL, trayW, boxWid, boxLen);
        let try3 = g1.count >= g2.count ? g1.positions : g2.positions;

        let all = [try1, try2, try3];
        all.sort((a, b) => b.length - a.length);
        bestPositions = all[0];
        bestCount = bestPositions.length;

        return { count: bestCount, positions: bestPositions };
    }
    
    function centerPositionsInTray(positions, trayW, trayD) {
        if (!positions.length) return positions;
        let minX = Infinity, minY = Infinity, maxX = -Infinity, maxY = -Infinity;
        positions.forEach(p => {
            minX = Math.min(minX, p.x);
            minY = Math.min(minY, p.y);
            maxX = Math.max(maxX, p.x + p.w);
            maxY = Math.max(maxY, p.y + p.h);
        });
        const offsetX = (trayW - (maxX - minX)) / 2 - minX;
        const offsetY = (trayD - (maxY - minY)) / 2 - minY;
        return positions.map(p => ({ ...p, x: p.x + offsetX, y: p.y + offsetY }));
    }
    
    // ========== 递归分层优化算法（用于交错堆叠） ==========
    function globalLayerOptimization2D(containerW, containerH, itemW, itemH, gap = 0, margin = 0, allowRotation = true) {
        if (!allowRotation) return null;
        const effectiveW = containerW - 2 * margin;
        const effectiveH = containerH - 2 * margin;
        const minItemDim = Math.min(itemW, itemH);
        const resultCache = new Map();

        function recursivePackRegion(regionW, regionH) {
            const cacheKey = `${regionW.toFixed(2)}_${regionH.toFixed(2)}`;
            if (resultCache.has(cacheKey)) return resultCache.get(cacheKey);
            if (regionW < minItemDim - 1e-9 || regionH < minItemDim - 1e-9) {
                const empty = { count: 0, items: [] };
                resultCache.set(cacheKey, empty);
                return empty;
            }

            let bestResult = { count: 0, items: [] };

            // 不分割，直接平铺
            const colsNormal = Math.floor((regionW + gap) / (itemW + gap));
            const rowsNormal = Math.floor((regionH + gap) / (itemH + gap));
            const countNormal = colsNormal * rowsNormal;
            if (countNormal > bestResult.count) {
                const items = [];
                for (let r = 0; r < rowsNormal; r++) {
                    for (let c = 0; c < colsNormal; c++) {
                        items.push({ x: c * (itemW + gap), y: r * (itemH + gap), width: itemW, height: itemH, rotated: false });
                    }
                }
                bestResult = { count: countNormal, items: items };
            }
            const colsRotated = Math.floor((regionW + gap) / (itemH + gap));
            const rowsRotated = Math.floor((regionH + gap) / (itemW + gap));
            const countRotated = colsRotated * rowsRotated;
            if (countRotated > bestResult.count) {
                const items = [];
                for (let r = 0; r < rowsRotated; r++) {
                    for (let c = 0; c < colsRotated; c++) {
                        items.push({ x: c * (itemH + gap), y: r * (itemW + gap), width: itemH, height: itemW, rotated: true });
                    }
                }
                bestResult = { count: countRotated, items: items };
            }

            // 上下分割
            const maxSplitRows = Math.floor(regionH / minItemDim);
            for (let splitRow = 1; splitRow < maxSplitRows; splitRow++) {
                const splitHNormal = splitRow * (itemH + gap) - gap;
                if (splitHNormal > 0 && splitHNormal < regionH - 1e-9) {
                    const top = recursivePackRegion(regionW, splitHNormal);
                    const bottom = recursivePackRegion(regionW, regionH - splitHNormal - gap);
                    const total = top.count + bottom.count;
                    if (total > bestResult.count) {
                        const merged = [...top.items];
                        bottom.items.forEach(it => merged.push({ ...it, y: it.y + splitHNormal + gap }));
                        bestResult = { count: total, items: merged };
                    }
                }
                const splitHRotated = splitRow * (itemW + gap) - gap;
                if (splitHRotated > 0 && splitHRotated < regionH - 1e-9) {
                    const top = recursivePackRegion(regionW, splitHRotated);
                    const bottom = recursivePackRegion(regionW, regionH - splitHRotated - gap);
                    const total = top.count + bottom.count;
                    if (total > bestResult.count) {
                        const merged = [...top.items];
                        bottom.items.forEach(it => merged.push({ ...it, y: it.y + splitHRotated + gap }));
                        bestResult = { count: total, items: merged };
                    }
                }
            }

            // 左右分割
            const maxSplitCols = Math.floor(regionW / minItemDim);
            for (let splitCol = 1; splitCol < maxSplitCols; splitCol++) {
                const splitWNormal = splitCol * (itemW + gap) - gap;
                if (splitWNormal > 0 && splitWNormal < regionW - 1e-9) {
                    const left = recursivePackRegion(splitWNormal, regionH);
                    const right = recursivePackRegion(regionW - splitWNormal - gap, regionH);
                    const total = left.count + right.count;
                    if (total > bestResult.count) {
                        const merged = [...left.items];
                        right.items.forEach(it => merged.push({ ...it, x: it.x + splitWNormal + gap }));
                        bestResult = { count: total, items: merged };
                    }
                }
                const splitWRotated = splitCol * (itemH + gap) - gap;
                if (splitWRotated > 0 && splitWRotated < regionW - 1e-9) {
                    const left = recursivePackRegion(splitWRotated, regionH);
                    const right = recursivePackRegion(regionW - splitWRotated - gap, regionH);
                    const total = left.count + right.count;
                    if (total > bestResult.count) {
                        const merged = [...left.items];
                        right.items.forEach(it => merged.push({ ...it, x: it.x + splitWRotated + gap }));
                        bestResult = { count: total, items: merged };
                    }
                }
            }

            resultCache.set(cacheKey, bestResult);
            return bestResult;
        }

        const final = recursivePackRegion(effectiveW, effectiveH);
        if (final.count === 0) return null;
        const finalItems = final.items.map(item => ({
            x: item.x + margin,
            y: item.y + margin,
            width: item.width,
            height: item.height,
            rotated: item.rotated
        }));
        return {
            count: finalItems.length,
            items: finalItems,
            utilization: (finalItems.length * itemW * itemH) / (effectiveW * effectiveH) * 100,
            method: "递归分层优化（任意层数嵌套）"
        };
    }

    // ========== 提取自第二个网页的四块回字堆叠算法（含辅助函数） ==========
    function checkValid(list, bL, bW) {
        let arr = [];
        for (let item of list) {
            if (item.x + item.l <= bL + 1e-9 && item.y + item.w <= bW + 1e-9) {
                arr.push(item);
            }
        }
        return arr;
    }
    function isOverlap(x1, y1, w1, h1, x2, y2, w2, h2) {
        return !(x1 + w1 <= x2 + 1e-9 || x2 + w2 <= x1 + 1e-9 || y1 + h1 <= y2 + 1e-9 || y2 + h2 <= y1 + 1e-9);
    }
    function genCornerOptions(maxL, maxW, itemL, itemW) {
        const eps = 1e-9;
        const opts = [];
        const maxC = Math.floor(maxL / itemL);
        const maxR = Math.floor(maxW / itemW);
        for (let c = 0; c <= maxC; c++) {
            for (let r = 0; r <= maxR; r++) {
                if (c === 0 && r === 0) { opts.push({ cols: 0, rows: 0, w: 0, h: 0, count: 0 }); continue; }
                if (c === 0 || r === 0) continue;
                opts.push({ cols: c, rows: r, w: c * itemL, h: r * itemW, count: c * r });
            }
        }
        return opts;
    }

    function fourBlockLayout(containerL, containerW, itemL, itemW) {
        const eps = 1e-9;
        const w = itemL;
        const h = itemW;
        const minItem = Math.min(w, h);

        if (containerL < minItem - eps || containerW < minItem - eps) {
            return { count: 0, positions: [] };
        }

        let bestTotal = -1;
        let bestConfig = null;
        const rotOptions = [false, true];

        for (const tlRot of rotOptions) {
            for (const trRot of rotOptions) {
                for (const blRot of rotOptions) {
                    for (const brRot of rotOptions) {
                        const tlSize = { l: tlRot ? h : w, w: tlRot ? w : h };
                        const trSize = { l: trRot ? h : w, w: trRot ? w : h };
                        const blSize = { l: blRot ? h : w, w: blRot ? w : h };
                        const brSize = { l: brRot ? h : w, w: brRot ? w : h };

                        const tlOpts = genCornerOptions(containerL, containerW, tlSize.l, tlSize.w);
                        const trOpts = genCornerOptions(containerL, containerW, trSize.l, trSize.w);
                        const blOpts = genCornerOptions(containerL, containerW, blSize.l, blSize.w);
                        const brOpts = genCornerOptions(containerL, containerW, brSize.l, brSize.w);

                        let curBestCount = -1;
                        let curTL = null, curTR = null, curBL = null, curBR = null;

                        for (const tl of tlOpts) {
                            for (const tr of trOpts) {
                                if (tl.w + tr.w > containerL + eps) continue;
                                for (const bl of blOpts) {
                                    if (tl.h + bl.h > containerW + eps) continue;
                                    const trX = containerL - tr.w;
                                    const blY = containerW - bl.h;
                                    if (isOverlap(trX, 0, tr.w, tr.h, 0, blY, bl.w, bl.h)) continue;
                                    for (const br of brOpts) {
                                        if (bl.w + br.w > containerL + eps) continue;
                                        if (tr.h + br.h > containerW + eps) continue;
                                        const brX = containerL - br.w;
                                        const brY = containerW - br.h;
                                        if (isOverlap(0, 0, tl.w, tl.h, brX, brY, br.w, br.h)) continue;
                                        const total = tl.count + tr.count + bl.count + br.count;
                                        if (total > curBestCount) {
                                            curBestCount = total;
                                            curTL = tl; curTR = tr; curBL = bl; curBR = br;
                                        }
                                    }
                                }
                            }
                        }

                        if (curBestCount > bestTotal) {
                            bestTotal = curBestCount;
                            bestConfig = { tl: curTL, tr: curTR, bl: curBL, br: curBR, tlSize, trSize, blSize, brSize };
                        }
                    }
                }
            }
        }

        if (!bestConfig || bestTotal <= 0) {
            return { count: 0, positions: [] };
        }

        const pos = [];
        for (let r = 0; r < bestConfig.tl.rows; r++) {
            for (let c = 0; c < bestConfig.tl.cols; c++) {
                pos.push({ x: c * bestConfig.tlSize.l, y: r * bestConfig.tlSize.w, l: bestConfig.tlSize.l, w: bestConfig.tlSize.w });
            }
        }
        const trBaseX = containerL - bestConfig.tr.w;
        for (let r = 0; r < bestConfig.tr.rows; r++) {
            for (let c = 0; c < bestConfig.tr.cols; c++) {
                pos.push({ x: trBaseX + c * bestConfig.trSize.l, y: r * bestConfig.trSize.w, l: bestConfig.trSize.l, w: bestConfig.trSize.w });
            }
        }
        const blBaseY = containerW - bestConfig.bl.h;
        for (let r = 0; r < bestConfig.bl.rows; r++) {
            for (let c = 0; c < bestConfig.bl.cols; c++) {
                pos.push({ x: c * bestConfig.blSize.l, y: blBaseY + r * bestConfig.blSize.w, l: bestConfig.blSize.l, w: bestConfig.blSize.w });
            }
        }
        const brBaseX = containerL - bestConfig.br.w;
        const brBaseY = containerW - bestConfig.br.h;
        for (let r = 0; r < bestConfig.br.rows; r++) {
            for (let c = 0; c < bestConfig.br.cols; c++) {
                pos.push({ x: brBaseX + c * bestConfig.brSize.l, y: brBaseY + r * bestConfig.brSize.w, l: bestConfig.brSize.l, w: bestConfig.brSize.w });
            }
        }

        const innerLeft = Math.max(bestConfig.tl.w, bestConfig.bl.w);
        const innerRight = Math.min(containerL - bestConfig.tr.w, containerL - bestConfig.br.w);
        const innerTop = Math.max(bestConfig.tl.h, bestConfig.tr.h);
        const innerBottom = Math.min(containerW - bestConfig.bl.h, containerW - bestConfig.br.h);
        const innerL = innerRight - innerLeft;
        const innerW = innerBottom - innerTop;

        if (innerL >= minItem - eps && innerW >= minItem - eps) {
            const midResult = fourBlockLayout(innerL, innerW, itemL, itemW);
            midResult.positions.forEach(p => {
                pos.push({
                    x: p.x + innerLeft,
                    y: p.y + innerTop,
                    l: p.l,
                    w: p.w
                });
            });
        }

        const valid = checkValid(pos, containerL, containerW);
        const positions = valid.map(p => ({ x: p.x, y: p.y, w: p.l, h: p.w }));
        return { count: positions.length, positions };
    }

    function getLayerLayout(trayL, trayW, boxL, boxW, pattern) {
        let res;
        switch (pattern) {
            case "grid":
                let a = gridPlacement(trayL, trayW, boxL, boxW);
                let b = gridPlacement(trayL, trayW, boxW, boxL);
                res = a.count >= b.count ? a : b;
                break;
            case "staggered": {
                const optResult = globalLayerOptimization2D(trayL, trayW, boxL, boxW, 0, 0, true);
                if (optResult && optResult.count > 0) {
                    const positions = optResult.items.map(item => ({
                        x: item.x,
                        y: item.y,
                        w: item.width,
                        h: item.height
                    }));
                    res = { count: optResult.count, positions: positions, gap: 0 };
                } else {
                    res = optimalMixedLayout(trayL, trayW, boxL, boxW);
                }
                break;
            }
            case "mixed": {
                const result = fourBlockLayout(trayL, trayW, boxL, boxW);
                if (result.count > 0) {
                    res = { count: result.count, positions: result.positions, gap: 0 };
                } else {
                    res = optimalMixedLayout(trayL, trayW, boxL, boxW);
                }
                break;
            }
            default:
                res = gridPlacement(trayL, trayW, boxL, boxW);
        }
        if (res && res.positions) {
            res.positions = centerPositionsInTray(res.positions, trayL, trayW);
        }
        return res;
    }

    function createPalletBase(trayW, trayL, trayH) {
        const g = new THREE.Group();
        const trayMat = new THREE.MeshStandardMaterial({ color: 0xe2b84a, roughness: 0.4 });
        const trayMesh = new THREE.Mesh(new THREE.BoxGeometry(trayL, trayH, trayW), trayMat);
        trayMesh.position.set(0, trayH / 2, 0);
        trayMesh.castShadow = true;
        trayMesh.receiveShadow = true;
        g.add(trayMesh);
        return g;
    }

    function compute(params) {
        const { trayL, trayW, trayH, trayLoad, boxL, boxW, boxH, boxWgt, productsPerBox, pattern, limitMode, customHeight } = params;
        let layer = getLayerLayout(trayL, trayW, boxL, boxW, pattern);
        if (!layer.count) return { error: "单层无法容纳任何纸箱" };
        let perLayerQty = layer.count;
        let perLayerWeight = perLayerQty * boxWgt;
        let effH = (customHeight > 0) ? customHeight : trayH;
        let maxH = Math.floor(effH / boxH);
        let maxW = (trayLoad <= 0) ? Infinity : Math.floor(trayLoad / perLayerWeight);
        let finalMax = (limitMode === "height") ? maxH : (limitMode === "weight") ? maxW : Math.min(maxH, maxW);
        if (finalMax <= 0) finalMax = 0;
        let layers = finalMax;
        let totalQty = layers * perLayerQty;
        let totalWeight = totalQty * boxWgt;
        let totalProducts = totalQty * productsPerBox;
        let areaUtil = (perLayerQty * (boxL * boxW) / (trayL * trayW) * 100).toFixed(1);
        let actualTotalHeight = trayH + layers * boxH;
        let warning = "";
        if (maxH <= 0) warning = "⚠️ 高度不足";
        else if (maxW <= 0 && trayLoad > 0) warning = "⚠️ 超重";
        else if (layers === 0) warning = "⚠️ 无法堆叠";
        return {
            success: true, perLayerQty, layers, totalQty, totalWeight: totalWeight.toFixed(1),
            totalProducts, productsPerBox, areaUtil, warning, layerPositions: layer.positions,
            actualTotalHeight, boxL, boxW, boxH, trayL, trayW, trayH
        };
    }

    // ========== 以下为集装箱装载相关函数 ==========
    function palletFloorArea(positions) { return positions.reduce((s,p)=>s+p.l*p.w,0); }
    function finalizePalletPackEdge(positions, L, W, swapCont) { 
        if(!positions.length) return {count:0,positions:[], floorArea:0}; 
        let pos=positions.map(p=>({...p})); 
        if(swapCont){ 
            pos=pos.map(p=>({x:p.z,z:p.x,l:p.w,w:p.l,rotated:!p.rotated})); 
        } 
        return {count:pos.length, positions:pos, floorArea:palletFloorArea(pos)}; 
    }
    
    function packUniformGrid(L,W,A,B,gap,swapCont){ 
        let best={count:0,positions:[], floorArea:0}; 
        for(let rot of [false,true]){ 
            let pL=rot?B:A, pW=rot?A:B; 
            let nx=Math.floor((L+gap)/(pL+gap)), ny=Math.floor((W+gap)/(pW+gap)); 
            if(nx>0&&ny>0){ 
                let pos=[]; 
                for(let j=0;j<ny;j++) for(let i=0;i<nx;i++) pos.push({x:i*(pL+gap),z:j*(pW+gap),l:pL,w:pW,rotated:rot}); 
                if(pos.length>best.count) best={count:pos.length, positions:pos, floorArea:pos.reduce((s,p)=>s+p.l*p.w,0)}; 
            } 
        } 
        return finalizePalletPackEdge(best.positions, L, W, swapCont); 
    }
    
    function packExhaustiveRowPermutations(L,W,A,B,gap,swapCont){ 
        let best={count:0,positions:[], floorArea:0}; 
        for(let rot of [false,true]){ 
            let pL=rot?B:A, pW=rot?A:B; 
            let nx=Math.floor((L+gap)/(pL+gap)); 
            if(nx<=0) continue; 
            let maxRows=Math.floor((W+gap)/(pW+gap)); 
            for(let rows=maxRows;rows>=1;rows--){ 
                let usedW=rows*pW+(rows-1)*gap; 
                if(usedW<=W+1e-4){ 
                    let pos=[]; 
                    for(let r=0;r<rows;r++) for(let i=0;i<nx;i++) pos.push({x:i*(pL+gap),z:r*(pW+gap),l:pL,w:pW,rotated:rot}); 
                    if(pos.length>best.count) best={count:pos.length, positions:pos, floorArea:pos.reduce((s,p)=>s+p.l*p.w,0)}; 
                    break; 
                } 
            } 
        } 
        return finalizePalletPackEdge(best.positions, L, W, swapCont); 
    }
    
    function doubleSidedLayout(contL, contW, palletL, palletW, gap) {
        const g = gap || 0;
        const maxSideCount = Math.min(12, Math.floor((contL + g) / (Math.min(palletL, palletW) + g)) + 2);
        let best = { count: 0, positions: [] };
        for (let nA = 0; nA <= maxSideCount; nA++) {
            for (let nB = 0; nB <= maxSideCount; nB++) {
                if (nA + nB === 0) continue;
                const rotationsA = new Array(nA).fill(false).map(() => false);
                const rotationsB = new Array(nB).fill(false).map(() => false);
                const generateRotations = (n, arr, idx, callback) => {
                    if (idx === n) { callback([...arr]); return; }
                    arr[idx] = false; generateRotations(n, arr, idx+1, callback);
                    arr[idx] = true; generateRotations(n, arr, idx+1, callback);
                };
                generateRotations(nA, rotationsA, 0, (rotA) => {
                    const buildSidePositions = (contL, palletLen, palletWid, gap, rotations) => {
                        let x = 0; const positions = [];
                        for (const rot of rotations) {
                            const l = rot ? palletWid : palletLen;
                            if (x + l > contL + 1e-4) return null;
                            positions.push({ x, l, rotated: rot, w: rot ? palletLen : palletWid });
                            x += l + gap;
                        }
                        return positions;
                    };
                    const sideA = buildSidePositions(contL, palletL, palletW, g, rotA);
                    if (!sideA) return;
                    generateRotations(nB, rotationsB, 0, (rotB) => {
                        const sideB = buildSidePositions(contL, palletL, palletW, g, rotB);
                        if (!sideB) return;
                        const checkOverlapAndWidth = (positionsA, positionsB, contW, gap) => {
                            for (const a of positionsA) {
                                for (const b of positionsB) {
                                    if (!(a.x + a.l <= b.x + 1e-4 || b.x + b.l <= a.x + 1e-4)) {
                                        const required = a.w + b.w + gap;
                                        if (required > contW + 1e-4) return false;
                                    }
                                }
                            }
                            return true;
                        };
                        if (checkOverlapAndWidth(sideA, sideB, contW, g)) {
                            const positions = sideA.map(p => ({ x: p.x, z: 0, l: p.l, w: p.w, rotated: p.rotated }));
                            sideB.forEach(p => positions.push({ x: p.x, z: contW - p.w, l: p.l, w: p.w, rotated: p.rotated }));
                            if (positions.length > best.count) best = { count: positions.length, positions, floorArea: positions.reduce((s,p)=>s+p.l*p.w,0) };
                        }
                    });
                });
            }
        }
        return best.count ? best : null;
    }
    function packSkylineBottomLeft(L,W,A,B,gap,swapCont){ 
        let placed=[]; 
        const canPlace=(pos, placed, L, W, gap)=>{ 
            if(pos.x<0||pos.z<0||pos.x+pos.l>L+1e-4||pos.z+pos.w>W+1e-4) return false; 
            for(let p of placed) if(!(pos.x+pos.l+gap<=p.x||p.x+p.l+gap<=pos.x||pos.z+pos.w+gap<=p.z||p.z+p.w+gap<=pos.z)) return false; 
            return true; 
        }; 
        const getPoints=()=>{ 
            let pts=[{x:0,z:0}]; 
            for(let p of placed){ 
                pts.push({x:p.x+p.l+gap,z:p.z}); 
                pts.push({x:p.x,z:p.z+p.w+gap}); 
            } 
            return pts; 
        }; 
        while(true){ 
            let bestPos=null, bestScore=Infinity; 
            for(let pt of getPoints()) for(let rot of [false,true]){ 
                let pos={x:pt.x,z:pt.z,l:rot?B:A,w:rot?A:B,rotated:rot}; 
                if(canPlace(pos,placed,L,W,gap)){ 
                    let score=pos.z*(L+1)+pos.x; 
                    if(score<bestScore){ 
                        bestScore=score; 
                        bestPos=pos; 
                    } 
                } 
            } 
            if(!bestPos) break; 
            placed.push(bestPos); 
        } 
        return finalizePalletPackEdge(placed, L, W, swapCont); 
    }
    
    function bestPalletLayoutMixed(contL, contW, palletL, palletW, gap) {
        let best = { count: 0, positions: [], floorArea: 0 };
        const doubleSide = doubleSidedLayout(contL, contW, palletL, palletW, gap);
        if (doubleSide && doubleSide.count > best.count) best = doubleSide;
        const legacy = [packUniformGrid, packExhaustiveRowPermutations, packSkylineBottomLeft];
        for(let swap of [false, true]){ 
            let L=swap?contW:contL, W=swap?contL:contW; 
            for(let fn of legacy){ 
                let res=fn(L,W,palletL,palletW,gap,swap); 
                if(res.count>best.count) best=res; 
                else if(res.count===best.count && res.floorArea>best.floorArea) best=res; 
            } 
        }
        if (!best.count) return { count: 0, positions: [] };
        return best;
    }
    
    function packStaggeredLayout(contL, contW, palletL, palletW, gap, rotateRow = false) {
        const g = gap;
        const positions = [];
        let totalCount = 0;
        let currentZ = 0;
        let currL = palletL;
        let currW = palletW;
        let isRotated = false;
        while (true) {
            const rowCount = Math.floor((contL + g) / (currL + g));
            if (rowCount <= 0) break;
            const rowUsedW = currW;
            if (currentZ + rowUsedW > contW + 1e-4) break;
            const offset = (totalCount % 2 === 1) ? (currL + g) / 2 : 0;
            for (let i = 0; i < rowCount; i++) {
                const x = offset + i * (currL + g);
                positions.push({ x: x, z: currentZ, l: currL, w: currW, rotated: isRotated });
            }
            totalCount += rowCount;
            currentZ += rowUsedW + g;
            if (rotateRow) { [currL, currW] = [currW, currL]; isRotated = !isRotated; }
        }
        const floorArea = positions.reduce((sum, p) => sum + p.l * p.w, 0);
        return { count: totalCount, positions: positions, floorArea: floorArea };
    }
    
    function bestPalletLayoutAdvanced(contL, contW, palletL, palletW, gap) {
        const g = gap || 0;
        const allPlans = [];
        const row1W = palletW, row2W = palletL;
        if (row1W + row2W + g <= contW) {
            const row1Count = Math.floor((contL + g) / (palletL + g));
            const row2Count = Math.floor((contL + g) / (palletW + g));
            const positions = [];
            for (let i = 0; i < row1Count; i++) positions.push({ x: i * (palletL + g), z: 0, l: palletL, w: palletW, rotated: false });
            for (let i = 0; i < row2Count; i++) positions.push({ x: i * (palletW + g), z: row1W + g, l: palletW, w: palletL, rotated: true });
            allPlans.push({ count: row1Count + row2Count, positions, floorArea: positions.reduce((s,p)=>s+p.l*p.w,0), strategy: "非对称双排" });
        }
        const revRow1W = palletL, revRow2W = palletW;
        if (revRow1W + revRow2W + g <= contW) {
            const revRow1Count = Math.floor((contL + g) / (palletW + g));
            const revRow2Count = Math.floor((contL + g) / (palletL + g));
            const positions = [];
            for (let i = 0; i < revRow1Count; i++) positions.push({ x: i * (palletW + g), z: 0, l: palletW, w: palletL, rotated: true });
            for (let i = 0; i < revRow2Count; i++) positions.push({ x: i * (palletL + g), z: revRow1W + g, l: palletL, w: palletW, rotated: false });
            allPlans.push({ count: revRow1Count + revRow2Count, positions, floorArea: positions.reduce((s,p)=>s+p.l*p.w,0), strategy: "反向非对称双排" });
        }
        const doubleSide = doubleSidedLayout(contL, contW, palletL, palletW, g);
        if (doubleSide && doubleSide.count > 0) allPlans.push({ ...doubleSide, strategy: "双面咬合" });
        const legacy = [packUniformGrid, packExhaustiveRowPermutations, packSkylineBottomLeft];
        for (let swap of [false, true]) {
            const L = swap ? contW : contL, W = swap ? contL : contW;
            for (let fn of legacy) {
                const res = fn(L, W, palletL, palletW, g, swap);
                if (res.count > 0) allPlans.push({ ...res, strategy: "规整网格" });
            }
        }
        const staggerSame = packStaggeredLayout(contL, contW, palletL, palletW, g, false);
        if (staggerSame.count > 0) allPlans.push({ ...staggerSame, strategy: "同向交错" });
        const staggerRotate = packStaggeredLayout(contL, contW, palletL, palletW, g, true);
        if (staggerRotate.count > 0) allPlans.push({ ...staggerRotate, strategy: "旋转交错" });
        if (allPlans.length === 0) return { count: 0, positions: [] };
        allPlans.sort((a,b) => b.count !== a.count ? b.count - a.count : b.floorArea - a.floorArea);
        return allPlans[0];
    }
    
    function getContainerLayoutByStrategy(contL, contW, palletL, palletW, gap, strategy) {
        if (strategy === 'staggered' || strategy === 'mixed') return bestPalletLayoutMixed(contL, contW, palletL, palletW, gap);
        else if (strategy === 'ring') return bestPalletLayoutAdvanced(contL, contW, palletL, palletW, gap);
        else return bestPalletLayoutMixed(contL, contW, palletL, palletW, gap);
    }
    
    function computeContainerLoading(palletResult, params, contL, contW, contH, gap, containerStrategy) {
        if(!palletResult?.success) return {error:"请先完成栈板计算"};
        const pL=params.trayL, pW=params.trayW, pH=palletResult.actualTotalHeight;
        if(!((pL <= contL && pW <= contW) || (pW <= contL && pL <= contW))) return {error:"栈板尺寸超过集装箱底面"};
        if(pH>contH) return {error:"栈板总高超过集装箱高度"};
        const maxStack = Math.floor(contH/pH);
        if(maxStack<=0) return {error:"高度不足一层"};
        let layout = getContainerLayoutByStrategy(contL, contW, pL, pW, gap, containerStrategy);
        if(!layout.count) return {error:"无法放置任何栈板"};
        if (layout.positions.length > 0) {
            let minX=Infinity,maxX=-Infinity,minZ=Infinity,maxZ=-Infinity;
            layout.positions.forEach(p=>{minX=Math.min(minX,p.x);maxX=Math.max(maxX,p.x+p.l);minZ=Math.min(minZ,p.z);maxZ=Math.max(maxZ,p.z+p.w);});
            const offsetX=(contL-(maxX-minX))/2-minX, offsetZ=(contW-(maxZ-minZ))/2-minZ;
            layout.positions=layout.positions.map(p=>({...p,x:p.x+offsetX,z:p.z+offsetZ}));
        }
        const totalPallets=layout.count*maxStack;
        const totalBoxes=totalPallets*palletResult.totalQty;
        const totalProducts=totalPallets*palletResult.totalProducts;
        const totalWeight=(totalPallets*parseFloat(palletResult.totalWeight)).toFixed(1);
        let usedArea=layout.floorArea||layout.positions.reduce((s,p)=>s+p.l*p.w,0);
        const floorUtil=(usedArea/(contL*contW)*100).toFixed(1);
        const volumeUtil=(totalPallets*pL*pW*pH/(contL*contW*contH)*100).toFixed(1);
        let strategyName='';
        if(containerStrategy==='staggered') strategyName='交错堆叠（混合智能择优）';
        else if(containerStrategy==='mixed') strategyName='混合朝向智能择优';
        else if(containerStrategy==='ring') strategyName='回字形堆叠';
        else strategyName='自动最优';
        return { success:true, palletsPerLayer:layout.count, stackLayers:maxStack, totalPallets, totalBoxes, totalProducts, totalWeight, floorUtil, volumeUtil, floorPositions:layout.positions, contL,contW,contH,gap, palletTotalHeight:pH, packStrategy:strategyName };
    }

    // ========== 3D 对象生成函数 ==========
    function createSolidBox(width, height, depth, position, drawEdges = true) {
        const group = new THREE.Group();
        const redMat = new THREE.MeshStandardMaterial({ color: 0xb0121a, roughness: 0.3, metalness: 0.1 });
        const blueMat = new THREE.MeshStandardMaterial({ color: 0x164f9a, roughness: 0.3, metalness: 0.1 });
        const darkMat = new THREE.MeshStandardMaterial({ color: 0xf0f0f0, roughness: 0.5 });
        let materials = (width >= depth) ? [redMat, redMat, darkMat, darkMat, blueMat, blueMat] : [blueMat, blueMat, darkMat, darkMat, redMat, redMat];
        const boxMesh = new THREE.Mesh(new THREE.BoxGeometry(width, height, depth), materials);
        boxMesh.position.copy(position);
        group.add(boxMesh);
        if (drawEdges) {
            const edgesGeo = new THREE.EdgesGeometry(boxMesh.geometry);
            const edgesMat = new THREE.LineBasicMaterial({ color: 0x000000 });
            const wireframe = new THREE.LineSegments(edgesGeo, edgesMat);
            boxMesh.add(wireframe);
        }
        return group;
    }

    function createFullPalletGroup(result, params, position, rotationY=0, drawEdges = true) { 
        const group = new THREE.Group(); 
        const {trayL,trayW,trayH,boxH}=params; 
        const palletBase = createPalletBase(trayW, trayL, trayH); 
        palletBase.position.set(0,0,0); 
        group.add(palletBase); 
        const startY = trayH; 
        for(let layerIdx=0; layerIdx<result.layers; layerIdx++) { 
            const yBase = startY + layerIdx*boxH; 
            for(let pos of result.layerPositions) { 
                const cx = pos.x + pos.w/2 - trayL/2; 
                const cz = pos.y + pos.h/2 - trayW/2; 
                group.add(createSolidBox(pos.w, boxH, pos.h, new THREE.Vector3(cx, yBase+boxH/2, cz), drawEdges)); 
            } 
        } 
        group.position.copy(position); 
        if(rotationY!==0) group.rotation.y = rotationY; 
        return group; 
    }

    function createContainerFrame(contL, contH, contW) { 
        const group = new THREE.Group(); 
        const color = 0xFFF160;
        const mat = new THREE.MeshBasicMaterial({ color: color, side: THREE.DoubleSide }); 
        const floorThick = 18;
        const floor = new THREE.Mesh(new THREE.BoxGeometry(contL, floorThick, contW), mat);
        floor.position.set(0, -floorThick/2, 0);
        floor.castShadow = true;
        group.add(floor);
        const backThick = 30;
        const back = new THREE.Mesh(new THREE.BoxGeometry(contL, contH, backThick), mat);
        back.position.set(0, contH/2, -contW/2 + backThick/2);
        back.castShadow = true;
        group.add(back);
        const rightThick = 18;
        const right = new THREE.Mesh(new THREE.BoxGeometry(rightThick, contH, contW), mat);
        right.position.set(contL/2 + rightThick/2, contH/2, 0);
        right.castShadow = true;
        group.add(right);
        return group; 
    }
    
    function createSimplePalletGroup(params, palletTotalHeight, position, rotationY = 0) {
        const group = new THREE.Group();
        const { trayL, trayW, trayH } = params;
        const palletBase = createPalletBase(trayW, trayL, trayH);
        palletBase.position.set(0, 0, 0);
        group.add(palletBase);
        const cargoHeight = palletTotalHeight - trayH;
        if (cargoHeight > 0.1) {
            const redMat = new THREE.MeshStandardMaterial({ color: 0xb0121a, roughness: 0.3, metalness: 0.1 });
            const blueMat = new THREE.MeshStandardMaterial({ color: 0x164f9a, roughness: 0.3, metalness: 0.1 });
            const darkMat = new THREE.MeshStandardMaterial({ color: 0xf0f0f0, roughness: 0.5 });
            let materials = (trayL >= trayW) ? [redMat, redMat, darkMat, darkMat, blueMat, blueMat] : [blueMat, blueMat, darkMat, darkMat, redMat, redMat];
            const cargoBox = new THREE.Mesh(new THREE.BoxGeometry(trayL, cargoHeight, trayW), materials);
            cargoBox.position.set(0, trayH + cargoHeight / 2, 0);
            cargoBox.castShadow = true;
            cargoBox.receiveShadow = true;
            group.add(cargoBox);
            const edgesGeo = new THREE.EdgesGeometry(cargoBox.geometry);
            const edgesMat = new THREE.LineBasicMaterial({ color: 0xaaaaaa, transparent: true, opacity: 0.5 });
            const wireframe = new THREE.LineSegments(edgesGeo, edgesMat);
            cargoBox.add(wireframe);
        }
        group.position.copy(position);
        if (rotationY !== 0) group.rotation.y = rotationY;
        return group;
    }
    
    function disposeGroup(g){ if(g) g.traverse(child=>{ if(child.geometry) child.geometry.dispose(); if(child.material) Array.isArray(child.material)?child.material.forEach(m=>m.dispose()):child.material.dispose(); }); scene.remove(g); }
    function update3DPalletScene(result,params){ if(currentGroup) disposeGroup(currentGroup); currentGroup=createFullPalletGroup(result,params,new THREE.Vector3(0,0,0),0, true); scene.add(currentGroup); }
    
    function update3DContainerScene(containerResult){ 
        if(currentGroup) disposeGroup(currentGroup); 
        currentGroup = new THREE.Group(); 
        currentGroup.add(createContainerFrame(containerResult.contL, containerResult.contH, containerResult.contW)); 
        for(let layer=0; layer<containerResult.stackLayers; layer++){ 
            const yBase = layer * containerResult.palletTotalHeight; 
            for(const fp of containerResult.floorPositions){ 
                const cx = fp.x + fp.l/2 - containerResult.contL/2; 
                const cz = fp.z + fp.w/2 - containerResult.contW/2; 
                const rot = fp.rotated ? Math.PI/2 : 0; 
                currentGroup.add(createSimplePalletGroup(currentParams, containerResult.palletTotalHeight, new THREE.Vector3(cx, yBase, cz), rot)); 
            } 
        } 
        scene.add(currentGroup); 
    }

    function getPattern(){ let r=document.getElementsByName('pattern'); for(let i of r) if(i.checked) return i.value; return 'grid'; }
    function getLimitMode(){ let r=document.getElementsByName('limitMode'); for(let i of r) if(i.checked) return i.value; return 'height'; }
    function getContainerStrategy(){ let r=document.getElementsByName('containerPattern'); for(let i of r) if(i.checked) return i.value; return 'staggered'; }
    
    function updateStatsUI(result){ if(!result.success){ dom.statsPanel.innerHTML=`<div style="grid-column:span2;color:#f87171;">${result.error}</div>`; dom.warningMsg.style.display='block'; dom.warningMsg.innerText=result.error; return; } dom.statsPanel.innerHTML=`<div class="stat-item"><div class="stat-value">${result.perLayerQty}</div><div class="stat-label">每层数量</div></div><div class="stat-item"><div class="stat-value">${result.layers}</div><div class="stat-label">堆叠层数</div></div><div class="stat-item"><div class="stat-value">${result.totalQty}</div><div class="stat-label">总箱数</div></div><div class="stat-item"><div class="stat-value">${result.totalWeight} kg</div><div class="stat-label">总重量</div></div><div class="stat-item"><div class="stat-value">${result.areaUtil}%</div><div class="stat-label">面积利用率</div></div><div class="stat-item"><div class="stat-value">${result.totalProducts}</div><div class="stat-label">产品总数</div></div>`; dom.warningMsg.style.display=result.warning?'block':'none'; if(result.warning) dom.warningMsg.innerText=result.warning; }
    function updateContainerStatsUI(cr){ if(!cr?.success){ dom.containerStatsPanel.innerHTML=`<div style="grid-column:span2;color:#f87171;">${cr?.error||'计算失败'}</div>`; dom.containerWarningMsg.style.display='block'; dom.containerWarningMsg.innerText=cr?.error||'计算失败'; return; } dom.containerStatsPanel.innerHTML=`<div class="stat-item"><div class="stat-value">${cr.palletsPerLayer}</div><div class="stat-label">每层栈板</div></div><div class="stat-item"><div class="stat-value">${cr.stackLayers}</div><div class="stat-label">层数</div></div><div class="stat-item"><div class="stat-value">${cr.totalPallets}</div><div class="stat-label">栈板总数</div></div><div class="stat-item"><div class="stat-value">${cr.totalBoxes}</div><div class="stat-label">总纸箱</div></div><div class="stat-item"><div class="stat-value">${cr.floorUtil}%</div><div class="stat-label">底面积利用</div></div><div class="stat-item"><div class="stat-value">${cr.volumeUtil}%</div><div class="stat-label">容积利用</div></div><div style="grid-column:span2;" class="stat-item"><div class="stat-value" style="font-size:0.85rem;">${cr.packStrategy||'自动最优'}</div><div class="stat-label">最优摆放策略</div></div>`; dom.containerWarningMsg.style.display='block'; dom.containerWarningMsg.style.color='#86efac'; dom.containerWarningMsg.style.borderLeftColor='#22c55e'; dom.containerWarningMsg.innerText=`✅ 最优装载：每层 ${cr.palletsPerLayer} 个 × ${cr.stackLayers} 层 = ${cr.totalPallets} 个栈板（底面积利用率 ${cr.floorUtil}%）`; }
    function onCalculate(){ const get=id=>parseFloat(document.getElementById(id).value)||0; const trayL=get('trayL'),trayW=get('trayW'),trayH=get('trayH'),trayLoad=get('trayLoad'),boxL=get('boxL'),boxW=get('boxW'),boxH=get('boxH'),boxWgt=get('boxWeight'),productsPerBox=parseInt(get('productsPerBox'))||1,customHeight=get('customHeight'); if([trayL,trayW,trayH,boxL,boxW,boxH].some(v=>v<=0)){ alert("尺寸必须为正"); return; } const params={trayL,trayW,trayH,trayLoad,boxL,boxW,boxH,boxWgt,productsPerBox, pattern:getPattern(), limitMode:getLimitMode(), customHeight}; const result=compute(params); currentResult=result; currentParams=params; currentContainerResult=null; updateStatsUI(result); if(result.success){ if(currentViewMode==='container') switchToPalletView(); else update3DPalletScene(result,params); dom.infoTrayL.innerText=params.trayL; dom.infoTrayW.innerText=params.trayW; dom.infoTotalHeight.innerText=result.actualTotalHeight; dom.infoContainerExtra.style.display='none'; dom.infoPanel.classList.remove('container-mode'); } else { if(currentGroup) disposeGroup(currentGroup); currentGroup=createPalletBase(trayW,trayL,trayH); scene.add(currentGroup); dom.infoTrayL.innerText=trayL; dom.infoTrayW.innerText=trayW; dom.infoTotalHeight.innerText=trayH; } }
    function onCalculateContainer(){ if(!currentResult?.success) onCalculate(); if(!currentResult?.success){ alert('请先完成栈板堆叠计算'); return; } const contL=parseFloat(dom.contL.value), contW=parseFloat(dom.contW.value), contH=parseFloat(dom.contH.value), gap=parseFloat(dom.contGap.value)||0; const strategy = getContainerStrategy(); if([contL,contW,contH].some(v=>v<=0)){ alert('集装箱尺寸无效'); return; } const cr=computeContainerLoading(currentResult,currentParams,contL,contW,contH,gap,strategy); currentContainerResult=cr; updateContainerStatsUI(cr); if(!cr.success) return; currentViewMode='container'; dom.viewContainerBtn.className='view-switch-btn container-active'; dom.viewPalletBtn.className='view-switch-btn'; update3DContainerScene(cr); dom.infoTrayL.innerText=cr.contL; dom.infoTrayW.innerText=cr.contW; dom.infoTotalHeight.innerText=cr.contH; dom.infoContainerExtra.style.display='block'; dom.infoContainerExtra.innerHTML=`<div class="label">🚢 栈板总数</div><div><span class="value">${cr.totalPallets}</span> 个 (${cr.stackLayers}层)</div><div class="label">📦 总纸箱 / 总产品</div><div><span class="value">${cr.totalBoxes}</span> 箱 &nbsp; <span class="value">${cr.totalProducts}</span> 个</div><div class="label">📐 摆放策略</div><div><span class="value" style="font-size:0.95rem;">${cr.packStrategy}</span></div>`; dom.infoPanel.classList.add('container-mode');
        const dist = Math.max(cr.contL * 2.8, cr.contH * 3.2, 12000);
        camera.position.set(dist * 0.7, dist * 0.5, dist * 0.75);
        controls.target.set(0, cr.contH / 2, 0);
    }
    function switchToPalletView(){ if(!currentResult?.success) return; currentViewMode='pallet'; dom.viewPalletBtn.className='view-switch-btn pallet-active'; dom.viewContainerBtn.className='view-switch-btn'; update3DPalletScene(currentResult,currentParams); dom.infoContainerExtra.style.display='none'; dom.infoPanel.classList.remove('container-mode'); dom.infoTrayL.innerText=currentParams.trayL; dom.infoTrayW.innerText=currentParams.trayW; dom.infoTotalHeight.innerText=currentResult.actualTotalHeight;
        camera.position.set(3200, 3000, 3800);
        controls.target.set(0, currentResult.actualTotalHeight / 2, 0);
    }
    function switchToContainerView(){ if(!currentContainerResult?.success){ if(currentResult?.success) onCalculateContainer(); if(!currentContainerResult?.success){ alert('请先计算集装箱装载'); return; } } currentViewMode='container'; dom.viewContainerBtn.className='view-switch-btn container-active'; dom.viewPalletBtn.className='view-switch-btn'; update3DContainerScene(currentContainerResult); dom.infoTrayL.innerText=currentContainerResult.contL; dom.infoTrayW.innerText=currentContainerResult.contW; dom.infoTotalHeight.innerText=currentContainerResult.contH; dom.infoContainerExtra.style.display='block'; dom.infoContainerExtra.innerHTML=`<div class="label">🚢 栈板总数</div><div><span class="value">${currentContainerResult.totalPallets}</span> 个</div><div class="label">摆放策略</div><div><span class="value">${currentContainerResult.packStrategy}</span></div>`; dom.infoPanel.classList.add('container-mode');
        const dist = Math.max(currentContainerResult.contL * 2.8, currentContainerResult.contH * 3.2, 12000);
        camera.position.set(dist * 0.7, dist * 0.5, dist * 0.75);
        controls.target.set(0, currentContainerResult.contH / 2, 0);
    }
    
    // ========== 导出图片叠加堆叠资讯表格 ==========
    function drawTable(sourceCanvas, rows, scale = 0.5) {
        const w = sourceCanvas.width, h = sourceCanvas.height;
        const off = document.createElement('canvas');
        off.width = w; off.height = h;
        const ctx = off.getContext('2d');
        ctx.drawImage(sourceCanvas, 0, 0);
        const baseFs = Math.max(14, Math.floor(h / 38));
        const fs = baseFs * scale;
        const lh = fs * 1.7;
        const p = fs * 0.9;
        ctx.font = `bold ${fs}px "Microsoft YaHei", "PingFang SC", sans-serif`;
        let c1 = 0, c2 = 0;
        rows.forEach(r => {
            c1 = Math.max(c1, ctx.measureText(r[0]).width);
            c2 = Math.max(c2, ctx.measureText(r[1]).width);
        });
        const tw = (c1 + c2 + p * 2 + fs * 0.8 * 2);
        const th = rows.length * lh + p * 2;
        const margin = Math.min(20, w * 0.03);
        const tx = w - tw - margin;
        const ty = h - th - margin;
        ctx.fillStyle = 'rgba(0,0,0,0.78)';
        ctx.shadowColor = 'rgba(0,0,0,0.5)';
        ctx.shadowBlur = 8;
        ctx.fillRect(tx, ty, tw, th);
        ctx.shadowBlur = 0;
        ctx.strokeStyle = '#ffffff';
        ctx.lineWidth = 1.5;
        ctx.strokeRect(tx, ty, tw, th);
        const mx = tx + p + c1 + fs * 0.8;
        ctx.beginPath();
        ctx.moveTo(mx, ty);
        ctx.lineTo(mx, ty + th);
        ctx.stroke();
        for (let i = 1; i < rows.length; i++) {
            const y = ty + p + i * lh;
            ctx.beginPath();
            ctx.moveTo(tx, y);
            ctx.lineTo(tx + tw, y);
            ctx.stroke();
        }
        ctx.fillStyle = '#ffffff';
        ctx.textBaseline = 'middle';
        rows.forEach((r, i) => {
            const ry = ty + p + i * lh + lh / 2;
            ctx.textAlign = 'left';
            ctx.fillText(r[0], tx + p, ry);
            ctx.fillText(r[1], mx + fs * 0.8, ry);
        });
        return off;
    }

    async function captureHighResCanvas(scale = 2.5) {
        if (!((currentViewMode === 'pallet' && currentResult?.success) || (currentViewMode === 'container' && currentContainerResult?.success))) return renderer.domElement;
        const ow = renderer.domElement.clientWidth, oh = renderer.domElement.clientHeight;
        let nw = Math.floor(ow * scale), nh = Math.floor(oh * scale);
        const max = 5000;
        if (nw > max || nh > max) { const r = Math.min(max / nw, max / nh); nw = Math.floor(nw * r); nh = Math.floor(nh * r); }
        renderer.setSize(nw, nh, false);
        renderer.setPixelRatio(1);
        camera.aspect = nw / nh;
        camera.updateProjectionMatrix();
        renderer.render(scene, camera);
        const src = renderer.domElement;
        let final;
        if (currentViewMode === 'container' && currentContainerResult?.success) {
            const rows = [
                ['集装箱尺寸', currentContainerResult.contL + ' x ' + currentContainerResult.contW + ' x ' + currentContainerResult.contH + ' mm'],
                ['栈板总数', currentContainerResult.totalPallets + ' 个 (' + currentContainerResult.stackLayers + '层)'],
                ['总纸箱数', currentContainerResult.totalBoxes + ' 箱'],
                ['总产品数', currentContainerResult.totalProducts + ' 个'],
                ['底面积利用', currentContainerResult.floorUtil + '%'],
                ['容积利用', currentContainerResult.volumeUtil + '%'],
                ['摆放策略', currentContainerResult.packStrategy || '自动最优']
            ];
            final = drawTable(src, rows, 0.5);
        } else if (currentResult?.success) {
            const rows = [
                ['堆叠层数', currentResult.layers + ' 层'],
                ['堆叠箱数', currentResult.totalQty + ' 箱'],
                ['托盘尺寸', currentParams.trayL + ' x ' + currentParams.trayW + ' mm'],
                ['堆叠总高', currentResult.actualTotalHeight + ' mm'],
                ['每箱产品数', currentResult.productsPerBox + ' 个'],
                ['产品总数', currentResult.totalProducts + ' 个']
            ];
            final = drawTable(src, rows, 0.5);
        } else {
            final = src;
        }
        renderer.setSize(ow, oh, false);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        camera.aspect = ow / oh;
        camera.updateProjectionMatrix();
        renderer.render(scene, camera);
        return final;
    }

    async function exportImage(format, quality = 0.95) {
        try {
            const cvs = await captureHighResCanvas(2.5);
            const data = cvs.toDataURL(format === 'png' ? 'image/png' : 'image/jpeg', quality);
            const a = document.createElement('a');
            a.download = `${currentViewMode === 'container' ? 'container' : 'stacking'}_export.${format === 'png' ? 'png' : 'jpg'}`;
            a.href = data;
            a.click();
        } catch (e) { alert('导出失败'); }
    }
    async function exportPDF() {
        try {
            const cvs = await captureHighResCanvas(2.5);
            const data = cvs.toDataURL('image/png', 1.0);
            const img = new Image();
            img.src = data;
            await new Promise(r => { img.onload = r; });
            const { jsPDF } = window.jspdf;
            const pdf = new jsPDF({ orientation: img.width > img.height ? 'landscape' : 'portrait', unit: 'mm', format: 'a4' });
            const pw = pdf.internal.pageSize.getWidth(), ph = pdf.internal.pageSize.getHeight();
            const r = Math.min(pw / img.width, ph / img.height);
            pdf.addImage(img, 'PNG', (pw - img.width * r) / 2, (ph - img.height * r) / 2, img.width * r, img.height * r);
            pdf.save(`${currentViewMode === 'container' ? 'container' : 'stacking'}_export.pdf`);
        } catch (e) { alert('PDF导出失败'); }
    }

    dom.calcBtn.addEventListener('click', onCalculate);
    dom.calcContainerBtn.addEventListener('click', onCalculateContainer);
    dom.viewPalletBtn.addEventListener('click', switchToPalletView);
    dom.viewContainerBtn.addEventListener('click', switchToContainerView);
    document.getElementById('exportPngBtn').addEventListener('click', () => exportImage('png'));
    document.getElementById('exportJpgBtn').addEventListener('click', () => exportImage('jpg', 0.92));
    document.getElementById('exportPdfBtn').addEventListener('click', exportPDF);

    document.getElementById('nextStrategyBtn').addEventListener('click', function() {
        if (currentViewMode === 'pallet') {
            const strategies = ['grid', 'staggered', 'mixed'];
            const current = getPattern();
            let idx = strategies.indexOf(current);
            if (idx === -1) idx = 0;
            const next = strategies[(idx + 1) % strategies.length];
            const radios = document.getElementsByName('pattern');
            for (let r of radios) {
                if (r.value === next) r.checked = true;
            }
            onCalculate();
        } else if (currentViewMode === 'container') {
            const strategies = ['staggered', 'mixed', 'ring'];
            const current = getContainerStrategy();
            let idx = strategies.indexOf(current);
            if (idx === -1) idx = 0;
            const next = strategies[(idx + 1) % strategies.length];
            const radios = document.getElementsByName('containerPattern');
            for (let r of radios) {
                if (r.value === next) r.checked = true;
            }
            onCalculateContainer();
        }
    });

    const presets = { '20ft': { l: 5800, w: 2350, h: 2380 }, '40ft': { l: 12032, w: 2350, h: 2380 }, '40hc': { l: 12032, w: 2350, h: 2690 } };
    document.querySelectorAll('.preset-btn').forEach(btn => {
        btn.addEventListener('click', () => {
            document.querySelectorAll('.preset-btn').forEach(b => b.classList.remove('active-preset'));
            btn.classList.add('active-preset');
            const p = btn.dataset.preset;
            if (presets[p]) { dom.contL.value = presets[p].l; dom.contW.value = presets[p].w; dom.contH.value = presets[p].h; dom.contGap.value = 20; }
        });
    });
    function resizeCanvas() { 
        const v = document.querySelector('.viewer'); 
        if (!v) return; 
        const w = v.clientWidth, h = v.clientHeight;
        renderer.setSize(w, h); 
        camera.aspect = w / h; 
        camera.updateProjectionMatrix(); 
        renderer.render(scene, camera); 
    }
    window.addEventListener('resize', resizeCanvas);
    window.addEventListener('orientationchange', () => setTimeout(resizeCanvas, 200));
    function animate() { 
        requestAnimationFrame(animate); 
        controls.update(); 
        renderer.render(scene, camera);
    }
    animate();
    setTimeout(() => { resizeCanvas(); onCalculate(); setTimeout(() => onCalculateContainer(), 200); }, 200);
</script>
</body>
</html>
