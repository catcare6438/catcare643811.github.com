<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>浪貓不浪 🐾 管理系統</title>
    <style>
        :root { --p: #ff8fab; --bg: #fffcfd; }
        body { font-family: -apple-system, sans-serif; background: var(--bg); margin: 0; padding-bottom: 50px; color: #444; }
        header { background: white; padding: 20px; text-align: center; border-bottom: 1px solid #eee; position: sticky; top: 0; z-index: 100; }
        .container { padding: 15px; display: grid; grid-template-columns: 1fr 1fr; gap: 15px; }
        .card { background: white; border-radius: 15px; overflow: hidden; box-shadow: 0 4px 10px rgba(0,0,0,0.05); cursor: pointer; border: 1px solid #f0f0f0; }
        .card img { width: 100%; height: 130px; object-fit: cover; }
        .card-body { padding: 10px; text-align: center; }
        .card-body h3 { margin: 0; font-size: 1rem; color: var(--p); }
        
        /* 彈出視窗樣式 */
        .overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.7); z-index: 1000; justify-content: center; align-items: center; }
        .modal { background: white; width: 90%; max-width: 400px; border-radius: 20px; padding: 20px; max-height: 80vh; overflow-y: auto; position: relative; }
        .info-row { border-bottom: 1px solid #f9f9f9; padding: 8px 0; font-size: 0.9rem; display: flex; justify-content: space-between; }
        .info-label { color: #999; font-weight: bold; min-width: 80px; }
        .info-value { color: #333; text-align: right; word-break: break-all; }
        .close-btn { background: var(--p); color: white; border: none; width: 100%; padding: 12px; border-radius: 10px; margin-top: 15px; font-weight: bold; }
    </style>
</head>
<body>

<header><h1 style="margin:0; font-size:1.2rem; color:var(--p);">🐾 浪貓不浪 貓咪健康檔案</h1></header>

<div id="app" class="container">
    <div style="grid-column: 1/-1; text-align:center; padding:50px;">讀取雲端全資料中...</div>
</div>

<div id="overlay" class="overlay" onclick="closeModal()">
    <div class="modal" onclick="event.stopPropagation()">
        <div id="modal-content"></div>
        <button class="close-btn" onclick="closeModal()">關閉視窗</button>
    </div>
</div>

<script>
    const GAS_URL = "https://script.google.com/macros/s/AKfycbxT_924TOVaiZc7-fEu0gITRgLZNvWmDwJn8xnTg75tcLmPD0eXOanYffdKBYphlvrX/exec";
    let catData = [];

    async function fetchData() {
        try {
            const res = await fetch(GAS_URL);
            catData = await res.json();
            let html = "";
            catData.forEach((cat, index) => {
                html += `
                <div class="card" onclick="showDetail(${index})">
                    <img src="${cat.img}">
                    <div class="card-body">
                        <h3>${cat.name}</h3>
                        <p style="font-size:0.8rem; margin:5px 0 0;">${cat.age || '未知年紀'}</p>
                    </div>
                </div>`;
            });
            document.getElementById('app').innerHTML = html || "暫無資料";
        } catch (e) {
            document.getElementById('app').innerHTML = "讀取失敗，請確認 GAS 權限。";
        }
    }

    function showDetail(index) {
        const cat = catData[index];
        const rows = [
            ["入住日期", cat.date],
            ["編號", cat.sn],
            ["性別/花色", cat.gender + " " + cat.color],
            ["年紀", cat.age],
            ["疫苗紀錄", cat.vax],
            ["驅蟲紀錄", cat.worm],
            ["結紮狀態", cat.fixed],
            ["體重紀錄", cat.weight]
        ];

        let contentHtml = `<h2 style="text-align:center; color:var(--p); margin-top:0;">${cat.name}</h2>`;
        rows.forEach(row => {
            if(row[1]) {
                contentHtml += `<div class="info-row"><span class="info-label">${row[0]}</span><span class="info-value">${row[1].replace(/\n/g, '<br>')}</span></div>`;
            }
        });
        
        document.getElementById('modal-content').innerHTML = contentHtml;
        document.getElementById('overlay').style.display = 'flex';
    }

    function closeModal() { document.getElementById('overlay').style.display = 'none'; }
    window.onload = fetchData;
</script>
</body>
</html>
