# Data

このフォルダは、共同分析で使うデータ置き場。

PMや作業者が迷わないように、現行分析で使うデータ、参照資料、未使用データを分けて管理する。

## 現在使う場所

現行Notebookが使うのは主にこの2つ。

```text
data/candidates/01_sorted/{dataset_name}/
data/processed/{dataset_name}/
```

新しい試合データを入れる時だけ、この入口も使う。

```text
data/candidates/00_inbox/
```

## 使うデータの流れ

```text
data/candidates/00_inbox/
  ↓ notebooks/00_sort_inbox_data.ipynb
data/candidates/01_sorted/{dataset_name}/
  ↓ 各担当者の前処理Notebook
data/processed/{dataset_name}/
  ↓ はるた/あつとの分析Notebook
各担当者の outputs/
```

## フォルダ構成

```text
data/
  candidates/      現在使う入力XML
  processed/       現在使う分析用CSV
  raw/             現時点では空。将来の元データ置き場
  interim/         現時点では空。将来の中間データ置き場
  98_reference/    参照資料。プログラム入力ではない
  99_unused/       現在の分析では使っていないデータ
```

## 現在使っている入力データ

現在の分析で使っている入力は、試合ごとのXML 2種類だけ。

```text
{match_id}_tracker_box_data.xml
{match_id}_tracker_box_metadata.xml
```

置き場所:

```text
data/candidates/01_sorted/{dataset_name}/01_tracking_core/{match_id}_tracker_box_data.xml
data/candidates/01_sorted/{dataset_name}/03_player_master/{match_id}_tracker_box_metadata.xml
```

用途:

- `tracker_box_data.xml`: 各フレームの選手座標、ボール座標、`ballStatus`
- `tracker_box_metadata.xml`: 試合情報、チーム情報、選手情報、GK判定、出場選手

現在保存している試合:

```text
2023-11-18_117092_tsukuba-b_vs_tsukuba-c1
2023-11-25_117093_tsukuba_vs_tsukuba-b
```

## 現在使っている処理済みデータ

はるた側の重心分析では、以下を使う。

```text
data/processed/{dataset_name}/team_centroids_by_frame.csv
data/processed/{dataset_name}/team_centroids_by_sequence.csv
```

検定・ANOVAで主に使うのは以下。

```text
data/processed/{dataset_name}/team_centroids_by_sequence.csv
```

## 参照資料

プログラムの入力ではないが、アルゴリズム確認や説明のために使う資料はここに置く。

```text
data/98_reference/papers/
```

現在の中身:

```text
0530.pdf
effective area algorthm.pdf
```

## 現在使っていないデータ

現行Notebookから参照されていないデータはここに分ける。

```text
data/99_unused/
```

現在の中身:

```text
data/99_unused/csv/FC徳島GPSデータ_202506~07 - FC徳島GPSデータ_202506~07.csv
data/99_unused/csv/各選手別_コンディション_データ - 6月のデータ.csv
```

これらはGPS分析やコンディション分析を始める時に、あらためて使う場所を決める。

## 新しい試合データを追加する手順

まず `00_inbox` にXMLを2つ入れる。

```text
data/candidates/00_inbox/
  {match_id}_tracker_box_data.xml
  {match_id}_tracker_box_metadata.xml
```

次に、共通Notebookを実行する。

```text
notebooks/00_sort_inbox_data.ipynb
```

実行すると、以下に自動整理される。

```text
data/candidates/01_sorted/{dataset_name}/
  00_README.md
  01_tracking_core/
    {match_id}_tracker_box_data.xml
  03_player_master/
    {match_id}_tracker_box_metadata.xml
```

## データ名の命名規則

データ名は以下で統一する。

```text
YYYY-MM-DD_{match_id}_{match_label}
```

例:

```text
2023-11-25_117093_tsukuba_vs_tsukuba-b
```

`match_label` は英数字、ハイフン、アンダースコアだけで書く。

## 入れなくてよいデータ

現時点の分析では、以下は使わない。

```text
イベントJSON
player_nodes.csv
padding_info.csv
homography.npy
mapx.npy
mapy.npy
keypoints.json
camera_intrinsics.npz
1st / 2nd のJSON
```

これらは、攻撃/守備判定をイベントベースに変える場合や、映像補正を行う場合にだけ検討する。
