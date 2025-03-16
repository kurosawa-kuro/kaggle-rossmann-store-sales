
# kaggle-rossmann-store-sales

Rossmann Store Sales は、ドイツのドラッグストア Rossmann の**店舗ごとの売上を予測**する Kaggle コンペです。本リポジトリでは、**DWH 的アプローチ（fact / dim 分割、ワイドテーブル化）**を活用したデータ処理と、XGBoost でのモデル学習を軸に、以下のステップで取り組んでいます。

## 1. 概要

- **データセット**  
  - `train.csv`: 日次売上データ (2013〜2015年)  
  - `store.csv`: 店舗のメタ情報 (競合店距離・オープン時期・Promo2など)  
- **目標**  
  - テスト期間の売上 (Sales) を予測し、RMSPE (Root Mean Squared Percentage Error) を最小化する。  
- **特徴**  
  - **DWH的手法**: `fact_sales`, `dim_store`, `dim_date` などに分割＆マージしやすい形へ変形し、ワイドテーブル(データマート)を作成  
  - **XGBoostベースライン**: シンプルな特徴量でまずモデルを動かし、その後ハイパラ調整や追加特徴量で精度を高める

---

## 2. リポジトリ構成

```bash
kaggle-rossmann-store-sales/
├── data/                    # Kaggleデータ (train.csv, store.csv, test.csvなど)を配置
├── notebooks/
│   ├── 01_data_explore.ipynb         # データ可視化 & EDA
│   ├── 02_data_cleaning.ipynb        # クリーニング & 前処理
│   ├── 03_feature_engineering.ipynb  # 特徴量生成
│   ├── 04_xgboost_baseline.ipynb     # ベースラインモデル構築
│   └── 05_improvement.ipynb          # 精度向上施策(ハイパラ調整, 追加特徴量など)
├── scripts/
│   ├── dwh_transform.py              # DWH的テーブル分割 & データマート化
│   ├── preprocessing.py              # 欠損補完や外れ値クリップ、ラグ特徴量などの関数
│   ├── train_xgb.py                  # XGBoostの学習スクリプト
│   └── ...
├── requirements.txt                  # 必要なPythonライブラリ
└── README.md                         # 本README
```

- **`notebooks/`**  
  - Notebook形式で、1) EDA, 2) クリーニング, 3) 特徴量生成, 4) ベースラインモデル、5) 精度向上 までの一連の流れを実行可能。  
- **`scripts/`**  
  - `dwh_transform.py`: データを `fact_sales`, `dim_store`, `dim_date` 的に分割 & ワイドテーブル化する仕組み。  
  - `preprocessing.py`: 欠損補完や外れ値クリップなど共通処理の関数集。  
  - `train_xgb.py`: Notebook外でXGBoostを学習する場合のサンプルスクリプト。  

---

## 3. セットアップ

### 3.1. 環境構築

```bash
# (例) venvの作成
python3 -m venv venv
source venv/bin/activate

# ライブラリインストール
pip install -r requirements.txt
```

### 3.2. データの配置

1. Kaggleから [Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales/data) のzipをダウンロード  
2. `train.csv`, `store.csv`, `test.csv` を `data/` フォルダ下に設置  

### 3.3. ノートブックを起動

```bash
jupyter notebook
# or jupyter lab
```

- `notebooks/01_data_explore.ipynb` から順番に実行し、EDA → データ変換 → 特徴量作成 → モデルトレーニング → 精度向上を進める。

---

## 4. ベースラインモデル構築

1. **EDA & DWH化**  
   - `01_data_explore.ipynb`: 欠損・統計量・売上傾向を確認  
   - `scripts/dwh_transform.py`: `fact_sales`, `dim_store`, `dim_date`などに分けてから、**データマート(ワイドテーブル)** を生成  
2. **クリーニング & 特徴量**  
   - `02_data_cleaning.ipynb` および `03_feature_engineering.ipynb`: 欠損0埋め、CompetitionDistanceクリップ、日付から Year/Month/WeekOfYear などを抽出、ラグ・移動平均などを追加  
3. **ベースライン学習 (XGBoost)**  
   - `04_xgboost_baseline.ipynb`:  
     - 時系列分割 (例: 2015-06 以前がtrain / 以降がval)  
     - `XGBRegressor` で Sales を回帰  
     - RMSE, MAE, RMSPE を評価し、ベースラインスコアを確認  

---

## 5. 精度向上 (追加の施策)

- **ノートブック: `05_improvement.ipynb`**  
  1. **ハイパーパラメータチューニング**  
     - GridSearchCV / RandomizedSearchCV / Optuna などで learning_rate, max_depth, subsample 等を調整  
  2. **さらなる特徴量**  
     - 連休フラグ、シーズナリティ（Quarter, Easterホリデー等）、競合オープン月数など  
     - ターゲットエンコーディング or カウントエンコーディング (StoreType, StateHoliday)  
  3. **時系列クロスバリデーション**  
     - 時系列に合わせた複数スライド分割でバリデーションし、汎化性能を厳密に測定  
  4. **他モデルとのアンサンブル**  
     - LightGBM / CatBoost / RandomForest などと組み合わせてスコアをさらに向上  

---

## 6. 結果

- **ベースラインRMSPE**: 例) `0.12` 前後  
- **チューニング後のRMSPE**: 例) `0.10` 付近まで向上  
- **DWHアーキテクチャの利点**  
  - fact/dim分割やワイドテーブル化により、**欠損補完や特徴量作成をシンプルに**行えた  
  - StateHoliday, Promo2, CompetitionOpenSinceYear/Month などをスムーズに結合し、実験の高速化に寄与

---

## 7. 今後の展望

1. **さらなる外部データ** (天気情報、イベントカレンダー、経済指標)  
2. **アンサンブル** (Neural Networkとの組み合わせやStacking)  
3. **MLOpsパイプライン** (Airflow/KubeflowでETL・学習を自動化)  
4. **クラウドDWH** (Redshift, BigQuery) を利用した大規模処理の検証

---

## 8. 参考リンク

- [Kaggle: Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales)  
- [XGBoost ドキュメント](https://xgboost.readthedocs.io/en/stable/)  
- [LightGBM / CatBoost 比較](https://lightgbm.readthedocs.io/)

---

## 9. ライセンス

- 本リポジトリのコードは [MIT License](LICENSE)  
- Kaggleデータの取り扱いは競技規約に準拠

---

**本リポジトリの最大の特徴**は、**DWH的な設計でデータを分割＆マート化**している点です。  
これにより、**欠損補完・特徴量エンジニアリングを効率的**に実装し、ベースラインからさらにハイパラ調整や追加特徴量で精度を高めるプロセスを明快に示しています。ぜひご参考ください。
