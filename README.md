<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>浪貓不浪 🐾 領養與生活誌</title>
    <style>
        :root { --p: #ff8fab; --bg: #fff6f8; --h: #ffd1dc; --b: #a2d2ff; --dark: #555; }
        body { font-family: "Microsoft JhengHei", sans-serif; background: var(--bg); margin: 0; color: var(--dark); line-height: 1.6; }
        
        header { background: var(--h); padding: 40px 15px; text-align: center; border-radius: 0 0 40px 40px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); }
        .nav { margin-top: 20px; display: flex; flex-wrap: wrap; justify-content: center; gap: 10px; }

        .btn { padding: 12px 24px; border-radius: 30px; border: none; cursor: pointer; font-weight: bold; transition: 0.3s; font-size: 14px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .btn-m { background: var(--p); color: white; }
        .btn-s { background: var(--b); color: white; }
        .btn:hover { transform: translateY(-3px); filter: brightness(1.1); }

        .container { padding: 30px 15px; display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 25px; max-width: 1200px; margin: 0 auto; }
        .card { background: white; border-radius: 25px; overflow: hidden; box-shadow: 0 10px 25px rgba(0,0,0,0.08); transition: 0.3s; text-align: center; }
        .card img { width: 100%; height: 250px; object-fit: cover; cursor: pointer; border-bottom: 4px solid var(--h); }
        .card-body { padding: 20px; }

        .overlay { display: none; position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.8); z-index: 1000; align-items: center; justify-content: center; backdrop-filter: blur(5px); }
        .modal { background: white; width: 90%; max-width: 500px; padding: 30px; border-radius: 30px; position: relative; max-height: 85vh; overflow-y: auto; }
        .close-btn { position: absolute; top: 15px; right: 20px; font-size: 28px; cursor: pointer; color: #ccc; }
        
        .form-group input, .form-group textarea, .form-group select { width: 100%; box-sizing: border-box; margin-bottom: 15px; padding: 12px; border-radius: 15px; border: 1px solid #eee; background: #fafafa; }
        
        .album-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px; margin-top: 15px; }
        .album-grid img { width: 100%; height: 150px; object-fit: cover; border-radius: 15px; }

        footer { text-align: center; padding: 50px; opacity: 0.6; font-size: 13px; }
    </style>
</head>
<body>

<header>
    <h1>🐾 浪貓不浪</h1>
    <p>讓愛延續，讓每一隻貓咪都有被寵溺的權利</p>
    <div class="nav">
        <button class="btn btn-m" onclick="loadCats('waiting')">🏠 正在等家</button>
        <button class="btn btn-m" onclick="loadCats('adopted')">🎊 幸福回娘家</button>
        <button class="btn btn-s" onclick="openAuthModal()">🔐 登入系統</button>
    </div>
</header>

<div id="display-area" class="container">
    <p style="text-align: center; grid-column: 1/-1;">同步雲端貓咪檔案中...</p>
</div>

<div id="modalOverlay" class="overlay">
    <div class="modal">
        <span class="close-btn" onclick="closeModal()">&times;</span>
        <h2 id="modalTitle" style="color: var(--p); text-align: center; margin-top: 0;">登入中心</h2>
        <div id="modalBody" class="form-group">
            </div>
    </div>
</div>

<footer>
    🐾 貓窩裏 到府寵物褓姆 ｜ 系統密碼：715715
</footer>

<script>
    /** --- 配置區 --- **/
    const WAITING_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vTf9z.../pub?output=csv"; 
    const ADOPTED_CSV = "https://docs.google.com/spreadsheets/d/e/2PACX-1vR_x.../pub?output=csv";
    const GAS_URL = "你的_GAS_部署網址"; 
    const ADMIN_PWD = "715715";

    let catsData = [];

    // 1. 載入資料
    async function loadCats(type) {
        const display = document.getElementById('display-area');
        display.innerHTML = "<p style='text-align:center; grid-column:1/-1;'>同步中...</p>";
        const url = (type === 'waiting') ? WAITING_CSV : ADOPTED_CSV;

        try {
            const res = await fetch(url);
            const csv = await res.text();
            const rows = csv.split('\n').slice(1);
            catsData = [];
            let html = "";

            rows.forEach(row => {
                const col = row.split(',');
                if (col.length < 5) return;
                const cat = { name: col[1], age: col[2], desc: col[4], img: col[5], status: type };
                catsData.push(cat);

                html += `
                    <div class="card">
                        <img src="${cat.img}" onclick="viewDetails('${cat.name}')">
                        <div class="card-body">
                            <h3>${cat.name}</h3>
                            <p>${cat.age}</p>
                            <button class="btn btn-s" style="width:100%" onclick="viewDetails('${cat.name}')">${type === 'waiting' ? '查看更多' : '查看生活照'}</button>
                        </div>
                    </div>`;
            });
            display.innerHTML = html;
        } catch (e) { display.innerHTML = "讀取失敗，請確認試算表已發佈。"; }
    }

    // 2. 登入視窗切換
    function openAuthModal() {
        showModal("身分登入", `
            <input type="text" id="loginUser" placeholder="管理員帳號 或 貓咪姓名">
            <input type="password" id="loginPwd" placeholder="登入密碼">
            <button class="btn btn-m" style="width:100%" onclick="handleLogin()">確認登入</button>
            <p style="font-size:12px; color:#999; text-align:center; margin-top:10px;">領養人預設密碼為：貓咪名+715</p>
        `);
    }

    // 3. 處理登入邏輯
    function handleLogin() {
        const user = document.getElementById('loginUser').value;
        const pwd = document.getElementById('loginPwd').value;

        if (user === "admin" && pwd === ADMIN_PWD) {
            showAdminPanel();
        } else if (pwd === user + "715") {
            showOwnerPanel(user);
        } else {
            alert("帳號或密碼錯誤！");
        }
    }

    // 4. 管理員介面
    function showAdminPanel() {
        showModal("管理員後台", `
            <select id="actionType"><option value="new">新增待領養貓咪</option><option value="notice">發布系統公告</option></select>
            <input id="fName" placeholder="貓咪姓名">
            <input id="fAge" placeholder="年齡/性別">
            <input id="fImg" placeholder="主要照片網址">
            <textarea id="fDesc" placeholder="個性描述"></textarea>
            <button class="btn btn-m" style="width:100%" onclick="sendToGas('new')">確認發布</button>
        `);
    }

    // 5. 領養人回娘家介面
    function showOwnerPanel(catName) {
        showModal(`${catName} 的家長你好！`, `
            <p style="font-size:14px;">分享 ${catName} 的近況給中途麻麻吧：</p>
            <input id="fImg" placeholder="上傳生活照網址">
            <textarea id="fDesc" placeholder="今天發生了什麼有趣的事？"></textarea>
            <input type="password" id="newPwd" placeholder="修改新密碼 (選填)">
            <button class="btn btn-s" style="width:100%" onclick="sendToGas('daily', '${catName}')">送出回娘家照片</button>
        `);
    }

    // 6. 通用發送函數
    function sendToGas(type, catName = "") {
        const data = {
            action: type,
            name: catName || document.getElementById('fName').value,
            img: document.getElementById('fImg').value,
            desc: document.getElementById('fDesc').value,
            age: document.getElementById('fAge')?.value || "",
            pwd: ADMIN_PWD // 實際上後端驗證用
        };

        alert("同步中，請稍候...");
        fetch(GAS_URL, { method: "POST", mode: "no-cors", body: JSON.stringify(data) })
        .then(() => {
            alert("同步成功！");
            closeModal();
            loadCats('waiting');
        });
    }

    // 輔助函數
    function showModal(title, body) {
        document.getElementById('modalOverlay').style.display = 'flex';
        document.getElementById('modalTitle').innerText = title;
        document.getElementById('modalBody').innerHTML = body;
    }
    function closeModal() { document.getElementById('modalOverlay').style.display = 'none'; }
    function viewDetails(name) {
        const cat = catsData.find(c => c.name === name);
        showModal(`${name} 的資訊`, `
            <img src="${cat.img}" style="width:100%; border-radius:15px; margin-bottom:15px;">
            <p><b>年齡：</b>${cat.age}</p>
            <p><b>故事：</b>${cat.desc}</p>
            <hr>
            <div class="album-grid" id="albumArea">讀取相簿中...</div>
            <button class="btn btn-m" style="width:100%; margin-top:15px;" onclick="window.open('https://line.me/R/msg/text/?我想了解貓咪：${name}')">LINE 預約/洽詢</button>
        `);
        // 此處可增加 fetch Daily CSV 的邏輯來填充相簿
        document.getElementById('albumArea').innerHTML = "<p style='font-size:12px; color:#999;'>尚無回娘家照片</p>";
    }

    window.onload = () => loadCats('waiting');
</script>
</body>
</html>
