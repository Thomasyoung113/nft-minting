<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // AJEH ATOMIC BURST</title>
    <style>
        :root { --primary: #00f2ff; --bg: #050505; --card: #0d0d0d; }
        body { 
            background: var(--bg); color: #fff; font-family: 'Space Grotesk', sans-serif; 
            margin: 0; display: flex; justify-content: center; align-items: center; min-height: 100vh;
        }
        
        .main-card {
            background: var(--card); border: 1px solid #1a1a1a; border-radius: 24px;
            padding: 40px; width: 100%; max-width: 450px; text-align: center;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }

        .header-box { margin-bottom: 30px; }
        .header-box h1 { font-size: 1rem; letter-spacing: 4px; color: var(--primary); margin: 0; }
        .header-box p { font-size: 0.7rem; color: #444; margin-top: 5px; }

        .stat-line { display: flex; justify-content: space-between; font-size: 0.75rem; margin-bottom: 20px; color: #666; font-family: monospace; }

        .input-group { background: #000; border: 1px solid #222; border-radius: 12px; padding: 15px; margin-bottom: 20px; }
        .input-group label { display: block; font-size: 0.6rem; color: #444; margin-bottom: 8px; text-transform: uppercase; }
        input { background: transparent; border: none; color: #fff; font-size: 1.5rem; text-align: center; width: 100%; outline: none; font-weight: bold; }

        #mainBtn { 
            background: #fff; color: #000; border: none; width: 100%; padding: 18px; 
            border-radius: 12px; font-weight: 800; cursor: pointer; transition: 0.2s;
        }
        #mainBtn:hover { transform: scale(1.02); }
        #mainBtn:disabled { opacity: 0.3; cursor: not-allowed; }

        #feedback { margin-top: 20px; font-size: 0.8rem; min-height: 20px; color: var(--primary); font-family: monospace; }
        #history { margin-top: 20px; font-size: 0.6rem; color: #333; max-height: 80px; overflow-y: auto; text-align: left; border-top: 1px solid #161616; padding-top: 10px; }
    </style>
</head>
<body>

<div class="main-card">
    <div class="header-box">
        <h1>ATOMIC BURST</h1>
        <p>TEMPO MODERATO // CHAIN ID 42431</p>
    </div>

    <div class="stat-line">
        <span>BAL: <span id="valBal" style="color:#fff">0.00</span> USD</span>
        <span>BLOCK: <span id="blockHeight" style="color:#fff">---</span></span>
    </div>

    <div id="ui-body" style="display: none;">
        <div class="input-group">
            <label>Burst Intensity (No. of Contracts)</label>
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
        if (!window.ethereum) return alert("Please install MetaMask!");
        
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
        } catch (e) {
            document.getElementById('feedback').innerText = "CONNECTION FAILED";
        }
    }

    async function updateStats() {
        if (!address) return;
        const [bal, height] = await Promise.all([
            provider.getBalance(address),
            provider.getBlockNumber()
        ]);
        document.getElementById('valBal').innerText = parseFloat(ethers.formatEther(bal)).toFixed(2);
        document.getElementById('blockHeight').innerText = height;
    }

    async function runBurst() {
        const count = parseInt(document.getElementById('mintCount').value);
        const btn = document.getElementById('mainBtn');
        btn.disabled = true;

        for(let i=0; i<count; i++) {
            try {
                document.getElementById('feedback').innerText = `DEPLOYING ${i+1}/${count}...`;
                const factory = new ethers.ContractFactory([], BYTECODE, signer);
                const tx = await factory.deploy();
                await tx.waitForDeployment();
                
                const log = document.createElement('div');
                log.innerText = `[${i+1}] DEPLOYED: ${await tx.getAddress()}`;
                document.getElementById('history').prepend(log);
                updateStats();
            } catch (e) {
                document.getElementById('feedback').innerText = "BURST HALTED";
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
