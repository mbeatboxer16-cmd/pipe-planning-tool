<!DOCTYPE html>

<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>配管計画ツール - 材料マスター管理</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

```
<!-- 即座に読み込まれる関数定義 -->
<script>
    // onclick属性から呼び出される関数を先に定義
    function switchTab(tabName) {
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
        
        event.target.classList.add('active');
        document.getElementById(tabName).classList.add('active');

        if (tabName === 'list') {
            if (typeof displayMaterials === 'function') displayMaterials();
            if (typeof updateStats === 'function') updateStats();
        }
    }
    
    // テンプレートデータ（グローバル）
    window.templates = {
        'bend_45_300': {
            materialType: '曲管',
            angle: 45,
            diameter: 300,
            effectiveLength: 450,
            radius: 600,
            insertLength: 120,
            unitPrice: 15000,
            maker: '積水樹脂',
            partNumber: 'ABC-300-45'
        },
        'bend_45_400': {
            materialType: '曲管',
            angle: 45,
            diameter: 400,
            effectiveLength: 600,
            radius: 800,
            insertLength: 150,
            unitPrice: 22000,
            maker: '積水樹脂',
            partNumber: 'ABC-400-45'
        },
        'bend_22_300': {
            materialType: '曲管',
            angle: 22.5,
            diameter: 300,
            effectiveLength: 300,
            radius: 600,
            insertLength: 120,
            unitPrice: 12000,
            maker: '積水樹脂',
            partNumber: 'ABC-300-22'
        },
        'bend_11_300': {
            materialType: '曲管',
            angle: 11.25,
            diameter: 300,
            effectiveLength: 200,
            radius: 600,
            insertLength: 120,
            unitPrice: 10000,
            maker: '積水樹脂',
            partNumber: 'ABC-300-11'
        }
    };
    
    function loadTemplate(templateId) {
        const template = window.templates[templateId];
        if (template) {
            document.getElementById('materialType').value = template.materialType;
            document.getElementById('angle').value = template.angle;
            document.getElementById('diameter').value = template.diameter;
            document.getElementById('effectiveLength').value = template.effectiveLength;
            document.getElementById('radius').value = template.radius;
            document.getElementById('insertLength').value = template.insertLength;
            document.getElementById('unitPrice').value = template.unitPrice;
            document.getElementById('maker').value = template.maker;
            document.getElementById('partNumber').value = template.partNumber;
            
            if (typeof showAlert === 'function') showAlert('テンプレートを読み込みました', 'success');
        }
    }
</script>

<style>
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
    }

    body {
        font-family: 'Hiragino Kaku Gothic ProN', 'Meiryo', sans-serif;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        min-height: 100vh;
        padding: 10px;
    }

    .container {
        max-width: 1200px;
        margin: 0 auto;
        background: white;
        border-radius: 10px;
        box-shadow: 0 10px 40px rgba(0,0,0,0.2);
        overflow: hidden;
    }

    .header {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 20px;
        text-align: center;
    }

    .header h1 {
        font-size: 24px;
        margin-bottom: 5px;
    }

    .header p {
        font-size: 14px;
        opacity: 0.9;
    }

    .tabs {
        display: flex;
        background: #f5f5f5;
        border-bottom: 2px solid #ddd;
    }

    .tab {
        flex: 1;
        padding: 15px;
        text-align: center;
        cursor: pointer;
        background: #f5f5f5;
        border: none;
        font-size: 16px;
        transition: all 0.3s;
    }

    .tab:hover {
        background: #e0e0e0;
    }

    .tab.active {
        background: white;
        border-bottom: 3px solid #667eea;
        font-weight: bold;
    }

    .content {
        padding: 20px;
    }

    .section {
        display: none;
    }

    .section.active {
        display: block;
    }

    .form-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
        margin-bottom: 20px;
    }

    .form-group {
        display: flex;
        flex-direction: column;
    }

    .form-group label {
        font-size: 14px;
        color: #555;
        margin-bottom: 5px;
        font-weight: bold;
    }

    .form-group input,
    .form-group select {
        padding: 10px;
        border: 2px solid #ddd;
        border-radius: 5px;
        font-size: 14px;
        transition: border-color 0.3s;
    }

    .form-group input:focus,
    .form-group select:focus {
        outline: none;
        border-color: #667eea;
    }

    .form-group input[type="number"] {
        text-align: right;
    }

    .button-group {
        display: flex;
        gap: 10px;
        margin-bottom: 20px;
        flex-wrap: wrap;
    }

    .btn {
        padding: 12px 24px;
        border: none;
        border-radius: 5px;
        font-size: 14px;
        font-weight: bold;
        cursor: pointer;
        transition: all 0.3s;
        display: inline-flex;
        align-items: center;
        gap: 5px;
    }

    .btn:hover {
        transform: translateY(-2px);
        box-shadow: 0 5px 15px rgba(0,0,0,0.2);
    }

    .btn-primary {
        background: #667eea;
        color: white;
    }

    .btn-success {
        background: #48bb78;
        color: white;
    }

    .btn-warning {
        background: #ed8936;
        color: white;
    }

    .btn-danger {
        background: #f56565;
        color: white;
    }

    .btn-secondary {
        background: #718096;
        color: white;
    }

    .table-container {
        overflow-x: auto;
        margin-top: 20px;
    }

    table {
        width: 100%;
        border-collapse: collapse;
        font-size: 13px;
    }

    th, td {
        padding: 12px 8px;
        text-align: left;
        border-bottom: 1px solid #ddd;
    }

    th {
        background: #f5f5f5;
        font-weight: bold;
        position: sticky;
        top: 0;
        z-index: 10;
    }

    tr:hover {
        background: #f9f9f9;
    }

    .action-buttons {
        display: flex;
        gap: 5px;
    }

    .btn-small {
        padding: 5px 10px;
        font-size: 12px;
    }

    .filter-section {
        background: #f9f9f9;
        padding: 15px;
        border-radius: 5px;
        margin-bottom: 20px;
    }

    .filter-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
        gap: 10px;
    }

    .stats-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
        margin-bottom: 20px;
    }

    .stat-card {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 20px;
        border-radius: 8px;
        text-align: center;
    }

    .stat-card h3 {
        font-size: 32px;
        margin-bottom: 5px;
    }

    .stat-card p {
        font-size: 14px;
        opacity: 0.9;
    }

    .alert {
        padding: 15px;
        border-radius: 5px;
        margin-bottom: 20px;
    }

    .alert-success {
        background: #c6f6d5;
        color: #22543d;
        border-left: 4px solid #48bb78;
    }

    .alert-error {
        background: #fed7d7;
        color: #742a2a;
        border-left: 4px solid #f56565;
    }

    .modal {
        display: none;
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        background: rgba(0,0,0,0.5);
        z-index: 1000;
        justify-content: center;
        align-items: center;
    }

    .modal.active {
        display: flex;
    }

    .modal-content {
        background: white;
        padding: 30px;
        border-radius: 10px;
        max-width: 500px;
        width: 90%;
        max-height: 90vh;
        overflow-y: auto;
    }

    .modal-header {
        font-size: 20px;
        font-weight: bold;
        margin-bottom: 20px;
        color: #667eea;
    }

    @media (max-width: 768px) {
        .header h1 {
            font-size: 20px;
        }

        .form-grid {
            grid-template-columns: 1fr;
        }

        .tab {
            font-size: 14px;
            padding: 12px 5px;
        }

        table {
            font-size: 11px;
        }

        th, td {
            padding: 8px 4px;
        }
    }

    .template-section {
        background: #fff3cd;
        padding: 15px;
        border-radius: 5px;
        margin-bottom: 20px;
        border-left: 4px solid #ffc107;
    }

    .template-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
        gap: 10px;
        margin-top: 10px;
    }

    .template-btn {
        padding: 10px;
        background: white;
        border: 2px solid #ffc107;
        border-radius: 5px;
        cursor: pointer;
        text-align: center;
        transition: all 0.3s;
    }

    .template-btn:hover {
        background: #ffc107;
        color: white;
    }

    .empty-state {
        text-align: center;
        padding: 40px;
        color: #999;
    }

    .empty-state img {
        width: 100px;
        opacity: 0.3;
        margin-bottom: 20px;
    }

    /* 3Dビューア */
    #viewer3D {
        width: 100%;
        height: 500px;
        background: #1a1a1a;
        border-radius: 8px;
        margin-bottom: 20px;
        position: relative;
        overflow: hidden;
    }

    .viewer-controls {
        position: absolute;
        top: 10px;
        right: 10px;
        background: rgba(255,255,255,0.9);
        padding: 10px;
        border-radius: 5px;
        font-size: 12px;
        z-index: 100;
    }

    .viewer-controls button {
        display: block;
        width: 100%;
        margin-bottom: 5px;
    }

    /* 複数ルート比較 */
    .route-card {
        background: white;
        border: 2px solid #ddd;
        border-radius: 8px;
        padding: 15px;
        margin-bottom: 15px;
        cursor: pointer;
        transition: all 0.3s;
    }

    .route-card:hover {
        border-color: #667eea;
        box-shadow: 0 5px 15px rgba(102,126,234,0.3);
    }

    .route-card.selected {
        border-color: #667eea;
        background: #f0f4ff;
    }

    .route-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 10px;
    }

    .route-badge {
        background: #667eea;
        color: white;
        padding: 5px 10px;
        border-radius: 5px;
        font-size: 12px;
        font-weight: bold;
    }

    .route-stats {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 10px;
        font-size: 13px;
    }

    .route-stat-item {
        text-align: center;
    }

    .route-stat-value {
        font-size: 18px;
        font-weight: bold;
        color: #667eea;
    }

    .route-stat-label {
        color: #666;
        font-size: 11px;
    }
</style>
```

</head>
<body>
    <div class="container">
        <div class="header">
            <h1>⚙️ 配管計画ツール</h1>
            <p>材料マスター管理システム</p>
        </div>

