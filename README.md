<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Auto Clicker with Area Selection</title>
<style>
body { font-family: Arial, sans-serif; background:#f0f0f0; margin:0; padding:20px; }
.card { background:white; border-radius:10px; padding:20px; max-width:600px; margin:auto; box-shadow:0 5px 15px rgba(0,0,0,0.1); }
h1 { margin:0 0 15px; font-size:1.5rem; }
.row { margin-bottom:12px; }
label { display:inline-block; width:150px; }
input[type="number"], input[type="range"] { width:100px; }
#selectionArea { position:absolute; border:2px dashed red; pointer-events:none; display:none; }
</style>
</head>
<body>

<div class="card">
  <h1>Auto Clicker (영역 선택 가능)</h1>
  <div class="row">
    <button id="startBtn">자동 클릭 시작</button>
    <button id="stopBtn">자동 클릭 중지</button>
  </div>
  <div class="row">
    <label for="delay">시작 지연 (초):</label>
    <input id="delay" type="number" min="0" value="0">
  </div>
  <div class="row">
    <label for="repeat">클릭 횟수:</label>
    <input id="repeat" type="number" min="1" value="10">
  </div>
  <div class="row">
    <label for="speed">속도 (ms / 클릭):</label>
    <input id="speed" type="number" min="10" value="200">
  </div>
  <div class="row">
    <p>영역 선택: 마우스로 드래그 → Enter로 고정/해제</p>
  </div>
</div>

<div id="selectionArea"></div>

<script>
let isDragging = false;
let startX = 0, startY = 0, endX = 0, endY = 0;
let areaFixed = false;
let selectedArea = null;
let autoInterval = null;
let clicksDone = 0;

const selectionArea = document.getElementById('selectionArea');
const startBtn = document.getElementById('startBtn');
const stopBtn = document.getElementById('stopBtn');

document.addEventListener('mousedown', (e) => {
  if (areaFixed) return;
  isDragging = true;
  startX = e.clientX;
  startY = e.clientY;
  selectionArea.style.left = startX + 'px';
  selectionArea.style.top = startY + 'px';
  selectionArea.style.width = '0px';
  selectionArea.style.height = '0px';
  selectionArea.style.display = 'block';
});

document.addEventListener('mousemove', (e) => {
  if (!isDragging) return;
  endX = e.clientX;
  endY = e.clientY;
  const x = Math.min(startX, endX);
  const y = Math.min(startY, endY);
  const w = Math.abs(startX - endX);
  const h = Math.abs(startY - endY);
  selectionArea.style.left = x + 'px';
  selectionArea.style.top = y + 'px';
  selectionArea.style.width = w + 'px';
  selectionArea.style.height = h + 'px';
});

document.addEventListener('mouseup', (e) => {
  if (!isDragging) return;
  isDragging = false;
  selectedArea = {
    x: Math.min(startX, endX),
    y: Math.min(startY, endY),
    width: Math.abs(startX - endX),
    height: Math.abs(startY - endY)
  };
});

window.addEventListener('keydown', (e) => {
  if (e.key === 'Enter') {
    if (!selectedArea) {
      alert('먼저 영역을 선택하세요.');
      return;
    }
    areaFixed = !areaFixed;
    if (areaFixed) {
      alert('영역이 고정되었습니다.');
      selectionArea.style.border = '2px solid green';
    } else {
      alert('영역 고정이 해제되었습니다.');
      selectionArea.style.border = '2px dashed red';
    }
  }
});

function clickAtRandomPosition(area) {
  const x = area.x + Math.random() * area.width;
  const y = area.y + Math.random() * area.height;
  const evt = new MouseEvent('click', {
    view: window,
    bubbles: true,
    cancelable: true,
    clientX: x,
    clientY: y
  });
  document.elementFromPoint(x, y)?.dispatchEvent(evt);
}

startBtn.addEventListener('click', () => {
  if (!selectedArea) {
    alert('영역을 선택하고 Enter로 고정하세요.');
    return;
  }
  const delay = parseInt(document.getElementById('delay').value) * 1000;
  const repeat = parseInt(document.getElementById('repeat').value);
  const speed = parseInt(document.getElementById('speed').value);

  clicksDone = 0;
  setTimeout(() => {
    autoInterval = setInterval(() => {
      if (clicksDone >= repeat) {
        clearInterval(autoInterval);
        return;
      }
      clickAtRandomPosition(selectedArea);
      clicksDone++;
    }, speed);
  }, delay);
});

stopBtn.addEventListener('click', () => {
  if (autoInterval) {
    clearInterval(autoInterval);
    autoInterval = null;
    clicksDone = 0;
  }
});
</script>
</body>
</html>
