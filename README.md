<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tempo NFT Portal</title>
    <style>
        body { font-family: 'Inter', system-ui, sans-serif; background: #0d1117; color: #c9d1d9; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; }
        .card { background: #161b22; border: 1px solid #30363d; border-radius: 12px; padding: 2rem; width: 350px; text-align: center; box-shadow: 0 8px 24px rgba(0,0,0,0.5); }
        h1 { color: #f0f6fc; margin-bottom: 0.2rem; font-size: 1.5rem; }
        #supplyDisplay { font-size: 0.9rem; color: #39d353; font-weight: bold; margin-bottom: 1rem; background: rgba(57, 211, 83, 0.1); padding: 5px; border-radius: 5px; display: none; }
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
    <div id="supplyDisplay">Total Minted: Loading...</div>
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
        } catch (e) { console.log("Supply check skipped"); }
    }

    async function connect() {
        if (!window.ethereum) {
            alert("Rabby not found. Please install the extension.");
            return;
        }

        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            
            // Trigger Rabby network add/switch
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
            const addr = await signer.getAddress();
            
            contract = new ethers.Contract(contractAddress, abi, signer);
            await refreshSupply();

            document.getElementById('status').innerText = `Connected: ${addr.slice(0,6)}...${addr.slice(-4)}`;
            document.getElementById('connectBtn').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
        } catch (err) {
            console.error(err);
            document.getElementById('feedback').innerHTML = `<p class="error">Check Rabby for notifications.</p>`;
        }
    }

    async function mint() {
        const qty = document.getElementById('qty').value;
        const feedback = document.getElementById('feedback');
        try {
            feedback.innerHTML = "<p>Confirming...</p>";
            const tx = await contract.mint(qty);
            feedback.innerHTML = "<p>Pending...</p>";
            await tx.wait();
            await refreshSupply();
            feedback.innerHTML = `<p style="color: #39d353">Success!</p>`;
        } catch (err) {
            feedback.innerHTML = `<p class="error">Error: ${err.action || "Rejected"}</p>`;
        }
    }

    // Attach functions to buttons manually (best for module type)
    document.getElementById('connectBtn').onclick = connect;
    document.getElementById('mintBtn').onclick = mint;
</script>

</body>
</html>
