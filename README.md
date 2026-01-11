<!DOCTYPE html>
<html>
<head>
    <title>Tempo NFT Minting Portal</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/index.umd.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; padding: 50px; background-color: #0d1117; color: white; }
        .container { background: #161b22; padding: 40px; border-radius: 20px; border: 1px solid #30363d; display: inline-block; max-width: 400px; }
        button { background: #238636; color: white; border: none; padding: 12px 24px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 16px; margin-top: 20px; width: 100%; }
        button:hover { background: #2ea043; }
        input { width: 80%; padding: 10px; margin: 10px 0; border-radius: 6px; border: 1px solid #30363d; background: #0d1117; color: white; text-align: center; }
        #status { color: #8b949e; margin-bottom: 20px; word-break: break-all; font-size: 14px; }
        .success { color: #39d353; font-weight: bold; }
    </style>
</head>
<body>

<div class="container">
    <h1>Tempo Minter</h1>
    <p id="status">Wallet Not Connected</p>
    <button id="connectBtn" onclick="connectWallet()">Connect Rabby Wallet</button>
    
    <div id="mintSection" style="display:none; margin-top: 30px;">
        <label>Quantity to Mint:</label><br>
        <input type="number" id="mintAmount" value="1" min="1">
        <button onclick="mintNFT()">Mint NFT</button>
    </div>
    
    <p id="txStatus"></p>
</div>

<script>
    let signer;
    let contract;
    const contractAddress = "0xD84465DC85e23f18dC60a5975b4562d6a5B6DcbB";
    const tempoChainId = "0xa5bd"; // 42429

    const abi = [
        "function mint(uint256 quantity) public",
        "function totalSupply() view returns (uint256)"
    ];

    async function connectWallet() {
        if (!window.ethereum) {
            alert("Rabby Wallet not found!");
            return;
        }

        try {
            const provider = new ethers.BrowserProvider(window.ethereum);
            await provider.send("eth_requestAccounts", []);
            
            // Switch to Tempo Testnet
            await window.ethereum.request({
                method: 'wallet_switchEthereumChain',
                params: [{ chainId: tempoChainId }],
            }).catch(async (err) => {
                if (err.code === 4902) {
                    await window.ethereum.request({
                        method: 'wallet_addEthereumChain',
                        params: [{
                            chainId: tempoChainId,
                            chainName: 'Tempo Testnet',
                            nativeCurrency: { name: 'USD', symbol: 'USD', decimals: 18 },
                            rpcUrls: ['https://rpc.testnet.tempo.xyz'],
                            blockExplorerUrls: ['https://explore.tempo.xyz']
                        }]
                    });
                }
            });

            signer = await provider.getSigner();
            contract = new ethers.Contract(contractAddress, abi, signer);
            
            const address = await signer.getAddress();
            document.getElementById("status").innerText = "Connected: " + address;
            document.getElementById("connectBtn").style.display = "none";
            document.getElementById("mintSection").style.display = "block";
        } catch (error) {
            console.error(error);
            alert("Failed to connect.");
        }
    }

    async function mintNFT() {
        const qty = document.getElementById("mintAmount").value;
        const txStatus = document.getElementById("txStatus");
        
        try {
            txStatus.innerText = "Confirm in Rabby...";
            const tx = await contract.mint(qty);
            txStatus.innerText = "Processing transaction...";
            await tx.wait();
            txStatus.innerHTML = `<span class="success">Mint Successful!</span><br><a href="https://explore.tempo.xyz/tx/${tx.hash}" target="_blank" style="color:#58a6ff">View Receipt</a>`;
        } catch (error) {
            txStatus.innerText = "Error: " + (error.reason || "Transaction Canceled");
        }
    }
</script>

</body>
</html># nft-minting
