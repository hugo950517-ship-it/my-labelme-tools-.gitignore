import os
import re
import json
import numpy as np
from scipy.interpolate import interp1d


def resample_polygon(points, num_points=50):
    """
    將多邊形頂點依弧長做等距重採樣 (Resampling)，確保不同點數的多邊形能進行一對一插值。
    """
    pts = np.array(points)
    if len(pts) < 3:
        return points
    
    # 計算相鄰點之間的歐式距離
    distances = np.sqrt(np.sum(np.diff(pts, axis=0, append=[pts[0]])**2, axis=1))
    cumulative_length = np.cumsum(distances)
    total_length = cumulative_length[-1]
    
    if total_length == 0:
        return points

    cumulative_length = np.insert(cumulative_length[:-1], 0, 0)
    target_lengths = np.linspace(0, total_length, num_points, endpoint=False)

    fx = interp1d(cumulative_length, pts[:, 0], kind='linear', fill_value="extrapolate")
    fy = interp1d(cumulative_length, pts[:, 1], kind='linear', fill_value="extrapolate")

    return np.column_stack((fx(target_lengths), fy(target_lengths))).tolist()


def interpolate_shapes(start_shapes, end_shapes, alpha, num_resample_points=50):
    """
    對兩個關鍵幀 (Keyframes) 中的 Labelme shape 進行線性插值 (Linear Interpolation)。
    """
    start_dict = {s['label']: s for s in start_shapes}
    end_dict = {s['label']: s for s in end_shapes}
    interpolated_shapes = []

    for label, s_shape in start_dict.items():
        if label in end_dict:
            e_shape = end_dict[label]
            pts_start = resample_polygon(s_shape['points'], num_resample_points)
            pts_end = resample_polygon(e_shape['points'], num_resample_points)

            # 線性插值計算
            interp_pts = (1 - alpha) * np.array(pts_start) + alpha * np.array(pts_end)

            new_shape = s_shape.copy()
            new_shape['points'] = interp_pts.tolist()
            interpolated_shapes.append(new_shape)

    return interpolated_shapes


def parse_filename_structure(str1, str2):
    """
    靈活解析兩張檔名之間的差異，精準提取影格遞增的數字變因 (支援位數變化如 B1 -> B198)。
    """
    def clean(s):
        s = s.strip().replace('[', '').replace(']', '')
        if not s.lower().endswith('.png'):
            s += '.png'
        return s

    f1, f2 = clean(str1), clean(str2)

    # 提取所有 "字母+數字" 的組合
    matches1 = list(re.finditer(r'([a-zA-Z]+)(\d+)', f1))
    matches2 = list(re.finditer(r'([a-zA-Z]+)(\d+)', f2))

    if len(matches1) != len(matches2):
        return {'valid': False}

    diff_index = -1
    for i, (m1, m2) in enumerate(zip(matches1, matches2)):
        if m1.group(2) != m2.group(2):
            diff_index = i
            break

    if diff_index == -1:
        return {'valid': False}

    target_m1 = matches1[diff_index]
    target_m2 = matches2[diff_index]

    prefix = f1[:target_m1.start(2)]
    suffix = f1[target_m1.end(2):]

    return {
        'valid': True,
        'prefix': prefix,
        'suffix': suffix,
        'num1': int(target_m1.group(2)),
        'num2': int(target_m2.group(2))
    }


