# 研究ロードマップ

## 研究テーマ

```text
チーム重心とEffective Areaを用いたシュート前攻撃危険度指標の提案
```

## 目的

従来のxGは、実際に打たれたシュートの得点確率を評価する指標である。
一方で、サッカーにはシュートに至らなかったが危険だった攻撃局面が多く存在する。

本研究では、チーム重心とEffective Area（EA）を使い、シュートが起きる前の攻撃状態を評価する。
最終的には、ある時刻のチーム配置と空間状態から、その後にシュートや危険エリア侵入が起きやすいかを説明する指標を作る。

## 基本仮説

```text
攻撃チームの重心が相手ゴール方向へ前進し、
守備チームの重心が自陣ゴール方向へ後退し、
攻撃EAが前方へ広がり、
守備EAが縮小している局面ほど、
その後にシュート・PA侵入・ゴール前侵入が発生しやすい。
```

## この研究で作りたいもの

最初からゴール期待値を作るのではなく、段階的に以下を作る。

```text
Step 1: EA・重心特徴量
Step 2: 未来イベントラベル
Step 3: 探索分析
Step 4: シンプル予測モデル
Step 5: Pre-shot Threat Index
```

最終成果物の候補:

```text
Pre-shot Threat Index
攻撃侵食度
Centroid-EA Threat
```

## データ保護方針

既存データは壊さない。

守るルール:

- `data/candidates/01_sorted/` は読み取り専用として扱う。
- `data/processed/` の既存CSVを上書きしない。
- `はるた/outputs/` の既存結果を上書きしない。
- 新しい検証結果は、必ず新しいフォルダか新しいファイル名で保存する。
- 実験用の中間ファイルは `data/interim/` または `はるた/outputs/experiments/` に保存する。
- GitHubには実データ・生成outputを上げない。

推奨する新規出力先:

```text
はるた/outputs/experiments/{dataset_name}/pre_shot_threat/
```

推奨する新規ファイル名:

```text
features_centroid_ea_by_frame.csv
labels_future_events_by_frame.csv
analysis_spearman_shot_in_3s.csv
analysis_mannwhitney_shot_in_3s.csv
model_logistic_shot_in_3s_summary.csv
pre_shot_threat_index_by_frame.csv
```

既存CSVを加工する場合も、元ファイルは変更せず、新しいCSVとして出す。

## 現在ある主要データ

重心:

```text
data/processed/{dataset_name}/team_centroids_by_frame.csv
data/processed/{dataset_name}/team_centroids_by_sequence.csv
```

EA:

```text
はるた/outputs/tables/{dataset_name}/effective_area.csv
```

入力XML:

```text
data/candidates/01_sorted/{dataset_name}/01_tracking_core/{match_id}_tracker_box_data.xml
data/candidates/01_sorted/{dataset_name}/03_player_master/{match_id}_tracker_box_metadata.xml
```

今後確認する必要があるもの:

```text
シュートイベント
ゴールイベント
ボール位置
PA侵入判定に使える座標
攻撃方向
```

## 分析設計

### 1. 特徴量

時刻 `t` のEA・重心から作る。

候補:

```text
att_centroid_x
att_centroid_y
def_centroid_x
def_centroid_y
centroid_gap_x
centroid_gap_y
centroid_distance
att_centroid_forward_speed
def_centroid_backward_speed
attacking_ea_m2
defensive_ea_m2
ea_ratio
attacking_ea_delta
defensive_ea_delta
ea_ratio_delta
```

重要な設計:

- 攻撃方向をそろえる。
- 攻撃方向に進むほど `forward_x` が大きくなるようにする。
- 前半・後半で攻撃方向が変わる場合は補正する。
- チームA/Bではなく、攻撃チーム/守備チーム基準で特徴量を作る。

### 2. 未来イベントラベル

時刻 `t` から未来 `N秒` 以内にイベントが起きるかを0/1で作る。

最初の候補:

```text
shot_in_1s
shot_in_3s
shot_in_5s
pa_entry_in_3s
final_third_entry_in_3s
goal_in_5s
```

最初の主目的変数は `shot_in_3s` を第一候補にする。

理由:

- 1秒は短すぎて配置変化の影響が出にくい可能性がある。
- 5秒は他の要因が入りすぎる可能性がある。
- 3秒は、配置状態と次の攻撃アクションの関係を見る初期値として扱いやすい。

### 3. 探索分析

最初は指標式を決めない。
EA・重心特徴量が未来イベントと関係しているかを見る。

使う分析:

```text
Spearman順位相関
Point-biserial相関
Mann-Whitney U検定
特徴量同士のPearson相関
```

使い分け:

- Spearman: 単調関係を見る。
- Point-biserial: 数値特徴量と0/1ラベルの関係を見る。
- Mann-Whitney U: シュートあり/なしで特徴量分布が違うか見る。
- Pearson: 特徴量同士の重複を確認する。

### 4. シンプルモデル

探索で関係がありそうなら、最初はロジスティック回帰で見る。

目的変数:

```text
shot_in_3s
```

説明変数の初期候補:

