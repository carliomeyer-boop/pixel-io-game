# pixel-io-game
<!DOCTYPE html>
<html>
<head>
  <title>.io Game</title>
  <style>
    body { margin:0; background:#111; overflow:hidden; }
    canvas { display:block; margin:auto; background:#000; }
    #menu {
      position:absolute;
      width:100%;
      height:100%;
      display:flex;
      flex-direction:column;
      justify-content:center;
      align-items:center;
      background:#111;
      color:white;
    }
  </style>
</head>
<body>

<div id="menu">
  <input id="name" placeholder="Name">
  <input id="room" placeholder="Room">
  <button onclick="start()">Play</button>
</div>

<canvas id="c" width="500" height="500"></canvas>

<script src="/socket.io/socket.io.js"></script>
<script>
const socket = io();

let players = {};
let player = { x:5, y:5 };

let keys = {};
document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

function start() {
  socket.emit("join", {
    name: name.value,
    roomId: room.value
  });

  menu.style.display = "none";
}

socket.on("state", data => {
  players = data;
});

function move() {
  if (keys["ArrowUp"]) player.y -= 0.1;
  if (keys["ArrowDown"]) player.y += 0.1;
  if (keys["ArrowLeft"]) player.x -= 0.1;
  if (keys["ArrowRight"]) player.x += 0.1;

  socket.emit("move", player);
}

function shoot() {
  socket.emit("shoot", player);
}

document.addEventListener("keydown", e => {
  if (e.key === " ") shoot();
});

function draw() {
  let c = document.getElementById("c");
  let ctx = c.getContext("2d");

  ctx.clearRect(0,0,500,500);

  for (let id in players) {
    let p = players[id];

    ctx.fillStyle = id === socket.id ? "#00ffcc" : "blue";
    ctx.fillRect(p.x*40, p.y*40, 20, 20);

    ctx.fillStyle = "white";
    ctx.fillText(p.name, p.x*40, p.y*40 - 5);

    ctx.fillStyle = "red";
    ctx.fillRect(p.x*40, p.y*40 - 10, p.hp/5, 3);
  }
}

function loop() {
  move();
  draw();
  requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
