<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>NITO Online Pro</title>
<script src="/socket.io/socket.io.js"></script>
<style>
body { margin:0; background:#111; color:#fff; font-family:sans-serif; overflow:hidden; }
#ui { position:fixed; top:10px; left:10px; background:#000c; padding:8px; border-radius:6px; }
canvas { background:#2b2b2b; display:block; }
#chat { position:fixed; bottom:0; width:100%; background:#000c; }
#messages { max-height:120px; overflow:auto; font-size:13px; padding:4px; }
input { width:100%; background:#222; color:#fff; border:none; padding:5px; }
</style>
</head>
<body>

<div id="ui">
<select id="role">
  <option value="criminal">خلافکار</option>
  <option value="police">پلیس</option>
</select>
<button onclick="shoot()">🔫 جرم</button>
<button onclick="toggleVehicle()">🚗 وسیله</button>
<div id="stats"></div>
</div>

<canvas id="game" width="800" height="500"></canvas>
<canvas id="map" width="200" height="125"
 style="position:fixed; top:10px; right:10px; border:1px solid #fff;"></canvas>

<div id="chat">
  <div id="messages"></div>
  <input id="msg" placeholder="چت...">
</div>

<script>
const socket = io();
const canvas = document.getElementById("game");
const c = canvas.getContext("2d");
const map = document.getElementById("map");
const m = map.getContext("2d");

const messages = document.getElementById("messages");
const msg = document.getElementById("msg");
const roleSelect = document.getElementById("role");
const stats = document.getElementById("stats");

let myId = null;
let me = null;
const players = {};
const keys = {};

onkeydown = e => keys[e.key] = true;
onkeyup = e => keys[e.key] = false;

socket.on("init", d => {
  myId = d.id;
  Object.assign(players, d.players);
  me = players[myId];
});

socket.on("playerJoined", p => players[p.id] = p);
socket.on("playerUpdate", p => players[p.id] = p);
socket.on("playerLeft", id => delete players[id]);

socket.on("crime", d => {
  if(me.role==="police")
    messages.innerHTML += `<div style="color:orange;">🚨 جرم گزارش شد</div>`;
});

socket.on("arrested", () => {
  messages.innerHTML += `<div style="color:red;">⛓️ دستگیر شدی</div>`;
  me.wanted = 0;
});

socket.on("chat", d => {
  messages.innerHTML += `<div><b>${d.role}</b>: ${d.text}</div>`;
  messages.scrollTop = messages.scrollHeight;
});

msg.onkeydown = e => {
  if(e.key==="Enter" && msg.value.trim()){
    socket.emit("chat", msg.value);
    msg.value="";
  }
};

roleSelect.onchange = ()=> me.role = roleSelect.value;

function shoot(){
  socket.emit("crime");
}

function toggleVehicle(){
  me.inVehicle = !me.inVehicle;
  me.vehicleType = me.inVehicle ? "car" : null;
}

function loop(){
  if(!me) return;

  let speed = me.inVehicle ? 5 : 3;
  if(keys.ArrowUp) me.y -= speed;
  if(keys.ArrowDown) me.y += speed;
  if(keys.ArrowLeft) me.x -= speed;
  if(keys.ArrowRight) me.x += speed;

  me.x = Math.max(0, Math.min(800, me.x));
  me.y = Math.max(0, Math.min(500, me.y));

  socket.emit("update", me);

  c.clearRect(0,0,800,500);
  for(let p of Object.values(players)){
    c.fillStyle =
      p.inVehicle ? "green" :
      p.role==="police" ? "blue" : "red";
    c.fillRect(p.x, p.y, p.inVehicle?25:20, p.inVehicle?25:20);

    if(me.role==="police" && p.role==="criminal" && p.wanted>0){
      let d = Math.hypot(me.x-p.x, me.y-p.y);
      if(d < 20){
        socket.emit("arrest", p.id);
      }
    }
  }

  m.clearRect(0,0,200,125);
  for(let p of Object.values(players)){
    m.fillStyle = p.role==="police"?"blue":"red";
    m.fillRect(p.x/4,p.y/4,4,4);
  }
  m.fillStyle="yellow";
  m.fillRect(me.x/4,me.y/4,5,5);

  stats.innerHTML = `
Level: ${me.level}<br>
XP: ${me.exp}<br>
Money: 💰${me.money}<br>
Wanted: ${me.wanted}
`;

  requestAnimationFrame(loop);
}
loop();
</script>
</body>
</html># Nitro
&lt;meta name="description" content="welcome to my website">
