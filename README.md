<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HDRP | Hostage Payout System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            background-color: #1a1c20;
            color: #ececed;
            font-family: 'Plus Jakarta Sans', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }
        .main-card {
            background-color: #22252a;
            width: 100%;
            max-width: 480px;
            border-radius: 12px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
            overflow: hidden;
            position: relative;
            border: 1px solid #2d3136;
        }
        .top-accent { height: 4px; background: #0074ff; width: 100%; }
        .fund-container {
            background: rgba(16, 185, 129, 0.05);
            border: 1px solid rgba(16, 185, 129, 0.2);
            border-radius: 10px;
            padding: 14px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .fund-container.exhausted {
            background: rgba(239, 68, 68, 0.05);
            border-color: rgba(239, 68, 68, 0.3);
        }
        .input-box {
            background: #2b2f35;
            border: 1px solid #3d424a;
            border-radius: 10px;
            padding: 12px 16px;
            width: 100%;
            color: white;
            outline: none;
            transition: border-color 0.2s;
        }
        .input-box:focus { border-color: #0074ff; }
        .label-text {
            color: #8b949e;
            font-size: 11px;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            margin-bottom: 8px;
            display: block;
        }
        .btn-confirm {
            background: #0074ff;
            color: white;
            font-weight: 800;
            text-transform: uppercase;
            padding: 16px;
            border-radius: 10px;
            width: 100%;
            letter-spacing: 1px;
            transition: background 0.2s;
        }
        .btn-confirm:hover { background: #0062d6; }
        .btn-support {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid #3d424a;
            color: #8b949e;
            font-weight: 600;
            font-size: 12px;
            text-transform: uppercase;
            padding: 10px;
            border-radius: 8px;
            width: 100%;
            text-align: center;
            transition: all 0.2s;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            text-decoration: none;
        }
        .btn-support:hover {
            background: rgba(255, 255, 255, 0.06);
            color: white;
            border-color: #4d535b;
        }
        .flag-icon {
            width: 22px;
            height: 14px;
            border-radius: 2px;
            cursor: pointer;
            opacity: 0.6;
            transition: opacity 0.2s;
        }
        .flag-icon.active { opacity: 1; }
        #confirmModal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.85);
            backdrop-filter: blur(4px);
            z-index: 50;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        .modal-card {
            background: #22252a;
            border: 1px solid #3d424a;
            border-radius: 16px;
            max-width: 400px;
            width: 100%;
            padding: 24px;
            text-align: center;
        }
    </style>
</head>
<body>

<div id="confirmModal">
    <div class="modal-card">
        <div class="w-12 h-12 bg-red-500/10 text-red-500 rounded-full flex items-center justify-center mx-auto mb-4">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
            </svg>
        </div>
        <h3 id="modal-title" class="text-xl font-bold text-white mb-3">Limit Überschreitung</h3>
        <p id="modal-text" class="text-gray-400 text-sm mb-6 leading-relaxed">
            Achtung: Deine Zahlung überschreitet das festgelegte Limit. Du kannst die Zahlung fortsetzen, allerdings wird nur der Betrag bis zum Limit verarbeitet.
        </p>
        <div class="space-y-3">
            <button id="modal-confirm" class="w-full py-3 bg-red-600 hover:bg-red-700 text-white font-bold rounded-lg transition uppercase text-xs tracking-widest">Bestätigen</button>
            <button id="modal-cancel" class="w-full py-3 bg-transparent hover:bg-white/5 text-gray-500 hover:text-white font-bold rounded-lg transition uppercase text-xs tracking-widest">Abbrechen</button>
        </div>
    </div>
</div>

<div class="main-card">
    <div class="top-accent"></div>
    
    <div class="p-10">
        <div class="text-center mb-8">
            <h1 class="text-4xl font-bold text-white mb-2">Hostage Payout</h1>
            <p id="ui-subtitle" class="text-gray-400 text-sm font-medium">Offizielle Dokumentation für Auszahlungen an Geiselnehmer</p>
        </div>

        <div id="budgetBox" class="fund-container mb-10">
            <span id="ui-fund-label" class="text-[11px] font-bold text-emerald-500 uppercase tracking-wider">Weekly Fund Balance</span>
            <span id="budgetDisplay" class="text-emerald-400 font-bold text-lg">Wird geladen...</span>
        </div>

        <div id="exhaustedWarning" class="hidden bg-red-500/10 border border-red-500/30 rounded-lg p-3 mb-6 flex items-center gap-3">
            <div class="text-red-500">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7 4a1 1 0 11-2 0 1 1 0 012 0zm-1-9a1 1 0 00-1 1v4a1 1 0 102 0V6a1 1 0 00-1-1z" clip-rule="evenodd" />
                </svg>
            </div>
            <p class="text-[11px] text-red-400 font-medium">Budget aufgebraucht. Weitere Auszahlungen müssen privat bestätigt werden.</p>
        </div>

        <form id="payoutForm" class="space-y-6">
            <div>
                <label id="ui-label-officer" class="label-text">Officer in Charge (ID)</label>
                <input type="text" id="officerId" placeholder="z.B. 83742" required class="input-box">
            </div>

            <div>
                <label id="ui-label-recipient" class="label-text">Recipient (ID)</label>
                <input type="text" id="receiverId" placeholder="z.B. 99283" required class="input-box">
            </div>

            <div class="grid grid-cols-2 gap-5">
                <div>
                    <label id="ui-label-hostages" class="label-text">Hostages</label>
                    <input type="number" id="hostageCount" placeholder="1-2" min="1" max="2" required class="input-box">
                </div>
                <div>
                    <label id="ui-label-amount" class="label-text">Amount (€)</label>
                    <input type="number" id="amount" placeholder="0.00" step="0.01" required class="input-box">
                </div>
            </div>

            <div>
                <label id="ui-label-evidence" class="label-text">Evidence URL</label>
                <input type="url" id="evidence" placeholder="https://discord.com/channels..." required class="input-box">
            </div>

            <div class="pt-4 space-y-3">
                <button type="submit" id="ui-btn-text" class="btn-confirm">Auszahlung Bestätigen</button>
                <a href="https://discord.com/channels/1170842061319188490/1170842061910593665" target="_blank" class="btn-support" id="ui-support-text">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
                    </svg>
                    Support Kontakt
                </a>
            </div>

            <div class="flex justify-between items-center pt-2">
                <div class="flex gap-2">
                    <img src="https://flagcdn.com/w40/gb.png" class="flag-icon" id="lang-en" onclick="setLanguage('en')">
                    <img src="https://flagcdn.com/w40/de.png" class="flag-icon active" id="lang-de" onclick="setLanguage('de')">
                </div>
                <span class="text-[10px] font-bold text-gray-500 uppercase">HPD Financial Hub</span>
            </div>
        </form>
    </div>
</div>

<script>
    // Lokale Konfiguration (Kein Firebase mehr nötig)
    const WEBHOOK_URL = "https://discord.com/api/webhooks/1353773517723930784/lkn3JdaWW3QYd6UIvOaJVWT6hzVgcc7kTEQWJVYNCquOTPhmNOHLEQgvWFGIyiq9HbPw";
    const MAX_BUDGET = 50000;
    const STORAGE_KEY = "hpd_hostage_budget";

    let currentBudget = MAX_BUDGET;
    let currentLanguage = 'de';
    let pendingData = null;

    const translations = {
        en: {
            subtitle: "Official documentation for hostage payouts",
            fund: "Weekly Fund Balance",
            officer: "Officer in Charge (ID)",
            recipient: "Recipient (ID)",
            hostages: "Hostages",
            amount: "Amount (€)",
            evidence: "Evidence URL",
            btn: "Confirm Payout",
            support: "Contact Support",
            modalTitle: "Limit Exceeded",
            modalText: "Attention: Your payment exceeds the set limit. You can continue, but only the amount up to the limit will be processed by the fund.",
            modalConfirm: "Confirm",
            modalCancel: "Cancel",
            budgetWarning: "Budget exhausted. Further payouts must be confirmed as personal expenses.",
            rangeError: "Hostage count must be between 1 and 2."
        },
        de: {
            subtitle: "Offizielle Dokumentation für Auszahlungen an Geiselnehmer",
            fund: "Wöchentliches Budget",
            officer: "Bearbeitender Beamter (ID)",
            recipient: "Empfänger (ID)",
            hostages: "Anzahl Geiseln",
            amount: "Betrag (€)",
            evidence: "Beweismittel URL",
            btn: "Auszahlung Bestätigen",
            support: "Support Kontakt",
            modalTitle: "Limit Überschreitung",
            modalText: "Achtung: Deine Zahlung überschreitet das festgelegte Limit. Du kannst die Zahlung fortsetzen, allerdings wird nur der Betrag bis zum Limit verarbeitet.",
            modalConfirm: "Bestätigen",
            modalCancel: "Abbrechen",
            budgetWarning: "Budget aufgebraucht. Weitere Auszahlungen müssen privat bestätigt werden.",
            rangeError: "Die Anzahl der Geiseln muss zwischen 1 und 2 liegen."
        }
    };

    // Budget lokal laden
    function initBudget() {
        const stored = localStorage.getItem(STORAGE_KEY);
        if (stored !== null) {
            currentBudget = parseFloat(stored);
        } else {
            localStorage.setItem(STORAGE_KEY, MAX_BUDGET.toString());
        }
        updateBudgetUI();
    }

    window.setLanguage = function(lang) {
        currentLanguage = lang;
        const t = translations[lang];
        document.getElementById('ui-subtitle').innerText = t.subtitle;
        document.getElementById('ui-fund-label').innerText = t.fund;
        document.getElementById('ui-label-officer').innerText = t.officer;
        document.getElementById('ui-label-recipient').innerText = t.recipient;
        document.getElementById('ui-label-hostages').innerText = t.hostages;
        document.getElementById('ui-label-amount').innerText = t.amount;
        document.getElementById('ui-label-evidence').innerText = t.evidence;
        document.getElementById('ui-btn-text').innerText = t.btn;
        document.getElementById('ui-support-text').innerHTML = `
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
            </svg>
            ${t.support}
        `;
        document.getElementById('modal-title').innerText = t.modalTitle;
        document.getElementById('modal-text').innerText = t.modalText;
        document.getElementById('modal-confirm').innerText = t.modalConfirm;
        document.getElementById('modal-cancel').innerText = t.modalCancel;
        document.getElementById('exhaustedWarning').querySelector('p').innerText = t.budgetWarning;

        document.getElementById('lang-en').classList.toggle('active', lang==='en');
        document.getElementById('lang-de').classList.toggle('active', lang==='de');
    }

    function updateBudgetUI() {
        const display = document.getElementById('budgetDisplay');
        const box = document.getElementById('budgetBox');
        const label = document.getElementById('ui-fund-label');
        const warning = document.getElementById('exhaustedWarning');

        display.innerText = currentBudget.toLocaleString('de-DE', { minimumFractionDigits:2 }) + "€";

        if(currentBudget <= 0) {
            box.classList.add('exhausted');
            display.classList.replace('text-emerald-400', 'text-red-500');
            label.classList.replace('text-emerald-500', 'text-red-500');
            warning.classList.remove('hidden');
        } else {
            box.classList.remove('exhausted');
            display.classList.replace('text-red-500', 'text-emerald-400');
            label.classList.replace('text-red-500', 'text-emerald-500');
            warning.classList.add('hidden');
        }
    }

    function handlePayout(data) {
        const fundPortion = Math.min(data.amount, currentBudget);
        const personalPortion = data.amount - fundPortion;
        const newBudget = Math.max(0, currentBudget - fundPortion);

        // Lokal speichern
        currentBudget = newBudget;
        localStorage.setItem(STORAGE_KEY, currentBudget.toString());
        
        updateBudgetUI();
        sendWebhook(data, fundPortion, personalPortion, currentBudget);
        document.getElementById('payoutForm').reset();
    }

    async function sendWebhook(data, fundPortion, personalPortion, budgetAfter) {
        const payload = { embeds:[{
            title: personalPortion > 0 ? "⚠️ Hostage Payout (Partially Private)" : "✅ Hostage Payout Log",
            color: personalPortion > 0 ? 15548997 : 2303786,
            fields:[
                {name:"Officer ID", value:data.officer, inline:true},
                {name:"Recipient ID", value:data.receiver, inline:true},
                {name:"Hostage Count", value:`${data.count}`, inline:true},
                {name:"Total Amount", value:`**${data.amount.toLocaleString('de-DE')} €**`, inline:false},
                {name:"Deducted from Budget", value:`${fundPortion.toLocaleString('de-DE')} €`, inline:true},
                {name:"Personal Share", value:`${personalPortion.toLocaleString('de-DE')} €`, inline:true},
                {name:"Evidence", value: `[View Evidence](${data.evidence})`, inline:false},
                {name:"Remaining Budget", value: `\`${budgetAfter.toLocaleString('de-DE')} €\``, inline:false}
            ],
            footer: { text: "HPD Financial Hub Service (Local Mode)" },
            timestamp: new Date().toISOString()
        }]};

        try { 
            await fetch(WEBHOOK_URL, {method:"POST", headers:{"Content-Type":"application/json"}, body:JSON.stringify(payload)}); 
        } catch(e) { console.warn("Webhook-Error"); }
    }

    document.getElementById('payoutForm').addEventListener('submit', e => {
        e.preventDefault();
        const amount = parseFloat(document.getElementById('amount').value);
        const hostages = parseInt(document.getElementById('hostageCount').value);

        if(hostages < 1 || hostages > 2) { 
            alert(translations[currentLanguage].rangeError); 
            return; 
        }

        const data = {
            officer: document.getElementById('officerId').value.trim(),
            receiver: document.getElementById('receiverId').value.trim(),
            count: hostages,
            amount: amount,
            evidence: document.getElementById('evidence').value
        };

        if(amount > currentBudget || currentBudget <= 0) {
            pendingData = data;
            document.getElementById('confirmModal').style.display = 'flex';
        } else {
            handlePayout(data);
        }
    });

    document.getElementById('modal-cancel').onclick = () => { document.getElementById('confirmModal').style.display = 'none'; };
    document.getElementById('modal-confirm').onclick = () => {
        if(pendingData) handlePayout(pendingData);
        document.getElementById('confirmModal').style.display = 'none';
    };

    // Init
    initBudget();
    setLanguage('de');
</script>

</body>
</html>
