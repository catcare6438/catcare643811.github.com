<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>浪貓不浪 🐾 官方 App</title>
    <style>
        :root { --p: #ff8fab; --bg: #fffcfd; --h: #ffd1dc; --b: #a2d2ff; --text: #444; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background: var(--bg); margin: 0; padding-bottom: 80px; -webkit-tap-highlight-color: transparent; }
        
        /* 頂部標題 */
        header { background: white; padding: 18px; text-align: center; border-bottom: 1px solid #f0f0f0; position: sticky; top: 0; z-index: 100; box-shadow: 0 2px 10px rgba(0,0,0,0.02); }
        header h1 { margin: 0; font-size: 1.3rem; color: var(--p); letter-spacing: 1px; }

        /* 內容容器 */
        .container { padding: 12px; display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; }
        
        /* 貓咪卡片 */
        .card { background: white; border-radius: 20px; overflow: hidden; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border: 1px solid #f8f8f8; display: flex; flex-direction: column; transition: 0.2s; }
        .card:active { transform: scale(0.96); opacity: 0.8; }
        .card img { width: 100%; height: 150px; object-fit: cover; background: #fdfdfd; }
        .card-body { padding: 10px; }
        .card h3 { margin: 0; font-size: 0.95rem; color: #333; }
        .card p { font-size: 0.75rem; color: #999; margin: 4px 0 0 0; }

        /* 底部 App 導覽列 */
        .bottom-nav { position: fixed; bottom: 0; left: 0; width: 100%; height: 70px; background: rgba(255,255,255,0.9); backdrop-filter: blur(15px); display: flex; justify-content: space-around; align-items: center; border-top: 1px solid #eee; z-index: 1000; padding-bottom: env(safe-area-inset-bottom); }
        .nav-item { background: none; border: none; color: #bbb; font-size: 0.7rem; cursor: pointer; display: flex; flex-direction: column; align-items: center; width: 33%; }
        .nav-item.active { color: var(--p); font-weight: bold; }
        .nav-icon { font-size: 1.4rem; margin-bottom: 3px; }

        /* 從底部彈出的詳細視窗 */
        .overlay { display: none; position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.5); z-index: 2000; align-items: flex-end; }
        .modal { background: white; width: 100%; padding: 25px 20px; border-radius: 30px 30px 0 0; animation: slideUp 0.3s ease-out; box-sizing: border-box; max-height: 85vh; overflow-y: auto; }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
        
        .btn-action { width: 100%; padding: 14px; border-radius: 20px; border: none; background: var(--p); color: white; font-weight: bold; font-size: 1rem; margin-top: 15px; box-shadow: 0 4px 10px rgba(255,143,171,0.3); }
        .loading { text-align: center; padding: 100px 20px; color: var(--p); grid-column: 1/-1; font-weight: bold; }
    </style>
</head>
<body>

<header><h1>🐾 浪貓不浪</h1></header>

<div id="display-area" class="container">
    <div class="loading">正在連線雲端資料庫...</div>
</div>

<nav class="bottom-nav">
    <button class="nav-item active" onclick="switchTab('waiting', this)">
        <span class="nav-icon">🏠</span>等家孩子
    </button>
    <button class="nav-item" onclick="switchTab('adopted', this)">
        <span class="nav-icon">🎉</span>幸福回娘家
    </button>
    <button class="nav-item" onclick="openLogin()">
        <span class="nav-icon">⚙️</span>管理後台
    </button>
</nav>

<div id="modalOverlay" class="overlay" onclick="if(event.target==this)closeModal()">
    <div class="modal">
        <div id="modalContent"></div>
    </div>
</div>

<script>
    /** --- 數據源配置 --- **/
    // 自動轉換為資料抓取格式 (CSV)
    const WAITING_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRCdE9UVNcCsau-k6E2wPWR8H8Mh76hMluxKjH8SZQLW2Vp7ZvZQSvDz8lRLhJwq4LLPXu0qaD9aQhR/pub?output=csv";
    const ADOPTED_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vSzApIyvMpCu12nSWMliqX3xa2_mDqKHtXaK1XtUgaAh0l_KNIMf0mCu7aj6UXzJYEaUa3BgIrCRW3O/pub?output=csv";

    let currentCats = [];

    async function fetchData(type) {
        const display = document.getElementById('display-area');
        const url = (type === 'waiting') ? WAITING_CSV : ADOPTED_CSV;
        
        try {
            // 加入隨機數防止快取，確保資料是最新的
            const response = await fetch(url + '&t=' + Date.now());
            const text = await response.text();
            
            // 解析 CSV 每一行
            const rows = text.split('\n').map(row => row.split(','));
            currentCats = [];
            let html = "";

            // 從第二列開始跑 (i=1)
            for (let i = 1; i < rows.length; i++) {
                const col = rows[i];
                if (col.length < 5) continue;

                const cat = {
                    name: col[1]?.replace(/"/g, "").trim(),
                    age: col[2]?.replace(/"/g, "").trim(),
                    desc: col[4]?.replace(/"/g, "").trim(),
                    img: col[5]?.replace(/"/g, "").trim() || "https://placehold.co/400x400?text=圖片載入中"
                };
                currentCats.push(cat);

                html += `
                    <div class="card" onclick="showCatDetail('${cat.name}')">
                        <img src="${cat.img}" onerror="this.src='https://placehold.co/400x300?text=喵~照片迷路了'">
                        <div class="card-body">
                            <h3>${cat.name}</h3>
                            <p>${cat.age}</p>
                        </div>
                    </div>`;
            }
            display.innerHTML = html || '<div class="loading">目前沒有貓咪資訊喔！</div>';
        } catch (err) {
            display.innerHTML = '<div class="loading" style="color:red">讀取失敗<br>請確認試算表已「發佈到網路」</div>';
        }
    }

    function switchTab(type, el) {
        document.querySelectorAll('.nav-item').forEach(i => i.classList.remove('active'));
        el.classList.add('active');
        fetchData(type);
    }

    function showCatDetail(name) {
        const cat = currentCats.find(c => c.name === name);
        if(!cat) return;
        
        const content = `
            <h2 style="color:var(--p); text-align:center; margin:0 0 15px 0;">${cat.name}</h2>
            <img src="${cat.img}" style="width:100%; border-radius:20px; box-shadow:0 4px 12px rgba(0,0,0,0.1); margin-bottom:15px;">
            <p style="font-size:1rem; color:#333; margin-bottom:8px;"><b>關於孩子：</b></p>
            <p style="color:#6
