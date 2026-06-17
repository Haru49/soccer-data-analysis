# FC徳島 共同分析プロジェクト

サッカーのトラッキングデータ、GPSデータ、コンディションデータを使った共同分析プロジェクト。

はるたとあつとが別テーマで開発しているため、PMはこのREADMEを入口にして、共通処理、各担当者のREADME、reports、outputsを確認する。

## 最初に見る場所

```text
1. README.md
2. data/README.md
3. notebooks/README.md
4. はるた/README.md
5. あつと/README.md
6. 各担当者の reports/
7. 各担当者の outputs/
```

## 担当フォルダ

```text
notebooks/ 共通前処理
はるた/     重心分析、ANOVA、Effective Area
あつと/     重心可視化、標本相関、ラグ相関、攻撃区間相関
data/       共通入力データ、整理済みデータ、参照資料、未使用データ
```

## 共通データの流れ

新しい試合データは、共通で以下の流れにする。

```text
data/candidates/00_inbox/
  ↓ notebooks/00_sort_inbox_data.ipynb で整理
data/candidates/01_sorted/{dataset_name}/
  ↓ 前処理
data/processed/{dataset_name}/
  ↓ 分析
{担当者}/outputs/tables/{dataset_name}/
{担当者}/outputs/figures/{dataset_name}/
  ↓ 要約
{担当者}/reports/
```

`dataset_name` は以下で統一する。

```text
YYYY-MM-DD_{match_id}_{match_label}
```

例:

```text
2023-11-25_117093_tsukuba_vs_tsukuba-b
```

## PM向け確認ポイント

共通処理と各担当者フォルダは、次の順番で確認する。

```text
notebooks/README.md
README.md
reports/
outputs/tables/
outputs/figures/
notebooks/
```

Notebookは実装・検証用、reportsは共有・報告用、outputsは分析結果の保存先として扱う。

## 共通ルール

- 入力データは `data/` に置く。
- 個人フォルダ内に元データを重複して置かない。
- 個人差がない前処理は `notebooks/` に置く。
- 現在使わないデータは `data/99_unused/` に置く。
- 参照資料は `data/98_reference/` に置く。
- 分析結果のCSVは `{担当者}/outputs/tables/{dataset_name}/` に置く。
- 図や画像は `{担当者}/outputs/figures/{dataset_name}/` に置く。
- PM向けの要約は `{担当者}/reports/` に置く。
- Notebookの中身が違うのは問題ないが、データの入口と出力先は揃える。

## 現在の状態

はるた側は、README、notebooks、outputs、reports の導線が整理済み。

あつと側は、PM向け入口として `あつと/notebooks/00_full_analysis_pipeline.ipynb` を追加した。既存の個別Notebookは検証用として残している。

共通前処理の `00_sort_inbox_data.ipynb` と共通依存関係の `requirements.txt` は、個人フォルダからルートへ移動済み。
