<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TEMPO // ATOMIC v4</title>
    <style>
        :root { --primary: #00f2ff; --secondary: #7000ff; --bg: #050505; }
        body { background: var(--bg); color: #fff; font-family: 'Space Grotesk', sans-serif; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; }
        .card { background: rgba(20, 20, 25, 0.95); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 28px; padding: 2.5rem; width: 380px; text-align: center; box-shadow: 0 0 50px rgba(112, 0, 255, 0.2); }
        h1 { font-size: 1.4rem; letter-spacing: 4px; background: linear-gradient(90deg, var(--primary), var(--secondary)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-transform: uppercase; margin-bottom: 2rem; }
        button { background: linear-gradient(135deg, var(--secondary), var(--primary)); border: none; color: white; padding: 20px; border-radius: 14px; width: 100%; font-weight: 900; text-transform: uppercase; cursor: pointer; }
        #feedback { margin-top: 20px; font-size: 0.8rem; color: var(--primary); min-height: 50px; }
        .history-item { background: rgba(255,255,255,0.05); padding: 8px; margin-top: 5px; border-radius: 5px; font-size: 0.7rem; font-family: monospace; display: flex; justify-content: space-between; }
    </style>
</head>
<body>

<div class="card">
    <h1>ATOMIC MINT</h1>
    <div id="ui">
        <button id="mainBtn">INITIALIZE LINK</button>
    </div>
    <div id="feedback"></div>
    <div id="history" style="margin-top: 20px; text-align: left;"></div>
    <p id="status" style="font-size: 0.6rem; color: #444; margin-top: 1.5rem;">STATUS: OFFLINE</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, state = "CONNECT";
    
    // We try multiple known Tempo RPCs
    const RPCS = [
        "https://rpc.moderato.testnet.tempo.xyz",
        "https://rpc.testnet.tempo.xyz",
        "https://tempo-testnet.rpc.thirdweb.com"
    ];

    async function handleAction() {
        const feedback = document.getElementById('feedback');
        const btn = document.getElementById('mainBtn');

        if (state === "CONNECT") {
            try {
                feedback.innerHTML = "Connecting to Rabby...";
                if (!window.ethereum) throw new Error("Rabby not found");

                // Try to add the network
                await window.ethereum.request({
                    method: 'wallet_addEthereumChain',
                    params: [{
                        chainId: "0x7A1", // 1953
                        chainName: 'Tempo Moderato',
                        rpcUrls: RPCS,
                        nativeCurrency: { name: 'TEMPO', symbol: 'TEMPO', decimals: 18 },
                        blockExplorerUrls: ["https://explore.tempo.xyz"]
                    }]
                });

                provider = new ethers.BrowserProvider(window.ethereum);
                await provider.send("eth_requestAccounts", []);
                signer = await provider.getSigner();
                
                state = "MINT";
                btn.innerText = "EXECUTE NEURAL LAUNCH";
                document.getElementById('status').innerText = "STATUS: ACTIVE";
                feedback.innerHTML = "Linked Successfully.";
            } catch (e) {
                feedback.innerHTML = `<span style="color:red">Error: ${e.message}</span>`;
            }
        } else {
            try {
                feedback.innerHTML = "Deploying fresh contract...";
                const ABI = ["constructor()"];
                const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033";
                
                const factory = new ethers.ContractFactory(ABI, BYTECODE, signer);
                const contract = await factory.deploy();
                feedback.innerHTML = "Awaiting block confirmation...";
                await contract.waitForDeployment();
                
                const addr = await contract.getAddress();
                feedback.innerHTML = "Deployment Success!";
                
                const item = document.createElement('div');
                item.className = 'history-item';
                item.innerHTML = `<span>${addr.slice(0,10)}...</span><a href="https://explore.tempo.xyz/address/${addr}" target="_blank" style="color:#00f2ff">VIEW</a>`;
                document.getElementById('history').appendChild(item);
            } catch (e) {
                feedback.innerHTML = `<span style="color:red">Deploy Failed: ${e.message.slice(0,40)}</span>`;
            }
        }
    }

    document.getElementById('mainBtn').onclick = handleAction;
</script>
</body>
</html>
