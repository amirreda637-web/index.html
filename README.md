<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <title>QR Attendance</title>

  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>

  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
      background: #f4f6f8;
    }

    h1 {
      margin-bottom: 5px;
    }

    #reader {
      max-width: 400px;
      margin: 20px auto;
      background: white;
      padding: 10px;
      border-radius: 15px;
    }

    #result {
      max-width: 400px;
      margin: 20px auto;
      padding: 15px;
      background: white;
      border-radius: 10px;
      font-size: 18px;
    }

    .success {
      color: green;
      font-weight: bold;
    }

    .error {
      color: red;
      font-weight: bold;
    }
  </style>
</head>

<body>

  <h1>QR Attendance</h1>

  <p>Scan QR Code</p>

  <div id="reader"></div>

  <div id="result">
    📷 Waiting for scan...
  </div>


<script>

const SCRIPT_URL =
"https://script.google.com/macros/s/AKfycbx1NXEUGfm459tBjtnA8wU8ie70-J14jaubbZsa4-xzlL8EmbDCxhFB8arYNvJZHSGZtQ/exec";


// ==========================================
// IMPORTANT:
// Prevent multiple scans at the same time
// ==========================================

let isProcessing = false;
let lastScanned = "";


function onScanSuccess(decodedText) {

  // If a scan is already being processed
  // ignore every new scan
  if (isProcessing) {
    return;
  }


  // Ignore the exact same QR immediately
  if (decodedText === lastScanned) {
    return;
  }


  // LOCK scanner
  isProcessing = true;
  lastScanned = decodedText;


  document.getElementById("result").innerHTML =
    "⏳ Processing...";


  fetch(
    SCRIPT_URL +
    "?id=" +
    encodeURIComponent(decodedText)
  )

  .then(response => response.json())

  .then(result => {

    if (result.success) {

      document.getElementById("result").innerHTML =
        '<div class="success">' +
        '✅ Attendance Recorded<br><br>' +
        'Name: ' + result.name + '<br>' +
        'Date: ' + result.date + '<br>' +
        'Time: ' + result.time +
        '</div>';

    }

    else {

      document.getElementById("result").innerHTML =
        '<div class="error">' +
        '❌ ' + result.message +
        '</div>';

    }


    // ==========================================
    // Allow scanning another person
    // ==========================================

    setTimeout(function() {

      isProcessing = false;
      lastScanned = "";

      document.getElementById("result").innerHTML =
        "📷 Ready for next scan";

    }, 2000);

  })


  .catch(error => {

    document.getElementById("result").innerHTML =
      '<div class="error">' +
      '❌ Connection Error' +
      '</div>';


    // Unlock if something went wrong
    isProcessing = false;
    lastScanned = "";

  });

}


function onScanFailure(error) {
  // Ignore normal scanning failures
}


// ==========================================
// START QR SCANNER
// ==========================================

const scanner = new Html5QrcodeScanner(
  "reader",
  {
    fps: 10,

    qrbox: {
      width: 250,
      height: 250
    }
  },

  false
);


scanner.render(
  onScanSuccess,
  onScanFailure
);

</script>

</body>
</html>
