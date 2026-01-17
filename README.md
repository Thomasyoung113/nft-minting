<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TEMPO // MASS DEPLOYER + REFUEL</title>
    <style>
        :root { --primary: #00f2ff; --secondary: #7000ff; --bg: #050505; }
        body { background: var(--bg); color: #fff; font-family: 'Space Grotesk', sans-serif; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; position: relative; }
        
        /* Top Right Faucet Button */
        .faucet-box { position: absolute; top: 20px; right: 20px; }
        #faucetBtn { 
            background: transparent; border: 1px solid var(--primary); color: var(--primary); 
            padding: 10px 15px; border-radius: 8px; font-size: 0.7rem; font-weight: bold; 
            cursor: pointer; transition: 0.3s; text-transform: uppercase; letter-spacing: 1px;
        }
        #faucetBtn:hover { background: var(--primary); color: #000; box-shadow: 0 0 15px var(--primary); }

        .card { background: rgba(20, 20, 25, 0.95); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 28px; padding: 2.5rem; width: 400px; text-align: center; box-shadow: 0 0 50px rgba(112, 0, 255, 0.2); }
        h1 { font-size: 1.4rem; letter-spacing: 4px; background: linear-gradient(90deg, var(--primary), var(--secondary)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-transform: uppercase; margin-bottom: 1.5rem; }
        
        .quantity-box { display: flex; align-items: center; justify-content: center; gap: 15px; margin-bottom: 20px; border: 1px solid rgba(255,255,255,0.1); padding: 10px; border-radius: 12px; }
        .quantity-box input { background: transparent; border: 1px solid var(--primary); color: var(--primary); text-align: center; width: 60px; font-weight: bold; border-radius: 5px; padding: 5px; outline: none; }
        
        button#mainBtn { background: linear-gradient(135deg, var(--secondary), var(--primary)); border: none; color: white; padding: 18px; border-radius: 12px; width: 100%; font-weight: 900; text-transform: uppercase; cursor: pointer; transition: 0.3s; }
        button:disabled { opacity: 0.4; }
        
        #feedback { margin-top: 20px; font-size: 0.8rem; color: var(--primary); min-height: 50px; }
        .history-list { max-height: 200px; overflow-y: auto; margin-top: 15px; }
        .history-item { background: rgba(255,255,255,0.05); padding: 8px; margin-top: 5px; border-radius: 6px; font-size: 0.6rem; font-family: monospace; display: flex; justify-content: space-between; align-items: center; }
    </style>
</head>
<body>

<div class="faucet-box">
    <button id="faucetBtn">⛽ Refuel Ajeh</button>
</div>

<div class="card">
    <h1>MASS DEPLOYER</h1>
    
    <div id="mintOptions" style="display:none;">
        <div class="quantity-box">
            <span>QUANTITY</span>
            <input type="number" id="mintCount" value="1" min="1" max="50">
        </div>
    </div>

    <div id="ui"><button id="mainBtn">INITIALIZE LINK</button></div>
    
    <div id="feedback"></div>
    <div id="history" class="history-list"></div>
    
    <p style="font-size:0.6rem; color:#444; margin-top:1.5rem;">AJEH 42431 // ATOMIC BURST MODE</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, state = "CONNECT";
    
    const CHAIN_ID_HEX = "0xA5BF"; 
    const RPC_URL = "https://rpc.moderato.tempo.xyz"; 
    const EXPLORER = "https://explore.tempo.xyz";
    const FAUCET_URL = "https://faucet.tempo.xyz"; // Official Faucet Link

    const ABI = ["constructor()"];
    const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033";

    async function handleAction() {
        const feedback = document.getElementById('feedback');
        const btn = document.getElementById('mainBtn');

        if (state === "CONNECT") {
            try {
                await window.ethereum.request({
                    method: 'wallet_addEthereumChain', params: [{
                        chainId: CHAIN_ID_HEX, chainName: 'Tempo Ajeh',
                        rpcUrls: [RPC_URL], nativeCurrency: { name: 'TEMPO', symbol: 'TEMPO', decimals: 18 },
                        blockExplorerUrls: [EXPLORER]
                    }]
                });
                provider = new ethers.BrowserProvider(window.ethereum);
                await provider.send("eth_requestAccounts", []);
                signer = await provider.getSigner();
                state = "MINT";
                btn.innerText = "START ATOMIC BURST";
                document.getElementById('mintOptions').style.display = 'block';
                feedback.innerHTML = "Link Established.";
            } catch (e) { feedback.innerHTML = "Error: " + e.message; }
        } else {
            const count = parseInt(document.getElementById('mintCount').value);
            btn.disabled = true;
            
            for(let i = 0; i < count; i++) {
                try {
                    feedback.innerHTML = `Deploying Instance ${i + 1} of ${count}...`;
                    const factory = new ethers.ContractFactory(ABI, BYTECODE, signer);
                    const contract = await factory.deploy();
                    
                    await contract.waitForDeployment();
                    const addr = await contract.getAddress();
                    
                    const item = document.createElement('div');
                    item.className = 'history-item';
                    item.innerHTML = `<span>#${i+1}: ${addr.slice(0,10)}...</span><a href="${EXPLORER}/address/${addr}" target="_blank" style="color:var(--primary)">[VIEW]</a>`;
                    document.getElementById('history').prepend(item);
                } catch (e) {
                    feedback.innerHTML = `<span style="color:red">Burst Interrupted at ${i+1}</span>`;
                    break;
                }
            }
            btn.disabled = false;
            feedback.innerHTML = "Mass Deployment Sequence Complete.";
        }
    }

    // Faucet Logic
    document.getElementById('faucetBtn').onclick = () => {
        // Opens official faucet in a new tab for refueling
        window.open(FAUCET_URL, '_blank');
    };

    document.getElementById('mainBtn').onclick = handleAction;
</script>
</body>
</html>
