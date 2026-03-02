<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>はい！カウンター</title>

<style>
body {
  font-family: -apple-system, BlinkMacSystemFont, sans-serif;
  text-align: center;
  background: #f4f4f4;
  margin: 0;
  padding: 30px;
}

h1 {
  font-size: 28px;
}

#count {
  font-size: 120px;
  font-weight: bold;
  margin: 40px 0;
}

button {
  font-size: 22px;
  padding: 15px 25px;
  margin: 10px;
  border-radius: 12px;
  border: none;
  background: #007aff;
  color: white;
}

button:active {
  background: #0051aa;
}

#status {
  margin-top: 20px;
  color: gray;
}
</style>
</head>

<body>

<h1>はい！カウンター</h1>
<div id="count">0</div>

<button onclick="start()">スタート</button>
<button onclick="resetCount()">リセット</button>

<div id="status">マイク待機中</div>

<script>
let count = 0;
let lastTrigger = 0;
let audioContext;
let analyser;
let microphone;

function start() {
  navigator.mediaDevices.getUserMedia({ audio: true })
    .then(stream => {
      audioContext = new (window.AudioContext || window.webkitAudioContext)();
      analyser = audioContext.createAnalyser();
      microphone = audioContext.createMediaStreamSource(stream);
      microphone.connect(analyser);
      analyser.fftSize = 256;

      document.getElementById("status").innerText = "監視中";
      listen();
    })
    .catch(err => {
      alert("マイクの許可が必要です");
    });
}

function listen() {
  const dataArray = new Uint8Array(analyser.frequencyBinCount);

  function detect() {
    analyser.getByteFrequencyData(dataArray);

    let volume = dataArray.reduce((a, b) => a + b) / dataArray.length;
    let now = Date.now();

    if (volume > 60 && now - lastTrigger > 800) {
      count++;
      document.getElementById("count").innerText = count;
      lastTrigger = now;

      if (navigator.vibrate) {
        navigator.vibrate(50);
      }
    }

    requestAnimationFrame(detect);
  }

  detect();
}

function resetCount() {
  count = 0;
  document.getElementById("count").innerText = count;
}
</script>

</body>
</html>