```
    <div class="tabs">
        <button class="tab active" onclick="switchTab('register')">📝 材料登録</button>
        <button class="tab" onclick="switchTab('list')">📋 材料一覧</button>
        <button class="tab" onclick="switchTab('calculate')">🧮 配管計算</button>
        <button class="tab" onclick="switchTab('import')">📥 データ管理</button>
    </div>

    <div class="content">
        <!-- 材料登録セクション -->
        <div id="register" class="section active">
            <div class="template-section">
                <h3>📌 よく使う規格テンプレート</h3>
                <div class="template-grid">
                    <div class="template-btn" onclick="loadTemplate('bend_45_300')">45度曲管<br>φ300</div>
                    <div class="template-btn" onclick="loadTemplate('bend_45_400')">45度曲管<br>φ400</div>
                    <div class="template-btn" onclick="loadTemplate('bend_22_300')">22.5度曲管<br>φ300</div>
                    <div class="template-btn" onclick="loadTemplate('bend_11_300')">11.25度曲管<br>φ300</div>
                </div>
            </div>

            <div id="alertContainer"></div>

            <h2 style="margin-bottom: 15px;">材料情報入力</h2>
            <form id="materialForm">
                <div class="form-grid">
                    <div class="form-group">
                        <label>材料種別 *</label>
                        <select id="materialType" required>
                            <option value="曲管">曲管</option>
                            <option value="直管">直管</option>
                            <option value="継手">継手</option>
                            <option value="その他">その他</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>曲管角度 (度)</label>
                        <input type="number" id="angle" step="0.01" placeholder="例: 45">
                    </div>
                    <div class="form-group">
                        <label>管径 (mm) *</label>
                        <input type="number" id="diameter" required placeholder="例: 300">
                    </div>
                    <div class="form-group">
                        <label>有効長 L (mm) *</label>
                        <input type="number" id="effectiveLength" required placeholder="例: 450">
                    </div>
                    <div class="form-group">
                        <label>曲率半径 R (mm)</label>
                        <input type="number" id="radius" placeholder="例: 600">
                    </div>
                    <div class="form-group">
                        <label>入り込み長さ (mm)</label>
                        <input type="number" id="insertLength" placeholder="例: 120">
                    </div>
                    <div class="form-group">
                        <label>単価 (円)</label>
                        <input type="number" id="unitPrice" placeholder="例: 15000">
                    </div>
                    <div class="form-group">
                        <label>メーカー</label>
                        <input type="text" id="maker" placeholder="例: 積水樹脂">
                    </div>
                    <div class="form-group">
                        <label>品番</label>
                        <input type="text" id="partNumber" placeholder="例: ABC-300-45">
                    </div>
                    <div class="form-group">
                        <label>備考</label>
                        <input type="text" id="notes" placeholder="備考">
                    </div>
                </div>

                <div class="button-group">
                    <button type="submit" class="btn btn-primary">💾 登録</button>
                    <button type="button" class="btn btn-secondary" onclick="clearForm()">🔄 クリア</button>
                </div>
            </form>
        </div>

        <!-- 材料一覧セクション -->
        <div id="list" class="section">
            <div class="stats-grid" id="statsGrid">
                <!-- 統計情報が動的に追加されます -->
            </div>

            <div class="filter-section">
                <h3 style="margin-bottom: 10px;">🔍 フィルター</h3>
                <div class="filter-grid">
                    <div class="form-group">
                        <label>材料種別</label>
                        <select id="filterType" onchange="filterMaterials()">
                            <option value="">すべて</option>
                            <option value="曲管">曲管</option>
                            <option value="直管">直管</option>
                            <option value="継手">継手</option>
                            <option value="その他">その他</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>管径</label>
                        <select id="filterDiameter" onchange="filterMaterials()">
                            <option value="">すべて</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>メーカー</label>
                        <select id="filterMaker" onchange="filterMaterials()">
                            <option value="">すべて</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>検索</label>
                        <input type="text" id="searchText" placeholder="品番、備考で検索" oninput="filterMaterials()">
                    </div>
                </div>
            </div>

            <div class="button-group">
                <button class="btn btn-success" onclick="exportToExcel()">📊 Excel出力</button>
                <button class="btn btn-secondary" onclick="exportToJSON()">💾 JSONダウンロード</button>
            </div>

            <div class="table-container">
                <table id="materialTable">
                    <thead>
                        <tr>
                            <th>種別</th>
                            <th>角度</th>
                            <th>管径<br>(mm)</th>
                            <th>有効長<br>(mm)</th>
                            <th>R<br>(mm)</th>
                            <th>入込長<br>(mm)</th>
                            <th>単価<br>(円)</th>
                            <th>メーカー</th>
                            <th>品番</th>
                            <th>操作</th>
                        </tr>
                    </thead>
                    <tbody id="materialTableBody">
                        <!-- データが動的に追加されます -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- 配管計算セクション -->
        <div id="calculate" class="section">
            <h2 style="margin-bottom: 15px;">配管ルート計算</h2>
            <p style="color: #666; margin-bottom: 20px;">2つの法線を曲管で接続するための計算を行います</p>

            <div id="calcAlertContainer"></div>

            <!-- 法線1入力 -->
            <div style="background: #f0f9ff; padding: 20px; border-radius: 8px; margin-bottom: 20px; border-left: 4px solid #0ea5e9;">
                <h3 style="color: #0369a1; margin-bottom: 15px;">📍 法線1（始点側）</h3>
                <div class="form-grid">
                    <div class="form-group">
                        <label>始点 X座標 (m)</label>
                        <input type="number" id="line1_startX" step="0.001" placeholder="0.000">
                    </div>
                    <div class="form-group">
                        <label>始点 Y座標 (m)</label>
                        <input type="number" id="line1_startY" step="0.001" placeholder="0.000">
                    </div>
                    <div class="form-group">
                        <label>始点 Z座標（標高） (m)</label>
                        <input type="number" id="line1_startZ" step="0.001" placeholder="0.000">
                    </div>
                    <div class="form-group">
                        <label>終点 X座標 (m)</label>
                        <input type="number" id="line1_endX" step="0.001" placeholder="10.000">
                    </div>
                    <div class="form-group">
                        <label>終点 Y座標 (m)</label>
                        <input type="number" id="line1_endY" step="0.001" placeholder="0.000">
                    </div>
                    <div class="form-group">
                        <label>終点 Z座標（標高） (m)</label>
                        <input type="number" id="line1_endZ" step="0.001" placeholder="-0.500">
                    </div>
                    <div class="form-group">
                        <label>方位角 (度) 自動計算</label>
                        <input type="number" id="line1_azimuth" step="0.01" readonly style="background: #f5f5f5;">
                    </div>
                    <div class="form-group">
                        <label>勾配 (%) 自動計算</label>
                        <input type="number" id="line1_gradient" step="0.01" readonly style="background: #f5f5f5;">
                    </div>
                    <div class="form-group">
                        <label>管径 (mm)</label>
                        <select id="line1_diameter">
                            <option value="">選択してください</option>
                        </select>
                    </div>
                </div>
            </div>

            <!-- 法線2入力 -->
            <div style="background: #fef3f2; padding: 20px; border-radius: 8px; margin-bottom: 20px; border-left: 4px solid #f97316;">
                <h3 style="color: #c2410c; margin-bottom: 15px;">📍 法線2（終点側）</h3>
                <div class="form-grid">
                    <div class="form-group">
                        <label>始点 X座標 (m)</label>
                        <input type="number" id="line2_startX" step="0.001" placeholder="20.000">
                    </div>
                    <div class="form-group">
                        <label>始点 Y座標 (m)</label>
                        <input type="number" id="line2_startY" step="0.001" placeholder="10.000">
                    </div>
                    <div class="form-group">
                        <label>始点 Z座標（標高） (m)</label>
                        <input type="number" id="line2_startZ" step="0.001" placeholder="-1.000">
                    </div>
                    <div class="form-group">
                        <label>終点 X座標 (m)</label>
                        <input type="number" id="line2_endX" step="0.001" placeholder="30.000">
                    </div>
                    <div class="form-group">
                        <label>終点 Y座標 (m)</label>
                        <input type="number" id="line2_endY" step="0.001" placeholder="10.000">
                    </div>
                    <div class="form-group">
                        <label>終点 Z座標（標高） (m)</label>
                        <input type="number" id="line2_endZ" step="0.001" placeholder="-1.500">
                    </div>
                    <div class="form-group">
                        <label>方位角 (度) 自動計算</label>
                        <input type="number" id="line2_azimuth" step="0.01" readonly style="background: #f5f5f5;">
                    </div>
                    <div class="form-group">
                        <label>勾配 (%) 自動計算</label>
                        <input type="number" id="line2_gradient" step="0.01" readonly style="background: #f5f5f5;">
                    </div>
                    <div class="form-group">
                        <label>管径 (mm)</label>
                        <select id="line2_diameter">
                            <option value="">選択してください</option>
                        </select>
                    </div>
                </div>
            </div>

            <!-- 計算条件 -->
            <div style="background: #f5f5f5; padding: 20px; border-radius: 8px; margin-bottom: 20px;">
                <h3 style="margin-bottom: 15px;">⚙️ 計算モード</h3>
                <div class="form-grid">
                    <div class="form-group">
                        <label>計算方法</label>
                        <select id="calculationMode" onchange="toggleCalculationMode()">
                            <option value="auto">自動最適化</option>
                            <option value="manual">曲管を手動指定</option>
                            <option value="s_curve">S字逆算（実務モード）</option>
                        </select>
                    </div>
                    <div class="form-group" id="optimizationCriteriaGroup">
                        <label>最適化基準</label>
                        <select id="optimizationCriteria">
                            <option value="cost">コスト最小</option>
                            <option value="length">距離最短</option>
                            <option value="bendCount">曲管数最小</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>最大勾配制限 (%)</label>
                        <input type="number" id="maxGradient" value="10" step="0.1">
                    </div>
                    <div class="form-group">
                        <label>最小土被り (m)</label>
                        <input type="number" id="minCover" value="0.6" step="0.1">
                    </div>
                </div>
            </div>

            <!-- S字逆算エリア -->
            <div id="sCurveArea" style="display: none; background: #e8f5e9; padding: 20px; border-radius: 8px; margin-bottom: 20px; border-left: 4px solid #4caf50;">
                <h3 style="margin-bottom: 15px;">📐 S字逆算設計（実務モード）</h3>
                <p style="color: #2e7d32; margin-bottom: 15px; font-size: 14px;">
                    中間直管の長さと使用曲管を指定すると、折れ点座標を自動計算します。<br>
                    右回りS字・左回りS字の両方の解を表示します。
                </p>
                
                <div class="form-grid" style="margin-bottom: 15px;">
                    <div class="form-group">
                        <label>曲管1を選択 *</label>
                        <select id="sCurveBend1">
                            <option value="">選択してください</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>曲管2を選択 *</label>
                        <select id="sCurveBend2">
                            <option value="">選択してください</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>中間直管の3D距離 (m) *</label>
                        <input type="number" id="middlePipeLength" step="0.001" placeholder="10.000" value="10">
                    </div>
                </div>

                <div style="background: #fff9c4; padding: 15px; border-radius: 5px; border-left: 3px solid #fbc02d; margin-top: 15px;">
                    <strong>💡 ヒント:</strong>
                    <ul style="margin: 10px 0 0 20px; font-size: 13px;">
                        <li>法線1・2の座標から自動で方向ベクトルを計算</li>
                        <li>指定した距離で折れ点座標を逆算</li>
                        <li>右回り・左回りの2パターンを表示</li>
                        <li>各区間の延長・高低差・勾配も自動計算</li>
                    </ul>
                </div>
            </div>

            <!-- 手動曲管指定エリア -->
            <div id="manualBendArea" style="display: none; background: #fff3cd; padding: 20px; border-radius: 8px; margin-bottom: 20px; border-left: 4px solid #ffc107;">
                <h3 style="margin-bottom: 15px;">🔧 曲管の手動指定</h3>
                <p style="color: #856404; margin-bottom: 15px; font-size: 14px;">
                    使用する曲管を順番に追加してください。3Dプレビューと直管延長が自動計算されます。
                </p>
                
                <div class="form-grid" style="margin-bottom: 15px;">
                    <div class="form-group">
                        <label>曲管を選択</label>
                        <select id="bendSelect">
                            <option value="">選択してください</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>回転方向</label>
                        <select id="bendDirection">
                            <option value="horizontal">水平方向</option>
                            <option value="vertical">垂直方向</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>回転軸</label>
                        <select id="bendAxis">
                            <option value="right">右回り</option>
                            <option value="left">左回り</option>
                            <option value="up">上向き</option>
                            <option value="down">下向き</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <button class="btn btn-primary" onclick="addManualBend()" style="margin-top: 20px;">➕ 曲管追加</button>
                    </div>
                </div>

                <!-- 追加された曲管リスト -->
                <div id="manualBendsList" style="margin-top: 20px;">
                    <h4 style="margin-bottom: 10px;">追加済み曲管</h4>
                    <div id="bendsListContainer"></div>
                </div>
            </div>

            <div class="button-group">
                <button class="btn btn-primary" onclick="calculatePipeRoute()">🧮 計算実行</button>
                <button class="btn btn-secondary" onclick="clearCalculation()">🔄 クリア</button>
                <button class="btn btn-success" onclick="showExample()">📝 サンプル入力</button>
            </div>

            <!-- 計算結果表示エリア -->
            <div id="calculationResults" style="display: none; margin-top: 30px;">
                <h3 style="margin-bottom: 15px;">📊 計算結果</h3>
                
                <!-- 3Dビューア -->
                <div style="background: white; padding: 20px; border-radius: 8px; border: 2px solid #ddd; margin-bottom: 20px;">
                    <h4 style="margin-bottom: 15px;">🎨 3Dプレビュー</h4>
                    <div id="viewer3D">
                        <div class="viewer-controls">
                            <button class="btn btn-small btn-secondary" onclick="resetCamera()">📷 視点リセット</button>
                            <button class="btn btn-small btn-secondary" onclick="toggleGrid()">📐 グリッド</button>
                            <button class="btn btn-small btn-secondary" onclick="toggleAxes()">🎯 軸表示</button>
                        </div>
                    </div>
                    <p style="font-size: 12px; color: #666; text-align: center;">マウスドラッグで回転、ホイールでズーム</p>
                </div>

                <!-- 複数ルート比較 -->
                <div style="background: white; padding: 20px; border-radius: 8px; border: 2px solid #ddd; margin-bottom: 20px;">
                    <h4 style="margin-bottom: 15px;">🔄 ルート比較（上位5案）</h4>
                    <div id="routeComparison">
                        <!-- 複数ルートカードが動的に表示されます -->
                    </div>
                </div>

                <div class="stats-grid" id="resultStats">
                    <!-- 統計情報が動的に表示されます -->
                </div>

                <div style="background: white; padding: 20px; border-radius: 8px; border: 2px solid #ddd; margin-bottom: 20px;">
                    <h4 style="margin-bottom: 15px;">📐 接続概要</h4>
                    <div id="connectionSummary" style="font-size: 14px;">
                        <!-- 接続概要が表示されます -->
                    </div>
                </div>

                <div class="table-container">
                    <h4 style="margin-bottom: 10px;">🔧 必要材料リスト</h4>
                    <table>
                        <thead>
                            <tr>
                                <th>No.</th>
                                <th>材料種別</th>
                                <th>角度</th>
                                <th>管径</th>
                                <th>品番</th>
                                <th>数量</th>
                                <th>単価</th>
                                <th>金額</th>
                            </tr>
                        </thead>
                        <tbody id="materialsListBody">
                            <!-- 材料リストが動的に表示されます -->
                        </tbody>
                    </table>
                </div>

                <div style="margin-top: 20px;">
                    <div class="button-group">
                        <button class="btn btn-success" onclick="exportCalculationToExcel()">📊 結果をExcel出力</button>
                        <button class="btn btn-primary" onclick="generateConstructionDrawing()">📐 施工図PDF生成</button>
                        <button class="btn btn-secondary" onclick="saveCalculation()">💾 計算結果を保存</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- データ管理セクション -->
        <div id="import" class="section">
            <h2 style="margin-bottom: 15px;">データのインポート/エクスポート</h2>
            
            <div style="margin-bottom: 30px; background: #e3f2fd; padding: 20px; border-radius: 8px; border-left: 4px solid #2196f3;">
                <h3>📸 AI承認図認識インポート</h3>
                <p style="color: #666; margin: 10px 0;">承認図の写真やPDFをアップロードすると、AIが材料データを自動認識して抽出します</p>
                <input type="file" id="approvalFileInput" accept="image/*,.pdf" style="margin-bottom: 10px;">
                <div class="button-group">
                    <button class="btn btn-primary" onclick="analyzeApprovalDocument()">🤖 AI解析実行</button>
                </div>
                <div id="analysisStatus" style="margin-top: 10px;"></div>
            </div>

            <div style="margin-bottom: 30px;">
                <h3>📥 JSONインポート</h3>
                <p style="color: #666; margin: 10px 0;">以前エクスポートしたJSONファイルを読み込みます</p>
                <input type="file" id="jsonFileInput" accept=".json" style="margin-bottom: 10px;">
                <div class="button-group">
                    <button class="btn btn-primary" onclick="importJSON()">📂 読み込み</button>
                </div>
            </div>

            <div style="margin-bottom: 30px;">
                <h3>📥 CSVインポート</h3>
                <p style="color: #666; margin: 10px 0;">CSV形式で材料を一括登録します</p>
                <textarea id="csvInput" rows="10" style="width: 100%; padding: 10px; border: 2px solid #ddd; border-radius: 5px; font-family: monospace;" placeholder="種別,角度,管径,有効長,R,入込長,単価,メーカー,品番,備考
```

曲管,45,300,450,600,120,15000,積水樹脂,ABC-300-45,
曲管,22.5,300,300,600,120,12000,積水樹脂,ABC-300-22,”></textarea>
<div class="button-group">
<button class="btn btn-primary" onclick="importCSV()">📊 CSV読み込み</button>
<button class="btn btn-secondary" onclick="downloadCSVTemplate()">📄 CSVテンプレート</button>
</div>
</div>

