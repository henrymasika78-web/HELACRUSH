<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HELACRUSH - Official</title>
    <style>
        :root {
            --bg-dark: #0f172a; --card-bg: #1e293b; --primary: #22c55e;
            --danger: #ef4444; --accent: #3b82f6; --gold: #f59e0b;
        }
        body { margin: 0; font-family: 'Inter', sans-serif; background: var(--bg-dark); color: white; }
        .container { max-width: 450px; margin: 20px auto; background: var(--card-bg); padding: 20px; border-radius: 16px; box-shadow: 0 20px 50px rgba(0,0,0,0.5); }
        
        /* Game Screen */
        .game-canvas {
            position: relative; height: 250px; background: #000; border-radius: 12px;
            overflow: hidden; border: 1px solid #334155; margin-bottom: 20px;
        }
        #multiplier {
            position: absolute; width: 100%; text-align: center; top: 40%;
            font-size: 4rem; font-weight: 900; color: var(--primary); z-index: 10;
        }
        #plane {
            position: absolute; bottom: 10%; left: 10%; width: 50px; transition: transform 0.1s linear;
        }

        /* UI Elements */
        nav { display: none; justify-content: space-around; padding: 15px; background: #000; position: sticky; top: 0; z-index: 100; }
        .btn { width: 100%; padding: 15px; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; margin-top: 10px; transition: 0.3s; }
        .btn-pay { background: var(--primary); color: white; }
        .btn-cashout { background: var(--gold); color: black; display: none; }
        .input-group { margin-bottom: 15px; }
        input { width: 100%; padding: 12px; border-radius: 8px; border: 1px solid #334155; background: #0f172a; color: white; box-sizing: border-box; }
        .section { display: none; }
        .active { display: block; }
        .history-tag { display: inline-block; padding: 4px 8px; background: #334155; border-radius: 4px; margin: 2px; font-size: 11px; }
    </style>
</head>
<body>

<nav id="navbar">
    <a onclick="showPage('game-page')">✈️ Play</a>
    <a onclick="showPage('wallet-page')">💰 Wallet</a>
    <a onclick="logout()">Logout</a>
</nav>

<div id="auth-page" class="container active">
    <h2 style="text-align:center; color: var(--primary)">HELACRUSH</h2>
    <div class="input-group">
        <input type="text" id="phone" placeholder="Safaricom Phone (254...)">
    </div>
    <div class="input-group">
        <input type="password" id="pin" placeholder="Game PIN">
    </div>
    <button class="btn btn-pay" onclick="handleLogin()">Login / Register</button>
</div>

<div id="game-page" class="container section">
    <div style="display:flex; justify-content: space-between; margin-bottom: 10px;">
        <span>Wallet: <b id="balance">0.00</b> KES</span>
    </div>

    <div class="game-canvas">
        <div id="multiplier">1.00x</div>
        <img id="plane" src="https://cdn-icons-png.flaticon.com/512/3122/3122315.png" alt="plane">
    </div>

    <input type="number" id="bet-amount" placeholder="Stake Amount (min 10)">
    <button id="bet-btn" class="btn btn-pay" onclick="placeBet()">BET</button>
    <button id="cashout-btn" class="btn btn-cashout" onclick="cashOut()">CASHOUT</button>

    <div id="history" style="margin-top: 20px;"></div>
</div>

<div id="wallet-page" class="container section">
    <h3>M-Pesa Deposit</h3>
    <input type="number" id="dep-amount" placeholder="Amount (min 10)">
    <button class="btn btn-pay" onclick="initiateDeposit()">STK PUSH DEPOSIT</button>
</div>

<script>
    let user = null;
    let gameActive = false;
    let multiplier = 1.00;
    let tickInterval;
    let currentCrashPoint = 0;

    // --- NAVIGATION ---
    function showPage(id) {
        document.querySelectorAll('.section, .active').forEach(p => p.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        if(id !== 'auth-page') document.getElementById('navbar').style.display = 'flex';
    }

    // --- AUTH (Local Simulation for this demo) ---
    function handleLogin() {
        const phone = document.getElementById('phone').value;
        if(phone.length < 10) return alert("Enter valid Safaricom number");
        
        user = JSON.parse(localStorage.getItem(phone)) || { phone, balance: 0, history: [] };
        localStorage.setItem(phone, JSON.stringify(user));
        
        document.getElementById('balance').innerText = user.balance.toFixed(2);
        showPage('game-page');
    }

    // --- REAL MONEY M-PESA LOGIC ---
    async function initiateDeposit() {
        const amount = document.getElementById('dep-amount').value;
        if(amount < 10) return alert("Min 10 KES");

        alert("Requesting STK Push to " + user.phone);
        
        // In production, you would fetch your Node.js/PHP API here:
        // const res = await fetch('/api/mpesa/stkpush', { method: 'POST', body: ... });
        
        // Simulating a successful M-Pesa Callback
        setTimeout(() => {
            user.balance += parseFloat(amount);
            updateUser();
            alert("Deposit Confirmed!");
            showPage('game-page');
        }, 3000);
    }

    // --- CORE GAME LOGIC ---
    function placeBet() {
        const amt = parseFloat(document.getElementById('bet-amount').value);
        if(amt > user.balance || amt < 10) return alert("Invalid Balance/Amount");

        user.balance -= amt;
        updateUser();
        
        // SECURE STEP: In a real app, you fetch the crash point from the server 
        // using an encrypted hash so the player can't see it in the code.
        currentCrashPoint = (Math.random() * 5 + 1).toFixed(2); 
        
        startFlight(amt);
    }

    function startFlight(bet) {
        gameActive = true;
        multiplier = 1.00;
        document.getElementById('bet-btn').style.display = 'none';
        document.getElementById('cashout-btn').style.display = 'block';
        
        tickInterval = setInterval(() => {
            multiplier += 0.01;
            document.getElementById('multiplier').innerText = multiplier.toFixed(2) + "x";
            
            // Move plane
            const p = document.getElementById('plane');
            p.style.transform = `translate(${multiplier * 20}px, -${multiplier * 10}px)`;

            if(multiplier >= currentCrashPoint) {
                crashGame(bet);
            }
        }, 100);
    }

    function cashOut() {
        if(!gameActive) return;
        clearInterval(tickInterval);
        
        const bet = parseFloat(document.getElementById('bet-amount').value);
        const win = bet * multiplier;
        user.balance += win;
        user.history.unshift({m: multiplier.toFixed(2), win: true});
        
        alert(`Won: KES ${win.toFixed(2)}`);
        resetUI();
    }

    function crashGame(bet) {
        clearInterval(tickInterval);
        gameActive = false;
        user.history.unshift({m: multiplier.toFixed(2), win: false});
        
        document.getElementById('multiplier').style.color = 'var(--danger)';
        document.getElementById('multiplier').innerText = "CRASHED";
        
        setTimeout(() => {
            resetUI();
        }, 2000);
    }

    function resetUI() {
        gameActive = false;
        updateUser();
        document.getElementById('bet-btn').style.display = 'block';
        document.getElementById('cashout-btn').style.display = 'none';
        document.getElementById('multiplier').style.color = 'var(--primary)';
        document.getElementById('multiplier').innerText = "1.00x";
        document.getElementById('plane').style.transform = `translate(0,0)`;
    }

    function updateUser() {
        localStorage.setItem(user.phone, JSON.stringify(user));
        document.getElementById('balance').innerText = user.balance.toFixed(2);
        
        const histDiv = document.getElementById('history');
        histDiv.innerHTML = user.history.slice(0, 10).map(h => 
            `<span class="history-tag" style="color:${h.win ? '#22c55e' : '#ef4444'}">${h.m}x</span>`
        ).join('');
    }

    function logout() { location.reload(); }
</script>
</body>
</html>
