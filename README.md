<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>加密帳密保險箱</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
    <style>
        body { font-family: -apple-system, sans-serif; background-color: #f0f2f5; padding: 20px; }
        .card { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 20px; }
        input, select, button { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; font-size: 16px; }
        button { background-color: #007AFF; color: white; border: none; font-weight: bold; }
        .item { border-bottom: 1px solid #eee; padding: 15px 0; }
        .high-tag { background-color: #FF3B30; color: white; padding: 2px 6px; border-radius: 4px; font-size: 12px; }
    </style>
</head>
<body>

    <h2>➕ 新增加密紀錄</h2>
    <div class="card">
        <input type="text" id="purpose" placeholder="用途 (如: 銀行)">
        <input type="text" id="account" placeholder="帳號">
        <input type="password" id="password" placeholder="要加密的密碼">
        <select id="importance">
            <option value="Normal">一般 (點擊直接看)</option>
            <option value="High">高度重要 (看時需驗證)</option>
        </select>
        <button onclick="addEntry()">加密並儲存</button>
    </div>

    <h2>📂 我的帳密清單</h2>
    <div class="card" id="listContainer"></div>

    <div style="margin-top: 50px; text-align: center;">
        <button style="background:none; color:#007AFF; font-size:14px;" onclick="setOrChangeMasterKey()">管理主密碼 (金鑰)</button>
    </div>

    <script>
        let myData = JSON.parse(localStorage.getItem('encrypted_data')) || [];
        let savedMasterKey = localStorage.getItem('master_key');

        refreshUI();

        // --- 核心：加密函式 ---
        function encrypt(text) {
            // 使用主密碼對文字進行 AES 加密
            return CryptoJS.AES.encrypt(text, savedMasterKey).toString();
        }

        // --- 核心：解密函式 ---
        function decrypt(cipherText) {
            try {
                // 使用主密碼嘗試解密
                const bytes = CryptoJS.AES.decrypt(cipherText, savedMasterKey);
                const originalText = bytes.toString(CryptoJS.enc.Utf8);
                return originalText || "【解鎖失敗：金鑰錯誤】";
            } catch (e) {
                return "【解鎖失敗】";
            }
        }

        function addEntry() {
            if (!savedMasterKey) return alert("請先在頁面底部設定「主密碼」才能開始存檔！");
            
            const p = document.getElementById('purpose').value;
            const a = document.getElementById('account').value;
            const pw = document.getElementById('password').value;
            const imp = document.getElementById('importance').value;

            if (!p || !pw) return alert("請填寫用途與密碼");

            // 重點：存入前先叫加密工具把密碼攪碎
            const encryptedPassword = encrypt(pw);

            myData.push({ id: Date.now(), purpose: p, account: a, password: encryptedPassword, importance: imp });
            saveData();
            refreshUI();
            
            document.querySelectorAll('input').forEach(i => i.value = '');
        }

        function showPassword(encryptedPw, importance) {
            if (!savedMasterKey) return alert("未設定主密碼");

            const input = prompt("請輸入主密碼進行解密：");
            if (input === savedMasterKey) {
                const result = decrypt(encryptedPw);
                alert("解密後的密碼為: " + result);
            } else {
                alert("驗證失敗！");
            }
        }

        function setOrChangeMasterKey() {
            if (savedMasterKey) {
                const old = prompt("請輸入舊主密碼：");
                if (old !== savedMasterKey) return alert("錯誤，拒絕修改");
            }
            const newKey = prompt("設定新主密碼 (遺失將無法救回資料)：");
            if (newKey && newKey.length >= 4) {
                localStorage.setItem('master_key', newKey);
                savedMasterKey = newKey;
                alert("主密碼設定成功！舊資料仍需用舊密碼解密。");
            }
        }

        function refreshUI() {
            const container = document.getElementById('listContainer');
            container.innerHTML = myData.length === 0 ? '<p>尚無紀錄</p>' : '';
            myData.forEach(item => {
                const div = document.createElement('div');
                div.className = 'item';
                div.innerHTML = `
                    <strong>${item.purpose}</strong> ${item.importance==='High'?'<span class="high-tag">重要</span>':''}<br>
                    <small>帳號: ${item.account}</small><br>
                    <small>密碼: <span style="color:#007AFF; cursor:pointer;" onclick="showPassword('${item.password}', '${item.importance}')">解密並查看</span></small>
                    <button style="width:auto; border:none; background:none; color:gray; font-size:12px; margin-top:5px;" onclick="deleteItem(${item.id})">[刪除]</button>
                `;
                container.appendChild(div);
            });
        }

        function saveData() { localStorage.setItem('encrypted_data', JSON.stringify(myData)); }
        function deleteItem(id) { if (confirm("刪除？")) { myData = myData.filter(i => i.id !== id); saveData(); refreshUI(); } }
    </script>
</body>
</html>
