<!DOCTYPE html>
<html lang="uz">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kalkulyator</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <div id="setup" class="overlay">
    <div class="setup-box">
      <h2>Sirli kodni o‘rnating</h2>
      <input type="password" id="setCode" placeholder="Masalan: 9630">
      <button id="saveCode">Saqlash</button>
    </div>
  </div>

  <div class="calculator">
    <input type="text" id="display" disabled>
    <div class="buttons">
      <button>7</button><button>8</button><button>9</button><button>/</button>
      <button>4</button><button>5</button><button>6</button><button>*</button>
      <button>1</button><button>2</button><button>3</button><button>-</button>
      <button>0</button><button>.</button><button id="equals">=</button><button>+</button>
      <button id="clear">C</button>
    </div>
  </div>

  <div id="recorder" class="hidden">
    <h2>🎙️ Diktofon</h2>
    <button id="recordBtn">🔴 Yozishni boshlash</button>
    <div id="recordingsList"></div>
  </div>

  <script src="script.js"></script>
</body>
</html>body {
  margin: 0;
  background-color: #000;
  color: #fff;
  font-family: Arial, sans-serif;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
}

.calculator {
  text-align: center;
  margin-top: 20px;
}

#display {
  width: 240px;
  height: 50px;
  font-size: 24px;
  text-align: right;
  background: #111;
  border: none;
  color: #fff;
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 8px;
}

button {
  width: 50px;
  height: 50px;
  font-size: 20px;
  margin: 5px;
  border-radius: 50%;
  background-color: #222;
  color: #fff;
  border: none;
}

button:hover {
  background-color: #333;
}

#recorder {
  margin-top: 20px;
  text-align: center;
}

.hidden {
  display: none;
}

.overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0,0,0,0.9);
  display: flex;
  align-items: center;
  justify-content: center;
}

.setup-box {
  background: #111;
  padding: 20px;
  border-radius: 10px;
  text-align: center;
}

.setup-box input {
  padding: 10px;
  font-size: 18px;
  border-radius: 5px;
  border: none;
  width: 150px;
}

.setup-box button {
  margin-top: 10px;
  padding: 10px 20px;
  border: none;
  background: #444;
  color: #fff;
  border-radius: 5px;
}const display = document.getElementById("display");
const buttons = document.querySelectorAll(".buttons button");
const equals = document.getElementById("equals");
const clear = document.getElementById("clear");
const recorder = document.getElementById("recorder");

let expression = "";
let mediaRecorder;
let audioChunks = [];

const storedCode = localStorage.getItem("secretCode");

// Sirli kod o‘rnatish
if (!storedCode) {
  document.getElementById("setup").style.display = "flex";
  document.getElementById("saveCode").onclick = () => {
    const code = document.getElementById("setCode").value.trim();
    if (code) {
      localStorage.setItem("secretCode", code);
      document.getElementById("setup").style.display = "none";
    }
  };
} else {
  document.getElementById("setup").style.display = "none";
}

// Kalkulyator tugmalari
buttons.forEach(btn => {
  btn.addEventListener("click", () => {
    if (btn.id === "equals") return;
    expression += btn.textContent;
    display.value = expression;
  });
});

equals.addEventListener("click", () => {
  const secretCode = localStorage.getItem("secretCode");
  if (expression === secretCode) {
    recorder.classList.remove("hidden");
  } else {
    try {
      display.value = eval(expression);
    } catch {
      display.value = "Xato";
    }
  }
  expression = "";
});

clear.addEventListener("click", () => {
  expression = "";
  display.value = "";
});

// 🎙️ Diktofon funksiyasi
const recordBtn = document.getElementById("recordBtn");
const recordingsList = document.getElementById("recordingsList");

let recordings = JSON.parse(localStorage.getItem("recordings") || "[]");
updateList();

recordBtn.onclick = async () => {
  if (mediaRecorder && mediaRecorder.state === "recording") {
    mediaRecorder.stop();
    recordBtn.textContent = "🔴 Yozishni boshlash";
  } else {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    mediaRecorder = new MediaRecorder(stream);
    audioChunks = [];
    mediaRecorder.ondataavailable = e => audioChunks.push(e.data);
    mediaRecorder.onstop = () => {
      const blob = new Blob(audioChunks, { type: "audio/webm" });
      const url = URL.createObjectURL(blob);
      const name = new Date().toLocaleString();
      recordings.push({ name, url });
      localStorage.setItem("recordings", JSON.stringify(recordings));
      updateList();
    };
    mediaRecorder.start();
    recordBtn.textContent = "⏹️ To‘xtatish";
  }
};

function updateList() {
  recordingsList.innerHTML = "";
  recordings.forEach((rec, i) => {
    const div = document.createElement("div");
    div.innerHTML = `
      <p>${rec.name}</p>
      <audio controls src="${rec.url}"></audio><br>
      <button onclick="deleteRecording(${i})">🗑️ O‘chirish</button>
    `;
    recordingsList.appendChild(div);
  });
}

window.deleteRecording = function (i) {
  recordings.splice(i, 1);
  localStorage.setItem("recordings", JSON.stringify(recordings));
  updateList();
};
