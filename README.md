<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>浪貓不浪</title>

<style>
body {font-family: sans-serif;background:#fff7f7;margin:0;}
header {text-align:center;padding:25px;background:#ffd6e0;}
.tabs {display:flex;justify-content:center;gap:10px;margin:10px;}
.tab {padding:8px 15px;background:#ffe4ec;border-radius:20px;cursor:pointer;}
.active {background:#ff8fab;color:white;}
.grid {display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:12px;padding:15px;}
.card {background:white;border-radius:15px;padding:10px;box-shadow:0 3px 8px rgba(0,0,0,0.08);}
.card img {width:100%;border-radius:10px;}
button {width:100%;margin-top:5px;border:none;padding:7px;border-radius:10px;background:#ff8fab;color:white;}
.footer {text-align:center;padding:30px;font-size:13px;color:#777;}
</style>

</head>
<body>

<header>
<h2>🐾 浪貓不浪</h2>
<p>讓每一隻貓，都有被愛的機會</p>
</header>

<div class="tabs">
<div class="tab active" onclick="switchTab('waiting')">等家中</div>
<div class="tab" onclick="switchTab('adopted')">已被愛</div>
<div class="tab" onclick="showFavorites()">❤️ 收藏</div>
</div>

<div id="list" class="grid"></div>

<div class="footer">
🐾 貓窩裏 到府寵物褓姆  
<br>用陪伴，讓牠在家也安心 💕
</div>

<script>

const SHEET_URL = "你的CSV網址";
const LINE_LINK = "https://line.me/R/ti/p/你的LINEID";

let cats = [];
let tab = "waiting";

fetch(SHEET_URL)
.then(r=>r.text())
.then(t=>{
  const rows = t.split("\n").slice(1);
  cats = rows.map(r=>{
    const c = r.split(",");
    return {
      id:c[0],name:c[1],age:c[2],
      personality:c[4],desc:c[5],
      img:c[7],status:c[8],
      adopt:c[9],adopter:c[10],
      share:c[11]
    }
  });
  render();
});

function switchTab(t){
  tab=t;
  document.querySelectorAll(".tab").forEach(e=>e.classList.remove("active"));
  event.target.classList.add("active");
  render();
}

function render(listData){
  const data = listData || cats.filter(c=> tab==="waiting"?c.status==="等待家":c.status==="已找到家");

  document.getElementById("list").innerHTML =
  data.map(c=>`
    <div class="card">
      <img src="${c.img}">
      <h4>${c.name}</h4>
      <p>${c.personality}</p>

      <button onclick="view('${c.share}')">看看我</button>
      <button onclick="fav('${c.id}')">❤️ 收藏</button>
      <button onclick="line('${c.name}')">LINE問我</button>
    </div>
  `).join("");
}

// ❤️ 收藏功能
function fav(id){
  let f = JSON.parse(localStorage.getItem("fav")||"[]");
  if(!f.includes(id)) f.push(id);
  localStorage.setItem("fav",JSON.stringify(f));
  alert("已收藏 ❤️");
}

function showFavorites(){
  const f = JSON.parse(localStorage.getItem("fav")||"[]");
  const favCats = cats.filter(c=>f.includes(c.id));
  render(favCats);
}

// LINE 帶入名字
function line(name){
  window.open(`${LINE_LINK}?text=我想詢問貓咪：${name}`);
}

// 查看分享頁
function view(id){
  location.href="?cat="+id;
}

// 分享頁 + 上傳功能
const url = new URLSearchParams(location.search);
const catId = url.get("cat");

if(catId){
  document.body.innerHTML="載入中...";

  fetch(SHEET_URL)
  .then(r=>r.text())
  .then(t=>{
    const rows = t.split("\n").slice(1);
    const c = rows.map(r=>r.split(",")).find(x=>x[11]==catId);

    const days = Math.floor((new Date()-new Date(c[9]))/(86400000));

    document.body.innerHTML=`
      <div style="padding:20px">
        <h2>${c[1]}</h2>
        <p>已被愛 ${days} 天 ❤️</p>
        <img src="${c[7]}" style="width:100%;border-radius:15px">
        <p>${c[5]}</p>
        <h4>現在的家人：${c[10]}</h4>

        <hr>
        <h3>📸 分享牠的日常</h3>

        <input id="imgInput" placeholder="貼上圖片網址">
        <textarea id="textInput" placeholder="寫點牠的生活..."></textarea>
        <button onclick="submitShare()">送出分享</button>

        <p id="msg"></p>
      </div>
    `;
  });
}

// 📸 上傳（用 Google Form）
function submitShare(){
  const img = document.getElementById("imgInput").value;
  const text = document.getElementById("textInput").value;

  fetch("你的GoogleForm網址",{
    method:"POST",
    mode:"no-cors",
    body:new URLSearchParams({
      "entry.圖片":img,
      "entry.文字":text
    })
  });

  document.getElementById("msg").innerText="已送出 💕";
}

</script>

</body>
</html>
