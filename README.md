<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // AJEH ATOMIC BURST</title>
    <style>
        :root { --primary: #00f2ff; --bg: #050505; --card: #0d0d0d; }
        body { 
            background: var(--bg); color: #fff; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
            margin: 0; display: flex; justify-content: center; align-items: center; min-height: 100vh;
        }
        
        .main-card {
            background: var(--card); border: 1px solid #1a1a1a; border-radius: 24px;
            padding: 40px; width: 100%; max-width: 400px; text-align: center;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }

        .header-box { margin-bottom: 30px; }
        .header-box h1 { font-size: 0.9rem; letter-spacing: 4px; color: var(--primary); margin: 0; text-transform: uppercase; }
        .header-box p { font-size: 0.6rem; color: #444; margin-top: 8px; letter-spacing: 1px; }

        .stat-line { display: flex; justify-content: space-between; font-size: 0.7rem; margin-bottom: 25px; color: #666; font-family: monospace; }

        .input-group { background: #000; border: 1px solid #1a1a1a; border-radius: 12px; padding: 15px; margin-bottom: 20px; }
        .input-group label { display: block; font-size: 0.55rem; color: #333; margin-bottom: 8px; text-transform: uppercase; font-weight: bold; }
        input { background: transparent; border: none; color: #fff; font-size: 1.4rem; text-align: center; width: 100%; outline: none; font-weight: bold; }

        #mainBtn { 
            background: #fff; color: #000; border: none; width: 100%; padding: 18px; 
            border-radius: 12px; font-weight: 900; cursor: pointer; transition: 0.2s;
            text-transform: uppercase; font-size: 0.8rem;
        }
        #mainBtn:hover { background: var(--primary); transform: translateY(-2px); }
        #mainBtn:disabled { opacity: 0.3; cursor: not-allowed; transform: none; }

        #feedback { margin-top: 20px; font-size: 0.75rem; min-height: 20px; color: var(--primary); font-family: monospace; text-transform: uppercase; }
        #history { margin-top: 20px; font-size: 0.6rem; color: #333; max-height: 80px; overflow-y: auto; text-align: left; border-top: 1px solid #161616; padding-top: 10px; font-family: monospace; }
        
        ::-webkit-scrollbar { width: 3px; }
        ::-webkit-scrollbar-thumb { background: #222; }
    </style>
</head>
<body>

<div class="main-card">
    <div class="header-box">
        <h1>ATOMIC BURST</h1>
        <p>TEMPO MODERATO // AJEH TESTNET</p>
    </div>

    <div class="stat-line">
        <span>WALLET: <span id="valBal" style="color:#fff">0.00</span> USD</span>
        <span>BLOCK: <span id="blockHeight" style="color:#fff">---</span></span>
    </div>

    <div id="ui-body" style="display: none;">
        <div class="input-group">
            <label>Burst Intensity</label>
            <input type="number" id="mintCount" value="1" min="1" max="100">
        </div>
    </div>

    <button id="mainBtn">CONNECT NEURAL LINK</button>
    
    <div id="feedback">Ready for connection.</div>
    <div id="history"></div>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, address, state = "CONNECT";
    const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033";

    async function init() {
        if (!window.ethereum) return alert("MetaMask/Rabby not found!");
        
        try {
            document.getElementById('feedback').innerText = "PENDING WALLET...";
            provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            address = await signer.getAddress();
            
            state = "MINT";
            document.getElementById('mainBtn').innerText = "EXECUTE BURST";
            document.getElementById('ui-body').style.display = "block";
            document.getElementById('feedback').innerText = "LINK ACTIVE: " + address.slice(0,6);
            
            updateStats();
        } catch (e) {
            document.getElementById('feedback').innerText = "CONNECTION REJECTED";
        }
    }

    async function updateStats() {
        if (!address) return;
        try {
            const [bal, height] = await Promise.all([
                provider.getBalance(address),
                provider.getBlockNumber()
            ]);
            document.getElementById('valBal').innerText = parseFloat(ethers.formatEther(bal)).toFixed(2);
            document.getElementById('blockHeight').innerText = height;
        } catch (e) { console.log("Stats update error"); }
    }

    async function runBurst() {
        const count = parseInt(document.getElementById('mintCount').value);
        const btn = document.getElementById('mainBtn');
        btn.disabled = true;

        for(let i=0; i<count; i++) {
            try {
                document.getElementById('feedback').innerText = `BURSTING ${i+1}/${count}...`;
                const factory = new ethers.ContractFactory([], BYTECODE, signer);
                const tx = await factory.deploy();
                
                // Track progress instantly
                const log = document.createElement('div');
                log.innerText = `[PENDING] Burst #${i+1}...`;
                document.getElementById('history').prepend(log);

                await tx.waitForDeployment();
                log.innerText = `[SUCCESS] Contract: ${await tx.getAddress()}`;
                updateStats();
            } catch (e) {
                document.getElementById('feedback').innerText = "BURST INTERRUPTED";
                break;
            }
        }
        btn.disabled = false;
        document.getElementById('feedback').innerText = "SEQUENCE COMPLETE";
    }

    document.getElementById('mainBtn').onclick = () => {
        if (state === "CONNECT") init();
        else runBurst();
    };

    setInterval(updateStats, 10000);
</script>
</body>
</html>