def run_interpolation():
    print("==================================================")
    print("     Labelme Keyframe Interpolation Tool          ")
    print("==================================================")

    # 1. 動態取得資料夾路徑 (避免硬編碼)
    input_dir = input("請輸入資料集資料夾路徑 (按下 Enter 預設為當前目錄): ").strip()
    dataset_dir = input_dir if input_dir else os.getcwd()

    if not os.path.exists(dataset_dir):
        print(f"❌ 錯誤：找不到指定的資料夾路徑 -> {dataset_dir}")
        return

    # 2. 取得檔名輸入
    first_file = input("請貼上「第一張」PNG 檔名: ").strip()
    last_file  = input("請貼上「最後一張」PNG 檔名: ").strip()
    print("--------------------------------------------------")

    if not first_file or not last_file:
        print("❌ 錯誤：檔名不能為空！")
        return

    diff_result = parse_filename_structure(first_file, last_file)

    if not diff_result['valid']:
        print("❌ 錯誤：無法定位第一張與最後一張變化的影格數字，請確認檔名結構是否一致！")
        return

    start_num = min(diff_result['num1'], diff_result['num2'])
    end_num   = max(diff_result['num1'], diff_result['num2'])
    prefix_str = diff_result['prefix']
    suffix_str = diff_result['suffix']

    pattern_str = re.escape(prefix_str) + r'(\d+)' + re.escape(suffix_str)
    frame_pattern = re.compile(rf'^{pattern_str}$', re.IGNORECASE)

    all_files = os.listdir(dataset_dir)
    img_dict = {}
    json_dict = {}

    for fname in all_files:
        img_match = frame_pattern.search(fname)
        if img_match:
            f_num = int(img_match.group(1))
            if start_num <= f_num <= end_num:
                img_dict[f_num] = fname
                json_name = os.path.splitext(fname)[0] + '.json'
                if json_name in all_files:
                    json_dict[f_num] = json_name

    sorted_frames = sorted(img_dict.keys())
    if not sorted_frames:
        print("\n❌ 在指定範圍內找不到對應的圖片檔！")
        return

    print(f"✅ 格式驗證成功！檔名結構特徵一致。")
    print(f"🔍 鎖定影格範圍: {start_num} ~ {end_num}")
    print(f"   符合圖片數量: {len(sorted_frames)} 張")

    keyframe_indices = sorted([f_num for f_num in img_dict if f_num in json_dict])
    print(f"📌 區間內關鍵幀 JSON 數量: {len(keyframe_indices)} 個")
    print(f"   已標註影格編號: {keyframe_indices}")

    if len(keyframe_indices) < 2:
        print("\n⚠️ 該區間內的關鍵幀 JSON 數量不足 2 個！請先在 Labelme 中至少標註該範圍內首尾 2 張關鍵幀。")
        return

    resample_pts = 60  # 插值時使用的多邊形重採樣點數
    generated_count = 0

    for i in range(len(keyframe_indices) - 1):
        start_f, end_f = keyframe_indices[i], keyframe_indices[i + 1]
        gap = end_f - start_f
        if gap > 1:
            print(f"\n🔄 處理區段: [{start_f} ➔ {end_f}] (自動補全中間 {gap - 1} 張)...")

            start_json_path = os.path.join(dataset_dir, json_dict[start_f])
            end_json_path = os.path.join(dataset_dir, json_dict[end_f])

            with open(start_json_path, 'r', encoding='utf-8') as f:
                start_data = json.load(f)
            with open(end_json_path, 'r', encoding='utf-8') as f:
                end_data = json.load(f)

            for step in range(1, gap):
                curr_f = start_f + step
                if curr_f not in img_dict or curr_f in json_dict:
                    continue

                target_img_name = img_dict[curr_f]
                target_json_name = os.path.splitext(target_img_name)[0] + '.json'
                target_json_path = os.path.join(dataset_dir, target_json_name)

                alpha = step / gap
                interp_shapes = interpolate_shapes(
                    start_data['shapes'], 
                    end_data['shapes'], 
                    alpha, 
                    num_resample_points=resample_pts
                )

                new_json_data = start_data.copy()
                new_json_data['imagePath'] = target_img_name
                new_json_data['shapes'] = interp_shapes

                with open(target_json_path, 'w', encoding='utf-8') as f:
                    json.dump(new_json_data, f, ensure_ascii=False, indent=2)

                print(f"  ├─ ✅ 已生成標註檔: {target_json_name}")
                generated_count += 1
                json_dict[curr_f] = target_json_name

    print(f"\n🎉 處理完畢！共成功生成 {generated_count} 個 JSON 檔。")


if __name__ == "__main__":
    run_interpolation()
