<html lang="en">
<head>
<meta charset="UTF-8">
<title>TECLADO | Entry Verification</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<script src="https://unpkg.com/html5-qrcode"></script>

<style>
body {
    font-family: 'Orbitron', sans-serif;
    background: radial-gradient(circle, #0f0c29, #302b63);
    color: #fff;
    text-align: center;
    padding: 20px;
}
h1 {
    color: #00f5ff;
}
#reader {
    width: 320px;
    margin: auto;
}
.result {
    margin-top: 20px;
    padding: 20px;
    border-radius: 15px;
    font-size: 1.2rem;
}
.valid { background: #00ff99; color: #000; }
.used { background: #ff9800; color: #000; }
.invalid { background: #ff0033; }
</style>
</head>

<body>

<h1>🎟️ TECLADO ENTRY SCAN</h1>
<p>Scan Participant QR Code</p>

<div id="reader"></div>
<div id="result"></div>

<script>
const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxsyQINALIabdlCcrxQBHxRQjckhKSXKYlI5cvnWb3VUrElDK6iMaRjYm1DpqDezXmuSQ/exec";

function onScanSuccess(decodedText) {

    html5QrcodeScanner.clear();

    const match = decodedText.match(/ID:(TECLADO-[\d\-]+)/);
    if (!match) {
        showResult("❌ Invalid QR", "invalid");
        return;
    }

    const pid = match[1];

    fetch(`${SCRIPT_URL}?pid=${pid}`)
        .then(res => res.json())
        .then(data => {

            if (data.status === "valid") {
                showResult(`✅ ENTRY ALLOWED<br><b>${data.name}</b><br>${data.event}`, "valid");
            } 
            else if (data.status === "used") {
                showResult(`⚠️ ALREADY USED<br><b>${data.name}</b>`, "used");
            } 
            else {
                showResult("❌ INVALID PASS", "invalid");
            }

            setTimeout(startScanner, 3000);
        });
}

function showResult(text, cls) {
    const box = document.getElementById("result");
    box.className = `result ${cls}`;
    box.innerHTML = text;
}

function startScanner() {
    document.getElementById("result").innerHTML = "";
    html5QrcodeScanner.render(onScanSuccess);
}

const html5QrcodeScanner = new Html5QrcodeScanner(
    "reader",
    { fps: 10, qrbox: 250 }
);

startScanner();
</script>

</body>
</html>
