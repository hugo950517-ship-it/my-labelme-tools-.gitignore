# my-labelme-tools
# Labelme Polygonal Keyframe Interpolation Tool

這個 Python 工具專門用於處理 **Labelme** 的多邊形 (Polygon) 標註資料。
透過幾何重採樣 (Resampling) 與線性插值 (Linear Interpolation) 演算法，可以在僅標註少數關鍵幀 (Keyframes) 的情況下，自動補全中間連續影格的 JSON 標註檔。

## 🌟 主要功能
- **動態檔名解析**：自動比對輸入檔名，支援 `_s1`, `_B1` 等複合檔名規則。
- **等距重採樣 (Resampling)**：將不同點數的多邊形歸一化至相同點數，確保插值滑順。
- **批量自動補全**：自動偵測區間內的標註 JSON，並補全中間所有缺失的影格標註。

## 🚀 使用方法
1. 安裝依賴庫：
   ```bash
   pip install numpy scipy
