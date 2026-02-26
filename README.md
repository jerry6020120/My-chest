<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的帳密記錄本</title>
    <style>
        body { font-family: -apple-system, sans-serif; background-color: #f0f2f5; padding: 20px; }
        .card { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 20px; }
        input, select, button { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; font-size: 16px; }
        button { background-color: #007AFF; color: white; border: none; font-weight: bold; }
        .item { border-bottom: 1px solid #eee; padding: 15px 0; }
        .high-tag { background-color: #FF3B30; color: white; padding: 2px 6px; border-radius: 4px; font-size: 12px; }
        .normal-tag { background-color: #8E8E93; color: white; padding: 2px 6px; border-radius: 4px; font-size: 12px; }
    </style>
</head>
<body>

    <h2>➕ 新增紀錄</h2>
    <div class="card">
        <input type="text" id="purpose" placeholder="用途 (如: 銀行、IG)">
        <input type="text" id="account" placeholder="帳號">
        <input type="password" id="password" placeholder="密碼">
        <select id="importance">
            <option value="Normal">一般 (點擊直接看)</option>
            <option value="High">高度重要 (看時要輸密碼)</option>
        </select>
        <button onclick="addEntry()">儲存到手機</button>
    </div>

    <h2>📂 我的帳密清單</h2>
    <div class="card" id="listContainer">
        </div>

    <div style="margin-top: 50px; text-align: center;">
        <button style="background:none; color:#007AFF; font-size:14px;" onclick="setOrChangeMasterKey()">管理/設定主密碼</button>
    </div>

    <script>
        // --- 1. 初始化資料 ---
        let myData = JSON.parse(localStorage.getItem('pass_data')) || [];
        let savedMasterKey = localStorage.getItem('master_key');

        // 頁面一打開就顯示清單
        refreshUI();

        // --- 2. 新增資料 ---
        function addEntry() {
            const p = document.getElementById('purpose').value;
            const a = document.getElementById('account').value;
            const pw = document.getElementById('password').value;
            const imp = document.getElementById('importance').value;

            if (!p || !pw) return alert("請填寫用途與密碼");

            myData.push({ id: Date.now(), purpose: p, account: a, password: pw, importance: imp });
            saveData();
            refreshUI();
            
            // 清空輸入
            document.getElementById('purpose').value = '';
            document.getElementById('account').value = '';
            document.getElementById('password').value = '';
        }

        // --- 3. 顯示清單邏輯 ---
        function refreshUI() {
            const container = document.getElementById('listContainer');
            container.innerHTML = myData.length === 0 ? '<p>尚無紀錄</p>' : '';

            myData.forEach(item => {
                const isHigh = item.importance === 'High';
                const div = document.createElement('div');
                div.className = 'item';
                div.innerHTML = `
                    <div>
                        <strong>${item.purpose}</strong> 
                        <span class="${isHigh ? 'high-tag' : 'normal-tag'}">${isHigh ? '重要' : '一般'}</span>
                    </div>
                    <small>帳號: ${item.account}</small><br>
                    <small>密碼: <span style="color:#007AFF; cursor:pointer;" onclick="showPassword('${item.password}', '${item.importance}')">**** (點擊查看)</span></small>
                    <button style="width:auto; border:none; background:none; color:gray; font-size:12px; margin-top:5px;" onclick="deleteItem(${item.id})">[刪除]</button>
                `;
                container.appendChild(div);
            });
        }

        // --- 4. 查看密碼的關鍵邏輯 ---
        function showPassword(pw, importance) {
            if (importance === 'High') {
                // 如果沒有設定過主密碼，先叫他去設定
                if (!savedMasterKey) {
                    alert("你還沒設定主密碼喔！請點擊下方的「管理主密碼」進行設定。");
                    return;
                }
                
                // 跳出輸入框
                const input = prompt("【安全驗證】請輸入主密碼：");
                if (input === savedMasterKey) {
                    alert("密碼為: " + pw);
                } else {
                    alert("密碼錯誤，無法查看！");
                }
            } else {
                // 一般程度，直接給看
                alert("密碼為: " + pw);
            }
        }

        // --- 5. 管理主密碼 ---
        function setOrChangeMasterKey() {
            const newKey = prompt("請設定新的主密碼 (至少4位)：");
            if (newKey && newKey.length >= 4) {
                localStorage.setItem('master_key', newKey);
                savedMasterKey = newKey;
                alert("主密碼設定成功！");
            } else {
                alert("設定取消或字數不足。");
            }
        }

        // 存檔與刪除
        function saveData() { localStorage.setItem('pass_data', JSON.stringify(myData)); }
        function deleteItem(id) {
            if (confirm("確定刪除？")) {
                myData = myData.filter(i => i.id !== id);
                saveData();
                refreshUI();
            }
        }
    </script>
</body>
</html>
