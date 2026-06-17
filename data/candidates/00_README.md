# Candidates

新しい試合データを置く場所。

## まず入れる場所

ダウンロード直後のデータは、まずここに置く。

```text
data/candidates/00_inbox/
```

## 整理後の場所

中身を確認したら、データ名を付けて以下へ移す。

```text
data/candidates/01_sorted/{dataset_name}/
```

データ名は以下の形式。

```text
YYYY-MM-DD_{match_id}_{match_label}
```

## 必要なファイル

1試合につき必要なのは2ファイルだけ。

```text
01_tracking_core/{match_id}_tracker_box_data.xml
03_player_master/{match_id}_tracker_box_metadata.xml
```

それ以外のイベントJSON、補正用ファイル、未使用CSVは、現時点の分析では入れなくてよい。
