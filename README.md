<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // NEURAL HISTORY</title>
    <style>
        :root {
            --primary: #00f2ff;
            --secondary: #7000ff;
            --bg: #050505;
            --card-bg: rgba(20, 20, 25, 0.95);
        }

        body {
            margin: 0;
            background-color: var(--bg);
            background-image: 
                linear-gradient(rgba(0, 242, 255, 0.05) 1px, transparent 1px),
                linear-gradient(90deg, rgba(0, 242, 255, 0.05) 1px, transparent 1px);
            background-size: 30px 30px;
            color: #fff;
            font-family: 'Space Grotesk', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }

        .card {
            background: var(--card-bg);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(0, 242, 255, 0.2);
            border-radius: 28px;
            padding: 2.5rem;
            width: 400px;
            text-align: center;
            box-shadow: 0 0 50px rgba(112, 0, 255, 0.2);
        }

        h1 {
            font-size: 1.4rem;
            letter-spacing: 4px;
            background: linear-gradient(90deg, var(--primary), var(--secondary));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
            margin-bottom: 1.5rem;
        }

        button {
            background: linear-gradient(135deg, var(--secondary), var(--primary));
            border: none;
            color: white;
            padding: 20px;
            border-radius: 14px;
            width: 100%;
            font-weight: 900;
            text-transform: uppercase;
            letter-spacing: 2px;
            cursor: pointer;
            transition: all 0.3s;
        }

        button:hover:not(:disabled) {
            box-shadow: 0 0 30px rgba(0, 242, 255, 0.5);
            transform: scale(1.02);
        }

        #history-container {
            margin-top: 2rem;
            text-align: left;
            border-top: 1px solid rgba(255,255,255,0.1);
            padding-top: 1.5rem;
        }

        .history-title {
            font-size: 0.7rem;
            color: var(--primary);
            text-transform: uppercase;
            letter-spacing: 2px;
            margin-bottom: 10px;
            display: flex;
            justify-content: space-between;
        }

        .history-list {
            max-height: 150px;
            overflow-y: auto;
            scrollbar-width: thin;
        }

        .history-item {
            background: rgba(255,255,255,0.03);
            border-radius: 8px;
            padding: 10px;
            margin-bottom: 8px;
            font-family: monospace;
            font-size: 0.65rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .history-item a {
            color: var(--primary);
            text-decoration: none;
            opacity: 0.8;
        }

        .history-item a:hover { opacity: 1; }

        #feedback { margin-top: 20px; font-size: 0.8rem; min-height: 40px;}
        #status { font-size: 0.65rem; color: #444; margin-top: 1.5rem; letter-spacing: 2px; text-transform: uppercase;}
        .error { color: #ff0055; font-size: 0.7rem; }
    </style>
</head>
<body>

<div class="card">
    <h1>ATOMIC MINT</h1>
    
    <div id="authUI">
        <button id="connectBtn">INITIALIZE LINK</button>
    </div>

    <div id="mintUI" style="display: none;">
        <button id="deployBtn">EXECUTE NEURAL LAUNCH</button>
    </div>

    <div id="feedback"></div>

    <div id="history-container" style="display: none;">
        <div class="history-title">
            <span>Recent Spawns</span>
            <span id="clearHistory" style="cursor:pointer; font-size: 0.5rem; opacity: 0.5;">[CLEAR]</span>
        </div>
        <div id="historyList" class="history-list"></div>
    </div>

    <p id="status">NETWORK // DISCONNECTED</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer;
    const CHAIN_ID_HEX = "0x7A1"; // 1953
    const RPC_URLS
