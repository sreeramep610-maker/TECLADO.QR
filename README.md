<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>TECLADO Entry Scanner</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&display=swap" rel="stylesheet">
  <style>
    body { 
      font-family: 'Orbitron', sans-serif; 
      background: linear-gradient(135deg, #0f0c29, #302b63, #24243e); 
      color: #fff; 
      padding: 10px; 
      margin: 0;
    }
    .container { 
      max-width: 100%; 
      margin: 0 auto; 
      text-align: center; 
      background: rgba(10,10,20,0.95); 
      border-radius: 20px; 
      padding: 20px; 
      border: 2px solid #ff4444; 
    }
    h1 { margin: 0 0 10px 0; font-size: 2.2em; }
    .status { 
      margin: 25px 0; 
      padding: 25px; 
      border-radius: 20px; 
      font-size: 20px; 
      min-height: 80px; 
      background: rgba(255,255,255,0.1); 
      transition: all 0.3s ease;
    }
    .success { background: rgba(0,255,153,0.3) !important; border: 3px solid #00ff99 !important; }
    .error { background: rgba(255,68,68,0.3) !important; border: 3px solid #ff4444 !important; }
    button { 
      padding: 18px; 
      border: none; 
      border-radius: 15px; 
      font-weight: bold; 
      font-size: 18px; 
      cursor: pointer; 
      margin: 5px;
      min-width: 140px;
    }
    .hidden { display: none !important; }
    .btn-start { background: linear-gradient(45deg, #00ff99, #00cc88); color: #000; }
    .btn-stop { background: linear-gradient(45deg, #ff4444, #cc3333); color: #fff; }
    .btn-file { background: linear-gradient(45deg, #ffd700, #ffaa00); color: #000; }
    .btn-config { background: linear-gradient(45deg, #9b59b6, #8e44ad); color: #fff; font-size: 16px; padding: 12px; }
    
    .permission-guide { 
      background: linear-gradient(90deg, #ff6b6b, #ffd93d); 
      border-radius: 15px; 
      padding: 20px; 
      margin: 15px 0; 
      font-size: 16px; 
      text-align: left; 
    }
    .steps { display: flex; flex-direction: column; gap: 10px; }
    .step { display: flex; align-items: flex-start; gap: 10px; }
    .step-number { 
      background: rgba(0,0,0,0.7); 
      color: #fff; 
      width: 30px; 
      height: 30px; 
      border-radius: 50%; 
      display: flex; 
      align-items: center; 
      justify-content: center; 
      font-weight: bold; 
      flex-shrink: 0; 
    }
    
    #qr-reader { width: 100%; max-width: 500px; margin: 20px auto; }
    .sheet-status { font-size: 14px; opacity: 0.8; margin-top: 10px; }
  </style>
</head>
<body>
  <div class="container">
    <h1>🎟 ENTRY SCANNER</h1>
    <p>TECLADO 2026 • CCSIT PALAKKAD</p>
    
    <!-- CONFIGURE GOOGLE SHEETS -->
    <div style="background: rgba(0,0,0,0.5); padding: 15px; border-radius: 15px; margin: 15px 0;">
      <button id="configSheet" class="btn-config">⚙️ CONFIGURE SHEETS</button>
      <div id="sheetConfig" class="hidden" style="margin-top: 15px;">
        <input id="sheetId" placeholder="Google Sheet ID" style="width: 100%; padding: 12px; margin: 10px 0; border-radius: 10px; border: 1px solid #444; background: #111; color: #fff;">
        <input id="sheetName" placeholder="Sheet Name (e.g. Entries)" style="width: 100%; padding: 12px; margin: 10px 0; border-radius: 10px; border: 1px solid #444; background: #111; color: #fff;">
        <button id="saveConfig" style="background: #00ff99; color: #000; padding: 12px 24px;">💾 SAVE</button>
      </div>
      <div id="sheetStatus" class="sheet-status"></div>
    </div>

    <!-- PERMISSION GUIDE -->
    <div id="permissionGuide" class="permission-guide">
      <h3>📱 CAMERA NOT WORKING?</h3>
      <div class="steps">
        <div class="step">
          <div class="step-number">1</div>
          <div><strong>Tap site info</strong> (🔒 next to URL) → Camera → <strong>Allow</strong></div>
        </div>
        <div class="step">
          <div class="step-number">2</div>
          <div><strong>Phone Settings</strong> → Apps → Chrome → Permissions → <strong>Camera ON</strong></div>
        </div>
        <div class="step">
          <div class="step-number">3</div>
          <div><strong>Try incognito</strong> or restart browser</div>
        </div>
      </div>
      <button onclick="hidePermissionGuide()" style="margin-top: 15px; padding: 10px 20px; background: #00ff99; color: #000; border: none; border-radius: 10px; font-weight: bold;">✅ I FIXED IT</button>
    </div>

    <div id="qr-reader"></div>
    
    <div style="display: flex; gap: 15px; justify-content: center; flex-wrap: wrap;">
      <button id="startScan" class="btn-start">🚀 START CAMERA</button>
      <button id="stopScan" class="hidden btn-stop">⏹ STOP</button>
      <button id="fileScan" class="btn-file">📸 PHOTO SCAN</button>
    </div>
    
    <div id="result" class="status">👆 Tap START CAMERA (allow permission!)</div>
  </div>

  <script>
    const html5QrCode = new Html5Qrcode("qr-reader");
    let isScanning = false;
    
    // GOOGLE SHEETS CONFIG
    let sheetConfig = JSON.parse(localStorage.getItem('sheetConfig') || '{}');
    
    // UPDATE SHEET STATUS
    function updateSheetStatus() {
      const statusEl = document.getElementById('sheetStatus');
      if (sheetConfig.scriptUrl) {
        statusEl.innerHTML = '✅ Sheets Connected';
        statusEl.style.color = '#00ff99';
      } else {
        statusEl.innerHTML = '⚠️ Configure Google Sheets first';
        statusEl.style.color = '#ffaa00';
      }
    }
    
    // SAVE SHEET CONFIG
    document.getElementById('saveConfig').onclick = () => {
      sheetConfig.sheetId = document.getElementById('sheetId').value;
      sheetConfig.sheetName = document.getElementById('sheetName').value || 'Entries';
      
      // Generate Apps Script Web App URL (you need to deploy this)
    //   sheetConfig.scriptUrl = `https://script.google.com/macros/s/AKfycbwyiwJUCC3fGnqgygtSRrQycFreEYLv3FTay4cTZF_m74Y-koQoQR21nk0AGWyuBkW8/exe?sheetId=${sheetConfig.sheetId}&sheetName=${sheetConfig.sheetName}`;
      
      localStorage.setItem('sheetConfig', JSON.stringify(sheetConfig));
      document.getElementById('sheetConfig').classList.add('hidden');
      updateSheetStatus();
      alert('✅ Configuration saved! Deploy the Google Apps Script with the sheet ID.');
    };
    
    // LOG TO GOOGLE SHEETS
    async function logToSheets(qrData) {
      if (!sheetConfig.scriptUrl) {
        return { success: false, message: 'Configure Sheets first!' };
      }
      
      const now = new Date().toLocaleString('en-IN', { timeZone: 'Asia/Kolkata' });
      const entryData = {
        timestamp: now,
        qr_code: qrData,
        scanner: 'TECLADO 2026',
        location: 'CCSIT PALAKKAD'
      };
      
      try {
        const response = await fetch(sheetConfig.scriptUrl, {
          method: 'POST',
          mode: 'no-cors', // Important for Google Apps Script
          body: JSON.stringify(entryData)
        });
        
        return { success: true, message: 'Entry logged!' };
      } catch (error) {
        return { success: false, message: 'Network error - check script URL' };
      }
    }
    
    // START SCANNING
    async function startScanning() {
      try {
        if (location.protocol !== 'https:' && location.hostname !== 'localhost') {
          alert('⚠️ Camera needs HTTPS or localhost. Use GitHub Pages/Netlify!');
          return;
        }
        
        document.getElementById('result').innerHTML = '🔄 Requesting camera permission...';
        
        const config = { 
          fps: 10, 
          qrbox: { width: 250, height: 250 },
          videoConstraints: {
            facingMode: 'environment',
            width: { ideal: 1280 },
            height: { ideal: 720 }
          }
        };
        
        await html5QrCode.start(
          config.videoConstraints,
          config,
          async (decodedText) => {
            console.log('✅ SCANNED:', decodedText);
            
            // LOG TO SHEETS
            const logResult = await logToSheets(decodedText);
            
            document.getElementById('result').innerHTML = 
              `✅ QR: ${decodedText.substring(0, 30)}...<br>` +
              `📅 ${new Date().toLocaleTimeString('en-IN')}<br>` +
              `<strong>${logResult.message}</strong>`;
            document.getElementById('result').className = 'status success';
            
            // AUTO RESTART AFTER 2s
            html5QrCode.stop().then(() => {
              setTimeout(startScanning, 2000);
            });
          },
          () => {}
        );
        
        isScanning = true;
        document.getElementById('startScan').classList.add('hidden');
        document.getElementById('stopScan').classList.remove('hidden');
        
      } catch (err) {
        console.error('Camera error:', err);
        document.getElementById('result').innerHTML = `❌ Camera denied<br><small>🔒 Check permission guide above</small>`;
        document.getElementById('result').className = 'status error';
      }
    }
    
    // UI EVENT LISTENERS
    document.getElementById('startScan').onclick = startScanning;
    document.getElementById('configSheet').onclick = () => {
      document.getElementById('sheetConfig').classList.toggle('hidden');
      document.getElementById('sheetId').value = sheetConfig.sheetId || '';
      document.getElementById('sheetName').value = sheetConfig.sheetName || 'Entries';
    };
    
    function hidePermissionGuide() {
      document.getElementById('permissionGuide').style.display = 'none';
    }
    
    // INIT
    updateSheetStatus();
  </script>
</body>
</html>
