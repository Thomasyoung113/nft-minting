<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // AJEH 42431</title>
    <style>
        :root { --primary: #00f2ff; --secondary: #7000ff; --bg: #050505; }
        body { background: var(--bg); color: #fff; font-family: 'Space Grotesk', sans-serif; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; }
        .card { background: rgba(20, 20, 25, 0.95); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 28px; padding: 2.5rem; width: 380px; text-align: center; box-shadow: 0 0 50px rgba(112, 0, 255, 0.2); }
        h1 { font-size: 1.4rem; letter-spacing: 4px; background: linear-gradient(90deg, var(--primary), var(--secondary)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-transform: uppercase; margin-bottom: 2rem; }
        button { background: linear-gradient(135deg, var(--secondary), var(--primary)); border: none; color: white; padding: 18px; border-radius: 12px; width: 100%; font-weight: 900; text-transform: uppercase; cursor: pointer; transition: 0.3s; }
        button:disabled { opacity: 0.4; cursor: not-allowed; }
        #feedback { margin-top: 20px; font-size: 0.8rem; color: var(--primary); min-height: 50px; }
        .history-item { background: rgba(255,255,255,0.05); padding: 8px; margin-top: 6px; border-radius: 6px; font-size: 0.65rem; font-family: monospace; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(0, 242, 255, 0.1); }
        #status { font-size: 0.6rem; color: #555; margin-top: 1.5rem; letter-spacing: 2px; }
    </style>
</head>
<body>

<div class="card">
    <h1>ATOMIC MINT</h1>
    <div id="ui"><button id="mainBtn">INITIALIZE LINK</button></div>
    <div id="feedback"></div>
    <div id="history" style="margin-top: 20px; text-align: left;"></div>
    <p id="status">STATUS: OFFLINE // AJEH 42431</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, state = "CONNECT";
    
    // AJEH NETWORK SPECIFICATIONS
    const CHAIN_ID_HEX = "0xA5BF"; // 42431 in Hex
    const RPC_URL = "https://rpc.moderato.tempo.xyz"; 
    const EXPLORER = "https://explore.tempo.xyz";

    async function handleAction() {
        const feedback = document.getElementById('feedback');
        const btn = document.getElementById('mainBtn');

        if (state === "CONNECT") {
            try {
                feedback.innerHTML = "Syncing Neural Node (Ajeh 42431)...";
                if (!window.ethereum) throw new Error("Rabby not detected.");

                // Requesting Network Switch/Add
                await window.ethereum.request({
                    method: 'wallet_addEthereumChain',
                    params: [{
                        chainId: CHAIN_ID_HEX,
                        chainName: 'Tempo Moderato (Ajeh)',
                        rpcUrls: [RPC_URL],
                        nativeCurrency: { name: 'TEMPO', symbol: 'TEMPO', decimals: 18 },
                        blockExplorerUrls: [EXPLORER]
                    }]
                });

                provider = new ethers.BrowserProvider(window.ethereum);
                await provider.send("eth_requestAccounts", []);
                signer = await provider.getSigner();
                
                state = "MINT";
                btn.innerText = "EXECUTE NEURAL LAUNCH";
                document.getElementById('status').innerText = "STATUS: ACTIVE // AJEH 42431";
                feedback.innerHTML = "Neural Link Established.";
            } catch (e) {
                console.error(e);
                feedback.innerHTML = `<span style="color:#ff0055">Error: ${e.message.slice(0, 50)}</span>`;
            }
        } else {
            try {
                btn.disabled = true;
                feedback.innerHTML = "Deploying fresh atomic contract...";
                
                // Minimalist contract to ensure fast deployment
                const ABI = ["constructor()"];
                const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033";
                
                const factory = new ethers.ContractFactory(ABI, BYTECODE, signer);
                const contract = await factory.deploy();
                
                feedback.innerHTML = "Broadcasting Neural Print...";
                await contract.waitForDeployment();
                
                const addr = await contract.getAddress();
                feedback.innerHTML = `<span style="color:#00f2ff">Launch Success!</span>`;
                
                // Update Local UI History
                const item = document.createElement('div');
                item.className = 'history-item';
                item.innerHTML = `<span>${addr.slice(0,12)}...</span><a href="${EXPLORER}/address/${addr}" target="_blank" style="color:var(--primary); text-decoration:none;">[EXPLORER]</a>`;
                document.getElementById('history').prepend(item);
                
            } catch (e) {
                console.error(e);
                feedback.innerHTML = `<span style="color:#ff0055">Launch Interrupted.</span>`;
            } finally {
                btn.disabled = false;
            }
        }
    }

    document.getElementById('mainBtn').onclick = handleAction;
</script>
</body>
</html>
