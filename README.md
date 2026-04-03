<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>浪貓不浪 🐾 官方 App</title>
    <style>
        :root { --p: #ff8fab; --bg: #fffcfd; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); margin: 0; padding-bottom: 90px; color: #444; }
        header { background: white; padding: 20px; text-align: center; border-bottom: 1px solid #eee; position: sticky; top: 0; z-index: 100; }
        header h1 { margin: 0; font-size: 1.2rem; color: var(--p); letter-spacing: 2px; }
        .container { padding: 15px; display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
        .card { background: white; border-radius: 20px; overflow: hidden; box-shadow: 0 4px 12px rgba(0,0,0,0.05); border: 1px solid #f5f5f5; text-align: center; }
        .card img { width: 100%; height: 140px; object-fit: cover; background: #ffeef2; }
        .card-info { padding: 12px; }
        .card-info h3 { margin: 0; font-size: 1rem; color: #333; }
        .card-info p { margin: 5px 0 0; font-size: 0.8rem; color: #999; }
        .nav { position: fixed; bottom: 0; left: 0; width: 100%; height: 75px; background: white; display: flex; justify-content: space-around; align-items: center; border-top: 1px solid #eee; }
        .nav-btn { background: none; border: none; color: #ccc; font-size: 0.75rem; display: flex; flex-direction: column; align-items: center; }
        .nav-btn.active { color: var(--p); font-weight: bold; }
        .overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); z-index: 1000; align-items: flex-end; }
        .modal { background: white; width: 100%; border-radius: 30px 30px 0 0; padding: 30px 20px; box-sizing: border-box; animation: slideUp 0.3s; }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
        input { width: 100%; padding: 15px; margin-bottom: 10px; border: 1px solid #eee; border-radius: 15px; box-sizing: border-box; }
        .btn-main { width: 100%; padding: 16px; border-radius: 25px; border: none; background: var(--p); color: white; font-weight: bold; }
    </style>
</head>
<body>

<header><h1>🐾 浪貓不浪</h1></header>

<div id="display" class="container">
    <div style="grid-column:1/-1; text-align:center; padding:50px;">連線資料庫中...</div>
</div>

<nav class="nav">
    <button class="nav-btn active"><span style="font-size:1.5rem">🏠</span><br>等家孩子</button>
    <button class="nav-btn" onclick="openAdmin()"><span style="font-size:1.5rem">⚙️</span><br>管理</button>
</nav>

<div id="overlay" class="overlay" onclick="closeModal()">
    <div class="modal" onclick="event.stopPropagation()" id="modal-content"></div>
</div>

<script>
    const GAS_URL = "https://script.google.com/macros/s/AKfycbxT_924TOVaiZc7-fEu0gITRgLZNvWmDwJn8xnTg75tcLmPD0eXOanYffdKBYphlvrX/exec";

    async function loadData() {
        const div = document.getElementById('display');
        try {
            const res = await fetch(GAS_URL);
            const cats = await res.json();
            let html = "";
            cats.forEach(cat => {
                html += `
                <div class="card" onclick="alert('${cat.name}\\n花色：${cat.color}\\n體重：${cat.weight}')">
                    <img src="${cat.img}">
                    <div class="card-info">
                        <h3>${cat.name}</h3>
                        <p>${cat.age}</p>
                    </div>
                </div>`;
            });
            div.innerHTML = html || "目前沒有資料";
        } catch (e) {
            div.innerHTML = "連線失敗，請檢查 GAS 是否部署為「所有人」";
        }
    }

    function openAdmin() {
        document.getElementById('modal-content').innerHTML = `
            <h2 style="text-align:center; margin-top:0;">新增貓咪</h2>
            <input id="n" placeholder="名字"><input id="c" placeholder="花色"><input id="a" placeholder="年紀"><input id="w" placeholder="體重">
            <button class="btn-main" onclick="sendCat()">確認發佈</button>
        `;
        document.getElementById('overlay').style.display = 'flex';
    }

    async function sendCat() {
        const d = { name: document.getElementById('n').value, color: document.getElementById('c').value, age: document.getElementById('a').value, weight: document.getElementById('w').value, pwd: '715715' };
        alert('同步中...');
        await fetch(GAS_URL, { method: 'POST', body: JSON.stringify(d) });
        location.reload();
    }

    function closeModal() { document.getElementById('overlay').style.display = 'none'; }
    window.onload = loadData;
</script>
</body>
</html>
