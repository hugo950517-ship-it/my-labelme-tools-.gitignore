# 🏷️ Labelme Keyframe Interpolation Tool (標註關鍵幀自動插值工具)

一款專為 **Labelme** 影像標註工具設計的 Python 自動化腳本。

在進行電腦視覺 (Computer Vision) 專題、超音波影像序列或連續影片標註時，手動繪製每個影格的多邊形 (Polygon) 極度耗時。本專案透過 **弧長等距重採樣 (Arc-Length Resampling)** 與 **幾何線性插值 (Linear Interpolation)**，只需手動標註少數幾張關鍵幀 (Keyframes)，即可自動補全區間內所有中間影格的 JSON 標註檔。



💡 專案開發動機 (Motivation)
在處理連續影格標註時，我們發現以下痛點：
1.人工標註耗時：數百張影格重複繪製多邊形，時間成本極高。
2.頂點數量不一致：不同影格手畫的點數不同（如第 1 幀 40 點、第 50 幀 70 點），無法直接進行點對點座標計算。
3.檔名結構複雜：序列檔名常包含多組數字與字母（如 ..._B1_s1.png 到 ..._B198_s1.png），傳統字串處理容易抓錯影格編號。



✨ 核心功能 (Features)
自動解析檔名結構：採用正規表達式 (Regex) 動態偵測序列檔名中遞增的影格數字，支援各種位數變化。
多邊形等距重採樣：自動將不同頂點數的多邊形依弧長歸一化至相同點數（預設 60 點），確保插值滑順不變形。
多關鍵幀連續補全：支援標註多張關鍵幀，程式會自動分段完成區間內的插值計算。
完全相容 Labelme：生成的 JSON 格式符合 Labelme 原生規範，可直接用 Labelme 開啟與人工微調。



🛠️ 技術架構與原理 (Methodology)頂點重採樣 (Resampling)：計算多邊形各邊歐式距離與累積周長，使用 scipy.interpolate.interp1d 依周長比例重新均勻採樣出 $N$ 個頂點。時序線性插值 (Interpolation)：計算當前影格相對於前後關鍵幀的時間權重 $\alpha = \frac{f_{\text{current}} - f_{\text{start}}}{f_{\text{end}} - f_{\text{start}}}$，並對所有頂點進行座標線性插值：

$$P_{\text{interp}} = (1 - \alpha) \cdot P_{\text{start}} + \alpha \cdot P_{\text{end}}$$



📁 專案檔案結構 (Project Structure)
├── .gitignore                         # Git 防護規則 (防止推送圖片與 JSON 數據)
├── README.md                          # 專案說明文件
├── requirements.txt                   # 套件依賴清單
└── labelme_keyframe_interpolation.py  # 核心插值主程式



🚀 快速開始 (Quick Start)
1. 安裝環境依賴
pip install -r requirements.txt



2. 準備資料
將影像序列放入資料夾中。
使用 Labelme 打開資料夾，標註該序列的第一張與最後一張關鍵幀（若中間有劇烈形變，可額外標註中間幀），並儲存 JSON 檔。
注意：各關鍵幀中對應物體的標籤名稱 (Label) 必須一致。



3. 執行程式
python labelme_keyframe_interpolation.py



4. 執行流程範例
   
==================================================

   Labelme Keyframe Interpolation Tool
         
==================================================

請輸入資料集資料夾路徑 
(按下 Enter 預設為當前目錄): 

請貼上「第一張」PNG 檔名: sample_B1_s1.png

請貼上「最後一張」PNG 檔名: sample_B198_s1.png



✅ 格式驗證成功！檔名結構特徵一致。
🔍 鎖定影格範圍: 1 ~ 198
符合圖片數量: 198 張
📌 區間內關鍵幀 JSON 數量: 2 個

🔄 處理區段: [1 ➔ 198] (自動補全中間 196 張)...
├─ ✅ 已生成標註檔: sample_B2_s1.json
├─ ✅ 已生成標註檔: sample_B3_s1.json
...
🎉 處理完畢！共成功生成 196 個 JSON 檔。



⚙️ 套件需求 (Requirements)
Python 3.8+

NumPy >= 1.20.0

SciPy >= 1.7.0

📜 授權 (License)
本專案採用 MIT License 釋出，歡迎自由修改與學習使用。 
