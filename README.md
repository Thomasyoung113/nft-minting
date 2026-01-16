<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // ATOMIC DEPLOYER</title>
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
        }

        .card {
            background: var(--card-bg);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(0, 242, 255, 0.2);
            border-radius: 28px;
            padding: 2.5rem;
            width: 380px;
            text-align: center;
            box-shadow: 0 0 50px rgba(112, 0, 255, 0.2);
        }

        h1 {
            font-size: 1.4rem;
            letter-spacing: 4px;
            background: linear-gradient(90deg, var(--primary), var(--secondary));
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            text-transform: uppercase;
            margin-bottom: 1rem;
        }

        .desc { font-size: 0.7rem; color: #888; margin-bottom: 2rem; line-height: 1.4; }

        button {
            background: linear-gradient(135deg, var(--secondary), var(--primary));
            border: none;
            color: white;
            padding: 20px;
            border-radius: 14px;
            width: 100%;
            font-weight: 900;
            text-transform: uppercase;
            letter-spacing: 2px;
            cursor: pointer;
            transition: all 0.3s;
        }

        button:hover {
            box-shadow: 0 0 30px rgba(0, 242, 255, 0.5);
            transform: scale(1.02);
        }

        #feedback { margin-top: 20px; font-size: 0.8rem; min-height: 40px;}
        .address-link { color: var(--primary); font-family: monospace; font-size: 0.7rem; word-break: break-all; }
        #status { font-size: 0.6rem; color: #444; margin-top: 1.5rem; letter-spacing: 2px; text-transform: uppercase;}
        .error { color: #ff0055; font-size: 0.7rem; }
    </style>
</head>
<body>

<div class="card">
    <h1>ATOMIC MINT</h1>
    <p class="desc">Each click spawns a unique, fresh NFT contract on Tempo Moderato.</p>
    
    <div id="authUI">
        <button id="connectBtn">Initialize Link</button>
    </div>

    <div id="mintUI" style="display: none;">
        <button id="deployBtn">Execute Neural Launch</button>
    </div>

    <div id="feedback"></div>
    <p id="status">NETWORK // DISCONNECTED</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer;
    
    // Updated based on your Rabby Error Screen
    const chainIdDecimal = 1953;
    const moderatoHex = "0x7A1"; // Hex for 1953
    const rpc = "https://rpc.testnet.tempo.xyz"; // Standard endpoint
    const explorer = "https://explore.tempo.xyz";

    // Minimal ERC721 Proxy Bytecode
    const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033"; 
    const ABI = ["constructor()"];

    async function connect() {
        const feedback = document.getElementById('feedback');
        const status = document.getElementById('status');
        
        if (!window.ethereum) {
            feedback.innerHTML = "<span class='error'>Rabby Wallet not detected.</span>";
            return;
        }

        try {
            provider = new ethers.BrowserProvider(window.ethereum);
            
            // Force Rabby to add/switch the network
            feedback.innerHTML = "Requesting Network Sync...";
            try {
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
            } catch (rpcErr) {
                console.warn("Network switch failed, likely already added.");
            }

            await provider.send("eth_requestAccounts", []);
            signer = await provider.getSigner();
            const addr = await signer.getAddress();
            
            document.getElementById('authUI').style.display = 'none';
            document.getElementById('mintUI').style.display = 'block';
            status.innerText = `TUNNEL ACTIVE // ${addr.slice(0,10)}...`;
            feedback.innerHTML = "";
            
        } catch (err) {
            console.error(err);
            feedback.innerHTML = `<span class='error'>Link Failed: ${err.message.slice(0, 50)}...</span>`;
        }
    }

    async function deployAsMint() {
        const feedback = document.getElementById('feedback');
        feedback.innerHTML = "Generating Atomic Contract...";

        try {
            const factory = new ethers.ContractFactory(ABI, BYTECODE, signer);
            
            feedback.innerHTML = "Awaiting Neural Signature...";
            const contract = await factory.deploy();
            
            feedback.innerHTML = "Broadcasting to Moderato...";
            await contract.waitForDeployment();
            
            const newAddress = await contract.getAddress();
            
            feedback.innerHTML = `
                <div style="color: var(--primary); margin-bottom: 5px;">SUCCESS: FRESH CONTRACT SPAWNED</div>
                <a href="${explorer}/address/${newAddress}" target="_blank" class="address-link">${newAddress}</a>
            `;
        } catch (err) {
            console.error(err);
            feedback.innerHTML = "<span class='error'>Deployment Interrupted.</span>";
        }
    }

    document.getElementById('connectBtn').onclick = connect;
    document.getElementById('deployBtn').onclick = deployAsMint;

    // Handle network/account changes
    if (window.ethereum) {
        window.ethereum.on('chainChanged', () => window.location.reload());
        window.ethereum.on('accountsChanged', () => window.location.reload());
    }
</script>

</body>
</html>
