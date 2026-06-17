# FC徳島 重心分析プロジェクト

11対11のトラッキングデータを使い、攻撃中・守備中のチーム重心位置を統計的に比較するプロジェクト。

## 進め方

まずはJupyter Notebookで1ステップずつ確認する。

処理はNotebook内にまとめる。上から順番に実行すれば、データ確認から検定まで追える形にする。

一括実行はしない。実行する人が、Notebookを1つずつ開いて確認する。

```text
../notebooks/00_sort_inbox_data.ipynb
notebooks/重心/00_data_check.ipynb
notebooks/重心/04_assumption_checks.ipynb
notebooks/重心/05_anova.ipynb
notebooks/重心/06_tukey_hsd.ipynb
```

## フォルダ

```text
../data/candidates/ 候補データの入口と整理先
../data/interim/    中間データ
../data/processed/  分析用データ
../notebooks/       共通Notebook
notebooks/          はるた分析Notebook
outputs/tables/     結果表
outputs/figures/    図
reports/            レポート
Dev/                Claude Code用の設計・プロンプト
```

詳しいデータの流れは `../data/README.md`、共通前処理は `../notebooks/README.md`、出力ファイルの扱いは `outputs/README.md` を参照する。

Notebookの実行順は `notebooks/README.md`、現在の分析結果の要約は `reports/01_ANALYSIS_SUMMARY.md` を参照する。

重心とEAの計算チェックは `reports/03_CENTROID_EA_VALIDATION.md` を参照する。

シュート前攻撃危険度指標の研究ロードマップは `reports/04_RESEARCH_ROADMAP.md` を参照する。

## 初めて見る人向け

まず以下の順番で読む。

```text
1. README.md
2. ../data/README.md
3. ../notebooks/README.md
4. notebooks/README.md
5. outputs/README.md
6. reports/01_ANALYSIS_SUMMARY.md
7. reports/03_CENTROID_EA_VALIDATION.md
8. reports/04_RESEARCH_ROADMAP.md
```

実行する場合は、まず以下のNotebookを見る。

```text
notebooks/重心/00_data_check.ipynb
notebooks/重心/04_assumption_checks.ipynb
notebooks/重心/05_anova.ipynb
notebooks/重心/06_tukey_hsd.ipynb
```

## データの置き方

新しい試合データは、まず `data/candidates/00_inbox/` に入れる。

その後、共通Notebook `../notebooks/00_sort_inbox_data.ipynb` を実行すると、試合ごとに `data/candidates/01_sorted/{dataset_name}/` の下へ自動整理される。

この分析で必要な入力ファイルは2種類だけ。

```text
{match_id}_tracker_box_data.xml
{match_id}_tracker_box_metadata.xml
```

整理後の置き場所は以下。

```text
data/candidates/01_sorted/{dataset_name}/01_tracking_core/{match_id}_tracker_box_data.xml
data/candidates/01_sorted/{dataset_name}/03_player_master/{match_id}_tracker_box_metadata.xml
```

イベントJSON、補正用ファイル、未使用CSVは現時点の分析では不要。

## 座標の扱い

トラッキングXMLの座標は `0〜1` の端基準として読む。

`00_data_check.ipynb` では、重心計算の前に以下のように中央基準へ変換する。

```text
x_centered = x - 0.5
y_centered = y - 0.5
```

そのため、出力CSVの `centroid_x`、`centroid_y` は基本的に `-0.5〜0.5` の範囲になる。

## 重心計算

標準は、GKを除いたフィールドプレーヤー10人の単純平均。

```python
CENTROID_METHOD = 'simple_mean'
```

重み付き重心を使う場合は、`00_data_check.ipynb` の設定セルで以下を変更する。

```python
CENTROID_METHOD = 'weighted'
POSITION_WEIGHTS = {
    'DF': 1.0,
    'MF': 1.0,
    'FW': 1.0,
}
```

## 時間指定分析

コーナーキックなど、特定の時間帯だけ分析したい場合は、`00_data_check.ipynb` の設定セルを変更する。

```python
# 全時間を使う
START_TIME_SECONDS = None
END_TIME_SECONDS = None

# 20分〜23分だけ使う
START_TIME_SECONDS = 20 * 60
END_TIME_SECONDS = 23 * 60
```

出力CSVには `start_time_seconds` と `end_time_seconds` が残る。

## 現在の分析対象

現在は以下のデータ名を標準入力として分析している。

```text
2023-11-25_117093_tsukuba_vs_tsukuba-b
```

入力データ:

```text
data/candidates/01_sorted/2023-11-25_117093_tsukuba_vs_tsukuba-b/01_tracking_core/117093_tracker_box_data.xml
data/candidates/01_sorted/2023-11-25_117093_tsukuba_vs_tsukuba-b/03_player_master/117093_tracker_box_metadata.xml
```

Notebookで前処理すると、以下が作成される。

```text
data/processed/2023-11-25_117093_tsukuba_vs_tsukuba-b/team_centroids_by_frame.csv
data/processed/2023-11-25_117093_tsukuba_vs_tsukuba-b/team_centroids_by_sequence.csv
outputs/tables/2023-11-25_117093_tsukuba_vs_tsukuba-b/assumption_checks.csv
outputs/tables/2023-11-25_117093_tsukuba_vs_tsukuba-b/anova.csv
```
