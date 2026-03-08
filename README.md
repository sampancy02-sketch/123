<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>變壓器 U、n 關係模擬器</title>
    <style>
        body {
            font-family: 'Microsoft YaHei', Arial, sans-serif;
            max-width: 800px;
            margin: 20px auto;
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #2c3e50;
            text-align: center;
        }
        .circuit-diagram {
            background-color: #ecf0f1;
            padding: 20px;
            border-radius: 5px;
            text-align: center;
            margin: 20px 0;
            font-family: monospace;
            font-size: 18px;
        }
        .control-panel {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin: 20px 0;
            padding: 15px;
            background-color: #f8f9fa;
            border-radius: 5px;
        }
        .control-item {
            margin: 10px 0;
        }
        label {
            display: inline-block;
            width: 100px;
            font-weight: bold;
        }
        input[type="range"] {
            width: 200px;
            vertical-align: middle;
        }
        .voltage-display {
            font-size: 24px;
            text-align: center;
            padding: 20px;
            background-color: #d4edda;
            border-radius: 5px;
            margin: 20px 0;
        }
        .voltage-display span {
            font-size: 32px;
            color: #155724;
            font-weight: bold;
        }
        .data-table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        .data-table th, .data-table td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: center;
        }
        .data-table th {
            background-color: #4CAF50;
            color: white;
        }
        button {
            background-color: #4CAF50;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
        }
        button:hover {
            background-color: #45a049;
        }
        .mode-selector {
            text-align: center;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>⚡ 變壓器 U、n 關係模擬器 ⚡</h1>
        
        <!-- 電路圖 -->
        <div class="circuit-diagram">
            <pre>
  交流電源     原線圈      鐵芯      副線圈
    ～────┬────nnnn────┼────nnnn────┬──── 負載
          │            │            │
        U₁=220V      n₁匝        n₂匝      U₂=?
            </pre>
        </div>

        <!-- 控制面板 -->
        <div class="control-panel">
            <div class="control-item">
                <label>原線圈 n₁:</label>
                <input type="range" id="n1" min="50" max="500" value="200" step="10">
                <span id="n1-value">200</span> 匝
            </div>
            <div class="control-item">
                <label>副線圈 n₂:</label>
                <input type="range" id="n2" min="50" max="500" value="100" step="10">
                <span id="n2-value">100</span> 匝
            </div>
            <div class="control-item">
                <label>輸入電壓 U₁:</label>
                <input type="range" id="u1" min="10" max="380" value="220" step="10">
                <span id="u1-value">220</span> V
            </div>
            <div class="control-item">
                <label>模式：</label>
                <select id="mode">
                    <option value="fixed_n1">固定 n₁ (改變 n₂)</option>
                    <option value="fixed_n2">固定 n₂ (改變 n₁)</option>
                </select>
            </div>
        </div>

        <!-- 電壓顯示 -->
        <div class="voltage-display">
            U₂ = <span id="u2-display">110.00</span> V
        </div>

        <!-- 數據記錄 -->
        <h3>📊 實驗數據記錄</h3>
        <button onclick="recordData()">📝 記錄當前數據</button>
        <button onclick="clearData()">🗑️ 清除記錄</button>
        <table class="data-table" id="data-table">
            <thead>
                <tr>
                    <th>n₁ (匝)</th>
                    <th>n₂ (匝)</th>
                    <th>U₁ (V)</th>
                    <th>U₂ (V)</th>
                    <th>n₁/n₂</th>
                    <th>U₁/U₂</th>
                </tr>
            </thead>
            <tbody id="table-body">
                <!-- 記錄會動態添加在這裡 -->
            </tbody>
        </table>

        <h3>📈 關係圖</h3>
        <canvas id="chart" width="600" height="300" style="border:1px solid #ddd; background-color: white;"></canvas>
    </div>

    <script>
        // 獲取 DOM 元素
        const n1Slider = document.getElementById('n1');
        const n2Slider = document.getElementById('n2');
        const u1Slider = document.getElementById('u1');
        const n1Value = document.getElementById('n1-value');
        const n2Value = document.getElementById('n2-value');
        const u1Value = document.getElementById('u1-value');
        const u2Display = document.getElementById('u2-display');
        const modeSelect = document.getElementById('mode');
        const tableBody = document.getElementById('table-body');
        const canvas = document.getElementById('chart');
        const ctx = canvas.getContext('2d');

        // 數據存儲
        let records = [];

        // 更新顯示的函數
        function updateUI() {
            // 獲取當前值
            let n1 = parseInt(n1Slider.value);
            let n2 = parseInt(n2Slider.value);
            let u1 = parseInt(u1Slider.value);

            // 根據模式調整（控制變量）
            if (modeSelect.value === 'fixed_n1') {
                // 固定 n1，改變 n2 時自動計算
                n1 = 200; // 固定為 200 匝
                n1Slider.value = 200;
                n1Value.textContent = 200;
            } else {
                // 固定 n2，改變 n1 時自動計算
                n2 = 100; // 固定為 100 匝
                n2Slider.value = 100;
                n2Value.textContent = 100;
            }

            // 計算 U₂ (理想變壓器公式)
            let u2 = u1 * (n2 / n1);
            
            // 更新顯示
            n1Value.textContent = n1;
            n2Value.textContent = n2;
            u1Value.textContent = u1;
            u2Display.textContent = u2.toFixed(2);

            // 繪製關係圖
            drawChart();
        }

        // 繪製關係圖
        function drawChart() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 設定座標軸
            ctx.beginPath();
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 1;
            // X 軸
            ctx.moveTo(50, 250);
            ctx.lineTo(550, 250);
            // Y 軸
            ctx.moveTo(50, 250);
            ctx.lineTo(50, 50);
            ctx.stroke();

            // X 軸標籤
            ctx.font = '12px Arial';
            ctx.fillStyle = '#333';
            if (modeSelect.value === 'fixed_n1') {
                ctx.fillText('n₂ (匝)', 280, 280);
            } else {
                ctx.fillText('n₁ (匝)', 280, 280);
            }
            ctx.fillText('U₂ (V)', 20, 40);

            // 如果沒有數據，不畫點
            if (records.length === 0) return;

            // 找出最大值用於縮放
            let maxX = modeSelect.value === 'fixed_n1' 
                ? Math.max(...records.map(r => r.n2))
                : Math.max(...records.map(r => r.n1));
            let maxY = Math.max(...records.map(r => r.u2));

            // 繪製數據點
            records.forEach(record => {
                let x, y;
                if (modeSelect.value === 'fixed_n1') {
                    x = 50 + (record.n2 / maxX) * 500;
                } else {
                    x = 50 + (record.n1 / maxX) * 500;
                }
                y = 250 - (record.u2 / maxY) * 200;

                ctx.beginPath();
                ctx.fillStyle = 'red';
                ctx.arc(x, y, 5, 0, 2 * Math.PI);
                ctx.fill();
            });
        }

        // 記錄數據
        function recordData() {
            let n1 = parseInt(n1Slider.value);
            let n2 = parseInt(n2Slider.value);
            let u1 = parseInt(u1Slider.value);
            let u2 = u1 * (n2 / n1);

            // 添加到記錄
            records.push({
                n1: n1,
                n2: n2,
                u1: u1,
                u2: u2,
                ratio_n: (n1/n2).toFixed(2),
                ratio_u: (u1/u2).toFixed(2)
            });

            // 更新表格
            updateTable();

            // 更新圖表
            drawChart();
        }

        // 更新表格
        function updateTable() {
            tableBody.innerHTML = '';
            records.forEach(record => {
                let row = tableBody.insertRow();
                row.insertCell().textContent = record.n1;
                row.insertCell().textContent = record.n2;
                row.insertCell().textContent = record.u1;
                row.insertCell().textContent = record.u2.toFixed(2);
                row.insertCell().textContent = record.ratio_n;
                row.insertCell().textContent = record.ratio_u;
            });
        }

        // 清除數據
        function clearData() {
            records = [];
            tableBody.innerHTML = '';
            drawChart();
        }

        // 添加事件監聽
        n1Slider.addEventListener('input', updateUI);
        n2Slider.addEventListener('input', updateUI);
        u1Slider.addEventListener('input', updateUI);
        modeSelect.addEventListener('change', updateUI);

        // 初始化
        updateUI();
    </script>
</body>
</html>
