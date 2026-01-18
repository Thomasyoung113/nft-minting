<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TEMPO // AJEH ATOMIC V12</title>
    <style>
        :root { --primary: #00f2ff; --secondary: #7000ff; --bg: #050505; }
        body { background: var(--bg); color: #fff; font-family: 'Space Grotesk', sans-serif; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; padding: 20px; position: relative; overflow-x: hidden; }
        
        .dashboard { position: absolute; top: 20px; right: 20px; display: flex; align-items: center; gap: 15px; }
        .balance-display { font-family: monospace; font-size: 0.8rem; color: var(--primary); background: rgba(0, 242, 255, 0.1); padding: 8px 12px; border-radius: 8px; border: 1px solid rgba(0, 242, 255, 0.3); }
        
        #faucetBtn { 
            background: linear-gradient(45deg, #ff0055, var(--secondary)); border: none; color: white; 
            padding: 10px 18px; border-radius: 8px; font-size: 0.7rem; font-weight: bold; 
            cursor: pointer; transition: 0.3s; text-transform: uppercase;
        }
        #faucetBtn:hover { transform: translateY(-2px); box-shadow: 0 5px 15px rgba(255, 0, 85, 0.4); }

        .card { background: rgba(20, 20, 25, 0.95); border: 1px solid rgba(0, 242, 255, 0.2); border-radius: 28px; padding: 2.5rem; width: 420px; text-align: center; box-shadow: 0 0 50px rgba(112, 0, 255, 0.2); z-index: 10; }
        h1 { font-size: 1.4rem; letter-spacing: 4px; background: linear-gradient(90deg, var(--primary), var(--secondary)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-transform: uppercase; margin-bottom: 1.5rem; }
        
        .quantity-box { display: flex; align-items: center; justify-content: center; gap: 15px; margin-bottom: 20px; border: 1px solid rgba(255,255,255,0.1); padding: 12px; border-radius: 12px; background: rgba(255,255,255,0.02); }
        .quantity-box input { background: #000; border: 1px solid var(--primary); color: var(--primary); text-align: center; width: 70px; font-weight: bold; border-radius: 5px; padding: 8px; outline: none; font-size: 1rem; }
        
        button#mainBtn { background: linear-gradient(135deg, var(--secondary), var(--primary)); border: none; color: white; padding: 20px; border-radius: 14px; width: 100%; font-weight: 900; text-transform: uppercase; cursor: pointer; transition: 0.3s; letter-spacing: 1px; }
        button:disabled { opacity: 0.3; cursor: wait; }
        
        #feedback { margin-top: 20px; font-size: 0.8rem; color: var(--primary); min-height: 50px; line-height: 1.5; }
        .history-list { max-height: 180px; overflow-y: auto; margin-top: 15px; scrollbar-width: thin; }
        .history-item { background: rgba(255,255,255,0.03); padding: 10px; margin-top: 6px; border-radius: 8px; font-size: 0.65rem; font-family: monospace; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(0, 242, 255, 0.1); }
    </style>
</head>
<body>

<div class="dashboard">
    <div class="balance-display" id="balContainer" style="display:none;">
        <span id="walletBal">0.00</span> USD
    </div>
    <button id="faucetBtn">CLAIM TEST USD</button>
</div>

<div class="card">
    <h1>AJEH MASS DEPLOY</h1>
    
    <div id="mintOptions" style="display:none;">
        <div class="quantity-box">
            <span>BURST COUNT</span>
            <input type="number" id="mintCount" value="1" min="1" max="100">
        </div>
    </div>

    <div id="ui"><button id="mainBtn">CONNECT NEURAL LINK</button></div>
    
    <div id="feedback"></div>
    <div id="history" class="history-list"></div>
    
    <p style="font-size:0.55rem; color:#555; margin-top:1.5rem; letter-spacing: 2px;">CHAIN ID 42431 // MODERATO UPGRADE</p>
</div>

<script type="module">
    import { ethers } from "https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js";

    let provider, signer, state = "CONNECT";
    
    const CHAIN_ID_HEX = "0xA5BF"; 
    const RPC_URL = "https://rpc.moderato.tempo.xyz"; 
    const EXPLORER = "https://explore.tempo.xyz";
    
    // Faucet and Token Specs from docs.tempo.xyz
    const FAUCET_PAGE = "https://docs.tempo.xyz/quickstart/faucet";
    const BYTECODE = "0x608060405234801561001057600080fd5b5061012b806100206000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80634f6d7ef314602d575b600080fd5b60336047565b6040518082815260200191505060405180910390f35b6000600190509056fea2646970667358221220ec7b274c4249a62292f7e8006e8e5f73906a77d3202951f2f01f440590a931a764736f6c63430008120033";
    const ABI = ["constructor()"];

    async function updateBalance() {
        if (!signer) return;
        try {
            const address = await signer.getAddress();
            const balance = await provider.getBalance(address);
            document.getElementById('walletBal').innerText = parseFloat(ethers.formatEther(balance)).toFixed(2);
            document.getElementById('balContainer').style.display = 'block';
        } catch (e) { console.warn("Balance update skipped"); }
    }

    async function handleAction() {
        const feedback = document.getElementById('feedback');
        const btn = document.getElementById('mainBtn');

        if (state === "CONNECT") {
            try {
                feedback.innerHTML = "Initializing Connection...";
                await window.ethereum.request({
                    method: 'wallet_addEthereumChain', params: [{
                        chainId: CHAIN_ID_HEX, chainName: 'Tempo Moderato',
                        rpcUrls: [RPC_URL], 
                        nativeCurrency: { name: 'USD', symbol: 'USD', decimals: 18 },
                        blockExplorerUrls: [EXPLORER]
                    }]
                });
                provider = new ethers.BrowserProvider(window.ethereum);
                await provider.send("eth_requestAccounts", []);
                signer = await provider.getSigner();
                
                state = "MINT";
                btn.innerText = "EXECUTE ATOMIC BURST";
                document.getElementById('mintOptions').style.display = 'block';
                feedback.innerHTML = "Neural Link Active.";
                updateBalance();
            } catch (e) { feedback.innerHTML = "Link Error: " + e.message; }
        } else {
            const count = parseInt(document.getElementById('mintCount').value);
            btn.disabled = true;
            
            for(let i = 0; i < count; i++) {
                try {
                    feedback.innerHTML = `BURSTING: ${i + 1} / ${count}<br><small>Confirm in Wallet</small>`;
                    const factory = new ethers.ContractFactory(ABI, BYTECODE, signer);
                    const contract = await factory.deploy();
                    
                    await contract.waitForDeployment();
                    const addr = await contract.getAddress();
                    
                    const item = document.createElement('div');
                    item.className = 'history-item';
                    item.innerHTML = `<span>ATOMIC #${i+1}: ${addr.slice(0,8)}...</span><a href="${EXPLORER}/address/${addr}" target="_blank" style="color:var(--primary); text-decoration:none;">[VIEW]</a>`;
                    document.getElementById('history').prepend(item);
                    updateBalance();
                } catch (e) {
                    feedback.innerHTML = `<span style="color:#ff0055">BURST TERMINATED: ${e.message.slice(0,40)}</span>`;
                    break;
                }
            }
            btn.disabled = false;
            feedback.innerHTML = "Full Deployment Sequence Confirmed.";
        }
    }

    // Faucet Direct Integration
    document.getElementById('faucetBtn').onclick = async () => {
        const feedback = document.getElementById('feedback');
        if (state === "CONNECT") {
            alert("Please connect your wallet first.");
            return;
        }
        
        // Since Tempo's new faucet is web-based to protect against bots, 
        // this triggers the direct link provided in the quickstart docs.
        window.open(FAUCET_PAGE, '_blank');
        feedback.innerHTML = "Refueling window opened. Claim USD and return.";
    };

    document.getElementById('mainBtn').onclick = handleAction;
</script>
</body>
</html>
