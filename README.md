<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>TECLADO | Entry Gate Scanner</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <script src="https://unpkg.com/html5-qrcode"></script>

  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #0f0c29;
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 10vh;
      padding: 100px;
    }

    .box {
      width: 100%;
      max-width: 50px;
      padding: 20px;
      background: rgba(0,0,0,0.7);
      border-radius: 16px;
      box-shadow: 0 0 20px rgba(255,0,0,0.4);
      text-align: center;
    }

    h1 {
      color: #ff2e2e;
      margin-top: 10px;
    }

    #reader {
      width: 100%;
      margin-top: 15px;
      border-radius: 12px;
      overflow: hidden;
    }

    .result {
      margin-top: 15px;
      padding: 12px;
      border-radius: 10px;
      font-size: 15px;
      display: none;
    }

    .success {
      background: #00ff99;
      color: #000;
    }

    .error {
      background: #ff3b3b;
      color: #fff;
    }
    .header {
  position: fixed;
  top: 20px;
  width: 100%;
  text-align: center;
  z-index: 10;
}

.header h1 {
  margin: 0;
  font-size: 24px;
  color: #ff2e2e;
}

.header p {
  margin: 6px 0 0;
  font-size: 14px;
  opacity: 0.85;
}

  </style>
</head>
<body>

  <div class="header">
    <h1>🎟️ TECLADO ENTRY SCAN</h1>
    <p>Scan Participant QR Code</p>
  </div>
  
  
  <div id="reader"></div>
  <div id="result"></div>
  
  <script>
  const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbyXaDmOdRSdIbTueS6XZeZmYvaq2lTSA-GrSoCAHHzDTUprodwhjGS-YqwIu1YW05hxWQ/exec";
  
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
