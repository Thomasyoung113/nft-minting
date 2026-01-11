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
            --card-bg: rgba(20, 20, 25, 0.8);
        }

        body {
            margin: 0;
            background-color: var(--bg);
            background-image: 
                linear-gradient(rgba(0, 242, 255, 0.05) 1px, transparent 1px),
                linear-gradient(90deg, rgba(0, 242, 255, 0.05) 1px, transparent 1px);
            background-size: 30px 30px;
            color: #fff;
            font-family: 'Space Grotesk', 'Syncopate', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            overflow: hidden;
        }

        /* Animated Glow Background */
        .glow {
            position: absolute;
            width: 300px;
            height: 300px;
            background: var(--secondary);
            filter: blur(120px);
            opacity: 0.3;
            z-index: -1;
            animation: move 10s infinite alternate;
        }

        @keyframes move {
            from { transform: translate(-50%, -50%); }
            to { transform: translate(50%, 50%); }
        }

        .card {
            background: var(--card-bg);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(0, 242, 255, 0.2);
            border-radius: 24px;
            padding: 3rem;
            width: 380px;
            text-align: center;
            box-shadow: 0 0 40px rgba(0, 0, 0, 0.8), inset 0 0 20px rgba(0, 242, 255, 0.05);
        }

        h1 {
            font-size: 1.8rem;
            letter-spacing: 4px;
            margin-bottom: 0.5rem;
            background: linear-gradient(90deg, var(--primary), var(--secondary));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
        }

        .network-tag {
            display: inline-block;
            font-size: 0.7rem;
            background: rgba(0, 242, 255, 0.1);
            color: var(--primary);
            padding: 4px 12px;
            border-radius: 100px;
            border: 1px solid var(--primary);
            margin-bottom: 2rem;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 2rem;
        }

        .stat-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 16px;
        }

        .stat-label { font-size: 0.6rem; color: #888; text-transform: uppercase; margin-bottom: 5px; display: block; }
        .stat-value { font-size: 1.2rem; font-weight: bold; color: #fff; }

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
            transition: transform 0.2s, box-shadow 0.2s;
            box-shadow: 0 4px 15px rgba(112, 0, 255, 0.4);
        }

        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0, 242, 255, 0.6);
        }

        input {
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(255, 255, 255, 0.1);
            color: white;
            padding: 12px;
            border-radius: 8px;
            width: 60%;
            margin-bottom: 15px;
            font-size: 1.2rem;
            text-align: center;
        }

        #status { font-size: 0.75rem; color: #666; margin-top: 1.5rem; }
        .success-msg { color: var(--primary); font-size: 0.9rem; margin-top: 10px; }
    </style>
</head>
<body>

<div class="glow"></div>

<div class="card">
    <h1>TEMPO MINT</h1>
    <div class="network-tag">● Testnet Active</div>
    
    <div id="statsUI" class="stats-grid" style="display: none;">
        <div class="stat-card">
            <span class="stat-label">Global Supply</span>
            <span id="totalSupply" class="stat-value">0</span>
        </div>
        <div class="stat-card">
            <span class="stat-label">Your Wallet</span>
            <span id="userBalance" class="stat-value">0</span>
        </div>
    </div>

    <button id="connectBtn">Initialize Connection</button>

    <div id="mintUI" style="display: none;">
        <input type="number" id="qty" value="1" min="1">
        <button id="mintBtn">Execute Mint</button>
    </div>

    <div id="feedback"></div>
    <p id="status">Secure tunnel: disconnected</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, contract, userAddress;
    const contractAddress = "0xD84465DC85e23f18dC60a5975b4562d6a5B6DcbB";
    const tempoHex = "0xa5bd";

    const abi = [
        "function mint(uint256 quantity) public",
        "function totalSupply() view returns (uint256)",
        "function balanceOf(address owner) view returns (uint256)"
    ];

    async function updateStats() {
        try {
            const total = await contract.totalSupply();
            const balance = await contract.balanceOf(userAddress);
            document.getElementById('totalSupply').innerText = total.toString();
            document.getElementById('userBalance').innerText = balance.toString();
            document.getElementById('statsUI').style.display = 'grid';
        } catch (e) { console.log("Stats update failed"); }
    }

    async function connect() {
        if (!window.ethereum) return alert("Install Rabby!");
        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            await window.ethereum.request({
                method: 'wallet_addEthereumChain',
                params: [{
                    chainId: tempoHex,
                    chainName: 'Tempo Testnet',
                    rpcUrls: ['https://rpc.testnet.tempo.xyz'],
                    nativeCurrency: { name: 'USD', symbol: 'USD', decimals: 18 }
                }]
            });
            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            userAddress = await signer.getAddress();
            contract = new ethers.Contract(contractAddress, abi, signer);
            
            await updateStats();
            document.getElementById('status').innerText = `Tunnel active: ${userAddress.slice(0,6)}...${userAddress.slice(-4)}`;
            document.getElementById('connectBtn').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
        } catch (err) { console.error(err); }
    }

    async function mint() {
        const qty = document.getElementById('qty').value;
        const feedback = document.getElementById('feedback');
        try {
            feedback.innerHTML = "<p>Authorizing...</p>";
            const tx = await contract.mint(qty);
            feedback.innerHTML = "<p>Transmitting to Tempo...</p>";
            await tx.wait();
            await updateStats();
            feedback.innerHTML = `<p class="success-msg">Mint Complete.</p>`;
        } catch (err) {
            feedback.innerHTML = `<p style="color:#ff0055">Signal Failed.</p>`;
        }
    }

    document.getElementById('connectBtn').onclick = connect;
    document.getElementById('mintBtn').onclick = mint;
</script>

</body>
</html>
