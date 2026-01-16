<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // NEURAL MINT</title>
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
            overflow: hidden;
        }

        .glow {
            position: absolute;
            width: 450px;
            height: 450px;
            background: var(--secondary);
            filter: blur(140px);
            opacity: 0.15;
            z-index: -1;
            animation: pulse 8s infinite alternate;
        }

        @keyframes pulse {
            from { transform: scale(1); opacity: 0.1; }
            to { transform: scale(1.2); opacity: 0.2; }
        }

        .card {
            background: var(--card-bg);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(0, 242, 255, 0.15);
            border-radius: 28px;
            padding: 2.5rem;
            width: 380px;
            text-align: center;
            box-shadow: 0 20px 80px rgba(0, 0, 0, 0.8);
        }

        h1 {
            font-size: 1.5rem;
            letter-spacing: 6px;
            margin-bottom: 0.5rem;
            background: linear-gradient(90deg, var(--primary), var(--secondary));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
        }

        .network-tag {
            display: inline-block;
            font-size: 0.6rem;
            background: rgba(0, 242, 255, 0.05);
            color: var(--primary);
            padding: 6px 16px;
            border-radius: 100px;
            border: 1px solid rgba(0, 242, 255, 0.3);
            margin-bottom: 2rem;
            letter-spacing: 2px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 2rem;
        }

        .stat-card {
            background: rgba(255, 255, 255, 0.02);
            border: 1px solid rgba(255, 255, 255, 0.08);
            padding: 15px 10px;
            border-radius: 16px;
        }

        .stat-label { font-size: 0.5rem; color: #666; text-transform: uppercase; display: block; margin-bottom: 5px; }
        .stat-value { font-size: 1.1rem; font-weight: bold; font-family: monospace; }

        .input-container {
            position: relative;
            display: inline-block;
            margin-bottom: 20px;
        }

        input {
            background: rgba(0, 0, 0, 0.5);
            border: 1px solid rgba(255, 255, 255, 0.1);
            color: var(--primary);
            padding: 12px 20px;
            border-radius: 12px;
            width: 120px;
            font-size: 1.3rem;
            text-align: center;
            outline: none;
            transition: border 0.3s;
        }

        input:focus { border-color: var(--primary); }

        button {
            background: linear-gradient(135deg, var(--secondary), var(--primary));
            border: none;
            color: white;
            padding: 18px;
            border-radius: 14px;
            width: 100%;
            font-weight: 900;
            text-transform: uppercase;
            letter-spacing: 3px;
            cursor: pointer;
            transition: all 0.3s;
        }

        button:hover {
            box-shadow: 0 0 30px rgba(0, 242, 255, 0.5);
            transform: translateY(-2px);
        }

        #txHistory {
            margin-top: 2rem;
            text-align: left;
            max-height: 120px;
            overflow-y: auto;
            border-top: 1px solid rgba(255,255,255,0.05);
            padding-top: 15px;
        }

        .tx-item { 
            font-size: 0.65rem; 
            margin-bottom: 8px; 
            font-family: monospace; 
            display: flex;
            justify-content: space-between;
            background: rgba(255,255,255,0.02);
            padding: 5px;
            border-radius: 4px;
        }

        .tx-link { color: var(--primary); text-decoration: none; opacity: 0.7; font-weight: bold; }
        .tx-link:hover { opacity: 1; text-decoration: underline; }

        #status { 
            font-size: 0.6rem; 
            color: #444; 
            margin-top: 1.5rem; 
            font-family: monospace; 
            cursor: pointer;
            transition: color 0.2s;
        }
        #status:hover { color: var(--primary); }

        .success-msg { color: var(--primary); text-shadow: 0 0 10px var(--primary); font-weight: bold; }
    </style>
</head>
<body>

<div class="glow"></div>