```
            <div style="margin-bottom: 30px;">
                <h3>🗑️ データ管理</h3>
                <div class="button-group">
                    <button class="btn btn-warning" onclick="backupData()">💾 バックアップ作成</button>
                    <button class="btn btn-danger" onclick="confirmClearAll()">🗑️ 全データ削除</button>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- AI解析結果確認モーダル -->
<div id="aiResultModal" class="modal">
    <div class="modal-content" style="max-width: 900px;">
        <div class="modal-header">AI解析結果の確認</div>
        <p style="color: #666; margin-bottom: 20px;">解析結果を確認し、必要に応じて修正してから登録してください</p>
        
        <div id="aiResultContent">
            <!-- 解析結果が動的に表示されます -->
        </div>

        <div class="button-group" style="margin-top: 20px;">
            <button class="btn btn-primary" onclick="importAIResults()">✅ すべて登録</button>
            <button class="btn btn-secondary" onclick="closeAIResultModal()">❌ キャンセル</button>
        </div>
    </div>
</div>

<!-- 編集モーダル -->
<div id="editModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">材料編集</div>
        <form id="editForm">
            <input type="hidden" id="editId">
            <div class="form-grid">
                <div class="form-group">
                    <label>材料種別</label>
                    <select id="editType">
                        <option value="曲管">曲管</option>
                        <option value="直管">直管</option>
                        <option value="継手">継手</option>
                        <option value="その他">その他</option>
                    </select>
                </div>
                <div class="form-group">
                    <label>角度 (度)</label>
                    <input type="number" id="editAngle" step="0.01">
                </div>
                <div class="form-group">
                    <label>管径 (mm)</label>
                    <input type="number" id="editDiameter">
                </div>
                <div class="form-group">
                    <label>有効長 (mm)</label>
                    <input type="number" id="editEffectiveLength">
                </div>
                <div class="form-group">
                    <label>R (mm)</label>
                    <input type="number" id="editRadius">
                </div>
                <div class="form-group">
                    <label>入込長 (mm)</label>
                    <input type="number" id="editInsertLength">
                </div>
                <div class="form-group">
                    <label>単価 (円)</label>
                    <input type="number" id="editUnitPrice">
                </div>
                <div class="form-group">
                    <label>メーカー</label>
                    <input type="text" id="editMaker">
                </div>
                <div class="form-group">
                    <label>品番</label>
                    <input type="text" id="editPartNumber">
                </div>
                <div class="form-group">
                    <label>備考</label>
                    <input type="text" id="editNotes">
                </div>
            </div>
            <div class="button-group" style="margin-top: 20px;">
                <button type="submit" class="btn btn-primary">💾 更新</button>
                <button type="button" class="btn btn-secondary" onclick="closeEditModal()">❌ キャンセル</button>
            </div>
        </form>
    </div>
</div>

<script>
    // グローバル関数として明示的にwindowに登録
    
    // データストレージ
    let materials = [];
    let editingId = null;
    let aiExtractedMaterials = [];
    let calculationResults = null;
    let allSolutions = [];
    let selectedSolutionIndex = 0;
    let manualBends = []; // 手動指定の曲管リスト
    let currentPipePath = null; // 現在の配管経路

    // 3D表示用変数
    let scene, camera, renderer, controls;
    let gridHelper, axesHelper;
    let pipeGroup;

    // 初期化
    document.addEventListener('DOMContentLoaded', function() {
        loadMaterials();
        displayMaterials();
        updateStats();
        updateDiameterOptions();
        setupCalculationListeners();
    });

    // フォールバック
    if (document.readyState !== 'loading') {
        setTimeout(function() {
            loadMaterials();
            displayMaterials();
            updateStats();
            updateDiameterOptions();
            setupCalculationListeners();
        }, 100);
    }

    // フォーム送信
    document.getElementById('materialForm').onsubmit = function(e) {
        e.preventDefault();
        
        const material = {
            id: Date.now(),
            materialType: document.getElementById('materialType').value,
            angle: parseFloat(document.getElementById('angle').value) || null,
            diameter: parseFloat(document.getElementById('diameter').value),
            effectiveLength: parseFloat(document.getElementById('effectiveLength').value),
            radius: parseFloat(document.getElementById('radius').value) || null,
            insertLength: parseFloat(document.getElementById('insertLength').value) || null,
            unitPrice: parseFloat(document.getElementById('unitPrice').value) || null,
            maker: document.getElementById('maker').value,
            partNumber: document.getElementById('partNumber').value,
            notes: document.getElementById('notes').value,
            createdAt: new Date().toISOString()
        };

        materials.push(material);
        saveMaterials();
        clearForm();
        showAlert('材料を登録しました', 'success');
    };

    // フォームクリア
    function clearForm() {
        document.getElementById('materialForm').reset();
    }

    // データ保存
    function saveMaterials() {
        localStorage.setItem('pipeMaterials', JSON.stringify(materials));
        updateDiameterOptions(); // 管径選択肢を更新
    }

    // データ読み込み
    function loadMaterials() {
        const saved = localStorage.getItem('pipeMaterials');
        if (saved) {
            materials = JSON.parse(saved);
        }
    }

    // 材料表示
    function displayMaterials() {
        const tbody = document.getElementById('materialTableBody');
        tbody.innerHTML = '';

        if (materials.length === 0) {
            tbody.innerHTML = '<tr><td colspan="10" class="empty-state">材料が登録されていません</td></tr>';
            return;
        }

        const filteredMaterials = getFilteredMaterials();

        filteredMaterials.forEach(material => {
            const row = tbody.insertRow();
            row.innerHTML = `
                <td>${material.materialType}</td>
                <td>${material.angle ? material.angle + '°' : '-'}</td>
                <td>${material.diameter}</td>
                <td>${material.effectiveLength}</td>
                <td>${material.radius || '-'}</td>
                <td>${material.insertLength || '-'}</td>
                <td>${material.unitPrice ? material.unitPrice.toLocaleString() : '-'}</td>
                <td>${material.maker || '-'}</td>
                <td>${material.partNumber || '-'}</td>
                <td class="action-buttons">
                    <button class="btn btn-primary btn-small" onclick="editMaterial(${material.id})">✏️</button>
                    <button class="btn btn-success btn-small" onclick="copyMaterial(${material.id})">📋</button>
                    <button class="btn btn-danger btn-small" onclick="deleteMaterial(${material.id})">🗑️</button>
                </td>
            `;
        });

        updateFilterOptions();
    }

    // フィルター用オプション更新
    function updateFilterOptions() {
        // 管径
        const diameters = [...new Set(materials.map(m => m.diameter))].sort((a, b) => a - b);
        const diameterSelect = document.getElementById('filterDiameter');
        const currentDiameter = diameterSelect.value;
        diameterSelect.innerHTML = '<option value="">すべて</option>';
        diameters.forEach(d => {
            const option = document.createElement('option');
            option.value = d;
            option.textContent = d + 'mm';
            diameterSelect.appendChild(option);
        });
        diameterSelect.value = currentDiameter;

        // メーカー
        const makers = [...new Set(materials.map(m => m.maker).filter(m => m))];
        const makerSelect = document.getElementById('filterMaker');
        const currentMaker = makerSelect.value;
        makerSelect.innerHTML = '<option value="">すべて</option>';
        makers.forEach(m => {
            const option = document.createElement('option');
            option.value = m;
            option.textContent = m;
            makerSelect.appendChild(option);
        });
        makerSelect.value = currentMaker;
    }

    // フィルター適用
    function getFilteredMaterials() {
        const filterType = document.getElementById('filterType').value;
        const filterDiameter = document.getElementById('filterDiameter').value;
        const filterMaker = document.getElementById('filterMaker').value;
        const searchText = document.getElementById('searchText').value.toLowerCase();

        return materials.filter(m => {
            if (filterType && m.materialType !== filterType) return false;
            if (filterDiameter && m.diameter != filterDiameter) return false;
            if (filterMaker && m.maker !== filterMaker) return false;
            if (searchText) {
                const searchTarget = (m.partNumber || '') + (m.notes || '');
                if (!searchTarget.toLowerCase().includes(searchText)) return false;
            }
            return true;
        });
    }

    function filterMaterials() {
        displayMaterials();
    }

    // 統計情報更新
    function updateStats() {
        const statsGrid = document.getElementById('statsGrid');
        const totalCount = materials.length;
        const bendCount = materials.filter(m => m.materialType === '曲管').length;
        const avgPrice = materials.filter(m => m.unitPrice).length > 0 
            ? Math.round(materials.filter(m => m.unitPrice).reduce((sum, m) => sum + m.unitPrice, 0) / materials.filter(m => m.unitPrice).length)
            : 0;

        statsGrid.innerHTML = `
            <div class="stat-card">
                <h3>${totalCount}</h3>
                <p>総材料数</p>
            </div>
            <div class="stat-card">
                <h3>${bendCount}</h3>
                <p>曲管種類数</p>
            </div>
            <div class="stat-card">
                <h3>¥${avgPrice.toLocaleString()}</h3>
                <p>平均単価</p>
            </div>
        `;
    }

    // 材料編集
    function editMaterial(id) {
        const material = materials.find(m => m.id === id);
        if (!material) return;

        document.getElementById('editId').value = material.id;
        document.getElementById('editType').value = material.materialType;
        document.getElementById('editAngle').value = material.angle || '';
        document.getElementById('editDiameter').value = material.diameter;
        document.getElementById('editEffectiveLength').value = material.effectiveLength;
        document.getElementById('editRadius').value = material.radius || '';
        document.getElementById('editInsertLength').value = material.insertLength || '';
        document.getElementById('editUnitPrice').value = material.unitPrice || '';
        document.getElementById('editMaker').value = material.maker || '';
        document.getElementById('editPartNumber').value = material.partNumber || '';
        document.getElementById('editNotes').value = material.notes || '';

        document.getElementById('editModal').classList.add('active');
    }

    // 編集フォーム送信
    document.getElementById('editForm').onsubmit = function(e) {
        e.preventDefault();
        
        const id = parseInt(document.getElementById('editId').value);
        const index = materials.findIndex(m => m.id === id);
        
        if (index !== -1) {
            materials[index] = {
                ...materials[index],
                materialType: document.getElementById('editType').value,
                angle: parseFloat(document.getElementById('editAngle').value) || null,
                diameter: parseFloat(document.getElementById('editDiameter').value),
                effectiveLength: parseFloat(document.getElementById('editEffectiveLength').value),
                radius: parseFloat(document.getElementById('editRadius').value) || null,
                insertLength: parseFloat(document.getElementById('editInsertLength').value) || null,
                unitPrice: parseFloat(document.getElementById('editUnitPrice').value) || null,
                maker: document.getElementById('editMaker').value,
                partNumber: document.getElementById('editPartNumber').value,
                notes: document.getElementById('editNotes').value,
                updatedAt: new Date().toISOString()
            };

            saveMaterials();
            displayMaterials();
            updateStats();
            closeEditModal();
            showAlert('材料を更新しました', 'success');
        }
    };

    function closeEditModal() {
        document.getElementById('editModal').classList.remove('active');
    }

    // 材料コピー
    function copyMaterial(id) {
        const material = materials.find(m => m.id === id);
        if (!material) return;

        const newMaterial = {
            ...material,
            id: Date.now(),
            partNumber: material.partNumber + ' (コピー)',
            createdAt: new Date().toISOString()
        };

        materials.push(newMaterial);
        saveMaterials();
        displayMaterials();
        updateStats();
        showAlert('材料を複製しました', 'success');
    }

    // 材料削除
    function deleteMaterial(id) {
        if (!confirm('この材料を削除してもよろしいですか?')) return;

        materials = materials.filter(m => m.id !== id);
        saveMaterials();
        displayMaterials();
        updateStats();
        showAlert('材料を削除しました', 'success');
    }

    // Excel出力
    function exportToExcel() {
        let csv = '材料種別,角度,管径(mm),有効長(mm),R(mm),入込長(mm),単価(円),メーカー,品番,備考\n';
        
        materials.forEach(m => {
            csv += `${m.materialType},${m.angle || ''},${m.diameter},${m.effectiveLength},${m.radius || ''},${m.insertLength || ''},${m.unitPrice || ''},${m.maker || ''},${m.partNumber || ''},${m.notes || ''}\n`;
        });

        const blob = new Blob(['\uFEFF' + csv], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = `材料マスター_${new Date().toISOString().slice(0,10)}.csv`;
        link.click();
    }

    // JSON出力
    function exportToJSON() {
        const dataStr = JSON.stringify(materials, null, 2);
        const blob = new Blob([dataStr], { type: 'application/json' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = `材料マスター_${new Date().toISOString().slice(0,10)}.json`;
        link.click();
    }

    // JSONインポート
    function importJSON() {
        const fileInput = document.getElementById('jsonFileInput');
        const file = fileInput.files[0];
        
        if (!file) {
            alert('ファイルを選択してください');
            return;
        }

        const reader = new FileReader();
        reader.onload = function(e) {
            try {
                const imported = JSON.parse(e.target.result);
                if (Array.isArray(imported)) {
                    if (confirm(`${imported.length}件のデータをインポートします。既存データは上書きされます。よろしいですか?`)) {
                        materials = imported;
                        saveMaterials();
                        displayMaterials();
                        updateStats();
                        showAlert(`${imported.length}件のデータをインポートしました`, 'success');
                    }
                } else {
                    alert('無効なJSONファイルです');
                }
            } catch (error) {
                alert('ファイルの読み込みに失敗しました: ' + error.message);
            }
        };
        reader.readAsText(file);
    }

    // CSVインポート
    function importCSV() {
        const csvText = document.getElementById('csvInput').value.trim();
        if (!csvText) {
            alert('CSVデータを入力してください');
            return;
        }

        const lines = csvText.split('\n');
        const imported = [];
        
        // ヘッダー行をスキップ
        for (let i = 1; i < lines.length; i++) {
            const line = lines[i].trim();
            if (!line) continue;

            const cols = line.split(',');
            if (cols.length >= 3) {
                imported.push({
                    id: Date.now() + i,
                    materialType: cols[0],
                    angle: parseFloat(cols[1]) || null,
                    diameter: parseFloat(cols[2]),
                    effectiveLength: parseFloat(cols[3]),
                    radius: parseFloat(cols[4]) || null,
                    insertLength: parseFloat(cols[5]) || null,
                    unitPrice: parseFloat(cols[6]) || null,
                    maker: cols[7] || '',
                    partNumber: cols[8] || '',
                    notes: cols[9] || '',
                    createdAt: new Date().toISOString()
                });
            }
        }

        if (imported.length > 0) {
            materials = materials.concat(imported);
            saveMaterials();
            displayMaterials();
            updateStats();
            document.getElementById('csvInput').value = '';
            showAlert(`${imported.length}件のデータをインポートしました`, 'success');
        } else {
            alert('有効なデータが見つかりませんでした');
        }
    }

    // CSVテンプレートダウンロード
    function downloadCSVTemplate() {
        const template = `材料種別,角度,管径(mm),有効長(mm),R(mm),入込長(mm),単価(円),メーカー,品番,備考
```

曲管,45,300,450,600,120,15000,積水樹脂,ABC-300-45,
曲管,22.5,300,300,600,120,12000,積水樹脂,ABC-300-22,
直管,,300,4000,,,8000,積水樹脂,STR-300,`;

```
        const blob = new Blob(['\uFEFF' + template], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = 'CSVテンプレート.csv';
        link.click();
    }

    // バックアップ
    function backupData() {
        exportToJSON();
        showAlert('バックアップを作成しました', 'success');
    }

    // 全データ削除確認
    function confirmClearAll() {
        if (confirm('すべてのデータを削除してもよろしいですか?\nこの操作は取り消せません。')) {
            if (confirm('本当に削除しますか?')) {
                materials = [];
                saveMaterials();
                displayMaterials();
                updateStats();
                showAlert('すべてのデータを削除しました', 'success');
            }
        }
    }

    // アラート表示
    function showAlert(message, type = 'success') {
        const alertContainer = document.getElementById('alertContainer');
        const alertClass = type === 'success' ? 'alert-success' : 'alert-error';
        
        const alert = document.createElement('div');
        alert.className = `alert ${alertClass}`;
        alert.textContent = message;
        
        alertContainer.innerHTML = '';
        alertContainer.appendChild(alert);

        setTimeout(() => {
            alert.remove();
        }, 3000);
    }

    // モーダル外クリックで閉じる
    document.getElementById('editModal').onclick = function(e) {
        if (e.target === this) {
            closeEditModal();
        }
    };

    // AI承認図認識機能
    async function analyzeApprovalDocument() {
        const fileInput = document.getElementById('approvalFileInput');
        const file = fileInput.files[0];
        
        if (!file) {
            alert('ファイルを選択してください');
            return;
        }

        const statusDiv = document.getElementById('analysisStatus');
        statusDiv.innerHTML = '<div class="alert alert-success">🤖 AI解析中...しばらくお待ちください</div>';

        try {
            // ファイルをBase64に変換
            const base64Data = await fileToBase64(file);
            
            // ファイルタイプを判定
            const mediaType = file.type === 'application/pdf' 
                ? 'application/pdf' 
                : file.type || 'image/jpeg';

            // Claude APIで解析
            const extractedData = await analyzeWithClaude(base64Data, mediaType);
            
            if (extractedData && extractedData.length > 0) {
                aiExtractedMaterials = extractedData;
                displayAIResults(extractedData);
                statusDiv.innerHTML = `<div class="alert alert-success">✅ ${extractedData.length}件の材料データを抽出しました</div>`;
            } else {
                statusDiv.innerHTML = '<div class="alert alert-error">❌ 材料データが見つかりませんでした</div>';
            }
        } catch (error) {
            console.error('解析エラー:', error);
            statusDiv.innerHTML = `<div class="alert alert-error">❌ 解析に失敗しました: ${error.message}</div>`;
        }
    }

    // ファイルをBase64に変換
    function fileToBase64(file) {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.onload = () => {
                const base64 = reader.result.split(',')[1];
                resolve(base64);
            };
            reader.onerror = () => reject(new Error('ファイルの読み込みに失敗しました'));
            reader.readAsDataURL(file);
        });
    }

    // Claude APIで承認図を解析
    async function analyzeWithClaude(base64Data, mediaType) {
        const isPDF = mediaType === 'application/pdf';
        
        const content = [
            {
                type: isPDF ? "document" : "image",
                source: {
                    type: "base64",
                    media_type: mediaType,
                    data: base64Data
                }
            },
            {
                type: "text",
                text: `この配管材料の承認図から、以下の情報を抽出してください：
