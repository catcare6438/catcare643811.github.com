<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>浪貓不浪 🐾 官方管理 App</title>
    <style>
        :root { --p: #ff8fab; --bg: #fffcfd; --gray: #999; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: var(--bg); margin: 0; padding-bottom: 90px; color: #333; -webkit-tap-highlight-color: transparent; }
        header { background: white; padding: 18px; text-align: center; border-bottom: 1px solid #eee; position: sticky; top: 0; z-index: 100; }
        header h1 { margin: 0; font-size: 1.3rem; color: var(--p); letter-spacing: 1px; }
        .container { padding: 15px; display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; }
        .card { background: white; border-radius: 20px; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border: 1px solid #f0f0f0; display: flex; flex-direction: column; transition: 0.2s; }
        .card:active { transform: scale(0.96); opacity: 0.9; }
        .card img { width: 100%; height: 160px; object-fit: cover; background: #f9f9f9; }
        .card-body { padding: 12px; text-align: center; }
        .card h3 { margin: 0; font-size: 0.95rem; color: #333; }
        .card p { font-size: 0.75rem; color: var(--gray); margin: 4px 0 0 0; }
        .bottom-nav { position: fixed; bottom: 0; left: 0; width: 100%; height: 75px; background: rgba(255,255,255,0.95); backdrop-filter: blur(10px); display: flex; justify-content: space-around; align-items: center; border-top: 1px solid #eee; z-index: 1000; padding-bottom: env(safe-area-inset-bottom); }
        .nav-item { background: none; border: none; color: #bbb; font-size: 0.7rem; display: flex; flex-direction: column; align-items: center; width: 33%; }
        .nav-item.active { color: var(--p); font-weight: bold; }
        .nav-icon { font-size: 1.5rem; margin-bottom: 4px; }
        .overlay { display: none; position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.6); z-index: 2000; align-items: flex-end; }
        .modal { background: white; width: 100%; padding: 30px 20px; border-radius: 30px 30px 0 0; animation: slideUp 0.3s ease-out; box-sizing: border-box; max-height: 85vh; overflow-y: auto; }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
        input, textarea { width: 100%; padding: 14px; margin-bottom: 12px; border: 1px solid #eee; border-radius: 15px; background: #f9f9f9; box-sizing: border-box; font-size: 1rem; }
        .btn-main { width: 100%; padding: 15px; border-radius: 25px; border: none; background: var(--p); color: white; font-weight: bold; font-size: 1rem; box-shadow: 0 4px 10px rgba(255,143,171,0.3); }
        .loading { text-align: center; padding: 100px 20px; color: var(--p); grid-column: 1/-1; font-weight: 500; }
    </style>
</head>
<body>

<header><h1>🐾 浪貓不浪</h1></header>

<div id="display-area" class="container">
    <div class="loading">正在連線雲端資料庫...</div>
</div>

<nav class="bottom-nav">
    <button class="nav-item active" onclick="switchTab('waiting', this)"><span class="nav-icon">🏠</span>等家孩子</button>
    <button class="nav-item" onclick="switchTab('adopted', this)"><span class="nav-icon">🎉</span>幸福生活</button>
    <button class="nav-item" onclick="openLogin()"><span class="nav-icon">⚙️</span>系統管理</button>
</nav>

<div id="modalOverlay" class="overlay" onclick="closeModal()">
    <div class="modal" onclick="event.stopPropagation()"><div id="modalContent"></div></div>
</div>

<script>
    // 已為您填入正確的 GAS 網址
    const GAS_URL = "https://script.google.com/macros/s/AKfycbxT_924TOVaiZc7-fEu0gITRgLZNvWmDwJn8xnTg75tcLmPD0eXOanYffdKBYphlvrX/exec"; 

    let cats = [];

    async function loadData(type) {
        const display = document.getElementById('display-area');
        display.innerHTML = '<div class="loading">正在抓取最新資料...</div>';
        try {
            const res = await fetch(`${GAS_URL}?type=${type}`);
            cats = await res.json();
            let html = "";
            cats.forEach(c => {
                html += `
                    <div class="card" onclick="showDetail('${c.name}')">
                        <img src="${c.img || 'https://placehold.co/400x300?text=喵'}" onerror="this.src='https://placehold.co/400x300?text=照片載入中'">
                        <div class="card-body"><h3>${c.name}</h3><p>${c.age}</p></div>
                    </div>`;
            });
            display.innerHTML = html || '<div class="loading">目前沒有資料喔！</div>';
        } catch (e) {
            display.innerHTML = '<div class="loading" style="color:red">連線失敗<br>請確認 GAS 網址與權限。</div>';
        }
    }

    function switchTab(type, btn) {
        document.querySelectorAll('.nav-item').forEach(i => i.classList.remove('active'));
        btn.classList.add('active');
        loadData(type);
    }

    function showDetail(name) {
        const c = cats.find(cat => cat.name === name);
        document.getElementById('modalContent').innerHTML = `
            <h2 style="text-align:center; color:var(--p); margin-top:0;">${c.name}</h2>
            <img src="${c.img}" style="width:100%; border-radius:20px; margin-bottom:15px;">
            <p style="color:#666; line-height:1.6;">${c.desc}</p>
            <button class="btn-main" onclick="window.open('https://line.me/R/msg/text/?你好，我想詢問貓咪：${c.name}')">💬 LINE 預約看貓</button>
            <p style="text-align:center; color:#ccc; margin-top:15px; font-size:0.8rem;" onclick="closeModal()">關閉詳情</p>
        `;
        document.getElementById('modalOverlay').style.display = 'flex';
    }

    function openLogin() {
        document.getElementById('modalContent').innerHTML = `
            <h2 style="text-align:center; margin-top:0;">系統管理</h2>
            <input type="text" id="user" placeholder="管理員帳號">
            <input type="password" id="pass" placeholder="密碼">
            <button class="btn-main" onclick="handleLogin()">確認進入</button>
        `;
        document.getElementById('modalOverlay').style.display = 'flex';
    }

    function handleLogin() {
        if (document.getElementById('user').value === 'admin' && document.getElementById('pass').value === '715715') {
            showAdminForm();
        } else { alert("帳密錯誤！"); }
    }

    function showAdminForm() {
        document.getElementById('modalContent').innerHTML = `
            <h2 style="text-align:center; margin-top:0;">新增貓咪檔案</h2>
            <input id="n" placeholder="貓咪姓名 (必填)">
            <input id="a" placeholder="年齡 / 性別">
            <input id="i" placeholder="照片網址 (URL)">
            <textarea id="d" placeholder="貓咪的故事描述..." rows="4"></textarea>
            <button class="btn-main" onclick="sendData()">確認發佈到雲端</button>
        `;
    }

    async function sendData() {
        const payload = { action: 'new', name: document.getElementById('n').value, age: document.getElementById('a').value, img: document.getElementById('i').value, desc: document.getElementById('d').value, pwd: '715715' };
        if(!payload.name) return alert("請填寫姓名");
        alert("同步中，請稍候...");
        try {
            await fetch(GAS_URL, { method: 'POST', body: JSON.stringify(payload) });
            alert("成功發佈！"); location.reload();
        } catch (e) { alert("同步失敗！"); }
    }

    function closeModal() { document.getElementById('modalOverlay').style.display = 'none'; }
    window.onload = () => loadData('waiting');
</script>
</body>
</html>
