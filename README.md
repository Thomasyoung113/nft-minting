<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' https://cdnjs.cloudflare.com; connect-src 'self' https://rpc.testnet.tempo.xyz https://*.tempo.xyz; style-src 'self' 'unsafe-inline';">
    
    <title>Tempo NFT Portal</title>
    
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/index.umd.min.js"></script>

    <style>
        body { font-family: 'Inter', system-ui, sans-serif; background: #0d1117; color: #c9d1d9; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .card { background: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 2rem; width: 350px; text-align: center; box-shadow: 0 8px 24px rgba(0,0,0,0.5); }
        h1 { color: #f0f6fc; margin-bottom: 0.2rem; font-size: 1.5rem; }
        #supplyDisplay { font-size: 0.9rem; color: #39d353; font-weight: bold; margin-bottom: 1rem; background: rgba(57, 211, 83, 0.1); padding: 5px; border-radius: 5px; display: none; }
        #status { font-size: 0.8rem; color: #8b949e; margin-bottom: 1.5rem; word-break: break-all; }
        button { background: #238636; color: white; border: 1px solid rgba(240,246,252,0.1); border-radius: 6px; padding: 12px; width: 100%; font-weight: 600; cursor: pointer; transition: 0.2s; margin-top: 10px; }
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
    <div id="supplyDisplay">Total Minted: Loading...</div>
    <p id="status">Please connect your wallet</p>
    
    <button id="connectBtn" onclick="connect()">Connect Wallet</button>

    <div id="mintUI" style="display: none;">
        <input type="number" id="qty" value="1" min="1">
        <button id="mintBtn" onclick="mint()">Mint NFT</button>
    </div>

    <div id="feedback"></div>
</div>

<script>
    // We use a self-invoking pattern to avoid global scope issues
    let provider, signer, contract;
    const contractAddress = "0xD84465DC85e23f18dC60a5975b4562d6a5B6DcbB";
    const tempoHex = "0xa5bd";

    const abi = [
        "function mint(uint256 quantity) public",
        "function totalSupply() view returns (uint256)"
    ];

    async function refreshSupply() {
        try {
            const count = await contract.totalSupply();
            const supplyEl = document.getElementById('supplyDisplay');
            supplyEl.innerText = `Total Minted: ${count.toString()}`;
            supplyEl.style.display = 'block';
        } catch (e) {
            console.warn("Could not fetch supply yet.");
        }
    }

    async function connect() {
        if (!window.ethereum) {
            alert("No wallet detected. Please install Rabby.");
            return;
        }

        try {
            // Using window.ethereum directly as the provider source
            provider = new ethers.BrowserProvider(window.ethereum);
            
            // Auto-switch to Tempo
            try {
                await window.ethereum.request({
                    method: 'wallet_switchEthereumChain',
                    params: [{ chainId: tempoHex }],
                });
            } catch (e) {
                if (e.code === 4902) {
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
                }
            }

            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            const addr = await signer.getAddress();
            
            contract = new ethers.Contract(contractAddress, abi, signer);

            await refreshSupply();

            document.getElementById('status').innerText = `Connected: ${addr.slice(0,6)}...${addr.slice(-4)}`;
            document.getElementById('connectBtn').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
        } catch (err) {
            console.error("Connection error:", err);
            document.getElementById('feedback').innerHTML = `<p class="error">Connection failed.</p>`;
        }
    }

    async function mint() {
        const btn = document.getElementById('mintBtn');
        const feedback = document.getElementById('feedback');
        const qty = document.getElementById('qty').value;

        try {
            btn.disabled = true;
            feedback.innerHTML = "<p>Check Wallet...</p>";
            
            const tx = await contract.mint(qty);
            feedback.innerHTML = "<p>Transaction pending...</p>";
            
            await tx.wait();
            await refreshSupply();
            
            feedback.innerHTML = `<p style="color: #39d353">Success!</p>
                                 <a class="explorer-link" href="https://explore.tempo.xyz/tx/${tx.hash}" target="_blank">View on Explorer</a>`;
        } catch (err) {
            feedback.innerHTML = `<p class="error">Mint failed: ${err.reason || "Rejected"}</p>`;
        } finally {
            btn.disabled = false;
        }
    }
</script>

</body>
</html>