```

【抽出項目】

- 材料種別（曲管、直管、継手など）
- 曲管角度（度）
- 管径（mm）
- 有効長（mm）
- 曲率半径R（mm）
- 入り込み長さ（mm）
- 単価（円）
- メーカー名
- 品番
- 備考

【出力形式】
必ずJSON配列形式で出力してください。他の説明は一切不要です。

[
{
“materialType”: “曲管”,
“angle”: 45,
“diameter”: 300,
“effectiveLength”: 450,
“radius”: 600,
“insertLength”: 120,
“unitPrice”: 15000,
“maker”: “積水樹脂”,
“partNumber”: “ABC-300-45”,
“notes”: “”
}
]

重要事項：

- 数値はすべて数字型で出力
- データが不明な項目はnullを設定
- 必ずJSON配列のみを出力（説明文やマークダウンは不要）
- バッククォートやコードブロックも不要`
  }
  ];
  
  ```
        const response = await fetch("https://api.anthropic.com/v1/messages", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                model: "claude-sonnet-4-20250514",
                max_tokens: 4000,
                messages: [
                    { role: "user", content: content }
                ]
            })
        });
  
        if (!response.ok) {
            throw new Error(`API Error: ${response.status}`);
        }
  
        const data = await response.json();
        const responseText = data.content[0].text;
        
        // JSONをパース（マークダウンのコードブロックを除去）
        let jsonText = responseText.trim();
        jsonText = jsonText.replace(/```json\n?/g, '').replace(/```\n?/g, '').trim();
        
        try {
            const parsed = JSON.parse(jsonText);
            return Array.isArray(parsed) ? parsed : [parsed];
        } catch (e) {
            console.error('JSON parse error:', e, jsonText);
            throw new Error('AI応答のパースに失敗しました');
        }
    }
  
    // AI解析結果を表示
    function displayAIResults(results) {
        const content = document.getElementById('aiResultContent');
        
        let html = '<div class="table-container"><table><thead><tr>';
        html += '<th>種別</th><th>角度</th><th>管径<br>(mm)</th><th>有効長<br>(mm)</th>';
        html += '<th>R<br>(mm)</th><th>入込長<br>(mm)</th><th>単価<br>(円)</th>';
        html += '<th>メーカー</th><th>品番</th><th>操作</th>';
        html += '</tr></thead><tbody>';
  
        results.forEach((item, index) => {
            html += `<tr id="aiRow${index}">
                <td><input type="text" value="${item.materialType || ''}" onchange="updateAIResult(${index}, 'materialType', this.value)" style="width:80px;"></td>
                <td><input type="number" value="${item.angle || ''}" onchange="updateAIResult(${index}, 'angle', this.value)" style="width:60px;"></td>
                <td><input type="number" value="${item.diameter || ''}" onchange="updateAIResult(${index}, 'diameter', this.value)" style="width:60px;"></td>
                <td><input type="number" value="${item.effectiveLength || ''}" onchange="updateAIResult(${index}, 'effectiveLength', this.value)" style="width:60px;"></td>
                <td><input type="number" value="${item.radius || ''}" onchange="updateAIResult(${index}, 'radius', this.value)" style="width:60px;"></td>
                <td><input type="number" value="${item.insertLength || ''}" onchange="updateAIResult(${index}, 'insertLength', this.value)" style="width:60px;"></td>
                <td><input type="number" value="${item.unitPrice || ''}" onchange="updateAIResult(${index}, 'unitPrice', this.value)" style="width:80px;"></td>
                <td><input type="text" value="${item.maker || ''}" onchange="updateAIResult(${index}, 'maker', this.value)" style="width:100px;"></td>
                <td><input type="text" value="${item.partNumber || ''}" onchange="updateAIResult(${index}, 'partNumber', this.value)" style="width:100px;"></td>
                <td><button class="btn btn-danger btn-small" onclick="removeAIResult(${index})">削除</button></td>
            </tr>`;
        });
  
        html += '</tbody></table></div>';
        content.innerHTML = html;
  
        document.getElementById('aiResultModal').classList.add('active');
    }
  
    // AI解析結果の更新
    function updateAIResult(index, field, value) {
        if (aiExtractedMaterials[index]) {
            if (field === 'angle' || field === 'diameter' || field === 'effectiveLength' || 
                field === 'radius' || field === 'insertLength' || field === 'unitPrice') {
                aiExtractedMaterials[index][field] = parseFloat(value) || null;
            } else {
                aiExtractedMaterials[index][field] = value;
            }
        }
    }
  
    // AI解析結果の削除
    function removeAIResult(index) {
        document.getElementById(`aiRow${index}`).remove();
        aiExtractedMaterials[index] = null;
    }
  
    // AI解析結果を一括登録
    function importAIResults() {
        const validResults = aiExtractedMaterials.filter(item => item !== null && item.diameter);
        
        if (validResults.length === 0) {
            alert('登録するデータがありません');
            return;
        }
  
        validResults.forEach(item => {
            materials.push({
                id: Date.now() + Math.random(),
                materialType: item.materialType || '曲管',
                angle: item.angle,
                diameter: item.diameter,
                effectiveLength: item.effectiveLength,
                radius: item.radius,
                insertLength: item.insertLength,
                unitPrice: item.unitPrice,
                maker: item.maker || '',
                partNumber: item.partNumber || '',
                notes: item.notes || 'AI抽出',
                createdAt: new Date().toISOString()
            });
        });
  
        saveMaterials();
        closeAIResultModal();
        
        // 一覧タブに切り替え
        document.querySelectorAll('.tab')[1].click();
        
        showAlert(`${validResults.length}件の材料を登録しました`, 'success');
        
        // 入力をクリア
        document.getElementById('approvalFileInput').value = '';
        document.getElementById('analysisStatus').innerHTML = '';
        aiExtractedMaterials = [];
    }
  
    // AI結果モーダルを閉じる
    function closeAIResultModal() {
        document.getElementById('aiResultModal').classList.remove('active');
    }
  
    // AI結果モーダルの外クリックで閉じる
    document.getElementById('aiResultModal').onclick = function(e) {
        if (e.target === this) {
            closeAIResultModal();
        }
    };
  
    // ========================================
    // 配管計算エンジン
    // ========================================
  
    // 管径選択肢を更新
    function updateDiameterOptions() {
        const diameters = [...new Set(materials.map(m => m.diameter))].sort((a, b) => a - b);
        const selects = ['line1_diameter', 'line2_diameter'];
        
        selects.forEach(selectId => {
            const select = document.getElementById(selectId);
            const currentValue = select.value;
            select.innerHTML = '<option value="">選択してください</option>';
            diameters.forEach(d => {
                const option = document.createElement('option');
                option.value = d;
                option.textContent = d + 'mm';
                select.appendChild(option);
            });
            select.value = currentValue;
        });
  
        // 曲管選択肢も更新
        updateBendSelectOptions();
    }
  
    // 曲管選択肢を更新
    function updateBendSelectOptions() {
        const bendSelect = document.getElementById('bendSelect');
        if (!bendSelect) return;
  
        const currentDiameter = document.getElementById('line1_diameter').value;
        bendSelect.innerHTML = '<option value="">選択してください</option>';
  
        if (!currentDiameter) return;
  
        const bends = materials.filter(m => 
            m.materialType === '曲管' && 
            m.diameter == currentDiameter &&
            m.angle !== null
        );
  
        bends.forEach(bend => {
            const option = document.createElement('option');
            option.value = bend.id;
            option.textContent = `${bend.angle}° - ${bend.partNumber || ''} (R=${bend.radius || '-'}mm)`;
            bendSelect.appendChild(option);
        });
  
        // S字モード用の曲管選択肢も更新
        updateSCurveBendOptions();
    }
  
    // S字モード用の曲管選択肢を更新
    function updateSCurveBendOptions() {
        const bend1Select = document.getElementById('sCurveBend1');
        const bend2Select = document.getElementById('sCurveBend2');
        if (!bend1Select || !bend2Select) return;
  
        const currentDiameter = document.getElementById('line1_diameter').value;
        
        [bend1Select, bend2Select].forEach(select => {
            const currentValue = select.value;
            select.innerHTML = '<option value="">選択してください</option>';
  
            if (!currentDiameter) return;
  
            const bends = materials.filter(m => 
                m.materialType === '曲管' && 
                m.diameter == currentDiameter &&
                m.angle !== null
            );
  
            bends.forEach(bend => {
                const option = document.createElement('option');
                option.value = bend.id;
                option.textContent = `${bend.angle}° - ${bend.partNumber || ''} (R=${bend.radius || '-'}mm, L=${bend.effectiveLength || '-'}mm)`;
                select.appendChild(option);
            });
  
            select.value = currentValue;
        });
    }
  
    // 計算画面のリスナー設定
    function setupCalculationListeners() {
        // 座標入力時に方位角と勾配を自動計算
        const line1Inputs = ['line1_startX', 'line1_startY', 'line1_startZ', 'line1_endX', 'line1_endY', 'line1_endZ'];
        const line2Inputs = ['line2_startX', 'line2_startY', 'line2_startZ', 'line2_endX', 'line2_endY', 'line2_endZ'];
        
        line1Inputs.forEach(id => {
            const input = document.getElementById(id);
            if (input) {
                input.addEventListener('input', () => calculateLineProperties(1));
            }
        });
        
        line2Inputs.forEach(id => {
            const input = document.getElementById(id);
            if (input) {
                input.addEventListener('input', () => calculateLineProperties(2));
            }
        });
  
        // 管径変更時に曲管選択肢を更新
        const diameterSelects = ['line1_diameter', 'line2_diameter'];
        diameterSelects.forEach(id => {
            const select = document.getElementById(id);
            if (select) {
                select.addEventListener('change', updateBendSelectOptions);
            }
        });
    }
  
    // 計算モード切替
    function toggleCalculationMode() {
        const mode = document.getElementById('calculationMode').value;
        const manualArea = document.getElementById('manualBendArea');
        const sCurveArea = document.getElementById('sCurveArea');
        const optimizationGroup = document.getElementById('optimizationCriteriaGroup');
  
        // 全部非表示
        manualArea.style.display = 'none';
        sCurveArea.style.display = 'none';
        optimizationGroup.style.display = 'block';
  
        if (mode === 'manual') {
            manualArea.style.display = 'block';
            optimizationGroup.style.display = 'none';
        } else if (mode === 's_curve') {
            sCurveArea.style.display = 'block';
            optimizationGroup.style.display = 'none';
            updateSCurveBendOptions();
        }
    }
  
    // 手動で曲管を追加
    function addManualBend() {
        const bendId = document.getElementById('bendSelect').value;
        const direction = document.getElementById('bendDirection').value;
        const axis = document.getElementById('bendAxis').value;
  
        if (!bendId) {
            showCalcAlert('曲管を選択してください', 'error');
            return;
        }
  
        const bend = materials.find(m => m.id == bendId);
        if (!bend) return;
  
        manualBends.push({
            material: bend,
            direction: direction,
            axis: axis,
            id: Date.now()
        });
  
        displayManualBendsList();
        calculateManualRoute();
        showCalcAlert('曲管を追加しました', 'success');
    }
  
    // 手動曲管リストを表示
    function displayManualBendsList() {
        const container = document.getElementById('bendsListContainer');
        
        if (manualBends.length === 0) {
            container.innerHTML = '<p style="color: #666; font-size: 14px;">まだ曲管が追加されていません</p>';
            return;
        }
  
        let html = '<div style="display: flex; flex-direction: column; gap: 10px;">';
        
        manualBends.forEach((bend, index) => {
            const directionText = bend.direction === 'horizontal' ? '水平' : '垂直';
            const axisText = {
                'right': '右回り',
                'left': '左回り',
                'up': '上向き',
                'down': '下向き'
            }[bend.axis];
  
            html += `
                <div style="background: white; padding: 15px; border-radius: 5px; border: 2px solid #ddd; display: flex; justify-content: space-between; align-items: center;">
                    <div>
                        <strong>${index + 1}. ${bend.material.angle}°曲管</strong><br>
                        <span style="font-size: 12px; color: #666;">
                            ${bend.material.partNumber || '-'} | 
                            R=${bend.material.radius || '-'}mm | 
                            ${directionText} ${axisText}
                        </span>
                    </div>
                    <div style="display: flex; gap: 5px;">
                        <button class="btn btn-small btn-secondary" onclick="moveBendUp(${index})">↑</button>
                        <button class="btn btn-small btn-secondary" onclick="moveBendDown(${index})">↓</button>
                        <button class="btn btn-small btn-danger" onclick="removeManualBend(${index})">削除</button>
                    </div>
                </div>
            `;
        });
  
        html += '</div>';
        container.innerHTML = html;
    }
  
    // 手動曲管を削除
    function removeManualBend(index) {
        manualBends.splice(index, 1);
        displayManualBendsList();
        calculateManualRoute();
    }
  
    // 曲管を上に移動
    function moveBendUp(index) {
        if (index === 0) return;
        [manualBends[index], manualBends[index - 1]] = [manualBends[index - 1], manualBends[index]];
        displayManualBendsList();
        calculateManualRoute();
    }
  
    // 曲管を下に移動
    function moveBendDown(index) {
        if (index === manualBends.length - 1) return;
        [manualBends[index], manualBends[index + 1]] = [manualBends[index + 1], manualBends[index]];
        displayManualBendsList();
        calculateManualRoute();
    }
  
    // 法線の方位角と勾配を計算
    function calculateLineProperties(lineNum) {
        const prefix = 'line' + lineNum + '_';
        const startX = parseFloat(document.getElementById(prefix + 'startX').value) || 0;
        const startY = parseFloat(document.getElementById(prefix + 'startY').value) || 0;
        const startZ = parseFloat(document.getElementById(prefix + 'startZ').value) || 0;
        const endX = parseFloat(document.getElementById(prefix + 'endX').value) || 0;
        const endY = parseFloat(document.getElementById(prefix + 'endY').value) || 0;
        const endZ = parseFloat(document.getElementById(prefix + 'endZ').value) || 0;
  
        // 水平距離
        const dx = endX - startX;
        const dy = endY - startY;
        const dz = endZ - startZ;
        const horizontalDist = Math.sqrt(dx * dx + dy * dy);
  
        // 方位角（北から時計回り、度）
        let azimuth = Math.atan2(dx, dy) * 180 / Math.PI;
        if (azimuth < 0) azimuth += 360;
  
        // 勾配（%）
        const gradient = horizontalDist > 0 ? (dz / horizontalDist) * 100 : 0;
  
        document.getElementById(prefix + 'azimuth').value = azimuth.toFixed(2);
        document.getElementById(prefix + 'gradient').value = gradient.toFixed(2);
    }
  
    // サンプルデータを入力
    function showExample() {
        const mode = document.getElementById('calculationMode').value;
  
        document.getElementById('line1_startX').value = 0;
        document.getElementById('line1_startY').value = 0;
        document.getElementById('line1_startZ').value = 100;
        document.getElementById('line1_endX').value = 10;
        document.getElementById('line1_endY').value = 0;
        document.getElementById('line1_endZ').value = 99.5;
  
        document.getElementById('line2_startX').value = 20;
        document.getElementById('line2_startY').value = 10;
        document.getElementById('line2_startZ').value = 99;
        document.getElementById('line2_endX').value = 30;
        document.getElementById('line2_endY').value = 10;
        document.getElementById('line2_endZ').value = 98.5;
  
        // 管径を設定（材料マスターにあれば）
        if (materials.length > 0) {
            const diameter = materials[0].diameter;
            document.getElementById('line1_diameter').value = diameter;
            document.getElementById('line2_diameter').value = diameter;
        }
  
        calculateLineProperties(1);
        calculateLineProperties(2);
  
        // S字モードの場合は中間直管の距離も設定
        if (mode === 's_curve') {
            document.getElementById('middlePipeLength').value = 10;
            updateSCurveBendOptions();
            
            // 45度曲管があれば自動選択
            const bend45 = materials.find(m => m.materialType === '曲管' && m.angle === 45);
            if (bend45) {
                document.getElementById('sCurveBend1').value = bend45.id;
                document.getElementById('sCurveBend2').value = bend45.id;
            }
        }
  
        showCalcAlert('サンプルデータを入力しました', 'success');
    }
  
    // 配管ルート計算のメイン関数
    function calculatePipeRoute() {
        const mode = document.getElementById('calculationMode').value;
  
        if (mode === 'manual') {
            calculateManualRoute(true);
        } else if (mode === 's_curve') {
            calculateSCurveRoute();
        } else {
            calculateAutoRoute();
        }
    }
  
    // 自動計算ルート
    function calculateAutoRoute() {
        // 入力値の取得
        const line1 = {
            startX: parseFloat(document.getElementById('line1_startX').value),
            startY: parseFloat(document.getElementById('line1_startY').value),
            startZ: parseFloat(document.getElementById('line1_startZ').value),
            endX: parseFloat(document.getElementById('line1_endX').value),
            endY: parseFloat(document.getElementById('line1_endY').value),
            endZ: parseFloat(document.getElementById('line1_endZ').value),
            azimuth: parseFloat(document.getElementById('line1_azimuth').value),
            gradient: parseFloat(document.getElementById('line1_gradient').value),
            diameter: parseFloat(document.getElementById('line1_diameter').value)
        };
  
        const line2 = {
            startX: parseFloat(document.getElementById('line2_startX').value),
            startY: parseFloat(document.getElementById('line2_startY').value),
            startZ: parseFloat(document.getElementById('line2_startZ').value),
            endX: parseFloat(document.getElementById('line2_endX').value),
            endY: parseFloat(document.getElementById('line2_endY').value),
            endZ: parseFloat(document.getElementById('line2_endZ').value),
            azimuth: parseFloat(document.getElementById('line2_azimuth').value),
            gradient: parseFloat(document.getElementById('line2_gradient').value),
            diameter: parseFloat(document.getElementById('line2_diameter').value)
        };
  
        // バリデーション
        if (isNaN(line1.diameter) || isNaN(line2.diameter)) {
            showCalcAlert('管径を選択してください', 'error');
            return;
        }
  
        if (line1.diameter !== line2.diameter) {
            showCalcAlert('現在のバージョンでは、両方の法線の管径を同じにする必要があります', 'error');
            return;
        }
  
        const optimizationCriteria = document.getElementById('optimizationCriteria').value;
  
        // 平面交角を計算
        let planarAngle = Math.abs(line2.azimuth - line1.azimuth);
        if (planarAngle > 180) planarAngle = 360 - planarAngle;
  
        // 高低差を計算
        const heightDiff = line2.startZ - line1.endZ;
  
        // 水平距離を計算
        const dx = line2.startX - line1.endX;
        const dy = line2.startY - line1.endY;
        const horizontalDist = Math.sqrt(dx * dx + dy * dy);
  
        // 最適な曲管の組み合わせを探索
        const solutions = findBendCombinations(planarAngle, heightDiff, horizontalDist, line1.diameter, optimizationCriteria);
  
        if (solutions.length === 0) {
            showCalcAlert('接続可能な曲管の組み合わせが見つかりませんでした', 'error');
            return;
        }
  
        // 最適解を表示
        const bestSolution = solutions[0];
        calculationResults = {
            line1,
            line2,
            planarAngle,
            heightDiff,
            horizontalDist,
            solution: bestSolution
        };
  
        allSolutions = solutions;
        selectedSolutionIndex = 0;
  
        displayCalculationResults();
        init3DViewer();
        render3DScene();
        showCalcAlert('計算が完了しました', 'success');
    }
  
    // 曲管の組み合わせを探索
    function findBendCombinations(planarAngle, heightDiff, horizontalDist, diameter, criteria) {
        const solutions = [];
        
        // 使用可能な曲管を取得
        const availableBends = materials.filter(m => 
            m.materialType === '曲管' && 
            m.diameter === diameter &&
            m.angle !== null
        );
  
        if (availableBends.length === 0) {
            return solutions;
        }
  
        // 曲管角度でソート
        availableBends.sort((a, b) => b.angle - a.angle);
  
        // 平面角度を解決する組み合わせを探索
        const planarCombinations = findAngleCombination(planarAngle, availableBends);
  
        planarCombinations.forEach(planarCombo => {
            // 縦断角度を解決する組み合わせを探索（簡易版）
            const verticalAngle = Math.abs(Math.atan(heightDiff / horizontalDist) * 180 / Math.PI);
            const verticalCombinations = findAngleCombination(verticalAngle, availableBends);
  
            verticalCombinations.forEach(verticalCombo => {
                // 材料リストを作成
                const materialsList = [];
                const addedBends = {};
  
                [...planarCombo, ...verticalCombo].forEach(bend => {
                    const key = `${bend.angle}_${bend.diameter}`;
                    if (!addedBends[key]) {
                        addedBends[key] = {
                            material: bend,
                            quantity: 0,
                            purpose: []
                        };
                    }
                    addedBends[key].quantity++;
                    addedBends[key].purpose.push(planarCombo.includes(bend) ? '平面' : '縦断');
                });
  
                Object.values(addedBends).forEach(item => {
                    materialsList.push({
                        material: item.material,
                        quantity: item.quantity,
                        purpose: item.purpose.join('+')
                    });
                });
  
                // コスト計算
                const totalCost = materialsList.reduce((sum, item) => {
                    return sum + (item.material.unitPrice || 0) * item.quantity;
                }, 0);
  
                // 総延長計算（簡易）
                const totalLength = materialsList.reduce((sum, item) => {
                    return sum + (item.material.effectiveLength || 0) * item.quantity;
                }, 0);
  
                solutions.push({
                    planarBends: planarCombo,
                    verticalBends: verticalCombo,
                    materialsList,
                    totalCost,
                    totalLength,
                    bendCount: materialsList.reduce((sum, item) => sum + item.quantity, 0)
                });
            });
        });
  
        // 最適化基準でソート
        if (criteria === 'cost') {
            solutions.sort((a, b) => a.totalCost - b.totalCost);
        } else if (criteria === 'length') {
            solutions.sort((a, b) => a.totalLength - b.totalLength);
        } else if (criteria === 'bendCount') {
            solutions.sort((a, b) => a.bendCount - b.bendCount);
        }
  
        return solutions.slice(0, 5); // 上位5件を返す
    }
  
    // 角度を曲管の組み合わせで実現
    function findAngleCombination(targetAngle, availableBends, maxBends = 5) {
        const combinations = [];
        const tolerance = 1.0; // 許容誤差 1度
  
        // 再帰的に組み合わせを探索
        function search(currentAngle, usedBends, depth) {
            if (depth > maxBends) return;
  
            if (Math.abs(currentAngle - targetAngle) <= tolerance) {
                combinations.push([...usedBends]);
                return;
            }
  
            if (currentAngle > targetAngle + tolerance) return;
  
            for (const bend of availableBends) {
                if (currentAngle + bend.angle <= targetAngle + tolerance) {
                    search(currentAngle + bend.angle, [...usedBends, bend], depth + 1);
                }
            }
        }
  
        search(0, [], 0);
  
        // 曲管数が少ない順にソート
        combinations.sort((a, b) => a.length - b.length);
  
        return combinations.slice(0, 10); // 上位10件
    }
  
    // 計算結果を表示
    function displayCalculationResults() {
        const result = calculationResults;
        document.getElementById('calculationResults').style.display = 'block';
  
        // 複数ルート比較を表示
        displayRouteComparison();
  
        // 統計情報
        const statsHTML = `
            <div class="stat-card" style="background: linear-gradient(135deg, #10b981 0%, #059669 100%);">
                <h3>${result.solution.bendCount}</h3>
                <p>必要曲管数</p>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);">
                <h3>${(result.solution.totalLength / 1000).toFixed(2)}m</h3>
                <p>総延長</p>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);">
                <h3>¥${result.solution.totalCost.toLocaleString()}</h3>
                <p>概算金額</p>
            </div>
        `;
        document.getElementById('resultStats').innerHTML = statsHTML;
  
        // 接続概要
        const summaryHTML = `
            <p><strong>平面交角:</strong> ${result.planarAngle.toFixed(2)}°</p>
            <p><strong>高低差:</strong> ${result.heightDiff.toFixed(3)}m</p>
            <p><strong>水平距離:</strong> ${result.horizontalDist.toFixed(3)}m</p>
            <p><strong>管径:</strong> φ${result.line1.diameter}mm</p>
        `;
        document.getElementById('connectionSummary').innerHTML = summaryHTML;
  
        // 材料リスト
        displayMaterialsList();
  
        // 結果エリアまでスクロール
        setTimeout(() => {
            document.getElementById('calculationResults').scrollIntoView({ behavior: 'smooth' });
        }, 100);
    }
  
    // 複数ルート比較を表示
    function displayRouteComparison() {
        const container = document.getElementById('routeComparison');
        container.innerHTML = '';
  
        allSolutions.forEach((solution, index) => {
            const card = document.createElement('div');
            card.className = 'route-card' + (index === selectedSolutionIndex ? ' selected' : '');
            card.onclick = () => selectSolution(index);
  
            const optimizationCriteria = document.getElementById('optimizationCriteria').value;
            let badgeText = '案 ' + (index + 1);
            if (index === 0) {
                if (optimizationCriteria === 'cost') badgeText = '最安値';
                else if (optimizationCriteria === 'length') badgeText = '最短距離';
                else if (optimizationCriteria === 'bendCount') badgeText = '最少曲管';
            }
  
            card.innerHTML = `
                <div class="route-header">
                    <span class="route-badge">${badgeText}</span>
                    <span style="font-size: 12px; color: #666;">クリックで選択</span>
                </div>
                <div class="route-stats">
                    <div class="route-stat-item">
                        <div class="route-stat-value">${solution.bendCount}</div>
                        <div class="route-stat-label">曲管数</div>
                    </div>
                    <div class="route-stat-item">
                        <div class="route-stat-value">${(solution.totalLength / 1000).toFixed(1)}m</div>
                        <div class="route-stat-label">総延長</div>
                    </div>
                    <div class="route-stat-item">
                        <div class="route-stat-value">¥${(solution.totalCost / 1000).toFixed(0)}k</div>
                        <div class="route-stat-label">概算金額</div>
                    </div>
                </div>
            `;
  
            container.appendChild(card);
        });
    }
  
    // ルートを選択
    function selectSolution(index) {
        selectedSolutionIndex = index;
        calculationResults.solution = allSolutions[index];
        displayCalculationResults();
        render3DScene();
    }
  
    // 材料リストを表示
    function displayMaterialsList() {
        const result = calculationResults;
        const tbody = document.getElementById('materialsListBody');
        tbody.innerHTML = '';
  
        result.solution.materialsList.forEach((item, index) => {
            const row = tbody.insertRow();
            const totalPrice = (item.material.unitPrice || 0) * item.quantity;
            row.innerHTML = `
                <td>${index + 1}</td>
                <td>${item.material.materialType}</td>
                <td>${item.material.angle}°</td>
                <td>φ${item.material.diameter}</td>
                <td>${item.material.partNumber || '-'}</td>
                <td>${item.quantity}</td>
                <td>${item.material.unitPrice ? '¥' + item.material.unitPrice.toLocaleString() : '-'}</td>
                <td>${totalPrice > 0 ? '¥' + totalPrice.toLocaleString() : '-'}</td>
            `;
        });
    }
  
    // 計算をクリア
    function clearCalculation() {
        const inputs = [
            'line1_startX', 'line1_startY', 'line1_startZ',
            'line1_endX', 'line1_endY', 'line1_endZ',
            'line2_startX', 'line2_startY', 'line2_startZ',
            'line2_endX', 'line2_endY', 'line2_endZ'
        ];
  
        inputs.forEach(id => {
            document.getElementById(id).value = '';
        });
  
        document.getElementById('line1_azimuth').value = '';
        document.getElementById('line1_gradient').value = '';
        document.getElementById('line2_azimuth').value = '';
        document.getElementById('line2_gradient').value = '';
        document.getElementById('line1_diameter').value = '';
        document.getElementById('line2_diameter').value = '';
  
        document.getElementById('calculationResults').style.display = 'none';
        calculationResults = null;
        
        // 手動ルートもクリア
        manualBends = [];
        currentPipePath = null;
        displayManualBendsList();
    }
  
    // 計算アラート表示
    function showCalcAlert(message, type = 'success') {
        const alertContainer = document.getElementById('calcAlertContainer');
        const alertClass = type === 'success' ? 'alert-success' : 'alert-error';
        
        const alert = document.createElement('div');
        alert.className = `alert ${alertClass}`;
        alert.textContent = message;
        
        alertContainer.innerHTML = '';
        alertContainer.appendChild(alert);
  
        setTimeout(() => {
            alert.remove();
        }, 3000);
    }
  
    // 計算結果をExcel出力
    function exportCalculationToExcel() {
        if (!calculationResults) return;
  
        const result = calculationResults;
        let csv = '配管ルート計算結果\n\n';
        csv += '法線情報\n';
        csv += '項目,法線1,法線2\n';
        csv += `始点X,${result.line1.startX},${result.line2.startX}\n`;
        csv += `始点Y,${result.line1.startY},${result.line2.startY}\n`;
        csv += `始点Z,${result.line1.startZ},${result.line2.startZ}\n`;
        csv += `終点X,${result.line1.endX},${result.line2.endX}\n`;
        csv += `終点Y,${result.line1.endY},${result.line2.endY}\n`;
        csv += `終点Z,${result.line1.endZ},${result.line2.endZ}\n`;
        csv += `方位角,${result.line1.azimuth},${result.line2.azimuth}\n`;
        csv += `勾配(%),${result.line1.gradient},${result.line2.gradient}\n\n`;
        
        csv += '計算結果サマリー\n';
        csv += `平面交角,${result.planarAngle.toFixed(2)}度\n`;
        csv += `高低差,${result.heightDiff.toFixed(3)}m\n`;
        csv += `水平距離,${result.horizontalDist.toFixed(3)}m\n`;
        csv += `必要曲管数,${result.solution.bendCount}\n`;
        csv += `総延長,${(result.solution.totalLength / 1000).toFixed(2)}m\n`;
        csv += `概算金額,${result.solution.totalCost.toLocaleString()}円\n\n`;
  
        csv += '必要材料リスト\n';
        csv += 'No,材料種別,角度,管径,品番,数量,単価,金額\n';
        
        result.solution.materialsList.forEach((item, index) => {
            const totalPrice = (item.material.unitPrice || 0) * item.quantity;
            csv += `${index + 1},${item.material.materialType},${item.material.angle}°,φ${item.material.diameter},${item.material.partNumber || '-'},${item.quantity},${item.material.unitPrice || '-'},${totalPrice > 0 ? totalPrice : '-'}\n`;
        });
  
        const blob = new Blob(['\uFEFF' + csv], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = `配管計算結果_${new Date().toISOString().slice(0,10)}.csv`;
        link.click();
    }
  
    // 計算結果を保存
    function saveCalculation() {
        if (!calculationResults) return;
  
        const dataStr = JSON.stringify(calculationResults, null, 2);
        const blob = new Blob([dataStr], { type: 'application/json' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = `配管計算_${new Date().toISOString().slice(0,10)}.json`;
        link.click();
  
        showCalcAlert('計算結果を保存しました', 'success');
    }
  
    // ========================================
    // S字逆算計算
    // ========================================
  
    function calculateSCurveRoute() {
        // 入力値の取得
        const line1 = {
            startX: parseFloat(document.getElementById('line1_startX').value),
            startY: parseFloat(document.getElementById('line1_startY').value),
            startZ: parseFloat(document.getElementById('line1_startZ').value),
            endX: parseFloat(document.getElementById('line1_endX').value),
            endY: parseFloat(document.getElementById('line1_endY').value),
            endZ: parseFloat(document.getElementById('line1_endZ').value),
            diameter: parseFloat(document.getElementById('line1_diameter').value)
        };
  
        const line2 = {
            startX: parseFloat(document.getElementById('line2_startX').value),
            startY: parseFloat(document.getElementById('line2_startY').value),
            startZ: parseFloat(document.getElementById('line2_startZ').value),
            endX: parseFloat(document.getElementById('line2_endX').value),
            endY: parseFloat(document.getElementById('line2_endY').value),
            endZ: parseFloat(document.getElementById('line2_endZ').value),
            diameter: parseFloat(document.getElementById('line2_diameter').value)
        };
  
        // バリデーション
        if (isNaN(line1.diameter) || isNaN(line2.diameter)) {
            showCalcAlert('管径を選択してください', 'error');
            return;
        }
  
        const bend1Id = document.getElementById('sCurveBend1').value;
        const bend2Id = document.getElementById('sCurveBend2').value;
        const middleLength = parseFloat(document.getElementById('middlePipeLength').value);
  
        if (!bend1Id || !bend2Id) {
            showCalcAlert('曲管1と曲管2を選択してください', 'error');
            return;
        }
  
        if (isNaN(middleLength) || middleLength <= 0) {
            showCalcAlert('中間直管の距離を入力してください', 'error');
            return;
        }
  
        const bend1 = materials.find(m => m.id == bend1Id);
        const bend2 = materials.find(m => m.id == bend2Id);
  
        if (!bend1 || !bend2) {
            showCalcAlert('曲管が見つかりません', 'error');
            return;
        }
  
        // S字計算（2つの解）
        const solutions = calculateSCurveSolutions(line1, line2, bend1, bend2, middleLength);
  
        if (solutions.length === 0) {
            showCalcAlert('S字接続の解が見つかりませんでした', 'error');
            return;
        }
  
        // 結果を保存
        allSolutions = solutions;
        selectedSolutionIndex = 0;
        calculationResults = {
            line1,
            line2,
            solution: solutions[0],
            mode: 's_curve'
        };
  
        displaySCurveResults();
        init3DViewer();
        renderSCurve3DScene();
        showCalcAlert(`${solutions.length}つの解が見つかりました`, 'success');
    }
  
    // S字の2つの解を計算
    function calculateSCurveSolutions(line1, line2, bend1, bend2, middleLength) {
        const solutions = [];
  
        // 法線1の方向ベクトル
        const v1 = {
            x: line1.endX - line1.startX,
            y: line1.endY - line1.startY,
            z: line1.endZ - line1.startZ
        };
        const len1 = Math.sqrt(v1.x**2 + v1.y**2 + v1.z**2);
        v1.x /= len1;
        v1.y /= len1;
        v1.z /= len1;
  
        // 法線2の方向ベクトル
        const v2 = {
            x: line2.endX - line2.startX,
            y: line2.endY - line2.startY,
            z: line2.endZ - line2.startZ
        };
        const len2 = Math.sqrt(v2.x**2 + v2.y**2 + v2.z**2);
        v2.x /= len2;
        v2.y /= len2;
        v2.z /= len2;
  
        // 曲管の有効長（mに変換）
        const L1 = (bend1.effectiveLength || 0) / 1000;
        const L2 = (bend2.effectiveLength || 0) / 1000;
  
        // パターンA: 右回りS字
        const solutionA = calculateSCurveSolution(line1, line2, v1, v2, bend1, bend2, middleLength, L1, L2, 'right');
        if (solutionA) {
            solutions.push({...solutionA, pattern: '右回りS字'});
        }
  
        // パターンB: 左回りS字
        const solutionB = calculateSCurveSolution(line1, line2, v1, v2, bend1, bend2, middleLength, L1, L2, 'left');
        if (solutionB) {
            solutions.push({...solutionB, pattern: '左回りS字'});
        }
  
        return solutions;
    }
  
    // 単一のS字解を計算
    function calculateSCurveSolution(line1, line2, v1, v2, bend1, bend2, middleLength, L1, L2, pattern) {
        // 曲管1後の方向ベクトル（簡易計算）
        const angle1Rad = bend1.angle * Math.PI / 180;
        const angle2Rad = bend2.angle * Math.PI / 180;
  
        // 平面角度と垂直角度に分解（簡易版）
        const horizontalAngle1 = angle1Rad;
        const horizontalAngle2 = angle2Rad;
  
        // 右回りか左回りかで符号を変える
        const sign = pattern === 'right' ? -1 : 1;
  
        // 曲管1後の方向（水平回転のみ簡易計算）
        const cos1 = Math.cos(sign * horizontalAngle1);
        const sin1 = Math.sin(sign * horizontalAngle1);
        const m = {
            x: v1.x * cos1 - v1.y * sin1,
            y: v1.x * sin1 + v1.y * cos1,
            z: v1.z // 簡易版：Z成分はそのまま
        };
        const lenM = Math.sqrt(m.x**2 + m.y**2 + m.z**2);
        m.x /= lenM;
        m.y /= lenM;
        m.z /= lenM;
  
        // 折れ点1（法線1終点 + 曲管1）
        const breakPoint1 = {
            x: line1.endX + v1.x * L1 * 0.5 + m.x * L1 * 0.5,
            y: line1.endY + v1.y * L1 * 0.5 + m.y * L1 * 0.5,
            z: line1.endZ + v1.z * L1 * 0.5 + m.z * L1 * 0.5
        };
  
        // 折れ点2（折れ点1 + 中間直管）
        const breakPoint2 = {
            x: breakPoint1.x + m.x * middleLength,
            y: breakPoint1.y + m.y * middleLength,
            z: breakPoint1.z + m.z * middleLength
        };
  
        // 曲管2後の方向（逆向きに計算）
        const cos2 = Math.cos(sign * horizontalAngle2);
        const sin2 = Math.sin(sign * horizontalAngle2);
        const m2 = {
            x: m.x * cos2 - m.y * sin2,
            y: m.x * sin2 + m.y * cos2,
            z: m.z
        };
        const lenM2 = Math.sqrt(m2.x**2 + m2.y**2 + m2.z**2);
        m2.x /= lenM2;
        m2.y /= lenM2;
        m2.z /= lenM2;
  
        // 法線2始点への到達確認
        const reachPoint = {
            x: breakPoint2.x + m.x * L2 * 0.5 + m2.x * L2 * 0.5,
            y: breakPoint2.y + m.y * L2 * 0.5 + m2.y * L2 * 0.5,
            z: breakPoint2.z + m.z * L2 * 0.5 + m2.z * L2 * 0.5
        };
  
        // 誤差計算
        const error = Math.sqrt(
            Math.pow(reachPoint.x - line2.startX, 2) +
            Math.pow(reachPoint.y - line2.startY, 2) +
            Math.pow(reachPoint.z - line2.startZ, 2)
        );
  
        // 各区間の延長計算
        const dist1 = Math.sqrt(
            Math.pow(line1.endX - line1.startX, 2) +
            Math.pow(line1.endY - line1.startY, 2) +
            Math.pow(line1.endZ - line1.startZ, 2)
        );
  
        const dist2 = Math.sqrt(
            Math.pow(line2.endX - line2.startX, 2) +
            Math.pow(line2.endY - line2.startY, 2) +
            Math.pow(line2.endZ - line2.startZ, 2)
        );
  
        // 中間直管の勾配
        const middleGradient = ((breakPoint2.z - breakPoint1.z) / middleLength) * 100;
  
        return {
            breakPoint1,
            breakPoint2,
            middleDirection: m,
            bend1,
            bend2,
            middleLength,
            error,
            segments: [
                {
                    name: '法線1',
                    length: dist1.toFixed(3),
                    heightDiff: (line1.endZ - line1.startZ).toFixed(3),
                    gradient: (((line1.endZ - line1.startZ) / dist1) * 100).toFixed(2)
                },
                {
                    name: '曲管1',
                    length: L1.toFixed(3),
                    material: bend1
                },
                {
                    name: '中間直管',
                    length: middleLength.toFixed(3),
                    heightDiff: (breakPoint2.z - breakPoint1.z).toFixed(3),
                    gradient: middleGradient.toFixed(2)
                },
                {
                    name: '曲管2',
                    length: L2.toFixed(3),
                    material: bend2
                },
                {
                    name: '法線2',
                    length: dist2.toFixed(3),
                    heightDiff: (line2.endZ - line2.startZ).toFixed(3),
                    gradient: (((line2.endZ - line2.startZ) / dist2) * 100).toFixed(2)
                }
            ],
            totalLength: dist1 + L1 + middleLength + L2 + dist2,
            totalCost: (bend1.unitPrice || 0) + (bend2.unitPrice || 0)
        };
    }
  
    // S字結果を表示
    function displaySCurveResults() {
        document.getElementById('calculationResults').style.display = 'block';
  
        // 複数解の比較表示
        const container = document.getElementById('routeComparison');
        container.parentElement.style.display = 'block';
        container.innerHTML = '';
  
        allSolutions.forEach((solution, index) => {
            const card = document.createElement('div');
            card.className = 'route-card' + (index === selectedSolutionIndex ? ' selected' : '');
            card.onclick = () => selectSCurveSolution(index);
  
            card.innerHTML = `
                <div class="route-header">
                    <span class="route-badge">${solution.pattern}</span>
                    <span style="font-size: 12px; color: #666;">誤差: ${(solution.error * 1000).toFixed(1)}mm</span>
                </div>
                <div style="margin-top: 10px; font-size: 13px;">
                    <p><strong>折れ点1:</strong> (${solution.breakPoint1.x.toFixed(3)}, ${solution.breakPoint1.y.toFixed(3)}, ${solution.breakPoint1.z.toFixed(3)})</p>
                    <p><strong>折れ点2:</strong> (${solution.breakPoint2.x.toFixed(3)}, ${solution.breakPoint2.y.toFixed(3)}, ${solution.breakPoint2.z.toFixed(3)})</p>
                    <p><strong>総延長:</strong> ${solution.totalLength.toFixed(3)}m</p>
                </div>
            `;
  
            container.appendChild(card);
        });
  
        const result = calculationResults;
        const solution = result.solution;
  
        // 統計情報
        const statsHTML = `
            <div class="stat-card" style="background: linear-gradient(135deg, #10b981 0%, #059669 100%);">
                <h3>2</h3>
                <p>使用曲管数</p>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);">
                <h3>${solution.totalLength.toFixed(2)}m</h3>
                <p>総延長</p>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);">
                <h3>¥${solution.totalCost.toLocaleString()}</h3>
                <p>曲管費用</p>
            </div>
        `;
        document.getElementById('resultStats').innerHTML = statsHTML;
  
        // 接続概要
        const summaryHTML = `
            <p><strong>パターン:</strong> ${solution.pattern}</p>
            <p><strong>折れ点1座標:</strong> X=${solution.breakPoint1.x.toFixed(3)}m, Y=${solution.breakPoint1.y.toFixed(3)}m, Z=${solution.breakPoint1.z.toFixed(3)}m</p>
            <p><strong>折れ点2座標:</strong> X=${solution.breakPoint2.x.toFixed(3)}m, Y=${solution.breakPoint2.y.toFixed(3)}m, Z=${solution.breakPoint2.z.toFixed(3)}m</p>
            <p><strong>接続誤差:</strong> ${(solution.error * 1000).toFixed(1)}mm</p>
        `;
        document.getElementById('connectionSummary').innerHTML = summaryHTML;
  
        // 区間リスト
        const tbody = document.getElementById('materialsListBody');
        tbody.innerHTML = '';
  
        solution.segments.forEach((segment, index) => {
            const row = tbody.insertRow();
            if (segment.material) {
                row.innerHTML = `
                    <td>${index + 1}</td>
                    <td>曲管</td>
                    <td>${segment.material.angle}°</td>
                    <td>φ${segment.material.diameter}</td>
                    <td>${segment.material.partNumber || '-'}</td>
                    <td>1</td>
                    <td>${segment.material.unitPrice ? '¥' + segment.material.unitPrice.toLocaleString() : '-'}</td>
                    <td><strong>${segment.length}m</strong></td>
                `;
            } else {
                row.innerHTML = `
                    <td>${index + 1}</td>
                    <td>直管</td>
                    <td>-</td>
                    <td>φ${result.line1.diameter}</td>
                    <td>${segment.name}</td>
                    <td>1</td>
                    <td>勾配${segment.gradient || 0}%</td>
                    <td><strong>${segment.length}m</strong></td>
                `;
            }
        });
  
        setTimeout(() => {
            document.getElementById('calculationResults').scrollIntoView({ behavior: 'smooth' });
        }, 100);
    }
  
    // S字解を選択
    function selectSCurveSolution(index) {
        selectedSolutionIndex = index;
        calculationResults.solution = allSolutions[index];
        displaySCurveResults();
        renderSCurve3DScene();
    }
  
    // ========================================
    // 手動ルート計算
    // ========================================
  
    function calculateManualRoute(showResults = false) {
        if (manualBends.length === 0 && showResults) {
            showCalcAlert('曲管を追加してください', 'error');
            return;
        }
  
        // 法線1の情報を取得
        const line1 = {
            startX: parseFloat(document.getElementById('line1_startX').value) || 0,
            startY: parseFloat(document.getElementById('line1_startY').value) || 0,
            startZ: parseFloat(document.getElementById('line1_startZ').value) || 0,
            endX: parseFloat(document.getElementById('line1_endX').value) || 0,
            endY: parseFloat(document.getElementById('line1_endY').value) || 0,
            endZ: parseFloat(document.getElementById('line1_endZ').value) || 0,
            diameter: parseFloat(document.getElementById('line1_diameter').value)
        };
  
        const line2 = {
            startX: parseFloat(document.getElementById('line2_startX').value) || 0,
            startY: parseFloat(document.getElementById('line2_startY').value) || 0,
            startZ: parseFloat(document.getElementById('line2_startZ').value) || 0,
            endX: parseFloat(document.getElementById('line2_endX').value) || 0,
            endY: parseFloat(document.getElementById('line2_endY').value) || 0,
            endZ: parseFloat(document.getElementById('line2_endZ').value) || 0,
            diameter: parseFloat(document.getElementById('line2_diameter').value)
        };
  
        // 配管経路を3D計算
        const path = calculate3DPipePath(line1, manualBends);
  
        // 直管の必要延長を計算
        const straightPipes = calculateStraightPipes(path, line1, line2);
  
        // 結果を保存
        currentPipePath = {
            path: path,
            straightPipes: straightPipes,
            line1: line1,
            line2: line2,
            manualBends: manualBends
        };
  
        if (showResults) {
            displayManualResults();
            init3DViewer();
            renderManual3DScene();
            showCalcAlert('計算が完了しました', 'success');
        } else if (scene) {
            renderManual3DScene();
        }
    }
  
    // 3D配管経路を計算
    function calculate3DPipePath(line1, bends) {
        const path = [];
  
        // 始点
        let currentPos = {
            x: line1.endX,
            y: line1.endY,
            z: line1.endZ
        };
  
        // 初期方向ベクトル（法線1の方向）
        let currentDir = {
            x: line1.endX - line1.startX,
            y: line1.endY - line1.startY,
            z: line1.endZ - line1.startZ
        };
        const len = Math.sqrt(currentDir.x**2 + currentDir.y**2 + currentDir.z**2);
        currentDir.x /= len;
        currentDir.y /= len;
        currentDir.z /= len;
  
        path.push({
            type: 'start',
            position: {...currentPos},
            direction: {...currentDir}
        });
  
        // 各曲管を適用
        bends.forEach((bend, index) => {
            const material = bend.material;
            const angleRad = material.angle * Math.PI / 180;
            const R = material.radius || 600; // デフォルトR=600mm
  
            // 曲管の始点
            path.push({
                type: 'bend_start',
                position: {...currentPos},
                direction: {...currentDir},
                material: material
            });
  
            // 曲管による方向変更を計算
            let newDir = {...currentDir};
  
            if (bend.direction === 'horizontal') {
                // 水平面での曲がり
                if (bend.axis === 'right') {
                    // 右回り（時計回り）
                    const cos = Math.cos(-angleRad);
                    const sin = Math.sin(-angleRad);
                    newDir.x = currentDir.x * cos - currentDir.y * sin;
                    newDir.y = currentDir.x * sin + currentDir.y * cos;
                } else {
                    // 左回り（反時計回り）
                    const cos = Math.cos(angleRad);
                    const sin = Math.sin(angleRad);
                    newDir.x = currentDir.x * cos - currentDir.y * sin;
                    newDir.y = currentDir.x * sin + currentDir.y * cos;
                }
            } else {
                // 垂直面での曲がり
                if (bend.axis === 'up') {
                    // 上向き
                    const horizontalLen = Math.sqrt(currentDir.x**2 + currentDir.y**2);
                    const cos = Math.cos(angleRad);
                    const sin = Math.sin(angleRad);
                    newDir.x = currentDir.x * cos;
                    newDir.y = currentDir.y * cos;
                    newDir.z = sin;
                } else {
                    // 下向き
                    const horizontalLen = Math.sqrt(currentDir.x**2 + currentDir.y**2);
                    const cos = Math.cos(angleRad);
                    const sin = Math.sin(angleRad);
                    newDir.x = currentDir.x * cos;
                    newDir.y = currentDir.y * cos;
                    newDir.z = -sin;
                }
            }
  
            // 方向ベクトルを正規化
            const newLen = Math.sqrt(newDir.x**2 + newDir.y**2 + newDir.z**2);
            newDir.x /= newLen;
            newDir.y /= newLen;
            newDir.z /= newLen;
  
            // 曲管の有効長を考慮して位置を更新
            const effectiveLength = (material.effectiveLength || 0) / 1000; // mmからmに変換
            currentPos.x += (currentDir.x + newDir.x) / 2 * effectiveLength;
            currentPos.y += (currentDir.y + newDir.y) / 2 * effectiveLength;
            currentPos.z += (currentDir.z + newDir.z) / 2 * effectiveLength;
  
            currentDir = newDir;
  
            // 曲管の終点
            path.push({
                type: 'bend_end',
                position: {...currentPos},
                direction: {...currentDir},
                material: material
            });
        });
  
        path.push({
            type: 'end',
            position: {...currentPos},
            direction: {...currentDir}
        });
  
        return path;
    }
  
    // 直管の必要延長を計算
    function calculateStraightPipes(path, line1, line2) {
        const straightPipes = [];
  
        // 法線1の終点から最初の曲管までの直管
        const dist1 = Math.sqrt(
            Math.pow(line1.endX - line1.startX, 2) +
            Math.pow(line1.endY - line1.startY, 2) +
            Math.pow(line1.endZ - line1.startZ, 2)
        );
        straightPipes.push({
            name: '法線1',
            length: dist1.toFixed(3),
            from: '始点',
            to: '曲管接続部'
        });
  
        // 最後の曲管から法線2の始点までの直管
        const lastPoint = path[path.length - 1].position;
        const distToLine2 = Math.sqrt(
            Math.pow(line2.startX - lastPoint.x, 2) +
            Math.pow(line2.startY - lastPoint.y, 2) +
            Math.pow(line2.startZ - lastPoint.z, 2)
        );
        straightPipes.push({
            name: '接続直管',
            length: distToLine2.toFixed(3),
            from: '曲管終端',
            to: '法線2始点'
        });
  
        // 法線2の直管
        const dist2 = Math.sqrt(
            Math.pow(line2.endX - line2.startX, 2) +
            Math.pow(line2.endY - line2.startY, 2) +
            Math.pow(line2.endZ - line2.startZ, 2)
        );
        straightPipes.push({
            name: '法線2',
            length: dist2.toFixed(3),
            from: '曲管接続部',
            to: '終点'
        });
  
        return straightPipes;
    }
  
    // 手動計算結果を表示
    function displayManualResults() {
        document.getElementById('calculationResults').style.display = 'block';
        document.getElementById('routeComparison').parentElement.style.display = 'none';
  
        const result = currentPipePath;
  
        // 統計情報
        const totalBendLength = manualBends.reduce((sum, bend) => 
            sum + (bend.material.effectiveLength || 0), 0) / 1000;
        const totalStraightLength = result.straightPipes.reduce((sum, pipe) => 
            sum + parseFloat(pipe.length), 0);
        const totalLength = totalBendLength + totalStraightLength;
  
        const totalCost = manualBends.reduce((sum, bend) => 
            sum + (bend.material.unitPrice || 0), 0);
  
        const statsHTML = `
            <div class="stat-card" style="background: linear-gradient(135deg, #10b981 0%, #059669 100%);">
                <h3>${manualBends.length}</h3>
                <p>使用曲管数</p>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);">
                <h3>${totalLength.toFixed(2)}m</h3>
                <p>総延長</p>
            </div>
            <div class="stat-card" style="background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);">
                <h3>¥${totalCost.toLocaleString()}</h3>
                <p>曲管費用</p>
            </div>
        `;
        document.getElementById('resultStats').innerHTML = statsHTML;
  
        // 接続概要
        const summaryHTML = `
            <p><strong>使用曲管数:</strong> ${manualBends.length}個</p>
            <p><strong>曲管総延長:</strong> ${totalBendLength.toFixed(3)}m</p>
            <p><strong>直管総延長:</strong> ${totalStraightLength.toFixed(3)}m</p>
            <p><strong>総延長:</strong> ${totalLength.toFixed(3)}m</p>
        `;
        document.getElementById('connectionSummary').innerHTML = summaryHTML;
  
        // 直管リストを表示
        const tbody = document.getElementById('materialsListBody');
        tbody.innerHTML = '';
  
        result.straightPipes.forEach((pipe, index) => {
            const row = tbody.insertRow();
            row.innerHTML = `
                <td>${index + 1}</td>
                <td>直管</td>
                <td>-</td>
                <td>φ${result.line1.diameter}</td>
                <td>${pipe.name}</td>
                <td>1</td>
                <td>-</td>
                <td><strong>${pipe.length}m</strong></td>
            `;
        });
  
        // 曲管リストを追加
        manualBends.forEach((bend, index) => {
            const row = tbody.insertRow();
            row.innerHTML = `
                <td>${result.straightPipes.length + index + 1}</td>
                <td>曲管</td>
                <td>${bend.material.angle}°</td>
                <td>φ${bend.material.diameter}</td>
                <td>${bend.material.partNumber || '-'}</td>
                <td>1</td>
                <td>${bend.material.unitPrice ? '¥' + bend.material.unitPrice.toLocaleString() : '-'}</td>
                <td>${(bend.material.effectiveLength / 1000).toFixed(3)}m</td>
            `;
        });
  
        setTimeout(() => {
            document.getElementById('calculationResults').scrollIntoView({ behavior: 'smooth' });
        }, 100);
    }
  
    // ========================================
    // 3D可視化機能
    // ========================================
  
    function init3DViewer() {
        const container = document.getElementById('viewer3D');
        
        // 既存のrendererがあれば削除
        if (renderer) {
            container.removeChild(renderer.domElement);
        }
  
        // シーンの作成
        scene = new THREE.Scene();
        scene.background = new THREE.Color(0x1a1a1a);
  
        // カメラの作成
        const width = container.clientWidth;
        const height = container.clientHeight;
        camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);
        camera.position.set(20, 20, 20);
        camera.lookAt(0, 0, 0);
  
        // レンダラーの作成
        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(width, height);
        container.appendChild(renderer.domElement);
  
        // ライトの追加
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);
  
        const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
        directionalLight.position.set(10, 10, 10);
        scene.add(directionalLight);
  
        // グリッドヘルパー
        gridHelper = new THREE.GridHelper(50, 50, 0x444444, 0x222222);
        scene.add(gridHelper);
  
        // 軸ヘルパー
        axesHelper = new THREE.AxesHelper(10);
        scene.add(axesHelper);
  
        // マウスコントロール（簡易版）
        let isDragging = false;
        let previousMousePosition = { x: 0, y: 0 };
  
        renderer.domElement.addEventListener('mousedown', (e) => {
            isDragging = true;
            previousMousePosition = { x: e.clientX, y: e.clientY };
        });
  
        renderer.domElement.addEventListener('mousemove', (e) => {
            if (isDragging) {
                const deltaX = e.clientX - previousMousePosition.x;
                const deltaY = e.clientY - previousMousePosition.y;
  
                camera.position.x += deltaX * 0.05;
                camera.position.y -= deltaY * 0.05;
                camera.lookAt(0, 0, 0);
  
                previousMousePosition = { x: e.clientX, y: e.clientY };
                render3DScene();
            }
        });
  
        renderer.domElement.addEventListener('mouseup', () => {
            isDragging = false;
        });
  
        renderer.domElement.addEventListener('wheel', (e) => {
            e.preventDefault();
            const delta = e.deltaY * 0.01;
            const distance = camera.position.length();
            const newDistance = Math.max(5, Math.min(100, distance + delta));
            camera.position.multiplyScalar(newDistance / distance);
            render3DScene();
        });
    }
  
    function render3DScene() {
        if (!calculationResults || !scene) return;
  
        // S字モードの場合は専用の表示
        if (calculationResults.mode === 's_curve') {
            renderSCurve3DScene();
            return;
        }
  
        // 既存の配管グループを削除
        if (pipeGroup) {
            scene.remove(pipeGroup);
        }
  
        pipeGroup = new THREE.Group();
  
        const result = calculationResults;
        const line1 = result.line1;
        const line2 = result.line2;
  
        // 法線1を描画（青）
        const line1Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line1.startX, line1.startZ, line1.startY),
            new THREE.Vector3(line1.endX, line1.endZ, line1.endY)
        ]);
        const line1Material = new THREE.LineBasicMaterial({ color: 0x3b82f6, linewidth: 3 });
        const line1Mesh = new THREE.Line(line1Geometry, line1Material);
        pipeGroup.add(line1Mesh);
  
        // 法線2を描画（オレンジ）
        const line2Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line2.startX, line2.startZ, line2.startY),
            new THREE.Vector3(line2.endX, line2.endZ, line2.endY)
        ]);
        const line2Material = new THREE.LineBasicMaterial({ color: 0xf97316, linewidth: 3 });
        const line2Mesh = new THREE.Line(line2Geometry, line2Material);
        pipeGroup.add(line2Mesh);
  
        // 接続部を描画（緑の破線）
        const connectionGeometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line1.endX, line1.endZ, line1.endY),
            new THREE.Vector3(line2.startX, line2.startZ, line2.startY)
        ]);
        const connectionMaterial = new THREE.LineDashedMaterial({ 
            color: 0x10b981, 
            linewidth: 2,
            dashSize: 0.5,
            gapSize: 0.3
        });
        const connectionMesh = new THREE.Line(connectionGeometry, connectionMaterial);
        connectionMesh.computeLineDistances();
        pipeGroup.add(connectionMesh);
  
        // 始点・終点マーカー
        const markerGeometry = new THREE.SphereGeometry(0.3, 16, 16);
        
        const startMarker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: 0x00ff00 }));
        startMarker.position.set(line1.startX, line1.startZ, line1.startY);
        pipeGroup.add(startMarker);
  
        const endMarker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: 0xff0000 }));
        endMarker.position.set(line2.endX, line2.endZ, line2.endY);
        pipeGroup.add(endMarker);
  
        // 曲管位置マーカー（黄色）
        const bendMarkerGeometry = new THREE.BoxGeometry(0.5, 0.5, 0.5);
        const bendMarker1 = new THREE.Mesh(bendMarkerGeometry, new THREE.MeshBasicMaterial({ color: 0xffd700 }));
        bendMarker1.position.set(line1.endX, line1.endZ, line1.endY);
        pipeGroup.add(bendMarker1);
  
        const bendMarker2 = new THREE.Mesh(bendMarkerGeometry, new THREE.MeshBasicMaterial({ color: 0xffd700 }));
        bendMarker2.position.set(line2.startX, line2.startZ, line2.startY);
        pipeGroup.add(bendMarker2);
  
        scene.add(pipeGroup);
  
        // カメラの位置を調整
        const centerX = (line1.startX + line2.endX) / 2;
        const centerY = (line1.startZ + line2.endZ) / 2;
        const centerZ = (line1.startY + line2.endY) / 2;
        
        const distance = 30;
        camera.position.set(centerX + distance, centerY + distance, centerZ + distance);
        camera.lookAt(centerX, centerY, centerZ);
  
        renderer.render(scene, camera);
    }
  
    // 手動ルート用の3D表示
    function renderManual3DScene() {
        if (!currentPipePath || !scene) return;
  
        // 既存の配管グループを削除
        if (pipeGroup) {
            scene.remove(pipeGroup);
        }
  
        pipeGroup = new THREE.Group();
  
        const result = currentPipePath;
        const line1 = result.line1;
        const line2 = result.line2;
        const path = result.path;
  
        // 法線1を描画（青）
        const line1Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line1.startX, line1.startZ, line1.startY),
            new THREE.Vector3(line1.endX, line1.endZ, line1.endY)
        ]);
        const line1Material = new THREE.LineBasicMaterial({ color: 0x3b82f6, linewidth: 3 });
        const line1Mesh = new THREE.Line(line1Geometry, line1Material);
        pipeGroup.add(line1Mesh);
  
        // 曲管経路を描画（緑の太線）
        const pathPoints = path.map(p => new THREE.Vector3(p.position.x, p.position.z, p.position.y));
        const pathGeometry = new THREE.BufferGeometry().setFromPoints(pathPoints);
        const pathMaterial = new THREE.LineBasicMaterial({ color: 0x10b981, linewidth: 5 });
        const pathMesh = new THREE.Line(pathGeometry, pathMaterial);
        pipeGroup.add(pathMesh);
  
        // 曲管位置にマーカー
        path.forEach(point => {
            if (point.type === 'bend_start' || point.type === 'bend_end') {
                const markerGeometry = new THREE.BoxGeometry(0.3, 0.3, 0.3);
                const color = point.type === 'bend_start' ? 0xffd700 : 0xff6b6b;
                const marker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: color }));
                marker.position.set(point.position.x, point.position.z, point.position.y);
                pipeGroup.add(marker);
            }
        });
  
        // 接続直管（黄色の破線）
        const lastPoint = path[path.length - 1].position;
        const connectionGeometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(lastPoint.x, lastPoint.z, lastPoint.y),
            new THREE.Vector3(line2.startX, line2.startZ, line2.startY)
        ]);
        const connectionMaterial = new THREE.LineDashedMaterial({ 
            color: 0xffd700, 
            linewidth: 2,
            dashSize: 0.5,
            gapSize: 0.3
        });
        const connectionMesh = new THREE.Line(connectionGeometry, connectionMaterial);
        connectionMesh.computeLineDistances();
        pipeGroup.add(connectionMesh);
  
        // 法線2を描画（オレンジ）
        const line2Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line2.startX, line2.startZ, line2.startY),
            new THREE.Vector3(line2.endX, line2.endZ, line2.endY)
        ]);
        const line2Material = new THREE.LineBasicMaterial({ color: 0xf97316, linewidth: 3 });
        const line2Mesh = new THREE.Line(line2Geometry, line2Material);
        pipeGroup.add(line2Mesh);
  
        // 始点・終点マーカー
        const markerGeometry = new THREE.SphereGeometry(0.3, 16, 16);
        
        const startMarker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: 0x00ff00 }));
        startMarker.position.set(line1.startX, line1.startZ, line1.startY);
        pipeGroup.add(startMarker);
  
        const endMarker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: 0xff0000 }));
        endMarker.position.set(line2.endX, line2.endZ, line2.endY);
        pipeGroup.add(endMarker);
  
        scene.add(pipeGroup);
  
        // カメラの位置を調整
        const centerX = (line1.startX + line2.endX) / 2;
        const centerY = (line1.startZ + line2.endZ) / 2;
        const centerZ = (line1.startY + line2.endY) / 2;
        
        const distance = 30;
        camera.position.set(centerX + distance, centerY + distance, centerZ + distance);
        camera.lookAt(centerX, centerY, centerZ);
  
        renderer.render(scene, camera);
    }
  
    // S字用の3D表示
    function renderSCurve3DScene() {
        if (!calculationResults || !scene || calculationResults.mode !== 's_curve') return;
  
        // 既存の配管グループを削除
        if (pipeGroup) {
            scene.remove(pipeGroup);
        }
  
        pipeGroup = new THREE.Group();
  
        const result = calculationResults;
        const solution = result.solution;
        const line1 = result.line1;
        const line2 = result.line2;
  
        // 法線1を描画（青）
        const line1Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line1.startX, line1.startZ, line1.startY),
            new THREE.Vector3(line1.endX, line1.endZ, line1.endY)
        ]);
        const line1Material = new THREE.LineBasicMaterial({ color: 0x3b82f6, linewidth: 3 });
        const line1Mesh = new THREE.Line(line1Geometry, line1Material);
        pipeGroup.add(line1Mesh);
  
        // 曲管1（黄色）
        const bend1Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line1.endX, line1.endZ, line1.endY),
            new THREE.Vector3(solution.breakPoint1.x, solution.breakPoint1.z, solution.breakPoint1.y)
        ]);
        const bend1Material = new THREE.LineBasicMaterial({ color: 0xffd700, linewidth: 4 });
        const bend1Mesh = new THREE.Line(bend1Geometry, bend1Material);
        pipeGroup.add(bend1Mesh);
  
        // 中間直管（緑の太線）
        const middleGeometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(solution.breakPoint1.x, solution.breakPoint1.z, solution.breakPoint1.y),
            new THREE.Vector3(solution.breakPoint2.x, solution.breakPoint2.z, solution.breakPoint2.y)
        ]);
        const middleMaterial = new THREE.LineBasicMaterial({ color: 0x10b981, linewidth: 5 });
        const middleMesh = new THREE.Line(middleGeometry, middleMaterial);
        pipeGroup.add(middleMesh);
  
        // 曲管2（オレンジ）
        const bend2Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(solution.breakPoint2.x, solution.breakPoint2.z, solution.breakPoint2.y),
            new THREE.Vector3(line2.startX, line2.startZ, line2.startY)
        ]);
        const bend2Material = new THREE.LineBasicMaterial({ color: 0xff6b6b, linewidth: 4 });
        const bend2Mesh = new THREE.Line(bend2Geometry, bend2Material);
        pipeGroup.add(bend2Mesh);
  
        // 法線2を描画（赤）
        const line2Geometry = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(line2.startX, line2.startZ, line2.startY),
            new THREE.Vector3(line2.endX, line2.endZ, line2.endY)
        ]);
        const line2Material = new THREE.LineBasicMaterial({ color: 0xf97316, linewidth: 3 });
        const line2Mesh = new THREE.Line(line2Geometry, line2Material);
        pipeGroup.add(line2Mesh);
  
        // マーカー
        const markerGeometry = new THREE.SphereGeometry(0.3, 16, 16);
        
        // 法線1始点（緑）
        const startMarker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: 0x00ff00 }));
        startMarker.position.set(line1.startX, line1.startZ, line1.startY);
        pipeGroup.add(startMarker);
  
        // 折れ点1（黄色）
        const break1Marker = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.5), new THREE.MeshBasicMaterial({ color: 0xffd700 }));
        break1Marker.position.set(solution.breakPoint1.x, solution.breakPoint1.z, solution.breakPoint1.y);
        pipeGroup.add(break1Marker);
  
        // 折れ点2（緑）
        const break2Marker = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.5), new THREE.MeshBasicMaterial({ color: 0x00ff00 }));
        break2Marker.position.set(solution.breakPoint2.x, solution.breakPoint2.z, solution.breakPoint2.y);
        pipeGroup.add(break2Marker);
  
        // 法線2終点（赤）
        const endMarker = new THREE.Mesh(markerGeometry, new THREE.MeshBasicMaterial({ color: 0xff0000 }));
        endMarker.position.set(line2.endX, line2.endZ, line2.endY);
        pipeGroup.add(endMarker);
  
        scene.add(pipeGroup);
  
        // カメラの位置を調整
        const centerX = (line1.startX + line2.endX) / 2;
        const centerY = (line1.startZ + line2.endZ) / 2;
        const centerZ = (line1.startY + line2.endY) / 2;
        
        const distance = 30;
        camera.position.set(centerX + distance, centerY + distance, centerZ + distance);
        camera.lookAt(centerX, centerY, centerZ);
  
        renderer.render(scene, camera);
    }
  
    function resetCamera() {
        if (camera) {
            camera.position.set(20, 20, 20);
            camera.lookAt(0, 0, 0);
            render3DScene();
        }
    }
  
    function toggleGrid() {
        if (gridHelper) {
            gridHelper.visible = !gridHelper.visible;
            render3DScene();
        }
    }
  
    function toggleAxes() {
        if (axesHelper) {
            axesHelper.visible = !axesHelper.visible;
            render3DScene();
        }
    }
  
    // ========================================
    // 施工図PDF生成機能
    // ========================================
  
    function generateConstructionDrawing() {
        if (!calculationResults) return;
  
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF();
  
        const result = calculationResults;
        const line1 = result.line1;
        const line2 = result.line2;
  
        // タイトル
        doc.setFontSize(18);
        doc.text('配管施工図', 105, 20, { align: 'center' });
  
        // 作成日
        doc.setFontSize(10);
        doc.text(`作成日: ${new Date().toLocaleDateString('ja-JP')}`, 20, 30);
  
        // 平面図
        doc.setFontSize(14);
        doc.text('平面図', 20, 45);
        doc.setLineWidth(0.5);
        doc.rect(15, 50, 180, 80);
  
        // 平面図を描画（簡易版）
        const scale = 5; // スケール
        const offsetX = 105;
        const offsetY = 90;
  
        // 法線1（青）
        doc.setDrawColor(59, 130, 246);
        doc.setLineWidth(1);
        doc.line(
            offsetX + line1.startX * scale,
            offsetY - line1.startY * scale,
            offsetX + line1.endX * scale,
            offsetY - line1.endY * scale
        );
  
        // 法線2（赤）
        doc.setDrawColor(249, 115, 22);
        doc.line(
            offsetX + line2.startX * scale,
            offsetY - line2.startY * scale,
            offsetX + line2.endX * scale,
            offsetY - line2.endY * scale
        );
  
        // 縦断図
        doc.setFontSize(14);
        doc.setDrawColor(0);
        doc.text('縦断図', 20, 145);
        doc.setLineWidth(0.5);
        doc.rect(15, 150, 180, 60);
  
        // 縦断図を描画（簡易版）
        const scaleH = 5; // 水平スケール
        const scaleV = 50; // 垂直スケール（誇張）
        const offsetX2 = 20;
        const offsetY2 = 180;
  
        const dist1 = Math.sqrt(
            Math.pow(line1.endX - line1.startX, 2) + 
            Math.pow(line1.endY - line1.startY, 2)
        );
        const dist2 = Math.sqrt(
            Math.pow(line2.endX - line2.startX, 2) + 
            Math.pow(line2.endY - line2.startY, 2)
        );
  
        // 法線1（青）
        doc.setDrawColor(59, 130, 246);
        doc.setLineWidth(1);
        doc.line(
            offsetX2,
            offsetY2 - (line1.startZ - 98) * scaleV,
            offsetX2 + dist1 * scaleH,
            offsetY2 - (line1.endZ - 98) * scaleV
        );
  
        // 法線2（赤）
        doc.setDrawColor(249, 115, 22);
        doc.line(
            offsetX2 + (dist1 + result.horizontalDist) * scaleH,
            offsetY2 - (line2.startZ - 98) * scaleV,
            offsetX2 + (dist1 + result.horizontalDist + dist2) * scaleH,
            offsetY2 - (line2.endZ - 98) * scaleV
        );
  
        // 計算結果サマリー
        doc.setDrawColor(0);
        doc.setFontSize(12);
        doc.text('計算結果サマリー', 20, 225);
        
        doc.setFontSize(10);
        let yPos = 235;
        doc.text(`平面交角: ${result.planarAngle.toFixed(2)}°`, 20, yPos);
        yPos += 7;
        doc.text(`高低差: ${result.heightDiff.toFixed(3)}m`, 20, yPos);
        yPos += 7;
        doc.text(`水平距離: ${result.horizontalDist.toFixed(3)}m`, 20, yPos);
        yPos += 7;
        doc.text(`管径: φ${line1.diameter}mm`, 20, yPos);
        yPos += 7;
        doc.text(`必要曲管数: ${result.solution.bendCount}個`, 20, yPos);
        yPos += 7;
        doc.text(`総延長: ${(result.solution.totalLength / 1000).toFixed(2)}m`, 20, yPos);
        yPos += 7;
        doc.text(`概算金額: ¥${result.solution.totalCost.toLocaleString()}`, 20, yPos);
  
        // 材料リスト（新ページ）
        doc.addPage();
        doc.setFontSize(14);
        doc.text('必要材料リスト', 20, 20);
  
        doc.setFontSize(9);
        const headers = ['No', '種別', '角度', '管径', '品番', '数量', '単価', '金額'];
        const headerX = [20, 35, 60, 75, 95, 135, 155, 175];
        
        yPos = 30;
        headers.forEach((header, i) => {
            doc.text(header, headerX[i], yPos);
        });
  
        doc.line(15, yPos + 2, 195, yPos + 2);
        yPos += 10;
  
        result.solution.materialsList.forEach((item, index) => {
            if (yPos > 280) {
                doc.addPage();
                yPos = 20;
            }
  
            const totalPrice = (item.material.unitPrice || 0) * item.quantity;
            const row = [
                (index + 1).toString(),
                item.material.materialType,
                item.material.angle + '°',
                'φ' + item.material.diameter,
                item.material.partNumber || '-',
                item.quantity.toString(),
                item.material.unitPrice ? '¥' + item.material.unitPrice.toLocaleString() : '-',
                totalPrice > 0 ? '¥' + totalPrice.toLocaleString() : '-'
            ];
  
            row.forEach((cell, i) => {
                doc.text(cell, headerX[i], yPos);
            });
  
            yPos += 7;
        });
  
        // PDF保存
        doc.save(`配管施工図_${new Date().toISOString().slice(0,10)}.pdf`);
        showCalcAlert('施工図PDFを生成しました', 'success');
    }
  ```
  
    </script>

</body>
</html>
