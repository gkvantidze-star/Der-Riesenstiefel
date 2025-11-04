# Der-Riesenstiefel
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>BootRun</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
  <header>
    <h1>👢 BootRun</h1>
    <nav>
      <a href="#upload">📸 Upload</a>
      <a href="#distance">📍 Entfernung</a>
      <a href="#game">🎮 Spiel</a>
    </nav>
  </header>

  <section id="upload">
    <h2>Bilder hochladen</h2>
    <input type="file" id="imageUpload" accept="image/*" multiple>
    <div id="gallery"></div>
  </section>

  <section id="distance">
    <h2>Entfernung anzeigen</h2>
    <button id="getDistance">Entfernung berechnen</button>
    <p id="distanceResult"></p>
  </section>

  <section id="game">
    <h2>🎮 Das Stiefel-Spiel</h2>
    <canvas id="bootGame" width="480" height="270"></canvas>
    <button id="restartGame">🔄 Neustart</button>
  </section>

  <footer>
    <p>© 2025 BootRun — by Ronny & Co.</p>
  </footer>

  <script src="script.js"></script>
  <script src="game.js"></script>
</body>
</html>

body {
  margin: 0;
  font-family: 'Poppins', sans-serif;
  background: linear-gradient(180deg, #0a0f1c, #14233b);
  color: white;
  text-align: center;
}

header {
  background: rgba(0, 0, 0, 0.5);
  padding: 1rem;
  position: sticky;
  top: 0;
  backdrop-filter: blur(8px);
}

nav a {
  color: #4fd1c5;
  margin: 0 1rem;
  text-decoration: none;
  font-weight: bold;
}

nav a:hover {
  color: white;
}

section {
  padding: 2rem;
  max-width: 800px;
  margin: auto;
}

#gallery img {
  width: 150px;
  margin: 10px;
  border-radius: 10px;
  box-shadow: 0 0 10px #4fd1c5;
}

button {
  background: #4fd1c5;
  color: #041218;
  border: none;
  padding: 0.6rem 1rem;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
}

button:hover {
  background: #3bc7b7;
}

canvas {
  background: #041218;
  border: 2px solid #4fd1c5;
  border-radius: 10px;
  margin-top: 1rem;
}

// 📸 Bild-Upload
document.getElementById('imageUpload').addEventListener('change', function (event) {
  const gallery = document.getElementById('gallery');
  gallery.innerHTML = '';
  for (const file of event.target.files) {
    const img = document.createElement('img');
    img.src = URL.createObjectURL(file);
    gallery.appendChild(img);
  }
});

// 📍 Entfernung berechnen (z. B. zur Schule)
document.getElementById('getDistance').addEventListener('click', () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(pos => {
      const userLat = pos.coords.latitude;
      const userLon = pos.coords.longitude;

      // Beispielpunkt: Schulkoordinaten (kannst du ändern)
      const schoolLat = 52.5200;
      const schoolLon = 13.4050;

      const R = 6371;
      const dLat = (schoolLat - userLat) * Math.PI / 180;
      const dLon = (schoolLon - userLon) * Math.PI / 180;
      const a = Math.sin(dLat/2)**2 + Math.cos(userLat*Math.PI/180)*Math.cos(schoolLat*Math.PI/180)*Math.sin(dLon/2)**2;
      const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
      const distance = R * c;

      document.getElementById('distanceResult').textContent = `Du bist etwa ${distance.toFixed(2)} km vom Treffpunkt entfernt.`;
    });
  } else {
    alert("Standortbestimmung wird von deinem Browser nicht unterstützt.");
  }
});

const canvas = document.getElementById('bootGame');
const ctx = canvas.getContext('2d');

let bootY = 200;
let velocity = 0;
let obstacles = [];
let gameOver = false;

function resetGame() {
  bootY = 200;
  velocity = 0;
  obstacles = [];
  gameOver = false;
}
document.getElementById('restartGame').addEventListener('click', resetGame);

function jump() {
  if (bootY >= 200) velocity = -10;
}
window.addEventListener('keydown', jump);

function loop() {
  if (gameOver) return;

  ctx.fillStyle = "#041218";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Boot
  ctx.fillStyle = "#4fd1c5";
  ctx.fillRect(50, bootY, 30, 30);

  // Bewegung
  velocity += 0.5;
  bootY += velocity;
  if (bootY > 200) bootY = 200;

  // Hindernisse
  if (Math.random() < 0.02) obstacles.push({ x: canvas.width, y: 210, w: 20, h: 20 });
  obstacles.forEach(o => {
    o.x -= 5;
    ctx.fillStyle = "#ff6b6b";
    ctx.fillRect(o.x, o.y, o.w, o.h);
    if (o.x < 80 && o.x + o.w > 50 && bootY + 30 > o.y) gameOver = true;
  });

  obstacles = obstacles.filter(o => o.x + o.w > 0);

  if (gameOver) {
    ctx.fillStyle = "white";
    ctx.font = "20px Poppins";
    ctx.fillText("Game Over! 😅", 170, 140);
  } else {
    requestAnimationFrame(loop);
  }
}
loop();