```text
att_centroid_x
def_centroid_x
centroid_gap_x
centroid_distance
attacking_ea_m2
defensive_ea_m2
ea_ratio
attacking_ea_delta
defensive_ea_delta
```

評価:

```text
ROC-AUC
PR-AUC
係数の符号
特徴量重要度
```

シュート発生は少数クラスになる可能性が高いので、Accuracyは主評価にしない。

### 5. 指標化

モデルや探索結果から、効きそうな特徴量を選ぶ。
その後、解釈しやすい式として `Pre-shot Threat Index` を作る。

最初から複雑にしない。

候補式の例:

```text
PSTI =
  z(攻撃重心の前進度)
+ z(守備重心の後退度)
+ z(攻守重心距離の変化)
+ z(攻撃EAの増加)
- z(守備EA)
+ z(EA比率)
```

実際の式は、探索分析とモデル結果を見てから決める。

## ロードマップ

### Phase 0: データ確認

目的:

```text
今あるデータで、シュート前危険度を検証できるか確認する。
```

やること:

- シュートイベントがどのファイルにあるか確認する。
- シュート時刻を `frame_number` または `match_time_s` に対応づけられるか確認する。
- 重心CSVとEA CSVを結合できるか確認する。
- 攻撃方向を補正できるか確認する。

成果物:

```text
はるた/reports/05_PRE_SHOT_DATA_AVAILABILITY.md
```

完了条件:

```text
shot_in_3s を作れるかどうかが判断できている。
```

### Phase 1: 特徴量テーブル作成

目的:

```text
t時点のEA・重心特徴量を1行1フレームで作る。
```

やること:

- 重心CSVとEA CSVを結合する。
- 攻撃チーム基準にそろえる。
- 攻撃方向をそろえる。
- EA比率、差分、速度系特徴量を作る。

成果物:

```text
はるた/outputs/experiments/{dataset_name}/pre_shot_threat/features_centroid_ea_by_frame.csv
```

完了条件:

```text
既存CSVを上書きせず、特徴量CSVが作成されている。
```

### Phase 2: 未来イベントラベル作成

目的:

```text
t時点の特徴量に対して、未来N秒以内のイベントを0/1で付与する。
```

やること:

- `shot_in_1s`
- `shot_in_3s`
- `shot_in_5s`
- 可能なら `pa_entry_in_3s`
- 可能なら `final_third_entry_in_3s`

成果物:

```text
はるた/outputs/experiments/{dataset_name}/pre_shot_threat/labels_future_events_by_frame.csv
```

完了条件:

```text
shot_in_3s の陽性数・陰性数が確認できている。
```

### Phase 3: 探索分析

目的:

```text
EA・重心特徴量が未来のシュート発生と関係するか確認する。
```

やること:

- Spearman順位相関
- Point-biserial相関
- Mann-Whitney U検定
- 特徴量同士の相関確認

成果物:

```text
はるた/reports/06_PRE_SHOT_EXPLORATORY_ANALYSIS.md
```

完了条件:

```text
どの特徴量が効きそうか、逆に弱そうかが整理されている。
```

### Phase 4: シンプルモデル

目的:

```text
EA・重心だけで shot_in_3s をどの程度説明できるか確認する。
```

やること:

- ロジスティック回帰
- ROC-AUC / PR-AUC
- 係数の符号確認
- 時間窓 1秒/3秒/5秒 の比較

成果物:

```text
はるた/reports/07_PRE_SHOT_MODEL_BASELINE.md
```

完了条件:

```text
EA・重心に予測情報があるか判断できている。
```

### Phase 5: 指標提案

目的:

```text
解釈しやすいPre-shot Threat Indexを提案する。
```

やること:

- 使う特徴量を絞る。
- 標準化方法を決める。
- 指標式を作る。
- 高スコア場面の可視化を行う。
- シュート前場面でスコアが上がるか確認する。

成果物:

```text
はるた/reports/08_PRE_SHOT_THREAT_INDEX.md
```

完了条件:

```text
指標の式、意味、限界、使い方が説明できる。
```

## 最初にやること

次の作業は Phase 0。

最初に確認すること:

```text
1. シュートイベントの所在
2. シュートイベントの時刻またはframe_number
3. 重心CSVとEA CSVの結合キー
4. 攻撃方向の補正方法
5. shot_in_3s が作れるか
```

この確認が終わるまで、モデル実装や指標式の確定はしない。

## 研究としての主張案

```text
既存のxGはシュート後の得点確率評価に強いが、シュートに至らない危険な攻撃局面を評価しにくい。
EPVやPitch Controlは強力だが、実装やデータ要求が重い。
そこで本研究では、チーム重心とEffective Areaを用いて、より簡易に計算可能なシュート前攻撃危険度指標を提案する。
```

## 注意点

- シュートイベントが少なすぎる場合、`shot_in_3s` だけでは検証が弱くなる。
- その場合は `PA侵入`、`final third侵入`、`ゴール前30m侵入` を代替目的変数にする。
- 1試合だけで結論を出しすぎない。
- まずは「可能性の検証」として扱う。
- 指標式は、探索分析とモデル結果を見てから決める。
