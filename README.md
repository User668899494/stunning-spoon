<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>罚斷注入--私人接口</title>
    <style>
        :root {
            --bg: #05080f;
            --surface: #0a1018;
            --surface2: #0d1520;
            --accent: #00e676;
            --accent2: #00c853;
            --accent-glow: #00ff7b;
            --danger: #ff1744;
            --danger-glow: #ff3d5c;
            --text: #e0e0e0;
            --text-secondary: #8a9bb5;
            --border: #1a2a3d;
            --gold: #ffb300;
            --blue-accent: #448aff;
            --purple: #b388ff;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'SF Mono', 'Fira Code', 'Consolas', 'Monaco', 'Courier New', monospace;
            background: var(--bg);
            color: var(--text);
            min-height: 100vh;
            overflow-x: hidden;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            -webkit-user-select: none;
        }

        #particleCanvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 0;
            pointer-events: none;
        }

        .main-container {
            position: relative;
            z-index: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        .screen {
            display: none;
            width: 100%;
            max-width: 520px;
            animation: fadeSlideIn 0.5s cubic-bezier(0.22, 0.61, 0.36, 1);
        }
        .screen.active {
            display: block;
        }

        @keyframes fadeSlideIn {
            from { opacity: 0; transform: translateY(30px); }
            to { opacity: 1; transform: translateY(0); }
        }
        @keyframes fadeSlideOut {
            from { opacity: 1; transform: translateY(0); }
            to { opacity: 0; transform: translateY(-20px); }
        }

        .card-input-container {
            background: var(--surface);
            border: 1px solid var(--border);
            border-radius: 20px;
            padding: 40px 30px;
            text-align: center;
            position: relative;
            overflow: hidden;
            box-shadow: 0 20px 60px rgba(0,0,0,0.6), 0 0 80px rgba(0,230,118,0.05);
        }
        .card-input-container::before {
            content: '';
            position: absolute;
            top: -50%; left: -50%;
            width: 200%; height: 200%;
            background: conic-gradient(from 0deg, transparent, rgba(0,230,118,0.04), transparent, rgba(0,180,100,0.03), transparent);
            animation: rotateBorder 8s linear infinite;
            z-index: 0;
        }
        @keyframes rotateBorder { to { transform: rotate(360deg); } }
        .card-input-inner { position: relative; z-index: 1; }
        .logo-icon {
            width: 70px; height: 70px; margin: 0 auto 20px;
            background: radial-gradient(circle, rgba(0,230,118,0.25) 0%, transparent 70%);
            border-radius: 50%; display: flex; align-items: center; justify-content: center; position: relative;
        }
        .logo-icon::after {
            content: '';
            width: 40px; height: 40px; border: 3px solid var(--accent);
            border-radius: 50%; border-top-color: transparent; animation: logoSpin 2s linear infinite;
        }
        @keyframes logoSpin { to { transform: rotate(360deg); } }
        .logo-shield {
            position: absolute; font-size: 28px; color: var(--accent-glow);
            filter: drop-shadow(0 0 12px rgba(0,255,123,0.7));
        }
        .title-text {
            font-size: 1.6em; font-weight: 700; letter-spacing: 2px; color: #fff;
            margin-bottom: 6px; text-shadow: 0 0 30px rgba(0,230,118,0.4);
        }
        .subtitle-text {
            font-size: 0.8em; color: var(--text-secondary); letter-spacing: 3px;
            margin-bottom: 28px; text-transform: uppercase;
        }
        .input-group { position: relative; margin-bottom: 20px; }
        .input-group input {
            width: 100%; padding: 16px 20px; background: var(--surface2); border: 2px solid var(--border);
            border-radius: 12px; color: #fff; font-size: 1.1em; letter-spacing: 4px; text-align: center;
            font-family: inherit; transition: all 0.3s; outline: none; caret-color: var(--accent);
        }
        .input-group input:focus {
            border-color: var(--accent); box-shadow: 0 0 25px rgba(0,230,118,0.2), inset 0 0 15px rgba(0,230,118,0.03);
        }
        .input-group input::placeholder { color: #3a4a5a; letter-spacing: 2px; }
        .input-group .glow-line {
            position: absolute; bottom: -1px; left: 50%; transform: translateX(-50%);
            width: 0; height: 2px; background: var(--accent-glow); transition: width 0.4s; border-radius: 1px;
        }
        .input-group input:focus ~ .glow-line { width: 70%; }
        .btn-submit {
            width: 100%; padding: 15px; background: linear-gradient(135deg, #006633, #009944, #006633);
            border: none; border-radius: 12px; color: #fff; font-size: 1.1em; font-weight: 700;
            letter-spacing: 3px; cursor: pointer; font-family: inherit; transition: all 0.3s;
            position: relative; overflow: hidden; text-shadow: 0 1px 2px rgba(0,0,0,0.4);
        }
        .btn-submit:hover {
            background: linear-gradient(135deg, #008044, #00b855, #008044);
            box-shadow: 0 8px 30px rgba(0,200,80,0.35); transform: translateY(-2px);
        }
        .btn-submit:active { transform: scale(0.96); transition: transform 0.1s; }
        .btn-submit::after {
            content: ''; position: absolute; top: -80%; left: -30%;
            width: 60%; height: 260%; background: rgba(255,255,255,0.08);
            transform: rotate(25deg); transition: left 0.6s;
        }
        .btn-submit:hover::after { left: 120%; }
        .error-msg {
            color: var(--danger); font-size: 0.85em; margin-top: 8px; min-height: 20px;
            letter-spacing: 1px; opacity: 0; transition: opacity 0.3s;
        }
        .error-msg.show { opacity: 1; animation: shake 0.5s ease; }
        @keyframes shake {
            0%,100% { transform: translateX(0); }
            20% { transform: translateX(-8px); }
            40% { transform: translateX(8px); }
            60% { transform: translateX(-5px); }
            80% { transform: translateX(5px); }
        }
        .hint-text { font-size: 0.7em; color: #3a4a58; letter-spacing: 1px; margin-top: 16px; }

        .toast-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            z-index: 9999; display: flex; align-items: flex-start;
            justify-content: center; pointer-events: none; padding-top: 30px;
        }
        .toast-card {
            background: rgba(10,18,26,0.95); border: 1px solid var(--accent);
            border-radius: 16px; padding: 16px 24px; display: flex;
            align-items: center; gap: 12px; box-shadow: 0 15px 50px rgba(0,0,0,0.7), 0 0 30px rgba(0,230,118,0.25);
            backdrop-filter: blur(20px); -webkit-backdrop-filter: blur(20px);
            animation: toastIn 0.5s cubic-bezier(0.22, 0.61, 0.36, 1);
            pointer-events: auto; max-width: 90vw;
        }
        .toast-card.toast-out { animation: toastOut 0.4s cubic-bezier(0.55, 0.06, 0.68, 0.19) forwards; }
        @keyframes toastIn {
            from { opacity: 0; transform: translateY(-50px) scale(0.9); }
            to { opacity: 1; transform: translateY(0) scale(1); }
        }
        @keyframes toastOut {
            from { opacity: 1; transform: translateY(0) scale(1); }
            to { opacity: 0; transform: translateY(-40px) scale(0.85); }
        }
        .toast-icon {
            width: 36px; height: 36px; border-radius: 50%;
            background: rgba(0,230,118,0.2); display: flex;
            align-items: center; justify-content: center; font-size: 18px;
            flex-shrink: 0; animation: toastPulse 1.5s ease-in-out infinite;
        }
        @keyframes toastPulse {
            0%,100% { box-shadow: 0 0 10px rgba(0,230,118,0.4); }
            50% { box-shadow: 0 0 25px rgba(0,230,118,0.8); }
        }
        .toast-text { color: #fff; font-size: 0.9em; letter-spacing: 1px; font-weight: 500; line-height: 1.3; }
        .toast-highlight { color: var(--accent-glow); font-weight: 700; }
        .toast-card.danger-toast { border-color: var(--danger); box-shadow: 0 15px 50px rgba(0,0,0,0.7), 0 0 35px rgba(255,23,68,0.3); }
        .toast-card.danger-toast .toast-icon { background: rgba(255,23,68,0.2); animation-name: toastPulseDanger; }
        @keyframes toastPulseDanger {
            0%,100% { box-shadow: 0 0 10px rgba(255,23,68,0.4); }
            50% { box-shadow: 0 0 30px rgba(255,60,90,0.9); }
        }
        .toast-card.danger-toast .toast-highlight { color: var(--danger-glow); }

        .loading-container {
            background: var(--surface); border: 1px solid var(--border);
            border-radius: 20px; padding: 40px 30px; text-align: center;
            box-shadow: 0 20px 60px rgba(0,0,0,0.6);
        }
        .loading-title {
            font-size: 1em; letter-spacing: 2px; color: var(--accent-glow);
            margin-bottom: 30px; animation: blinkText 1.5s ease-in-out infinite;
        }
        @keyframes blinkText { 0%,100% { opacity: 1; } 50% { opacity: 0.5; } }
        .loading-spinner-row {
            display: flex; align-items: center; gap: 8px;
            justify-content: center; margin-bottom: 20px;
            font-size: 0.75em; color: var(--text-secondary); letter-spacing: 1px;
        }
        .spinner-dot {
            width: 6px; height: 6px; border-radius: 50%;
            background: var(--accent); animation: dotBounce 1.2s ease-in-out infinite;
        }
        .spinner-dot:nth-child(2) { animation-delay: 0.2s; }
        .spinner-dot:nth-child(3) { animation-delay: 0.4s; }
        @keyframes dotBounce {
            0%,80%,100% { transform: scale(0.6); opacity: 0.4; }
            40% { transform: scale(1.4); opacity: 1; }
        }
        .progress-bar-outer {
            width: 100%; height: 8px; background: var(--surface2);
            border-radius: 4px; overflow: hidden; position: relative;
            border: 1px solid var(--border);
        }
        .progress-bar-inner {
            height: 100%; background: linear-gradient(90deg, #006633, var(--accent), var(--accent-glow));
            border-radius: 4px; width: 0%; transition: width 0.08s linear;
            position: relative; box-shadow: 0 0 20px rgba(0,230,118,0.5);
        }
        .progress-bar-inner::after {
            content: ''; position: absolute; right: 0; top: 50%; transform: translateY(-50%);
            width: 20px; height: 20px; border-radius: 50%; background: var(--accent-glow);
            box-shadow: 0 0 25px rgba(0,255,123,0.9);
        }
        .progress-percent { margin-top: 12px; font-size: 0.85em; color: var(--accent-glow); letter-spacing: 1px; }
        .loading-log {
            margin-top: 16px; font-size: 0.7em; color: #4a5a6a; letter-spacing: 1px;
            min-height: 18px; text-align: left; padding-left: 10px; border-left: 2px solid #1a2a3d;
        }

        .success-container {
            background: var(--surface); border: 1px solid var(--border);
            border-radius: 20px; padding: 35px 25px 30px; text-align: center;
            box-shadow: 0 20px 60px rgba(0,0,0,0.6), 0 0 60px rgba(0,200,80,0.06);
        }
        .faceid-animation { width: 100px; height: 100px; margin: 0 auto 20px; position: relative; }
        .faceid-ring {
            width: 100px; height: 100px; border-radius: 50%;
            border: 4px solid transparent; border-top-color: var(--accent-glow);
            border-right-color: var(--accent); animation: faceidSpin 0.8s linear infinite;
            position: absolute; top: 0; left: 0;
        }
        .faceid-ring.ring-done {
            animation: none; border: 4px solid var(--accent); border-top-color: var(--accent);
            border-right-color: var(--accent); transition: all 0.4s ease;
            box-shadow: 0 0 30px rgba(0,230,118,0.6);
        }
        @keyframes faceidSpin { to { transform: rotate(360deg); } }
        .faceid-check {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%) scale(0);
            font-size: 50px; color: var(--accent-glow);
            transition: transform 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            filter: drop-shadow(0 0 15px rgba(0,255,123,0.8));
        }
        .faceid-check.show { transform: translate(-50%, -50%) scale(1); }
        .success-title {
            font-size: 2.2em; font-weight: 900; letter-spacing: 6px; color: #fff;
            margin-bottom: 6px; text-shadow: 0 0 40px rgba(0,230,118,0.5);
        }
        .success-subtitle {
            font-size: 0.8em; color: var(--accent); letter-spacing: 2px; margin-bottom: 28px; opacity: 0.85;
        }
        .feature-buttons { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
        .feature-btn {
            padding: 16px 12px; background: var(--surface2); border: 1px solid var(--border);
            border-radius: 14px; color: #fff; cursor: pointer; font-family: inherit;
            font-size: 0.85em; letter-spacing: 2px; font-weight: 600; transition: all 0.3s;
            position: relative; overflow: hidden; text-align: center;
        }
        .feature-btn:hover { border-color: var(--accent); box-shadow: 0 0 25px rgba(0,230,118,0.2); transform: translateY(-2px); background: #111d2a; }
        .feature-btn:active { transform: scale(0.95); transition: transform 0.1s; }
        .feature-btn .btn-icon { font-size: 1.6em; display: block; margin-bottom: 6px; }
        .feature-btn.accent-btn { border-color: rgba(0,230,118,0.4); background: rgba(0,180,80,0.08); }
        .feature-btn.danger-btn { border-color: rgba(255,23,68,0.3); background: rgba(255,23,68,0.04); }
        .feature-btn.danger-btn:hover { border-color: var(--danger); box-shadow: 0 0 25px rgba(255,23,68,0.2); }
        .feature-btn.purple-btn { border-color: rgba(179,136,255,0.3); background: rgba(179,136,255,0.04); }
        .feature-btn.purple-btn:hover { border-color: var(--purple); box-shadow: 0 0 25px rgba(179,136,255,0.2); }

        .hook-container {
            background: var(--surface); border: 1px solid var(--border);
            border-radius: 20px; padding: 35px 25px 30px; text-align: center;
            box-shadow: 0 20px 60px rgba(0,0,0,0.6);
        }
        .hook-title { font-size: 1.3em; letter-spacing: 3px; color: #fff; margin-bottom: 8px; text-shadow: 0 0 25px rgba(179,136,255,0.5); }
        .hook-subtitle { font-size: 0.75em; color: var(--text-secondary); letter-spacing: 2px; margin-bottom: 28px; }
        .hook-buttons { display: flex; flex-direction: column; gap: 14px; }
        .hook-btn {
            padding: 18px 20px; border-radius: 14px; border: 1px solid var(--border);
            cursor: pointer; font-family: inherit; font-size: 0.9em; letter-spacing: 2px;
            font-weight: 600; transition: all 0.3s; color: #fff; text-align: center;
        }
        .hook-btn.hack-btn { background: rgba(255,23,68,0.06); border-color: rgba(255,23,68,0.4); }
        .hook-btn.hack-btn:hover { background: rgba(255,23,68,0.14); border-color: var(--danger); box-shadow: 0 0 30px rgba(255,23,68,0.3); transform: translateY(-2px); }
        .hook-btn.crazy-btn { background: rgba(255,140,0,0.06); border-color: rgba(255,140,0,0.4); }
        .hook-btn.crazy-btn:hover { background: rgba(255,140,0,0.14); border-color: #ff8c00; box-shadow: 0 0 30px rgba(255,140,0,0.35); transform: translateY(-2px); }
        .hook-btn:active { transform: scale(0.95); transition: transform 0.1s; }
        .btn-back {
            display: inline-block; margin-top: 20px; padding: 10px 24px; background: transparent;
            border: 1px solid var(--border); border-radius: 20px; color: var(--text-secondary);
            cursor: pointer; font-family: inherit; font-size: 0.8em; letter-spacing: 2px; transition: all 0.3s;
        }
        .btn-back:hover { border-color: #fff; color: #fff; }

        .hardware-container {
            background: var(--surface); border: 1px solid var(--border);
            border-radius: 20px; padding: 35px 25px 30px; text-align: center;
            box-shadow: 0 20px 60px rgba(0,0,0,0.6);
        }
        .hardware-title { font-size: 1.3em; letter-spacing: 3px; color: #fff; margin-bottom: 6px; text-shadow: 0 0 25px rgba(68,138,255,0.5); }
        .hardware-subtitle { font-size: 0.75em; color: var(--text-secondary); letter-spacing: 2px; margin-bottom: 20px; }
        .slider-group { margin: 20px 0; }
        .slider-label-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px; font-size: 0.8em; letter-spacing: 1px; color: var(--text-secondary); }
        .slider-value-display { font-size: 2em; font-weight: 900; color: var(--blue-accent); text-shadow: 0 0 20px rgba(68,138,255,0.6); letter-spacing: 1px; transition: color 0.2s; }
        .custom-range-wrapper { position: relative; width: 100%; padding: 10px 0; }
        input[type="range"].custom-slider {
            -webkit-appearance: none; appearance: none; width: 100%; height: 10px;
            border-radius: 5px; background: var(--surface2); outline: none;
            border: 1px solid var(--border); cursor: pointer; transition: all 0.2s;
        }
        input[type="range"].custom-slider::-webkit-slider-thumb {
            -webkit-appearance: none; appearance: none; width: 36px; height: 36px;
            border-radius: 50%; background: radial-gradient(circle at 40% 40%, #5a9aff, #1a5dc7);
            border: 3px solid #fff; cursor: grab; box-shadow: 0 0 25px rgba(68,138,255,0.7), 0 4px 15px rgba(0,0,0,0.4); transition: all 0.2s;
        }
        input[type="range"].custom-slider::-webkit-slider-thumb:active { cursor: grabbing; box-shadow: 0 0 40px rgba(68,138,255,1), 0 6px 20px rgba(0,0,0,0.5); transform: scale(1.1); }
        .slider-ticks { display: flex; justify-content: space-between; font-size: 0.65em; color: #3a4a58; letter-spacing: 1px; margin-top: 4px; padding: 0 6px; }
        .param-info {
            margin-top: 16px; padding: 12px 16px; background: var(--surface2); border-radius: 10px;
            font-size: 0.7em; color: var(--text-secondary); letter-spacing: 1px; border: 1px solid var(--border);
            text-align: left; line-height: 1.5;
        }
        .param-info span { color: var(--blue-accent); font-weight: 600; }

        .crazy-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            z-index: 9990; pointer-events: none; background: transparent; transition: background 0.3s;
        }
        .crazy-overlay.active { animation: crazyFlash 0.3s ease-in-out infinite; background: rgba(255,0,0,0.06); }
        @keyframes crazyFlash {
            0%,100% { background: rgba(255,0,0,0.04); }
            25% { background: rgba(255,80,0,0.1); }
            50% { background: rgba(255,0,80,0.07); }
            75% { background: rgba(255,140,0,0.09); }
        }
        .screen-shake { animation: screenShake 0.15s ease-in-out infinite; }
        @keyframes screenShake {
            0%,100% { transform: translate(0,0); }
            25% { transform: translate(-3px,2px); }
            50% { transform: translate(3px,-1px); }
            75% { transform: translate(-1px,-2px); }
        }

        @media (max-width: 480px) {
            .card-input-container { padding: 28px 18px; border-radius: 16px; }
            .title-text { font-size: 1.3em; }
            .success-title { font-size: 1.6em; letter-spacing: 3px; }
            .feature-buttons { gap: 8px; }
            .feature-btn { padding: 12px 8px; font-size: 0.75em; letter-spacing: 1px; }
            .faceid-ring, .faceid-animation { width: 75px; height: 75px; }
            .faceid-check { font-size: 38px; }
        }
    </style>
</head>
<body>

<canvas id="particleCanvas"></canvas>
<div class="crazy-overlay" id="crazyOverlay"></div>
<div class="toast-overlay" id="toastOverlay" style="display:none;"></div>

<div class="main-container" id="mainContainer">
    <!-- 屏幕1：卡密输入 -->
    <div class="screen active" id="screenWelcome">
        <div class="card-input-container">
            <div class="card-input-inner">
                <div class="logo-icon"><span class="logo-shield">🛡️</span></div>
                <div class="title-text">罚斷服务</div>
                <div class="subtitle-text">· DUST SERVICE ·</div>
                <div class="input-group">
                    <input type="text" id="cardInput" placeholder="请输入卡密" maxlength="20" autocomplete="off">
                    <div class="glow-line"></div>
                </div>
                <button class="btn-submit" id="btnSubmit">验 证 卡 密</button>
                <div class="error-msg" id="errorMsg"></div>
                <div class="hint-text">🔒 安全加密通道 · AES-256</div>
            </div>
        </div>
    </div>

    <!-- 屏幕2：加载 -->
    <div class="screen" id="screenLoading">
        <div class="loading-container">
            <div class="loading-title">⚡ 正在尝试入侵 ACE 服务器</div>
            <div class="loading-spinner-row"><span>建立连接</span><div class="spinner-dot"></div><div class="spinner-dot"></div><div class="spinner-dot"></div></div>
            <div class="progress-bar-outer"><div class="progress-bar-inner" id="progressBarInner"></div></div>
            <div class="progress-percent" id="progressPercent">0%</div>
            <div class="loading-log" id="loadingLog">[~] 初始化漏洞扫描...</div>
        </div>
    </div>

    <!-- 屏幕3：成功 -->
    <div class="screen" id="screenSuccess">
        <div class="success-container">
            <div class="faceid-animation">
                <div class="faceid-ring" id="faceidRing"></div>
                <div class="faceid-check" id="faceidCheck">✓</div>
            </div>
            <div class="success-title">罚斷注入连接成功</div>
            <div class="success-subtitle">● 服务器连接正常</div>
            <div class="feature-buttons">
                <button class="feature-btn accent-btn" id="btnAimbot"><span class="btn-icon">🎯</span>自瞄</button>
                <button class="feature-btn" id="btnBreakpoint"><span class="btn-icon">🔗</span>断点子追</button>
                <button class="feature-btn purple-btn" id="btnHardware"><span class="btn-icon">💾</span>硬件子追</button>
                <button class="feature-btn danger-btn" id="btnHook"><span class="btn-icon">🪝</span>Hook一下</button>
            </div>
        </div>
    </div>

    <!-- 屏幕4：Hook -->
    <div class="screen" id="screenHook">
        <div class="hook-container">
            <div class="hook-title">🪝 Hook 选项</div>
            <div class="hook-subtitle">选择执行模式</div>
            <div class="hook-buttons">
                <button class="hook-btn hack-btn" id="btnHackInvade">💀 罚斷入侵</button>
                <button class="hook-btn crazy-btn" id="btnCrazyMode">🔥 疯狂状态</button>
            </div>
            <button class="btn-back" id="btnBackFromHook">← 返回上级</button>
        </div>
    </div>

    <!-- 屏幕5：硬件 -->
    <div class="screen" id="screenHardware">
        <div class="hardware-container">
            <div class="hardware-title">💾 硬件子追</div>
            <div class="hardware-subtitle">追踪灵敏度调节</div>
            <div class="slider-group">
                <div class="slider-label-row"><span>追踪强度</span><span class="slider-value-display" id="sliderValueDisplay">50</span><span>%</span></div>
                <div class="custom-range-wrapper"><input type="range" class="custom-slider" id="hardwareSlider" min="0" max="100" value="50" step="1"></div>
                <div class="slider-ticks"><span>0</span><span>25</span><span>50</span><span>75</span><span>100</span></div>
            </div>
            <div class="param-info">📡 <span>当前参数：</span>硬件追踪延迟 <span id="paramLatency">15ms</span> | 锁定范围 <span id="paramRange">50m</span></div>
            <button class="btn-back" id="btnBackFromHardware">← 返回上级</button>
        </div>
    </div>
</div>

<script>
(function() {
    const particleCanvas = document.getElementById('particleCanvas');
    const ctx = particleCanvas.getContext('2d');
    const mainContainer = document.getElementById('mainContainer');
    const crazyOverlay = document.getElementById('crazyOverlay');
    const toastOverlay = document.getElementById('toastOverlay');

    const screenWelcome = document.getElementById('screenWelcome');
    const screenLoading = document.getElementById('screenLoading');
    const screenSuccess = document.getElementById('screenSuccess');
    const screenHook = document.getElementById('screenHook');
    const screenHardware = document.getElementById('screenHardware');

    const cardInput = document.getElementById('cardInput');
    const btnSubmit = document.getElementById('btnSubmit');
    const errorMsg = document.getElementById('errorMsg');
    const progressBarInner = document.getElementById('progressBarInner');
    const progressPercent = document.getElementById('progressPercent');
    const loadingLog = document.getElementById('loadingLog');
    const faceidRing = document.getElementById('faceidRing');
    const faceidCheck = document.getElementById('faceidCheck');
    const hardwareSlider = document.getElementById('hardwareSlider');
    const sliderValueDisplay = document.getElementById('sliderValueDisplay');
    const paramLatency = document.getElementById('paramLatency');
    const paramRange = document.getElementById('paramRange');

    let isCrazyMode = false;
    let loadingInterval = null;
    let currentScreen = 'welcome';

    // ============ 樱花粒子系统 ============
    let petals = [];
    const PETAL_COUNT = 60;

    function initPetals() {
        petals = [];
        for (let i = 0; i < PETAL_COUNT; i++) {
            petals.push({
                x: Math.random() * particleCanvas.width,
                y: Math.random() * -particleCanvas.height,
                vx: (Math.random() - 0.5) * 0.8,
                vy: 0.5 + Math.random() * 2.5,
                size: 8 + Math.random() * 14,
                rotation: Math.random() * Math.PI * 2,
                rotationSpeed: (Math.random() - 0.5) * 0.03,
                opacity: 0.5 + Math.random() * 0.5,
                swayOffset: Math.random() * Math.PI * 2,
                swaySpeed: 0.01 + Math.random() * 0.03
            });
        }
    }

    function drawPetals() {
        ctx.clearRect(0, 0, particleCanvas.width, particleCanvas.height);
        const baseColor = isCrazyMode ? [220, 60, 70] : [255, 140, 160];

        for (let p of petals) {
            p.x += p.vx + Math.sin(p.swayOffset) * 0.3;
            p.y += p.vy;
            p.swayOffset += p.swaySpeed;
            p.rotation += p.rotationSpeed;

            if (p.y > particleCanvas.height + 50) {
                p.y = -50;
                p.x = Math.random() * particleCanvas.width;
                p.vy = 0.5 + Math.random() * 2.5;
            }
            if (p.x < -50) p.x = particleCanvas.width + 50;
            if (p.x > particleCanvas.width + 50) p.x = -50;

            ctx.save();
            ctx.translate(p.x, p.y);
            ctx.rotate(p.rotation);
            ctx.globalAlpha = p.opacity;

            const s = p.size * 0.6;
            ctx.fillStyle = `rgba(${baseColor[0]}, ${baseColor[1]}, ${baseColor[2]}, ${p.opacity})`;
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.bezierCurveTo(-s, -s * 0.8, -s * 0.8, -s * 1.6, 0, -s * 1.4);
            ctx.bezierCurveTo(s * 0.8, -s * 1.6, s, -s * 0.8, 0, 0);
            ctx.fill();

            ctx.fillStyle = `rgba(255, ${Math.floor(baseColor[1]*0.8)}, ${Math.floor(baseColor[2]*0.8)}, ${p.opacity*0.6})`;
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.bezierCurveTo(-s*0.6, -s*0.4, -s*0.5, -s*1.0, 0, -s*0.9);
            ctx.bezierCurveTo(s*0.5, -s*1.0, s*0.6, -s*0.4, 0, 0);
            ctx.fill();
            ctx.restore();
        }
        requestAnimationFrame(drawPetals);
    }

    function resizeCanvas() {
        particleCanvas.width = window.innerWidth;
        particleCanvas.height = window.innerHeight;
        initPetals();
    }
    window.addEventListener('resize', resizeCanvas);
    resizeCanvas();
    initPetals();
    drawPetals();

    // ============ 音效与Toast ============
    function playBeepSound(freq=800, duration=0.15) {
        try {
            const ac = new (window.AudioContext||window.webkitAudioContext)();
            const osc = ac.createOscillator();
            const gain = ac.createGain();
            osc.connect(gain); gain.connect(ac.destination);
            osc.type = 'sine'; osc.frequency.setValueAtTime(freq, ac.currentTime);
            gain.gain.setValueAtTime(0.12, ac.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.001, ac.currentTime+duration);
            osc.start(); osc.stop(ac.currentTime+duration);
        } catch(e){}
    }
    function playGoSound() {
        try {
            const ac = new (window.AudioContext||window.webkitAudioContext)();
            const osc1 = ac.createOscillator(); const gain1 = ac.createGain();
            osc1.connect(gain1); gain1.connect(ac.destination);
            osc1.type='square'; osc1.frequency.setValueAtTime(150,ac.currentTime); osc1.frequency.linearRampToValueAtTime(300,ac.currentTime+0.15);
            gain1.gain.setValueAtTime(0.25,ac.currentTime); gain1.gain.exponentialRampToValueAtTime(0.001,ac.currentTime+0.35);
            osc1.start(); osc1.stop(ac.currentTime+0.35);
            const osc2 = ac.createOscillator(); const gain2 = ac.createGain();
            osc2.connect(gain2); gain2.connect(ac.destination);
            osc2.type='sawtooth'; osc2.frequency.setValueAtTime(400,ac.currentTime); osc2.frequency.linearRampToValueAtTime(600,ac.currentTime+0.2);
            gain2.gain.setValueAtTime(0.08,ac.currentTime); gain2.gain.exponentialRampToValueAtTime(0.001,ac.currentTime+0.3);
            osc2.start(); osc2.stop(ac.currentTime+0.3);
            setTimeout(()=>{
                if('speechSynthesis' in window) {
                    const u = new SpeechSynthesisUtterance('go'); u.lang='en-US'; u.rate=1.3; u.pitch=1.1; u.volume=0.9;
                    speechSynthesis.cancel(); speechSynthesis.speak(u);
                }
            },80);
        } catch(e){
            try { if('speechSynthesis' in window) { const u=new SpeechSynthesisUtterance('go'); u.lang='en-US'; speechSynthesis.speak(u); } } catch(e2){ playBeepSound(500,0.4); }
        }
    }
    function showToast(msg, highlight, duration=2500, danger=false) {
        return new Promise(resolve => {
            toastOverlay.style.display='flex';
            const card = document.createElement('div');
            card.className = 'toast-card' + (danger?' danger-toast':'');
            card.innerHTML = `<div class="toast-icon">${danger?'💀':'✅'}</div><div class="toast-text">${msg.replace(highlight||'', `<span class="toast-highlight">${highlight||''}</span>`)}</div>`;
            toastOverlay.appendChild(card);
            playBeepSound(danger?200:800, danger?0.3:0.15);
            setTimeout(()=>{
                card.classList.add('toast-out');
                setTimeout(()=>{ card.remove(); if(toastOverlay.children.length===0) toastOverlay.style.display='none'; resolve(); },400);
            }, duration);
        });
    }

    function switchScreen(target) {
        const allScreens = [screenWelcome, screenLoading, screenSuccess, screenHook, screenHardware];
        const targetEl = document.getElementById('screen'+target.charAt(0).toUpperCase()+target.slice(1));
        const current = allScreens.find(s=>s.classList.contains('active'));
        if(current === targetEl) return;
        if(current) {
            current.style.animation = 'fadeSlideOut 0.35s cubic-bezier(0.55, 0.06, 0.68, 0.19) forwards';
            setTimeout(()=>{
                current.classList.remove('active'); current.style.animation='';
                allScreens.forEach(s=>s.style.display='none');
                if(targetEl) { targetEl.style.display='block'; targetEl.classList.add('active'); targetEl.style.animation='fadeSlideIn 0.5s cubic-bezier(0.22, 0.61, 0.36, 1)'; setTimeout(()=>{ targetEl.style.animation=''; },500); }
            },300);
        } else {
            allScreens.forEach(s=>s.style.display='none');
            if(targetEl) { targetEl.style.display='block'; targetEl.classList.add('active'); targetEl.style.animation='fadeSlideIn 0.5s cubic-bezier(0.22, 0.61, 0.36, 1)'; setTimeout(()=>{ targetEl.style.animation=''; },500); }
        }
        currentScreen = target;
    }

    btnSubmit.addEventListener('click', async ()=>{
        if(cardInput.value.trim() === 'FD66') {
            errorMsg.classList.remove('show');
            playBeepSound(1000,0.1);
            await showToast('检测到是<span class="toast-highlight">开发者用户</span>已为你自动屏蔽','开发者用户',2200);
            switchScreen('loading');
            startLoading();
        } else {
            errorMsg.textContent='✗ 卡密错误，请重新输入'; errorMsg.classList.add('show');
            playBeepSound(200,0.2);
            cardInput.style.borderColor='#ff1744'; cardInput.style.boxShadow='0 0 20px rgba(255,23,68,0.3)';
            setTimeout(()=>{ cardInput.style.borderColor=''; cardInput.style.boxShadow=''; errorMsg.classList.remove('show'); },1800);
        }
    });
    cardInput.addEventListener('keydown', e=>{ if(e.key==='Enter') btnSubmit.click(); });

    function startLoading() {
        let progress=0;
        const logs = ['[~] 初始化漏洞扫描...','[+] 发现ACE服务器端口 443','[+] 尝试SSL握手绕过...','[~] 注入探测数据包...','[+] 目标响应延迟 23ms','[!] 检测到WAF防火墙...','[+] WAF规则绕过成功','[~] 枚举服务器用户...','[+] 发现管理员凭证','[+] 提权漏洞利用中...','[!] 获取root访问权限','[+] 建立持久化后门...','[~] 清理入侵痕迹...','[✓] ACE服务器入侵完成'];
        const duration=4500, stepTime=60;
        let step=0;
        if(loadingInterval) clearInterval(loadingInterval);
        loadingInterval = setInterval(()=>{
            step++;
            progress = Math.min(100, Math.floor((step/(duration/stepTime))*(0.85+Math.random()*0.15)*100));
            progressBarInner.style.width=progress+'%'; progressPercent.textContent=progress+'%';
            loadingLog.textContent = logs[Math.min(Math.floor((progress/100)*logs.length), logs.length-1)];
            if(Math.random()<0.3 && progress<95) playBeepSound(600+Math.random()*600,0.03);
            if(progress>=100) {
                clearInterval(loadingInterval); loadingInterval=null;
                progressBarInner.style.width='100%'; progressPercent.textContent='100%'; loadingLog.textContent='[✓] ACE服务器入侵完成';
                playBeepSound(1200,0.2);
                setTimeout(()=>{ switchScreen('success'); triggerSuccessAnim(); },600);
            }
        }, stepTime);
    }

    function triggerSuccessAnim() {
        faceidRing.classList.remove('ring-done'); faceidCheck.classList.remove('show');
        faceidRing.style.animation='faceidSpin 0.8s linear infinite';
        setTimeout(()=>{
            faceidRing.style.animation='none'; faceidRing.classList.add('ring-done');
            faceidCheck.classList.add('show'); playBeepSound(1500,0.25); setTimeout(()=>playBeepSound(2000,0.2),150);
        },1800);
    }

    document.getElementById('btnAimbot').addEventListener('click', ()=>{ playBeepSound(900,0.12); showToast('🎯 自瞄功能已激活','自瞄',1800); });
    document.getElementById('btnBreakpoint').addEventListener('click', ()=>{ playBeepSound(900,0.12); showToast('🔗 断点子追已开启','断点子追',1800); });
    document.getElementById('btnHardware').addEventListener('click', ()=>{ playBeepSound(900,0.12); switchScreen('hardware'); updateHardware(); });
    document.getElementById('btnHook').addEventListener('click', ()=>{ playBeepSound(700,0.12); switchScreen('hook'); });

    document.getElementById('btnHackInvade').addEventListener('click', async ()=>{
        playBeepSound(400,0.2);
        await showToast('💀 开启成功，感受罚斷的愤怒吧！','开启成功',2800,true);
        playGoSound();
        mainContainer.classList.add('screen-shake');
        setTimeout(()=>mainContainer.classList.remove('screen-shake'),600);
    });
    document.getElementById('btnCrazyMode').addEventListener('click', ()=>{
        isCrazyMode = !isCrazyMode;
        const btn = document.getElementById('btnCrazyMode');
        if(isCrazyMode) {
            crazyOverlay.classList.add('active'); mainContainer.classList.add('screen-shake');
            btn.textContent='🔥 疯狂状态 [ON]'; btn.style.borderColor='#ff8c00'; btn.style.boxShadow='0 0 30px rgba(255,140,0,0.5)'; btn.style.background='rgba(255,100,0,0.2)';
            playBeepSound(300,0.3); showToast('🔥 疯狂状态已激活！','疯狂状态',2000,true);
        } else {
            crazyOverlay.classList.remove('active'); mainContainer.classList.remove('screen-shake');
            btn.textContent='🔥 疯狂状态'; btn.style.borderColor='rgba(255,140,0,0.4)'; btn.style.boxShadow=''; btn.style.background='rgba(255,140,0,0.06)';
            playBeepSound(600,0.15);
        }
    });
    document.getElementById('btnBackFromHook').addEventListener('click', ()=>{ playBeepSound(700,0.1); switchScreen('success'); setTimeout(triggerSuccessAnim,300); });

    function updateHardware() {
        const val = parseInt(hardwareSlider.value);
        sliderValueDisplay.textContent = val;
        paramLatency.textContent = Math.max(1, Math.round(50-val*0.45))+'ms';
        paramRange.textContent = Math.round(val*1.6+5)+'m';
        sliderValueDisplay.style.color = val<30?'#448aff':val<60?'#ffb300':'#ff5252';
    }
    hardwareSlider.addEventListener('input', ()=>{ updateHardware(); if(Math.random()<0.25) playBeepSound(300+parseInt(hardwareSlider.value)*8,0.02); });
    hardwareSlider.addEventListener('change', ()=>{ playBeepSound(500+parseInt(hardwareSlider.value)*6,0.08); });
    document.getElementById('btnBackFromHardware').addEventListener('click', ()=>{ playBeepSound(700,0.1); switchScreen('success'); setTimeout(triggerSuccessAnim,300); });

    setTimeout(()=>cardInput.focus(),500);
    updateHardware();

    document.addEventListener('keydown', e=>{
        if(e.key==='Escape') {
            if(currentScreen==='hook') document.getElementById('btnBackFromHook').click();
            else if(currentScreen==='hardware') document.getElementById('btnBackFromHardware').click();
        }
    });
})();
</script>
</body>
</html>
