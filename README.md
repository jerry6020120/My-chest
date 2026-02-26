# My-chest
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的密碼保險箱</title>
    <style>
        /* 簡單的 CSS 讓介面在 iPhone 12 上看起來像 App */
        body { font-family: -apple-system, sans-serif; background: #f4f4f9; padding: 20px; }
        .card { background: white; padding: 15px; border-radius: 12px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); margin-bottom: 20px; }
        h2 { color: #333; font-size: 1.2rem; }
        input, select, button { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; }
        button { background: #007AFF; color: white; border: none; font-weight: bold; cursor: pointer; }
        .item { border-bottom: 1px solid #eee; padding: 10px 0; display: flex; justify-content: space-between; align-items: center; }
        .importance { font-size: 0.8rem; padding: 2px 6px; border-radius: 4px; color: white; }
        .high { background: #FF3B30; } /* 高重要度用紅色 */
        .normal { background: #34C759; } /* 一般用綠色 */
    </style>
</head>
<body>

    <h2>🔐 新增帳號密碼</h2>
    <div class="card">
        <input type="text" id="purpose" placeholder="用途 (例如：富邦銀行)">
        <input type="text" id="account" placeholder="帳號">
        <input type="password" id="password" placeholder="密碼">
        <select id="importance">
            <option value="Normal">一般程度 (直接查看)</option>
            <option value="High">高度重要 (需二次驗證)</option>
        </select>
        <button onclick="addEntry()">儲存到手機</button>
    </div>

    <h2>📂 我的帳密清單</h2>
    <div class="card">
        <select id="filter" onchange="displayEntries()">
            <option value="All">顯示全部</option>
            <option value="High">只看高度重要</option>
            <option value="Normal">只看一般</option>
        </select>
        <div id="listContainer">
            </div>
    </div>

    <script>
        // 這部分邏輯類似 C 語言的 Array 管理
        let entries = [];

        function addEntry() {
            const entry = {
                purpose: document.getElementById('purpose').value,
                account: document.getElementById('account').value,
                password: document.getElementById('password').value,
                importance: document.getElementById('importance').value
            };
            
            if(!entry.purpose) return alert("請輸入用途");
            
            entries.push(entry);
            displayEntries();
            
            // 清空輸入框
            document.getElementById('purpose').value = '';
            document.getElementById('account').value = '';
            document.getElementById('password').value = '';
        }

        function displayEntries() {
            const container = document.getElementById('listContainer');
            const filterValue = document.getElementById('filter').value;
            container.innerHTML = '';

            entries.forEach(item => {
                if (filterValue !== 'All' && item.importance !== filterValue) return;

                const div = document.createElement('div');
                div.className = 'item';
                div.innerHTML = `
                    <div>
                        <strong>${item.purpose}</strong><br>
                        <small>${item.account}</small>
                    </div>
                    <span class="importance ${item.importance === 'High' ? 'high' : 'normal'}">
                        ${item.importance}
                    </span>
                `;
                container.appendChild(div);
            });
        }
    </script>
</body>
</html>
