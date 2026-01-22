<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // AJEH ATOMIC V21</title>
    <style>
        :root { --primary: #00f2ff; --bg: #050505; --card: #0d0d0d; --accent: #ff0055; }
        body { 
            background: var(--bg); color: #fff; font-family: 'Segoe UI', sans-serif; 
            margin: 0; display: flex; flex-direction: column; justify-content: center; align-items: center; min-height: 100vh;
        }
        
        .main-card {
            background: var(--card); border: 1px solid #1a1a1a; border-radius: 24px;
            padding: 40px; width: 100%; max-width: 420px; text-align: center;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            margin-bottom: 20px;
        }

        .header-box { margin-bottom: 25px; }
        .header-box h1 { font-size: 0.9rem; letter-spacing: 4px; color: var(--primary); margin: 0; text-transform: uppercase; }
        
        .nav-actions { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 25px; }
        .nav-btn { 
            background: #111; border: 1px solid #222; color: #888; 
            padding: 12px; border-radius: 10px; font-size: 0.6rem; cursor: pointer;
            text-decoration: none; font-weight: bold; transition: 0.2s; text-align: center;
        }
        .nav-btn:hover { border-color: var(--primary); color: #fff; }
        .refuel-btn { border-color: var(--accent); color: var(--accent); }

        .stat-line { display: flex; justify-content: space-between; font-size: 0.7rem; margin-bottom: 20px; color: #666; font-family: monospace; }

        .input-group { background: #000; border: 1px solid #1a1a1a; border-radius: 12px; padding: 15px; margin-bottom: 20px; }
        input { background: transparent; border: none; color: #fff; font-size: 1.4rem; text-align: center; width: 100%; outline: none; font-weight: bold; }

        #mainBtn { 
            background: #fff; color: #000; border: none; width: 100%; padding: 18px; 
            border-radius: 12px; font-weight: 900; cursor: pointer; transition: 0.2s;
            text-transform: uppercase; font-size: 0.8rem;
        }
        #mainBtn:hover { background: var(--primary); transform: translateY(-2px); }

        #feedback { margin-top: 20px; font-size: 0.75rem; min-height: 20px; color: var(--primary); font-family: monospace; }
        #history { margin-top: 20px; font-size: 0.6rem; color: #444; max-height: 100px; overflow-y: auto; text-align: left; border-top: 1px solid #161616; padding-top: 10px; }
        .history-item { display: flex; justify-content: space-between; padding: 6px 0; border-bottom: 1px solid #111; }
        .history-item a { color: var(--primary); text-decoration: none; }

        /* Footer Note Style */
        .footer-note { 
            font-size: 0.65rem; color: #333; letter-spacing: 2px; text-transform: uppercase; font-weight: bold; 
        }
        .footer-note a { 
            color: #555; text-decoration: none; transition: 0.3s; 
        }
        .footer-note a:hover { 
            color: var(--primary); text-shadow: 0 0 8px var(--primary); 
        }
    </style>
</head>
<body>

<div class="main-card">
    <div class="header-box">
        <h1>ATOMIC BURST</h1>
    </div>

    <div class="nav-actions">
        <button id="massRefuel" class="nav-btn refuel-btn">MASS REFUEL</button>
        <a href="https://explore.tempo.xyz" target="_blank" class="nav-btn">EXPLORER</a>
    </div>

    <div class="stat-line">
        <span>BAL: <span id="valBal" style="color:#fff">0.00</span> USD</span>
        <span>BLOCK: <span id="blockHeight" style="color:#fff">---</span></span>
    </div>

    <div id="ui-body" style="display: none;">
        <div class="input-group">
            <input type="number" id="mintCount" value="1" min="1" max="100">
        </div>
    </div>

    <button id="mainBtn">CONNECT NEURAL LINK</button>
    
    <div id="feedback">Ready for connection.</div>
    <div id="history"></div>
</div>

<div class="footer-note">
    Built By <a href="https://t.me/thomas_young" target="_blank">THOMASYOUNG</a>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, address, state = "CONNECT";
    const EXPLORER_URL = "https://explore.tempo.xyz";
    const ASSETS = [
        "0x20c0000000000000000000000000000000000000", 
        "0x20c0000000000000000000000000000000000001", 
        "0x20c0000000000000000000000000000000000002", 
        "0x20c0000000000000000000000000000000000003"
    ];

    async function init() {
        if (!window.ethereum) return alert("MetaMask not found!");
        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            address = await signer.getAddress();
            state = "MINT";
            document.getElementById('mainBtn').innerText = "EXECUTE BURST";
            document.getElementById('ui-body').style.display = "block";
            document.getElementById('feedback').innerText = "LINK ACTIVE";
            updateStats();
        } catch (e) { document.getElementById('feedback').innerText = "LINK FAILED"; }
    }

    async function massRefuel() {
        if (!signer) return alert("Connect wallet first!");
        document.getElementById('feedback').innerText = "STARTING MASS REFUEL...";
        for (const asset of ASSETS) {
            try {
                const tx = { to: asset, value: 0 };
                const response = await signer.sendTransaction(tx);
                await response.wait();
            } catch (e) { console.log("Refuel attempt failed"); }
        }
        updateStats();
        document.getElementById('feedback').innerText = "REFUEL SEQUENCE DONE";
    }

    async function updateStats() {
        if (!address) return;
        try {
            const [bal, height] = await Promise.all([provider.getBalance(address), provider.getBlockNumber()]);
            document.getElementById('valBal').innerText = parseFloat(ethers.formatEther(bal)).toFixed(2);
            document.getElementById('blockHeight').innerText = height;
        } catch (e) { }
    }

    async function runBurst() {
        const count = parseInt(document.getElementById('mintCount').value);
        const btn = document.getElementById('mainBtn');
        btn.disabled = true;
        
        for(let i=0; i<count; i++) {
            let attempt = 0;
            let success = false;
            
            while(attempt < 3 && !success) {
                try {
                    document.getElementById('feedback').innerText = `BURSTING ${i+1}/${count} (Try ${attempt+1})...`;
                    const factory = new ethers.ContractFactory([], "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033", signer);
                    const tx = await factory.deploy();
                    await tx.waitForDeployment();
                    
                    const addr = await tx.getAddress();
                    const item = document.createElement('div');
                    item.className = 'history-item';
                    item.innerHTML = `<span>#${i+1} OK</span> <a href="${EXPLORER_URL}/address/${addr}" target="_blank">VIEW</a>`;
                    document.getElementById('history').prepend(item);
                    success = true;
                } catch (e) {
                    attempt++;
                    if(attempt >= 3) break;
                    await new Promise(r => setTimeout(r, 2000)); // Wait 2s before retry
                }
            }
            updateStats();
        }
        btn.disabled = false;
        document.getElementById('feedback').innerText = "SEQUENCE COMPLETE";
    }

    document.getElementById('mainBtn').onclick = () => (state === "CONNECT" ? init() : runBurst());
    document.getElementById('massRefuel').onclick = massRefuel;
    setInterval(updateStats, 10000);
</script>
</body>
</html>
