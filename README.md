<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // MODERATO MINT</title>
    <style>
        :root {
            --primary: #00f2ff;
            --secondary: #7000ff;
            --bg: #050505;
            --card-bg: rgba(20, 20, 25, 0.9);
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
            overflow-x: hidden;
        }

        .glow {
            position: absolute;
            width: 400px;
            height: 400px;
            background: var(--secondary);
            filter: blur(150px);
            opacity: 0.2;
            z-index: -1;
            animation: move 15s infinite alternate;
        }

        @keyframes move {
            from { transform: translate(-30%, -30%); }
            to { transform: translate(30%, 30%); }
        }

        .card {
            background: var(--card-bg);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(0, 242, 255, 0.2);
            border-radius: 24px;
            padding: 2.5rem;
            width: 400px;
            text-align: center;
            box-shadow: 0 0 50px rgba(0, 0, 0, 0.9);
        }

        h1 {
            font-size: 1.6rem;
            letter-spacing: 5px;
            margin-bottom: 0.5rem;
            background: linear-gradient(90deg, var(--primary), var(--secondary));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
        }

        .network-tag {
            display: inline-block;
            font-size: 0.65rem;
            background: rgba(0, 242, 255, 0.1);
            color: var(--primary);
            padding: 5px 15px;
            border-radius: 100px;
            border: 1px solid var(--primary);
            margin-bottom: 1.5rem;
            text-transform: uppercase;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 1.5rem;
        }

        .stat-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.1);
            padding: 12px;
            border-radius: 12px;
        }

        .stat-label { font-size: 0.55rem; color: #888; text-transform: uppercase; display: block; margin-bottom: 4px; }
        .stat-value { font-size: 1.1rem; font-weight: bold; color: #fff; }

        input {
            background: rgba(0, 0, 0, 0.4);
            border: 1px solid rgba(0, 242, 255, 0.3);
            color: white;
            padding: 12px;
            border-radius: 8px;
            width: 80px;
            margin-bottom: 15px;
            font-size: 1.2rem;
            text-align: center;
            outline: none;
        }

        button {
            background: linear-gradient(45deg, var(--secondary), var(--primary));
            border: none;
            color: white;
            padding: 16px;
            border-radius: 12px;
            width: 100%;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        button:hover {
            transform: scale(1.02);
            box-shadow: 0 0 20px rgba(0, 242, 255, 0.4);
        }

        #txHistory {
            margin-top: 1.5rem;
            text-align: left;
            font-size: 0.7rem;
            max-height: 100px;
            overflow-y: auto;
            border-top: 1px solid rgba(255,255,255,0.1);
            padding-top: 10px;
        }

        .tx-item { color: #888; margin-bottom: 5px; font-family: monospace; }
        .success-msg { color: var(--primary); font-weight: bold; }
        #status { font-size: 0.7rem; color: #555; margin-top: 1rem; }
    </style>
</head>
<body>

<div class="glow"></div>

<div class="card">
    <h1>NEURAL MINT</h1>
    <div class="network-tag">● MODERATO TESTNET</div>
    
    <div id="statsUI" class="stats-grid" style="display: none;">
        <div class="stat-card">
            <span class="stat-label">Global Supply</span>
            <span id="totalSupply" class="stat-value">--</span>
        </div>
        <div class="stat-card">
            <span class="stat-label">Your Wallet</span>
            <span id="userBalance" class="stat-value">--</span>
        </div>
    </div>

    <button id="connectBtn">Connect Neural Link</button>

    <div id="mintUI" style="display: none;">
        <input type="number" id="qty" value="1" min="1">
        <button id="mintBtn">Authorize Mint</button>
        <div id="txHistory"></div>
    </div>

    <div id="feedback" style="margin-top: 15px; font-size: 0.85rem;"></div>
    <p id="status">Status: Offline</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, contract, userAddress;
    
    // Config for Tempo Moderato
    const contractAddress = "0xD84465DC85e23f18dC60a5975b4562d6a5B6DcbB";
    const moderatoChainId = "0x7A1"; 
    const moderatoRpc = "https://rpc.moderato.testnet.tempo.xyz";

    const abi = [
        "function mint(uint256 quantity) public",
        "function totalSupply() view returns (uint256)",
        "function balanceOf(address owner) view returns (uint256)"
    ];

    function logTx(hash) {
        const history = document.getElementById('txHistory');
        const entry = document.createElement('div');
        entry.className = 'tx-item';
        entry.innerHTML = `> TX: ${hash.slice(0,10)}...${hash.slice(-8)}`;
        history.prepend(entry);
    }

    async function updateStats() {
        try {
            const [total, balance] = await Promise.all([
                contract.totalSupply(),
                contract.balanceOf(userAddress)
            ]);
            document.getElementById('totalSupply').innerText = total.toString();
            document.getElementById('userBalance').innerText = balance.toString();
            document.getElementById('statsUI').style.display = 'grid';
        } catch (e) { console.error("Sync Error:", e); }
    }

    async function connect() {
        if (!window.ethereum) return alert("Rabby Wallet required for Neural Link.");
        
        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            
            // Network Switch/Add Logic
            try {
                await window.ethereum.request({
                    method: 'wallet_switchEthereumChain',
                    params: [{ chainId: moderatoChainId }],
                });
            } catch (err) {
                if (err.code === 4902) {
                    await window.ethereum.request({
                        method: 'wallet_addEthereumChain',
                        params: [{
                            chainId: moderatoChainId,
                            chainName: 'Tempo Moderato',
                            rpcUrls: [moderatoRpc],
                            nativeCurrency: { name: 'TEMPO', symbol: 'TEMPO', decimals: 18 }
                        }]
                    });
                }
            }

            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            userAddress = await signer.getAddress();
            contract = new ethers.Contract(contractAddress, abi, signer);
            
            await updateStats();
            
            document.getElementById('status').innerText = `Active: ${userAddress.slice(0,6)}...${userAddress.slice(-4)}`;
            document.getElementById('connectBtn').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
            
        } catch (err) { console.error(err); }
    }

    async function mint() {
        const qty = document.getElementById('qty').value;
        const feedback = document.getElementById('feedback');
        try {
            feedback.innerHTML = "Submitting to Moderato...";
            const tx = await contract.mint(qty);
            
            feedback.innerHTML = "Processing Neural Print...";
            logTx(tx.hash);
            
            await tx.wait();
            await updateStats();
            feedback.innerHTML = `<span class="success-msg">Mint Confirmed.</span>`;
        } catch (err) {
            feedback.innerHTML = `<span style="color:#ff0055">Authorization Failed.</span>`;
        }
    }

    // Auto-refresh on wallet changes
    if (window.ethereum) {
        window.ethereum.on('accountsChanged', () => window.location.reload());
        window.ethereum.on('chainChanged', () => window.location.reload());
    }

    document.getElementById('connectBtn').onclick = connect;
    document.getElementById('mintBtn').onclick = mint;
</script>

</body>
</html>
