# Common Notebooks

個人の分析内容に依存しない、共同プロジェクト全体で使うNotebookを置く。

## Notebook

```text
00_sort_inbox_data.ipynb
```

## 役割

`data/candidates/00_inbox/` に入れた1試合分のXMLを、共通の整理済みデータ置き場へコピーする。

入力:

```text
data/candidates/00_inbox/{match_id}_tracker_box_data.xml
data/candidates/00_inbox/{match_id}_tracker_box_metadata.xml
```

出力:

```text
data/candidates/01_sorted/{dataset_name}/01_tracking_core/{match_id}_tracker_box_data.xml
data/candidates/01_sorted/{dataset_name}/03_player_master/{match_id}_tracker_box_metadata.xml
```

## 使うタイミング

新しい試合データを追加した時だけ実行する。

実行後に表示される `DATASET_NAME`, `MATCH_DATE`, `MATCH_ID`, `MATCH_LABEL` を、各担当者の分析Notebookに設定する。

## 個人フォルダとの違い

このフォルダは共通処理用。

```text
notebooks/   共通前処理
はるた/       はるた個人の分析
あつと/       あつと個人の分析
```