<div class="card">
    <h1>TEMPO MINT</h1>
    <div class="network-tag">MODERATO TESTNET</div>
    
    <div id="statsUI" class="stats-grid" style="display: none;">
        <div class="stat-card">
            <span class="stat-label">Global Supply</span>
            <span id="totalSupply" class="stat-value">0</span>
        </div>
        <div class="stat-card">
            <span class="stat-label">Your Balance</span>
            <span id="userBalance" class="stat-value">0</span>
        </div>
    </div>

    <button id="connectBtn">Link Neural Interface</button>

    <div id="mintUI" style="display: none;">
        <div class="input-container">
            <input type="number" id="qty" value="1" min="1">
        </div>
        <button id="mintBtn">Execute Mint</button>
        <div id="txHistory"></div>
    </div>

    <div id="feedback" style="margin-top: 15px; font-size: 0.8rem; min-height: 1.2rem;"></div>
    <p id="status" title="Click to copy address">SIGNAL // OFFLINE</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, contract, userAddress;
    
    // Moderato Build Config
    const contractAddress = "0xD84465DC85e23f18dC60a5975b4562d6a5B6DcbB";
    const moderatoHex = "0x7A1"; 
    const rpc = "https://rpc.moderato.testnet.tempo.xyz";
    const explorer = "https://explore.tempo.xyz"; // Updated explorer link

    const abi = [
        "function mint(uint256 quantity) public",
        "function totalSupply() view returns (uint256)",
        "function balanceOf(address owner) view returns (uint256)"
    ];

    function addTxToUI(hash) {
        const history = document.getElementById('txHistory');
        const item = document.createElement('div');
        item.className = 'tx-item';
        item.innerHTML = `
            <span>> HASH: ${hash.slice(0,6)}...${hash.slice(-4)}</span>
            <a href="${explorer}/tx/${hash}" target="_blank" class="tx-link">[EXPLORE]</a>
        `;
        history.prepend(item);
    }

    async function updateStats() {
        if (!contract) return;
        try {
            const [total, balance] = await Promise.all([
                contract.totalSupply(),
                contract.balanceOf(userAddress)
            ]);
            document.getElementById('totalSupply').innerText = total.toString();
            document.getElementById('userBalance').innerText = balance.toString();
            document.getElementById('statsUI').style.display = 'grid';
        } catch (e) { console.warn("Stats sync error - node might be rate limiting."); }
    }

    async function connect() {
        if (!window.ethereum) return alert("Neural Link: Wallet not detected. Please install Rabby.");
        
        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            
            try {
                await window.ethereum.request({
                    method: 'wallet_switchEthereumChain',
                    params: [{ chainId: moderatoHex }],
                });
            } catch (err) {
                if (err.code === 4902) {
                    await window.ethereum.request({
                        method: 'wallet_addEthereumChain',
                        params: [{
                            chainId: moderatoHex,
                            chainName: 'Tempo Moderato',
                            rpcUrls: [rpc],
                            nativeCurrency: { name: 'TEMPO', symbol: 'TEMPO', decimals: 18 },
                            blockExplorerUrls: [explorer]
                        }]
                    });
                }
            }

            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            userAddress = await signer.getAddress();
            contract = new ethers.Contract(contractAddress, abi, signer);
            
            await updateStats();
            
            document.getElementById('status').innerText = `TUNNEL // ${userAddress.slice(0,6)}...${userAddress.slice(-4)}`;
            document.getElementById('connectBtn').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
            
        } catch (err) { console.error("Connection failed", err); }
    }

    async function mint() {
        const qty = document.getElementById('qty').value;
        const feedback = document.getElementById('feedback');
        if (qty < 1) return;

        try {
            feedback.innerHTML = "Authenticating Signal...";
            const tx = await contract.mint(qty);
            
            feedback.innerHTML = "Broadcasting to Moderato...";
            addTxToUI(tx.hash);
            
            await tx.wait();
            await updateStats();
            feedback.innerHTML = `<span class="success-msg">Neural Print Confirmed.</span>`;
        } catch (err) {
            console.error(err);
            feedback.innerHTML = `<span style="color:#ff0055">Signal Interrupted.</span>`;
        }
    }

    // Copy Address Logic
    document.getElementById('status').onclick = () => {
        if (userAddress) {
            navigator.clipboard.writeText(userAddress);
            const oldText = document.getElementById('status').innerText;
            document.getElementById('status').innerText = "COPIED TO CLIPBOARD";
            setTimeout(() => document.getElementById('status').innerText = oldText, 2000);
        }
    };

    if (window.ethereum) {
        window.ethereum.on('accountsChanged', () => window.location.reload());
        window.ethereum.on('chainChanged', () => window.location.reload());
    }

    document.getElementById('connectBtn').onclick = connect;
    document.getElementById('mintBtn').onclick = mint;
</script>

</body>
</html>
