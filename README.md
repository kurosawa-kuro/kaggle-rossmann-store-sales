# kaggle-rossmann-store-sales

Rossmann Store Sales は、ドイツのドラッグストア Rossmann の**店舗ごとの売上を予測**する Kaggle コンペです。本リポジトリでは、**データのクリーニングから特徴量エンジニアリング、XGBoost を使ったベースラインモデルの構築まで**の流れを一通りまとめています。

## 1. 概要

- **データセット**:  
  - `train.csv` (店舗×日ごとの売上 / 2013-2015)  
  - `store.csv` (店舗のメタ情報: 競合店舗距離・オープン時期、Promo2 の有無 等)  
- **目的**:  
  - テスト期間の売上 (Sales) を予測。評価指標は Root Mean Squared Percentage Error (RMSPE)。  
- **アプローチ**:  
  1. データの読み込み & 結合  
  2. 欠損値・外れ値処理  
  3. 日付・販促データを活用した特徴量作成  
  4. XGBoost でベースラインモデル学習  
  5. RMSE / MAE などを測定 & Kaggle に提出

## 2. リポジトリ構成

```bash
kaggle-rossmann-store-sales/
├── data/               # ローカルのデータ（train.csv, store.csv, test.csv等）を置く想定
├── notebooks/
│   ├── 01_data_explore.ipynb          # データの可視化 & EDA用ノートブック
│   ├── 02_data_cleaning.ipynb         # クリーニング & 前処理
│   ├── 03_feature_engineering.ipynb   # 特徴量生成
│   └── 04_xgboost_baseline.ipynb      # XGBoostベースラインモデル
├── scripts/
│   ├── preprocessing.py               # 欠損補完や外れ値クリップ、ラグ特徴量などをまとめた関数
│   ├── train_xgb.py                   # XGBoostの学習スクリプト
│   └── ...
├── requirements.txt                   # Python依存ライブラリ
└── README.md                          # 本README
```

### 主なファイル・フォルダ

- **`notebooks/`**  
  - それぞれ Jupyter Notebook 形式で、EDA・特徴量生成・モデリングをステップごとに整理。
- **`scripts/`**  
  - Notebookではなくスクリプト形式で書いた ETL / モデル学習用コードを配置。
- **`data/`**  
  - ローカルでデータを管理するためのフォルダ。Git 管理に含めない場合は `.gitignore` などで無視設定。
- **`requirements.txt`**  
  - このリポジトリで使用する Python ライブラリ。`pandas`, `numpy`, `scikit-learn`, `xgboost`, `jupyter` など。

## 3. セットアップ

### 3.1. 環境構築

```bash
# 仮想環境の作成 (venv 例)
python3 -m venv venv
source venv/bin/activate

# ライブラリのインストール
pip install -r requirements.txt
```

### 3.2. データの配置

1. Kaggle から [Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales/data) データをダウンロード
2. `train.csv`, `test.csv`, `store.csv` を `data/` フォルダに配置

### 3.3. ノートブックの実行

```bash
jupyter notebook
# or jupyter lab
```

- ブラウザで `notebooks/01_data_explore.ipynb` を開いて実行。
- 手順通りに欠損補完や特徴量エンジニアリングを行う。

## 4. 取り組み手順

### 4.1. データの可視化 (EDA)

- `notebooks/01_data_explore.ipynb`
  - `train.csv` と `store.csv` をマージ
  - 欠損や基本統計量、売上分布、曜日別・Promo の有無での比較などを可視化

### 4.2. データクリーニング & 変形

- `notebooks/02_data_cleaning.ipynb`  
  - Open や Promo の欠損を 0 で補完  
  - CompetitionDistance の極端に大きい値をクリップ  
  - Date を datetime 型へ変換し、Year, Month, WeekOfYear などを追加

### 4.3. 特徴量エンジニアリング

- `notebooks/03_feature_engineering.ipynb`  
  - ラグ特徴量や移動平均 (例: 過去7日・14日・30日の売上平均)  
  - PromoInterval や StateHoliday 情報を One-Hot or Label Encoding  
  - CompetitionOpenSince や Promo2Since から経過月数など

### 4.4. モデル学習 (XGBoost ベースライン)

- `notebooks/04_xgboost_baseline.ipynb`
  - 学習データ / バリデーションデータの時系列分割 (例: 2015-06 以降をテストなど)  
  - `XGBRegressor` で学習 (RMSE / MAE を確認)  
  - 改善策: ハイパーパラメータチューニング, 追加特徴量, 学習データのクリップ等

### 4.5. 予測の提出 (Kaggle)

- 学習済みモデルを使い `test.csv` に対して予測 → `submission.csv` を生成  
- Kaggle にアップロードし、**RMSPE** スコアを確認

## 5. 結果

- **ベースラインの RMSPE**: (例) `0.12` 程度  
- **改善の方向**:
  - Holidy や SchoolHoliday, CompetitionOpenSinceYear/Month の計算ロジックを最適化  
  - 時系列クロスバリデーションを取り入れ、過去のラグ特徴量を増やす  
  - LightGBM, CatBoost との比較検討

## 6. 今後の展望

- **より高度な特徴量** (イベントカレンダー、天気情報など) を追加  
- **ハイパラ自動チューニング** (Optuna などを使う)  
- **アンサンブル** (複数モデルを組み合わせ)  
- **MLOps** 視点でのパイプライン化 (Airflow, Kubeflow, など)

## 7. 参考リンク

- [Kaggle: Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales)  
- [XGBoost 公式ドキュメント](https://xgboost.readthedocs.io/en/stable/)  
- [CatBoost / LightGBM など他のブースティングライブラリの比較](https://lightgbm.readthedocs.io/)

## 8. ライセンス

本リポジトリのソースコードは、特に明記がない限り [MIT License](LICENSE) とします。  
Kaggle のデータは競技規約に従って個別に利用してください。
