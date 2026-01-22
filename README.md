<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // AJEH ATOMIC V16</title>
    
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:title" content="TEMPO // AJEH ATOMIC BURST">
    <meta name="twitter:description" content="Mass deployer and auto-faucet trigger for Tempo Testnet (42431).">
    <meta name="twitter:image" content="https://yourusername.github.io/your-repo/og-image.png">

    <style>
        :root { --primary: #00f2ff; --secondary: #7000ff; --bg: #050505; --success: #44ff44; }
        body { background: var(--bg); color: #fff; font-family: 'Space Grotesk', sans-serif; display: flex; flex-direction: column; align-items: center; min-height: 100vh; margin: 0; padding: 20px; overflow-x: hidden; }
        
        /* Top Navigation/Stats */
        .dashboard { width: 100%; max-width: 1000px; display: flex; justify-content: space-between; align-items: center; margin-bottom: 30px; flex-wrap: wrap; gap: 15px; }
        
        .net-stats { display: flex; align-items: center; gap: 10px; font-size: 0.75rem; color: var(--success); text-transform: uppercase; letter-spacing: 1px; }
        .pulse-dot { width: 8px; height: 8px; background: var(--success); border-radius: 50%; box-shadow: 0 0 10px var(--success); animation: pulse 1.5s infinite; }
        
        .balance-display { font-family: monospace; color: var(--primary); background: rgba(0, 242, 255, 0.1); padding: 10px 20px; border-radius: 12px; border: 1px solid var(--primary); }
        
        /* Layout */
        .main-grid { display: grid; grid-template-columns: 1fr 320px; gap: 20px; max-width: 1000px; width: 100%; }
        
        @media (max-width: 800px) { .main-grid { grid-template-columns: 1fr; } }

        .card { background: rgba(20, 20, 25, 0.98); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 24px; padding: 2rem; text-align: center; box-shadow: 0 0 40px rgba(112, 0, 255, 0.1); }
        .leaderboard { background: #0a0a0c; border: 1px solid var(--secondary); border-radius: 24px; padding: 1.5rem; height: fit-content; }
        
        h1 { font-size: 1.1rem; letter-spacing: 3px; color: var(--primary); text-transform: uppercase; margin-bottom: 20px; }
        .rank-item { display: flex; justify-content: space-between; padding: 12px 0; border-bottom: 1px solid #222; font-size: 0.85rem; font-family: monospace; }

        /* Controls */
        .quantity-box { background: rgba(255,255,255,0.02); padding: 15px; border-radius: 15px; border: 1px solid #222; margin-bottom: 20px; }
        input { background: #000; border: 1px solid var(--primary); color: #fff; padding: 12px; width: 100px; text-align: center; border-radius: 8px; font-size: 1.2rem; font-weight: bold; outline: none; }
        
        button { cursor: pointer; border-radius: 12px; font-weight: 900; transition: 0.3s; text-transform: uppercase; border: none; letter-spacing: 1px; }
        #mainBtn { background: linear-gradient(135deg, var(--secondary), var(--primary)); color: white; padding: 20px; width: 100%; font-size: 1rem; box-shadow: 0 4px 15px rgba(112, 0, 255, 0.3); }
        #shareBtn { background: #fff; color: #000; padding: 15px; width: 100%; margin-top: 15px; display: none; }
        #faucetBtn { background: #ff0055; color: white; padding: 10px 20px; font-size: 0.7rem; }
        
        #feedback { margin-top: 20px; font-size: 0.85rem; color: var(--primary); min-height: 40px; }
        #history { max-height: 200px; overflow-y: auto; margin-top: 20px; font-size: 0.65rem; font-family: monospace; border-top: 1px solid #222; padding-top: 10px; }

        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.3; } 100% { opacity: 1; } }
        ::-webkit-scrollbar { width: 5px; }
        ::-webkit-scrollbar-thumb { background: var(--secondary); border-radius: 10px; }
    </style>
</head>
<body>

<div class="dashboard">
    <div class="net-stats">
        <div class="pulse-dot"></div>
        AJEH HEIGHT: <span id="blockHeight">SYNCING...</span>
    </div>
    <div class="balance-display"><span id="walletBal">0.00</span> USD</div>
    <button id="faucetBtn" onclick="window.open('https://docs.tempo.xyz/quickstart/faucet', '_blank')">REFUEL</button>
</div>

<div class="main-grid">
    <div class="card">
        <h1>ATOMIC DEPLOYER</h1>
        
        <div id="mintOptions" style="display:none;">
            <div class="quantity-box">
                <div style="font-size: 0.7rem; margin-bottom: 10px; opacity: 0.6;">BURST INTENSITY</div>
                <input type="number" id="mintCount" value="1" min="1" max="100">
            </div>
        </div>
        
        <button id="mainBtn">CONNECT NEURAL LINK</button>
        <button id="shareBtn">SHARE BURST ON X</button>
        
        <div id="feedback">Initialize connection to begin ajeh print.</div>
        <div id="history"></div>
    </div>

    <div class="leaderboard">
        <h1>TOP WHALES</h1>
        <div id="leaderboard-ui">Scanning Ajeh Network...</div>
    </div>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";
    import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js/+esm';

    // REPLACE THESE WITH YOUR ACTUAL SUPABASE KEYS
    const supabase = createClient('YOUR_SUPABASE_URL', 'YOUR_SUPABASE_KEY');

    let provider, signer, address, state = "CONNECT";
    const CHAIN_ID_HEX = "0xA5BF"; 
    const RPC_URL = "https://rpc.moderato.tempo.xyz";
    const EXPLORER = "https://explore.tempo.xyz";
    const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033";

    async function syncScore(count) {
        if (!address) return;
        await supabase.from('leaderboard').upsert({ wallet: address, count: count });
        loadLeaderboard();
    }

    async function loadLeaderboard() {
        try {
            const { data } = await supabase.from('leaderboard').select('*').order('count', { ascending: false }).limit(8);
            if (data) {
                document.getElementById('leaderboard-ui').innerHTML = data.map((e, i) => `
                    <div class="rank-item">
                        <span>#${i+1} ${e.wallet.slice(0,6)}...${e.wallet.slice(-4)}</span>
                        <span style="color:var(--primary)">${e.count}</span>
                    </div>
                `).join('');
            }
        } catch (e) { console.error("Leaderboard Error", e); }
    }

    async function updateNetwork() {
        if (!provider) return;
        try {
            const h = await provider.getBlockNumber();
            document.getElementById('blockHeight').innerText = h;
        } catch (e) { document.getElementById('blockHeight').innerText = "DELAYED"; }
    }

    async function updateBal() {
        if (!address) return;
        const b = await provider.getBalance(address);
        document.getElementById('walletBal').innerText = parseFloat(ethers.formatEther(b)).toFixed(2);
    }

    async function handleAction() {
        const fb = document.getElementById('feedback');
        const btn = document.getElementById('mainBtn');

        if (state === "CONNECT") {
            try {
                fb.innerText = "Connecting...";
                await window.ethereum.request({
                    method: 'wallet_addEthereumChain', params: [{
                        chainId: CHAIN_ID_HEX, chainName: 'Tempo Moderato',
                        rpcUrls: [RPC_URL], nativeCurrency: { name: 'USD', symbol: 'USD', decimals: 18 },
                        blockExplorerUrls: [EXPLORER]
                    }]
                });
                provider = new ethers.BrowserProvider(window.ethereum);
                await provider.send("eth_requestAccounts", []);
                signer = await provider.getSigner();
                address = await signer.getAddress();
                
                state = "MINT";
                btn.innerText = "EXECUTE BURST";
                document.getElementById('mintOptions').style.display = 'block';
                fb.innerText = "Link Active: " + address.slice(0,6) + "..." + address.slice(-4);
                
                const { data } = await supabase.from('leaderboard').select('count').eq('wallet', address).single();
                window.currentTotal = data ? data.count : 0;
                
                updateBal();
                updateNetwork();
                loadLeaderboard();
            } catch (e) { fb.innerText = "Link Failed: " + e.message; }
        } else {
            const burst = parseInt(document.getElementById('mintCount').value);
            btn.disabled = true;
            let success = 0;

            for(let i=0; i<burst; i++) {
                try {
                    fb.innerHTML = `PRINTING ${i+1}/${burst}...<br><span style="font-size:0.6rem">CONFIRM IN WALLET</span>`;
                    const factory = new ethers.ContractFactory([], BYTECODE, signer);
                    const tx = await factory.deploy();
                    await tx.waitForDeployment();
                    success++;
                    
                    const item = document.createElement('div');
                    item.innerHTML = `[${new Date().toLocaleTimeString().split(' ')[0]}] PRINTED: ${await tx.getAddress()}`;
                    document.getElementById('history').prepend(item);
                    updateBal();
                } catch (e) { 
                    fb.innerText = "Burst Interrupted.";
                    break; 
                }
            }
            
            window.currentTotal += success;
            await syncScore(window.currentTotal);
            
            btn.disabled = false;
            fb.innerText = `Sequence Confirmed: ${success} New Prints.`;
            document.getElementById('shareBtn').style.display = 'block';
        }
    }

    document.getElementById('mainBtn').onclick = handleAction;
    document.getElementById('shareBtn').onclick = () => {
        const txt = encodeURIComponent(`I've printed ${window.currentTotal} contracts on Tempo Ajeh! 🚀🚀\n\nWho's catching up?`);
        window.open(`https://twitter.com/intent/tweet?text=${txt}&url=${window.location.href}`, '_blank');
    };

    // Initial Syncs
    loadLeaderboard();
    setInterval(updateNetwork, 5000);
</script>
</body>
</html>
