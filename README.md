# ECG分析システム
> 10,756人分のECGデータの可視化・解析システム

## 概要

### 処理済みデータ
- **患者データ**: 10,756人
- **ECG解析データ**: 10,756件  
- **波形データ**: 226,905,000データポイント
- **データ形式**: Apache Parquet

### 患者属性
- **年齢**: 平均55.7歳（範囲: 0〜95歳）
- **性別分布**: 男性51.5% (5,538人), 女性47.2% (5,076人), その他1.3% (142人)

### ECG測定値
- **心拍数**: 平均76.7回/分（範囲: 19〜206）
- **PR間隔**: 平均166.7ms（範囲: 76〜354）  
- **QRS幅**: 平均98.8ms（範囲: 46〜322）
- **QT間隔**: 平均398.1ms（範囲: 196〜680）

## クイックスタート

### 1. リポジトリのクローン
```bash
git clone https://github.com/shvalin1/ECG_Analysis.git
cd ECG_Analysis
```

### 2. 環境セットアップ
```bash
# Python仮想環境の作成（推奨）
python -m venv ecg_env
source ecg_env/bin/activate  # Linux/Mac
# または Windows: ecg_env\Scripts\activate

# 必要ライブラリのインストール
pip install -r requirements.txt
```

### 3. データのダウンロード
```bash
# dataディレクトリを作成
mkdir -p data
```

**Google Driveから手動ダウンロード：**
1. google driveにアクセス
2. 以下の3つのファイルをダウンロード：
   - `patient_data.parquet`
   - `ecg_analysis.parquet`
   - `waveform_data.parquet`
3. ダウンロードしたファイルを `data/` フォルダに配置

### 4. Jupyter Notebookの起動
```bash
jupyter notebook ecg_visualization.ipynb
```

## ディレクトリ構造

```
ECG_Analysis/
├── data/                          # 処理済みデータ（Google Driveからダウンロード）
│   ├── patient_data.parquet      # 患者基本情報
│   ├── ecg_analysis.parquet      # ECG解析結果
│   └── waveform_data.parquet     # 12誘導波形データ
├── ecg_visualization.ipynb       # メインの解析ノートブック
├── requirements.txt              # 必要ライブラリ
└── README.md                     # このファイル
```

## データ構造詳細

### 1. 患者データ (patient_data.parquet)
```
Columns: recording_id, patient_id, exam_datetime, gender, age, height, weight
Shape: (10,756, 7)
Size: ~104.5KB

データ例:
- recording_id: "000012995508_20230117215637" (ECG記録ID)
- patient_id: "000012995508" (患者ID)
- exam_datetime: "20230117215637" (検査日時)
- gender: "M" / "F" / "O" (性別: 男性/女性/その他)
- age: 74.0 (年齢)
- height: 165.0 (身長cm) ※欠損値あり
- weight: 70.0 (体重kg) ※欠損値あり
```

### 2. ECG解析データ (ecg_analysis.parquet)
```
Columns: recording_id, patient_id, heart_rate, pr_interval, qrs_duration, 
         qt_interval, qtc_interval, qrs_axis, p_axis, t_axis, rv5, sv1, 
         rv6, rv5_plus_sv1, minnesota_code
Shape: (10,756, 15)
Size: ~167.9KB

主要項目:
- recording_id: ECG記録ID
- patient_id: 患者ID
- heart_rate: 心拍数 (bpm)
- pr_interval: PR間隔 (ms)
- qrs_duration: QRS幅 (ms)
- qt_interval: QT間隔 (ms)
- qtc_interval: 補正QT間隔 (ms)
- qrs_axis: QRS軸 (度)
- p_axis: P軸 (度)
- t_axis: T軸 (度)
- rv5: V5誘導R波振幅 (mV)
- sv1: V1誘導S波振幅 (mV)
- rv6: V6誘導R波振幅 (object型、全て欠損)
- rv5_plus_sv1: RV5 + SV1 (mV)
- minnesota_code: ミネソタコード (診断分類)
```

### 3. 波形データ (waveform_data.parquet)
```
Columns: recording_id, lead_name, sequence_number, value
Shape: (226,905,000, 4)
Size: ~369.6MB

データ形式:
- recording_id: ECG記録ID
- lead_name: 誘導名 ("I", "II", "III", "aVR", "aVL", "aVF", "V1"〜"V6")
- sequence_number: 時系列シーケンス番号 (0〜)
- value: 波形振幅値 (μV)

サンプリング:
- 周波数: 1000Hz想定
- 記録時間: 約60秒
- 誘導数: 12誘導
- 患者あたり平均データポイント数: 約21,100
```

## 主要な分析結果

### 患者分布
- **総患者数**: 10,756人
- **平均年齢**: 55.7歳
- **性別**: 男性 51.5%, 女性 47.2%, その他 1.3%

### ECG指標
- **心拍数**: 76.7 ± 19.2 bpm
- **QRS幅**: 98.8 ± 25.7 ms
- **QT間隔**: 398.1 ± 51.0 ms
- **PR間隔**: 166.7 ± 34.7 ms

### ミネソタコード
- **記録率**: 88.4% (9,511件)
- **ユニークコード数**: 176種類
- **最頻出コード**: 9-4-1 (1,057件, 11.1%)

## トラブルシューティング

### データファイルが見つからない場合
1. `data/` フォルダが存在するか確認：
   ```bash
   ls -la data/
   ```
2. 必要な3つのファイルがすべて存在するか確認：
   - `patient_data.parquet`
   - `ecg_analysis.parquet`
   - `waveform_data.parquet`

### データダウンロード時の注意点
- Google Driveからダウンロード時、ファイル名が変更される場合があります
- 元のファイル名に戻してください
- ファイルサイズの目安：
  - `patient_data.parquet`: ~104KB
  - `ecg_analysis.parquet`: ~168KB
  - `waveform_data.parquet`: ~370MB

### 依存関係のエラー
```bash
# ライブラリの再インストール
pip install --upgrade -r requirements.txt

# または個別インストール
pip install pandas numpy matplotlib seaborn plotly pyarrow jupyter japanize-matplotlib
```
