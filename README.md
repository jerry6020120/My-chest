<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的密碼保險箱</title>
    
    <style>
        /* CSS：這裡是設計外觀的地方（就像是幫房子裝潢） */
        body { 
            font-family: -apple-system, sans-serif; /* 使用 iPhone 的系統字體 */
            background-color: #f4f4f9; /* 背景色：淡灰色 */
            padding: 20px; 
            color: #333; 
        }
        .card { 
            background: white; 
            padding: 15px; 
            border-radius: 12px; /* 圓角效果 */
            box-shadow: 0 2px 10px rgba(0,0,0,0.1); /* 陰影，讓區塊看起來浮起來 */
            margin-bottom: 20px; 
        }
        input, select, button { 
            width: 100%; /* 寬度佔滿 */
            padding: 12px; 
            margin: 8px 0; 
            border: 1px solid #ddd; 
            border-radius: 8px; /* 讓輸入框也是圓角的 */
            box-sizing: border-box; /* 確保邊框不會把寬度撐破 */
            font-size: 16px; /* iPhone 上字體大於 16px 才不會在輸入時自動放大畫面 */
        }
        button { 
            background-color: #007AFF; /* iPhone 經典藍色 */
            color: white; 
            border: none; 
            font-weight: bold; 
        }
        .item { 
            border-bottom: 1px solid #eee; /* 每筆資料之間的一條細線 */
            padding: 15px 0; 
        }
        .importance-tag {
            font-size: 12px;
            padding: 2px 6px;
            border-radius: 4px;
            color: white;
            margin-left: 5px;
        }
        .high { background-color: #FF3B30; } /* 重要：紅色 */
        .normal { background-color: #34C759; } /* 一般：綠色 */
    </style>
</head>
<body>

    <h2>🔐 新增帳號密碼</h2>
    <div class="card">
        <input type="text" id="purpose" placeholder="用途 (例如: LINE、國泰銀行)">
        <input type="text" id="account" placeholder="帳號">
        <input type="password" id="password" placeholder="密碼">
        
        <label for="importance">重要程度：</label>
        <select id="importance">
            <option value="Normal">一般 (點擊即可看)</option>
            <option value="High">高度重要 (未來需加密)</option>
        </select>
        
        <button onclick="addEntry()">點我儲存到手機</button>
    </div>

    <h2>📂 我的清單</h2>
    <div class="card" id="listContainer">
        </div>

    <script>
        /* JavaScript：這裡是 App 的大腦（邏輯處理） */

        // 1. 【讀取舊資料】：一打開 App，先去手機的保險箱 (localStorage) 把以前存的資料拿出來
        // 如果沒有資料，就準備一個空的清單 []
        let myData = JSON.parse(localStorage.getItem('mySavedPasswords')) || [];

        // 2. 【顯示資料】：把讀取到的資料畫在螢幕上
        refreshUI();

        // 3. 【新增動作】：按下儲存按鈕後會發生的事
        function addEntry() {
            // 從畫面上的輸入框抓取文字
            const p = document.getElementById('purpose').value;
            const a = document.getElementById('account').value;
            const pw = document.getElementById('password').value;
            const imp = document.getElementById('importance').value;

            // 檢查有沒有漏填（至少要寫用途跟密碼）
            if (p === '' || pw === '') {
                alert("「用途」跟「密碼」不能空著喔！");
                return; // 停止往下執行
            }

            // 把這筆資料打包成一個小方塊
            const newRecord = {
                id: Date.now(), // 用時間當作身份證字號，確保每筆資料 ID 不同
                purpose: p,
                account: a,
                password: pw,
                importance: imp
            };

            // 把小方塊放進清單裡
            myData.push(newRecord);

            // 把更新後的清單，存進手機的硬碟裡 (localStorage)
            // JSON.stringify 是把清單變成文字，因為手機硬碟只看得懂文字
            localStorage.setItem('mySavedPasswords', JSON.stringify(myData));

            // 更新畫面、清空輸入框
            refreshUI();
            clearInputs();
        }

        // 4. 【刷新畫面】：把最新的清單顯示出來
        function refreshUI() {
            const container = document.getElementById('listContainer');
            
            // 先清空原本畫面上顯示的東西
            container.innerHTML = '';

            // 如果沒有資料，顯示提示文字
            if (myData.length === 0) {
                container.innerHTML = '<p style="color:gray;">目前還沒存任何密碼喔！</p>';
                return;
            }

            // 針對清單裡的每一筆資料，產出對應的 HTML 文字
            myData.forEach(function(item) {
                const colorClass = (item.importance === 'High') ? 'high' : 'normal';
                
                const itemHtml = `
                    <div class="item">
                        <strong>${item.purpose}</strong> 
                        <span class="importance-tag ${colorClass}">${item.importance}</span>
                        <br>
                        <small>帳號：${item.account}</small><br>
                        <small>密碼：<span style="color:#007AFF;" onclick="alert('您的密碼是：' + '${item.password}')">**** (點我顯示)</span></small>
                        <br>
                        <button style="width:auto; padding:5px 10px; background:none; color:red; border:1px solid red; margin-top:10px;" onclick="deleteItem(${item.id})">刪除</button>
                    </div>
                `;
                // 把這段文字塞進畫面的容器裡
                container.innerHTML += itemHtml;
            });
        }

        // 5. 【清空輸入框】：存完後把框框洗乾淨
        function clearInputs() {
            document.getElementById('purpose').value = '';
            document.getElementById('account').value = '';
            document.getElementById('password').value = '';
        }

        // 6. 【刪除動作】：點擊刪除按鈕時執行
        function deleteItem(targetId) {
            if (confirm("確定要刪除這筆紀錄嗎？")) {
                // 過濾掉那個 ID 的資料，剩下的留下來
                myData = myData.filter(item => item.id !== targetId);
                // 存回手機硬碟並重新整理畫面
                localStorage.setItem('mySavedPasswords', JSON.stringify(myData));
                refreshUI();
            }
        }
    </script>
</body>
</html>
