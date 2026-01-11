<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tempo NFT Portal</title>
    <style>
        body { font-family: 'Inter', system-ui, sans-serif; background: #0d1117; color: #c9d1d9; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .card { background: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 2rem; width: 350px; text-align: center; box-shadow: 0 8px 24px rgba(0,0,0,0.5); }
        h1 { color: #f0f6fc; margin-bottom: 0.5rem; font-size: 1.5rem; }
        .stats-box { display: flex; justify-content: space-around; margin-bottom: 1.5rem; gap: 10px; }
        .stat-item { background: rgba(57, 211, 83, 0.1); padding: 10px; border-radius: 8px; flex: 1; }
        .stat-label { font-size: 0.7rem; color: #8b949e; text-transform: uppercase; display: block; }
        .stat-value { font-size: 1.1rem; color: #39d353; font-weight: bold; }
        #status { font-size: 0.8rem; color: #8b949e; margin-bottom: 1.5rem; word-break: break-all; }
        button { background: #238636; color: white; border: none; border-radius: 6px; padding: 12px; width: 100%; font-weight: 600; cursor: pointer; transition: 0.2s; margin-top: 10px; }
        button:hover { background: #2ea043; }
        button:disabled { background: #21262d; color: #484f58; cursor: not-allowed; }
        input { background: #0d1117; border: 1px solid #30363d; color: white; border-radius: 6px; padding: 10px; width: 80%; text-align: center; margin-bottom: 10px; font-size: 1rem; }
        .explorer-link { color: #58a6ff; text-decoration: none; font-size: 0.8rem; margin-top: 15px; display: block; }
        .error { color: #f85149; font-size: 0.8rem; margin-top: 10px; }
    </style>
</head>
<body>

<div class="card">
    <h1>Tempo Minter</h1>
    
    <div id="statsUI" class="stats-box" style="display: none;">
        <div class="stat-item">
            <span class="stat-label">Total Minted</span>
            <span id="totalSupply" class="stat-value">0</span>
        </div>
        <div class="stat-item">
            <span class="stat-label">Your NFTs</span>
            <span id="userBalance" class="stat-value">0</span>
        </div>
    </div>

    <p id="status">Wallet Not Connected</p>
    
    <button id="connectBtn">Connect Wallet</button>

    <div id="mintUI" style="display: none;">
        <input type="number" id="qty" value="1" min="1">
        <button id="mintBtn">Mint NFT</button>
    </div>

    <div id="feedback"></div>
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
            document.getElementById('statsUI').style.display = 'flex';
        } catch (e) { console.log("Stats fetch failed"); }
    }

    async function connect() {
        if (!window.ethereum) {
            alert("Rabby not found!");
            return;
        }

        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            
            await window.ethereum.request({
                method: 'wallet_addEthereumChain',
                params: [{
                    chainId: tempoHex,
                    chainName: 'Tempo Testnet',
                    rpcUrls: ['https://rpc.testnet.tempo.xyz'],
                    nativeCurrency: { name: 'USD', symbol: 'USD', decimals: 18 },
                    blockExplorerUrls: ['https://explore.tempo.xyz']
                }]
            });

            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            userAddress = await signer.getAddress();
            
            contract = new ethers.Contract(contractAddress, abi, signer);
            await updateStats();

            document.getElementById('status').innerText = `Connected: ${userAddress.slice(0,6)}...${userAddress.slice(-4)}`;
            document.getElementById('connectBtn').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
        } catch (err) {
            console.error(err);
        }
    }

    async function mint() {
        const qty = document.getElementById('qty').value;
        const feedback = document.getElementById('feedback');
        const mintBtn = document.getElementById('mintBtn');
        try {
            mintBtn.disabled = true;
            feedback.innerHTML = "<p>Confirming...</p>";
            const tx = await contract.mint(qty);
            feedback.innerHTML = "<p>Pending on Tempo...</p>";
            await tx.wait();
            await updateStats();
            feedback.innerHTML = `<p style="color: #39d353">Mint Successful!</p>`;
        } catch (err) {
            feedback.innerHTML = `<p class="error">Mint failed: ${err.reason || "Rejected"}</p>`;
        } finally {
            mintBtn.disabled = false;
        }
    }

    document.getElementById('connectBtn').onclick = connect;
    document.getElementById('mintBtn').onclick = mint;
</script>

</body>
</html>
