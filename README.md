以下は、Kaggleの **Rossmann Store Sales** コンペを扱うリポジトリ [(kaggle-rossmann-store-sales)](https://github.com/kurosawa-kuro/kaggle-rossmann-store-sales) の README サンプルです。  
本リポジトリでは、**データウェアハウス（DWH）的な視点でテーブル分割・マート化を行い、効率的にデータを結合・変換して** XGBoost を使ったベースラインモデルを構築しています。  

---

# kaggle-rossmann-store-sales

Rossmann Store Sales は、ドイツのドラッグストア Rossmann の**店舗ごとの売上を予測**する Kaggle コンペです。本リポジトリでは、**DWHアーキテクチャを意識したデータ整形**（欠損値・外れ値処理、特徴量エンジニアリング）から、XGBoost を使ったベースラインモデルの学習・評価までの流れをカバーしています。

## 1. 概要

- **データセット**  
  - `train.csv`: 2013〜2015年の店舗×日ごとの売上（Sales）  
  - `store.csv`: 店舗メタ情報（競合店距離、オープン時期、Promo2など）  
- **目的**  
  - テスト期間の売上予測を行い、評価指標の RMSPE (Root Mean Squared Percentage Error) を最小化する。  
- **DWH的アプローチのポイント**  
  1. **ローデータ**(train, store)を「fact_sales」「dim_store」などのテーブルに分割・管理しやすい形へ。  
  2. **データマート**を作成して分析・モデリング用に横持ち（ワイドテーブル）化。  
  3. **Notebook**で特徴量を生成し、XGBoostでベースラインモデルを構築。  
  4. **RMSPE**を測定＆提出。

---

## 2. リポジトリ構成

```bash
kaggle-rossmann-store-sales/
├── data/               # ローカルのデータ（train.csv, store.csv, test.csv等）を配置
├── notebooks/
│   ├── 01_data_explore.ipynb          # データの可視化 & EDA
│   ├── 02_data_cleaning.ipynb         # クリーニング & 前処理
│   ├── 03_feature_engineering.ipynb   # 特徴量生成
│   └── 04_xgboost_baseline.ipynb      # XGBoostベースラインモデル
├── scripts/
│   ├── dwh_transform.py               # DWH的なテーブル分割 & データマート作成
│   ├── preprocessing.py               # 欠損補完や外れ値クリップ、ラグ特徴量など
│   ├── train_xgb.py                   # XGBoostの学習スクリプト
│   └── ...
├── requirements.txt                   # Python依存ライブラリ一覧
└── README.md                          # 本README
```

### 主なファイル・フォルダ

- **`notebooks/`**  
  - EDA、データクリーニング、特徴量エンジニアリング、モデル学習などをステップごとに実施。  
- **`scripts/`**  
  - `dwh_transform.py`: **DWH的テーブル設計**やワイドテーブル作成を行うスクリプト。  
  - `preprocessing.py`: 欠損値補完、外れ値クリップ、日付変換など前処理用関数をまとめたファイル。  
  - `train_xgb.py`: Notebook以外の形でXGBoostを実行するサンプル。
- **`data/`**  
  - ダウンロードしたKaggleデータを配置。（大容量ファイルは.gitignore推奨）
- **`requirements.txt`**  
  - pandas, numpy, scikit-learn, xgboost, jupyter など必要なPythonライブラリ。

---

## 3. セットアップ

### 3.1 環境構築

```bash
# 仮想環境作成 (例: venv)
python3 -m venv venv
source venv/bin/activate

# ライブラリのインストール
pip install -r requirements.txt
```

### 3.2 データの配置

1. [Kaggle: Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales/data) から  
   `train.csv`, `test.csv`, `store.csv` を入手  
2. `data/` フォルダ下に配置

### 3.3 ノートブックの実行

```bash
jupyter notebook
# or jupyter lab
```

- `notebooks/01_data_explore.ipynb` から順に実行し、EDA → 前処理 → 特徴量生成 → 学習を進める。

---

## 4. 取り組み手順

### 4.1 データ可視化・EDA

- `notebooks/01_data_explore.ipynb`
  - `train.csv`と`store.csv`をざっくり確認  
  - 欠損箇所、売上の分布、曜日ごとの傾向、Promoの影響などを可視化し、分析方針を検討

### 4.2 **DWH 的な整形** & 前処理

- `scripts/dwh_transform.py` (または `notebooks/02_data_cleaning.ipynb`)  
  - 「fact_sales」「dim_store」「dim_date」に分割しやすい設計  
  - **データマート (wide table)** を作成し、Notebookで扱いやすい形に変換  
  - 欠損値・外れ値処理 (CompetitionDistanceの中央値補完やクリップなど)

### 4.3 特徴量エンジニアリング

- `notebooks/03_feature_engineering.ipynb`
  - 日付から `Year, Month, WeekOfYear, DayOfWeek` 等を抽出  
  - StateHoliday, PromoInterval をOne-Hot or LabelEncoder  
  - **ラグ・移動平均**(例: 過去7日, 14日, 30日売上) で時系列情報を追加  
  - DWHで作ったテーブルに計算列を加えて再度データマート化

### 4.4 モデル学習 (XGBoost ベースライン)

- `notebooks/04_xgboost_baseline.ipynb`
  - 時系列でtrain/validに分割 (例: 2015-06以前を学習、以降を検証)  
  - `XGBRegressor` を用いて Sales を回帰  
  - RMSE, MAE, RMSPE を評価し、結果をNotebook上で可視化

### 4.5 予測の提出 (Kaggle)

- 学習済みモデルを使い、`test.csv` に対して予測 → `submission.csv` 出力  
- Kaggle へ提出し、Leaderboard でRMSPEを確認

---

## 5. 結果

- **ベースライン RMSPE**: 例として `0.12` 前後  
- **DWH構築の効果**:
  - 複数データソースを結合しやすく、fact/dim分割で**欠損管理や特徴量生成の効率**が向上  
  - ラグ特性やPromo特性をマスタ情報と容易に結合し、EDAやモデリングをシンプルに
- **今後の改善**:
  - 時系列クロスバリデーションを厳密に導入  
  - 休日・イベントカレンダーとの連携  
  - LightGBM, CatBoostの比較、ハイパラ最適化、アンサンブル

---

## 6. 今後の展望

1. **拡張された特徴量** (天気データや特別セール日などの外部情報)  
2. **ハイパーパラメータ自動チューニング** (Optuna, Hyperopt)  
3. **アンサンブル** (RF, LightGBM, Neural Networkなどと組み合わせ)  
4. **MLOps化** (Airflow や Kubeflow, AWS Glue でパイプラインを管理)

---

## 7. 参考リンク

- [Kaggle: Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales)  
- [XGBoostドキュメント](https://xgboost.readthedocs.io/en/stable/)  
- [LightGBM / CatBoostとの比較](https://lightgbm.readthedocs.io/)

---

## 8. ライセンス

本リポジトリのソースコードは、特に明記がない限り [MIT License](LICENSE) とします。  
Kaggleデータは競技規約に準じて個別に利用してください。

---

**本リポジトリ最大の特徴は、DWH的なテーブル設計とデータマート化**により、データの結合やクリーニングを効率的に行っている点です。  
この仕組みのおかげで、複雑な欠損処理や外部マスタ情報の取り込みを整理しやすく、EDA〜モデル学習までスムーズに進められます。ぜひご活用ください。
