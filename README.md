<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // AJEH ATOMIC V17</title>
    <style>
        :root { --primary: #00f2ff; --bg: #030303; --card: #0d0d0d; --text: #888; }
        body { background: var(--bg); color: #fff; font-family: 'Inter', sans-serif; margin: 0; padding: 20px; display: flex; flex-direction: column; align-items: center; }
        
        /* Bento Grid Layout */
        .bento-grid { display: grid; grid-template-columns: 2fr 1fr; grid-template-rows: auto auto; gap: 15px; max-width: 900px; width: 100%; }
        .glass-card { background: var(--card); border: 1px solid #1a1a1a; border-radius: 16px; padding: 20px; position: relative; overflow: hidden; }
        
        /* Subtle Leaderboard Styling */
        .subtle-leaderboard { font-size: 0.75rem; color: var(--text); }
        .whale-row { display: flex; justify-content: space-between; padding: 6px 0; border-bottom: 1px solid #161616; transition: 0.3s; }
        .whale-row:hover { color: var(--primary); background: rgba(0,242,255,0.02); }
        .whale-address { font-family: monospace; }
        
        /* Inputs & Buttons */
        .intensity-pill { background: #000; border: 1px solid #222; border-radius: 30px; padding: 5px 15px; display: inline-flex; align-items: center; gap: 10px; margin-bottom: 20px; }
        input { background: transparent; border: none; color: var(--primary); width: 40px; font-weight: bold; text-align: center; font-size: 1rem; outline: none; }
        
        #mainBtn { background: #fff; color: #000; width: 100%; padding: 15px; border-radius: 12px; font-weight: 800; cursor: pointer; transition: 0.2s; border: none; }
        #mainBtn:hover { transform: translateY(-2px); box-shadow: 0 5px 15px rgba(255,255,255,0.2); }
        #mainBtn:disabled { opacity: 0.5; cursor: not-allowed; }

        .status-light { width: 6px; height: 6px; border-radius: 50%; display: inline-block; margin-right: 5px; }
        .online { background: #00ff88; box-shadow: 0 0 8px #00ff88; }

        #history { font-size: 0.6rem; color: #444; margin-top: 15px; height: 60px; overflow-y: auto; text-align: left; }
    </style>
</head>
<body>

<div class="bento-grid">
    <div class="glass-card" style="grid-row: span 2;">
        <div style="font-size: 0.7rem; letter-spacing: 2px; margin-bottom: 20px;">
            <span class="status-light online"></span> TEMPO MODERATO // ATOMIC BURST
        </div>
        
        <div class="intensity-pill">
            <span style="font-size: 0.6rem; color: #555;">BURST INTENSITY</span>
            <input type="number" id="mintCount" value="1" min="1" max="100">
        </div>

        <button id="mainBtn">CONNECT WALLET</button>
        <div id="feedback" style="margin-top: 15px; font-size: 0.8rem; color: var(--text);">Neural Link: Waiting...</div>
        <div id="history"></div>
    </div>

    <div class="glass-card" style="padding: 15px;">
        <div style="font-size: 0.6rem; color: #555;">NETWORK STATUS</div>
        <div style="font-size: 1rem; font-family: monospace; color: var(--primary); margin-top: 5px;">
            <span id="blockHeight">---</span> <span style="font-size: 0.6rem; color:#333;">BLOCKS</span>
        </div>
    </div>

    <div class="glass-card subtle-leaderboard">
        <div style="font-size: 0.6rem; color: #555; margin-bottom: 10px;">GLOBAL RANKINGS</div>
        <div id="leaderboard-ui">Syncing...</div>
    </div>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";
    import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm';

    const supabase = createClient('YOUR_SUPABASE_URL', 'YOUR_SUPABASE_KEY');
    let provider, signer, address, state = "CONNECT";

    // Re-check for existing connection on GitHub Pages load
    window.onload = async () => {
        if (window.ethereum && window.ethereum.selectedAddress) {
            initWallet();
        }
        loadLeaderboard();
    };

    async function initWallet() {
        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []); // Forces popup
            signer = await provider.getSigner();
            address = await signer.getAddress();
            
            state = "MINT";
            document.getElementById('mainBtn').innerText = "EXECUTE ATOMIC BURST";
            document.getElementById('feedback').innerText = "LINKED: " + address.slice(0,6) + "...";
            updateHeight();
        } catch (e) { console.error(e); }
    }

    async function handleAction() {
        if (state === "CONNECT") {
            await initWallet();
        } else {
            const count = parseInt(document.getElementById('mintCount').value);
            const btn = document.getElementById('mainBtn');
            btn.disabled = true;
            
            for(let i=0; i<count; i++) {
                try {
                    document.getElementById('feedback').innerText = `Deploying ${i+1}/${count}...`;
                    const factory = new ethers.ContractFactory([], "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033", signer);
                    const tx = await factory.deploy();
                    await tx.waitForDeployment();
                    
                    // Log history
                    const log = document.createElement('div');
                    log.innerText = `> ${await tx.getAddress()}`;
                    document.getElementById('history').prepend(log);
                } catch (e) { break; }
            }
            
            // Sync with Supabase (Silent)
            const { data } = await supabase.from('leaderboard').select('count').eq('wallet', address).single();
            const newCount = (data ? data.count : 0) + count;
            await supabase.from('leaderboard').upsert({ wallet: address, count: newCount });
            
            btn.disabled = false;
            document.getElementById('feedback').innerText = "Burst Complete.";
            loadLeaderboard();
        }
    }

    async function loadLeaderboard() {
        const { data } = await supabase.from('leaderboard').select('*').order('count', { ascending: false }).limit(5);
        if (data) {
            document.getElementById('leaderboard-ui').innerHTML = data.map(e => `
                <div class="whale-row">
                    <span class="whale-address">${e.wallet.slice(0,4)}...${e.wallet.slice(-2)}</span>
                    <span>${e.count}</span>
                </div>
            `).join('');
        }
    }

    async function updateHeight() {
        if (!provider) return;
        const h = await provider.getBlockNumber();
        document.getElementById('blockHeight').innerText = h;
    }

    document.getElementById('mainBtn').onclick = handleAction;
    setInterval(updateHeight, 10000);
</script>
</body>
</html>
