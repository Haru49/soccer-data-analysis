# Effective Area プログラムまとめ

## 対象ファイル

```text
notebooks/effective_area/00_effective_area.ipynb
```

## 何をするプログラムか

トラッキングXMLから選手座標を読み取り、任意フレームまたはサンプリングした複数フレームについて、攻撃側・守備側の Effective Area を計算する。

現状のプログラムが直接出しているのは「得点期待値」そのものではなく、攻撃EA・守備EAの面積である。
得点期待値として使うには、EA面積を得点確率へ変換するモデルや式を追加する必要がある。

## 入力ファイル

```text
data/candidates/01_sorted/{dataset_name}/01_tracking_core/{match_id}_tracker_box_data.xml
data/candidates/01_sorted/{dataset_name}/03_player_master/{match_id}_tracker_box_metadata.xml
```

現在の設定:

```text
DATASET_NAME = 2023-11-25_117093_tsukuba_vs_tsukuba-b
```

## 出力ファイル

```text
outputs/tables/{dataset_name}/effective_area.csv
outputs/figures/{dataset_name}/effective_area/frame_{frame_number}.png
```

現在は `effective_area.csv` と図ファイルを生成済み。

現在の `effective_area.csv` の概要:

```text
行数: 3790
サンプリング: 有効フレーム25枚に1枚
守備EA平均: 127.515 m2
攻撃EA平均: 1111.105 m2
守備EA 0 の行数: 437
攻撃EA 0 の行数: 0
```

## 処理の流れ

1. 設定セルで試合データ、出力先、EAパラメータを決める。
2. metadata XMLからピッチサイズ、チーム情報、選手情報を読む。
3. tracking XMLから各フレームの選手座標を読む。
4. 座標を `0〜1` の正規化座標からメートル単位へ変換する。
5. `ballStatus` から攻撃チーム・守備チームを決める。
6. 守備チームの選手3人組から三角形を作る。
7. 周囲長が36m未満の守備三角形だけを採用し、その和集合を守備EAとする。
8. 攻撃チームの選手3人組から三角形を作り、守備EAや採用済み攻撃EAと重ならない三角形だけ採用する。
9. 採用した攻撃三角形の和集合を攻撃EAとする。
10. 単一フレームを可視化し、必要なら複数フレームをサンプリングしてCSV出力する。

## 主要変数

```text
MATCH_DATE
MATCH_ID
MATCH_LABEL
DATASET_NAME
```

分析する試合データを指定する。

```text
TRACKING_XML
METADATA_XML
```

入力XMLのパス。

```text
OUTPUT_DIR
FIGURE_DIR
```

CSVと図の出力先。

```text
PERIMETER_THRESHOLD = 36.0
```

守備三角形の採用閾値。個別の守備三角形の周囲長が36m未満なら採用し、36m以上なら守備EAに含めない。

```text
INCLUDE_GK = True
```

GKを含めて11人で計算するかどうか。Trueなら11C3 = 165個の三角形候補を作る。

```text
VALID_BALL_STATUSES = ('HOME', 'AWAY')
```

攻撃・守備が判定できるフレームだけを対象にする。

```text
DEMO_FRAME_INDEX
```

単一フレームのデモ表示で、何番目の有効フレームを使うか。

```text
SAMPLE_EVERY_N_FRAMES
```

全フレーム集計時のサンプリング間隔。25なら有効フレームの25個に1個を計算する。

## 主要関数

```text
load_tracking_metadata()
```

metadata XMLからピッチサイズ、チーム、選手情報を読み込む。

```text
load_player_positions_by_frame()
```

tracking XMLから各フレームの選手座標を読み、メートル単位に変換する。

```text
compute_defensive_ea()
```

守備チームの選手3人組から三角形を作り、周囲長36m未満の三角形だけの和集合を返す。

```text
compute_attacking_ea()
```

攻撃チームの選手3人組から三角形を作り、守備EAや採用済み攻撃EAと重ならない三角形の和集合を返す。

```text
compute_effective_area()
```

守備EAと攻撃EAをまとめて計算し、面積や採用三角形数を返す。

```text
plot_effective_area()
```

ピッチ上に守備EA、攻撃EA、選手位置を描画する。

```text
process_all_frames()
```

複数フレームについてEAを計算し、CSV保存用のDataFrameを作る。

## 出力CSVの主な列

```text
frame_number
match_time_ms
match_time_s
event_period
ball_status
```

どの時刻・どのフレームかを示す。

```text
attacking_team_id
defending_team_id
attacking_team_label
defending_team_label
```

攻撃チーム・守備チームを示す。

```text
n_att_players
n_def_players
```

計算に使った攻撃側・守備側の選手数。

```text
defensive_ea_m2
attacking_ea_m2
```

守備EA・攻撃EAの面積。単位は平方メートル。

```text
n_defensive_triangles
n_attacking_triangles
```

採用された三角形の数。

## 説明用の言い方

このプログラムは、選手3人で作れる三角形を使って、チームが支配しているようなエリアを数値化するものです。
守備側は、近い3人組で作れる周囲長36m未満の三角形を集めて守備EAを作ります。
攻撃側は、守備EAと重ならない三角形を順番に採用して攻撃EAを作ります。
最終的に、攻撃EAと守備EAの面積を平方メートルで出します。

現状では、得点期待値そのものではなく、得点期待値の材料になりそうなEA面積を計算しています。
得点期待値にするには、EA面積やゴール位置、シュート情報などを使って得点確率へ変換する追加モデルが必要です。

## 注意点

- READMEには以前 `08_effective_area.ipynb` と書かれていたが、実ファイルは `00_effective_area.ipynb`。
- 全フレーム計算は重いため、`SAMPLE_EVERY_N_FRAMES` でサンプリングしている。
- 攻撃EAの三角形採用順は `itertools.combinations` の順番に依存している。
- 得点期待値として報告する場合は、EA面積から得点確率へ変換するロジックを別途定義する必要がある。
