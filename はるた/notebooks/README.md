# Notebooks

分析作業のメインフォルダ。

```text
はるた/notebooks/
  重心/            ← 重心分析（ANOVA）
  effective_area/  ← Effective Area 計算
```

共通前処理は個人フォルダではなく、プロジェクトルートの `notebooks/00_sort_inbox_data.ipynb` を使う。

---

## 実行順（重心分析）

```text
../../notebooks/00_sort_inbox_data.ipynb  ← 新しいデータを入れた時だけ実行
重心/00_data_check.ipynb
重心/04_assumption_checks.ipynb
重心/05_anova.ipynb
重心/06_tukey_hsd.ipynb
```

## 実行順（Effective Area）

 ```text
../../notebooks/00_sort_inbox_data.ipynb  ← 新しいデータを入れた時だけ実行
effective_area/00_effective_area.ipynb
```

一括実行はしない。1つずつ開き、出力を確認してから次に進む。

---

## `重心/` フォルダ

GKを除いた各チーム10人の平均座標（チーム重心）を攻撃/守備ラベルで比較する。

| Notebook | 役割 |
| --- | --- |
| `00_data_check.ipynb` | 前処理・重心計算・CSV出力 |
| `04_assumption_checks.ipynb` | Shapiro-Wilk + Levene まとめ実行 |
| `05_anova.ipynb` | 一元配置分散分析 |
| `06_tukey_hsd.ipynb` | Tukey HSD多重比較 |

通常使わない旧版・単体確認用Notebookは `99_` 始まりにしている。

| Notebook | 役割 |
| --- | --- |
| `99_01_centroid_visualization.ipynb` | 重心の可視化（旧/補助） |
| `99_02_shapiro_wilk_check.ipynb` | Shapiro-Wilk検定（単体確認用） |
| `99_03_levene_check.ipynb` | Levene検定（単体確認用） |
| `99_07_full_analysis_pipeline.ipynb` | 旧パイプライン |

主な入力: `data/candidates/01_sorted/{dataset_name}/`
主な出力: `data/processed/{dataset_name}/`、`outputs/tables/{dataset_name}/`

---

## `effective_area/` フォルダ

任意の時刻の選手座標から守備・攻撃 Effective Area を計算する。

| Notebook | 役割 |
| --- | --- |
| `00_effective_area.ipynb` | EA計算・可視化・CSV出力 |
| `99_01_effective_area_viz.ipynb` | EA可視化の旧/補助Notebook |

アルゴリズム:
- 11C3 = 165個の三角形を生成（GK含む11人）
- 守備EA: 周囲長が閾値（36m）**未満**の守備三角形だけを採用し、和集合を計算
- 攻撃EA: 守備EA・採用済み攻撃EAと重ならない三角形を順次追加

主な設定（Notebookの設定セル）:
- `PERIMETER_THRESHOLD`: 守備三角形の採用閾値（デフォルト 36.0m、これ未満の三角形を採用）
- `INCLUDE_GK`: True固定（論文通り GK含む11人）

主な入力: `data/candidates/01_sorted/{dataset_name}/`
主な出力: `outputs/tables/{dataset_name}/effective_area.csv`、`outputs/figures/{dataset_name}/effective_area/`

---

## 現在の標準分析対象

```text
DATASET_NAME = 2023-11-25_117093_tsukuba_vs_tsukuba-b
```

## 注意

- データごとに `{dataset_name}` フォルダへ出力するため、別試合の結果を上書きしにくい。
- Notebook内で処理を完結させる方針なので、外部の `src/` フォルダは使わない。
