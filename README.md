Markdown# 🏷️ Labelme Keyframe Interpolation Tool

![Python Version](https://img.shields.io/badge/Python-3.8%2B-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)
![Labelme Support](https://img.shields.io/badge/Labelme-Polygon%20Interpolation-orange.svg)

**Labelme Keyframe Interpolation Tool** 是一款專門為 [Labelme](https://github.com/wkentaro/labelme) 影像標註工具設計的自動化輔助套件。

在處理超音波影像序列、連續影片影格或生醫影像標註時，逐影格手動繪製多邊形 (Polygon) 極度耗時。本工具透過 **弧長等距重採樣 (Equidistant Arc-Length Resampling)** 與 **幾何線性插值 (Linear Interpolation)**，讓使用者只需手動標註少數幾張關鍵幀（Keyframes），即可自動補全區間內所有中間影格的 JSON 標註檔。
📸 視覺化展示 (Visual Overview)Plaintext[ 關鍵幀 Frame 1 (JSON) ] ──┐
                          ├──► [ 弧長等距重採樣 (Resampling) ] ──► [ 時間權重 α 線性插值 ] ──► [ 自動生成 Frame 2~197 (JSON) ]
[ 關鍵幀 Frame 198 (JSON) ] ─┘
✨ 特色與優勢 (Key Features)動態檔名結構自動解析 (Dynamic Filename Pattern Parsing)：自動比對輸入的首尾檔名，精準提取影格遞增的數字變因。完整支援位數變化的複雜檔名（如 ..._B1_s1.png ➔ ..._B198_s1.png 或 frame_1.png ➔ frame_020.png）。多邊形等距重採樣 (Equidistant Polygon Resampling)：解決不同影格中多邊形頂點數量不一致（如首幀 45 點、尾幀 80 點）無法直接計算對應點的問題。自動依弧長比例歸一化至相同頂點數（預設 60 點），確保插值邊界過渡平滑自然。無硬編碼與高移植性 (Dynamic Pathing & Portable)：不再寫死絕對路徑，支援互動式輸入路徑，預設直接抓取執行腳本當前目錄。原生相容性 (100% Labelme Compatible)：生成的 JSON 檔案結構完全符合 Labelme 原生規範（包含 imagePath 與 shapes 點陣列），可直接用 Labelme 開啟微調或匯入訓練管線。📐 演算法原理 (How It Works)1. 弧長等距重採樣 (Equidistant Arc-Length Resampling)給定多邊形頂點集合 $P = \{p_1, p_2, \dots, p_n\}$：計算相鄰頂點間的歐式距離，累積得到總周長 $L$。在 $[0, L)$ 區間內取 $N$ 個等距目標長度（預設 $N=60$）。利用 scipy.interpolate.interp1d 進行線性外插與補值，重建 $N$ 個均勻分佈的頂點座標。2. 時序線性插值 (Temporal Linear Interpolation)對於關鍵幀 $A$ (影格 $f_A$) 與關鍵幀 $B$ (影格 $f_B$) 之間的中間影格 $f_x$：$$\alpha = \frac{f_x - f_A}{f_B - f_A}$$$$P_{f_x} = (1 - \alpha) \cdot P_{f_A} + \alpha \cdot P_{f_B}$$📁 建議專案架構 (Project Structure)Plaintextlabelme-keyframe-interpolation/
├── .gitignore                         # 防護規則：禁止將圖片與標註 JSON 上傳至 GitHub
├── README.md                          # 專案詳細說明文件
├── requirements.txt                   # Python 依賴套件清單
└── labelme_keyframe_interpolation.py  # 自動插值核心主程式
🚀 快速開始 (Quick Start)1. 安裝環境與依賴庫請確認系統已安裝 Python 3.8 以上版本，並執行：Bashpip install -r requirements.txt
2. 準備標註資料將圖片放置於目標資料夾中。使用 Labelme 開啟資料夾，至少標註該序列第一張與最後一張關鍵幀（以及中間任何動態劇烈變化的影格），並儲存 JSON 檔。注意：不同關鍵幀中，欲進行插值對應的形狀必須保持相同的 Label 名稱。3. 執行程式Bashpython labelme_keyframe_interpolation.py
💡 使用範例與終端機輸出 (Usage Example)Plaintext==================================================
     Labelme Keyframe Interpolation Tool          
==================================================
請輸入資料集資料夾路徑 (按下 Enter 預設為當前目錄): 
請貼上「第一張」PNG 檔名: sample_seq_B1_s1.png
請貼上「最後一張」PNG 檔名: sample_seq_B198_s1.png
✅ 格式驗證成功！檔名結構特徵一致。🔍 鎖定影格範圍: 1 ~ 198符合圖片數量: 198 張📌 區間內關鍵幀 JSON 數量: 2 個已標註影格編號: [1, 198]🔄 處理區段: [1 ➔ 198] (自動補全中間 196 張)...├─ ✅ 已生成標註檔: sample_seq_B2_s1.json├─ ✅ 已生成標註檔: sample_seq_B3_s1.json...├─ ✅ 已生成標註檔: sample_seq_B197_s1.json🎉 處理完畢！共成功生成 196 個 JSON 檔。⚙️ 核心函數與參數說明 (API Reference)resample_polygon(points, num_points=60)points (list): 原始多邊形的 2D 頂點座標串列 [[x1, y1], [x2, y2], ...]。num_points (int): 重採樣後的頂點總數，預設為 60。若標註物體形狀極度複雜，可調整為 100 或更高。parse_filename_structure(str1, str2)str1 / str2 (str): 第一張與最後一張圖片檔名。回傳值 (dict): 包含字串前綴 (prefix)、字串後綴 (suffix) 與影格編號範圍 (num1, num2)。❓ 常見問題與除錯 (FAQ & Troubleshooting)Q1: 程式提示「無法定位第一張與最後一張變化的影格數字」？答：請檢查兩張圖片檔名結構是否一致，僅允許代表影格數量的數字部分不同。例如 img_01.png 與 img_50.png 可以正常匹配；但若為 A_01.png 與 B_50.png，因字母與數字同時變化，演算法會判定無效。Q2: 為什麼執行後跳出「關鍵幀 JSON 數量不足 2 個」？答：自動插值演算法至少需要前後各 1 張手動標註好的 JSON 檔做為插值錨點（Keyframe）。請先開啟 Labelme 完成首尾影格的標註並儲存後再執行。Q3: 產出的 JSON 標註如果形狀有偏差怎麼辦？答：由於線性插值假設物體運動與形變為勻速，若中間影格有劇烈形變，可以在形變最大的中間影格手動再標註一張 JSON（作為第三個關鍵幀），重新執行程式後，演算法會自動分成兩段區間進行插值補全！📜 授權聲明 (License)本專案基於 MIT License 開源發布，歡迎自由修改與商業/學術使用。
