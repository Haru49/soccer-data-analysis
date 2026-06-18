# シュート前攻撃危険度指標 要件定義

## 1. 研究名称

```text
チーム重心とEffective Areaを用いたシュート前攻撃危険度指標の提案
```

作業名称:

```text
Centroid-EA Shot Probability
CE-xS
```

## 2. 基礎文献

```text
Beyond Expected Goals:
A Probabilistic Framework for Shot Occurrences in Soccer
```

- Authors: Jonathan Pipping-Gamón, Tianshu Feng, R. Paul Sabin
- arXiv: https://arxiv.org/abs/2512.00203
- Version: v2, 2026-01-26
- Status: preprint。Journal of Quantitative Analysis in Sportsへ投稿中と記載されている

論文では、以下を別々に推定している。

```text
xS(t) = 時刻tから次の1秒以内にシュートが起きる確率
xG(t) = 時刻tでシュートした場合にゴールになる確率
xG+(t) = xS(t) × xG(t)
```

本研究の第一段階では、xG+全体を再現しない。
チーム重心とEAを使って、xSに相当する部分を検証する。

## 3. 研究目的

時刻 `t` のチーム重心とEAから、同じ攻撃ポゼッション中の次の1秒以内にシュートが発生する確率を推定する。

主な研究質問:

```text
チーム重心とEAは、次の1秒以内のシュート発生を説明できるか。
```

追加の研究質問:

```text
1. 重心だけ、EAだけ、両方を使った場合で性能は変わるか。
2. ボール位置などの基本特徴量に、重心・EAを追加する価値はあるか。
3. 1秒、3秒、5秒で有効な特徴量は変わるか。
4. 高危険度と判定された場面は、実際の映像・配置として妥当か。
```

## 4. 研究の主張範囲

第一段階で主張するのは、以下までとする。

```text
EA・重心がシュート前状態の説明・予測に使える可能性
```

第一段階では、以下を主張しない。

```text
ゴール確率そのものを推定できる
既存xGより優れている
他リーグ・他チームにも一般化できる
選手個人の能力を評価できる
```

## 5. 対象データ

初期対象:

```text
2023-11-25_117093_tsukuba_vs_tsukuba-b
```

使用候補:

```text
data/processed/{dataset_name}/team_centroids_by_frame.csv
はるた/outputs/tables/{dataset_name}/effective_area.csv
data/candidates/01_sorted/{dataset_name}/... のイベントデータ
data/candidates/01_sorted/{dataset_name}/... のトラッキングデータ
```

既存データは変更・上書きしない。

## 6. 分析単位

基本単位は「攻撃チーム視点の1フレーム」とする。

1行に以下を持つ。

```text
試合
ハーフ
フレーム
時刻
ポゼッション
攻撃チーム
守備チーム
EA・重心特徴量
未来シュートラベル
```

結合キー:

```text
(event_period, frame_number)
```

`frame_number` はハーフごとにリセットされるため、単独では使用しない。
`match_time_ms` でも一致を検証する。

## 7. 対象局面

論文は、明確なボール保持があり、攻撃側がアタッキングサードにいる局面を対象にしている。

本研究では、次の2段階で確認する。

```text
Dataset A: 明確な攻撃ポゼッションがある全局面
Dataset B: 明確な攻撃ポゼッションがあり、ボールがアタッキングサードにある局面
```

Dataset Bを論文に近い主分析候補とする。
ただし、ボール位置と攻撃方向を正しく取得できることが前提。

## 8. 目的変数

### 主目的変数

```text
shot_in_1s
```

定義:

```text
時刻tから1秒以内に、
時刻tで攻撃中のチームが、
同じポゼッション内でシュートした場合に1。
それ以外は0。
```

### 感度分析

```text
shot_in_3s
shot_in_5s
```

1秒は基礎文献との比較用。
3秒・5秒は、EA・重心のようなチーム配置指標に必要な時間幅を確認するために使う。

### 将来の補助ラベル

```text
pa_entry_in_3s
final_third_entry_in_3s
goal_in_5s
```

これらは初期必須ではない。

## 9. ラベル作成ルール

- シュート時刻をフレームまたはミリ秒へ変換する。
- ラベル窓の終点は `t + N秒` とする。
- 現在の攻撃チームとシュートしたチームが一致することを確認する。
- ラベル窓内にポゼッションが変わった場合、変更後のシュートは現在状態の正例にしない。
- ハーフ終了をまたがない。
- 試合終了をまたがない。
- シュート発生後のフレームを、同じシュートの正例として重複利用しない。
- 時刻同期の許容誤差を記録する。
- 1秒ラベルは時刻ずれの影響を受けやすいため、3秒・5秒でも結果を確認する。

ポゼッション変更やデータ欠損で未来1秒を完全に観測できない行は、単純な0ではなく分析対象外候補とする。

## 10. 説明変数

### 重心特徴量

```text
att_centroid_forward_x
att_centroid_y
def_centroid_forward_x
def_centroid_y
centroid_gap_forward
centroid_gap_lateral
centroid_distance
att_centroid_forward_velocity
def_centroid_backward_velocity
centroid_gap_delta
```

### EA特徴量

```text
attacking_ea_m2
defensive_ea_m2
ea_share
ea_difference_m2
attacking_ea_delta
defensive_ea_delta
ea_share_delta
n_attacking_triangles
n_defensive_triangles
```

EA比率はゼロ除算を避けるため、以下を第一候補とする。

```text
ea_share =
attacking_ea_m2 /
(attacking_ea_m2 + defensive_ea_m2)
```

両方が0の場合は欠損扱いとする。

### 基本比較用特徴量

取得できる場合:

```text
ball_distance_to_goal
ball_angle_to_goal
ball_speed
ball_x
ball_y
```

基礎文献ではボールからゴールまでの距離が重要だった。
EA・重心の価値を示すには、ボール特徴量だけのモデルとの比較が必要。

## 11. 座標・方向

すべて攻撃方向基準に変換する。

```text
攻撃方向へ進むほど forward_x が大きい
相手ゴール中央を共通の基準点とする
前半・後半のサイド変更を補正する
```

チームA/Bの固定座標のままモデルへ入れない。

重心は正規化座標、EAは平方メートルなので、モデル入力前に標準化する。

## 12. 時系列特徴量

差分・速度は、直前行との差だけで作らない。

条件:

- 同じ試合
- 同じハーフ
- 同じポゼッション
- 同じ攻撃チーム

速度は実際の時間差で割る。

```text
velocity = (value_t - value_previous) / delta_time_seconds
```

EAは現状25有効フレームごとのサンプリングなので、時間差が一定とは限らない。
各行の実時間差を必ず使う。

## 13. モデル比較

最初は解釈しやすいロジスティック回帰を基準にする。

比較するモデル:

```text
M0: ボール基本特徴量のみ
M1: 重心特徴量のみ
M2: EA特徴量のみ
M3: 重心 + EA
M4: ボール基本特徴量 + 重心 + EA
```

EA・重心の追加価値は、主に `M0` と `M4` の差で判断する。

初期段階ではXGBoostを必須にしない。
ロジスティック回帰で関係が確認できた後、非線形モデルを追加する。

## 14. 探索分析

モデルの前に以下を確認する。

```text
正例・負例数
特徴量の欠損率
特徴量の分布
Spearman順位相関
Point-biserial相関
Mann-Whitney U検定
特徴量同士のPearson / Spearman相関
```

相関だけで結論を出さない。

## 15. データ分割

フレームをランダムに分割しない。
隣接フレームはほぼ同じ状態なので、情報漏洩が起きる。

分割単位:

```text
第一候補: 試合単位
第二候補: ポゼッション単位
```

現状はEAが1試合分しかないため、試合外汎化は評価できない。
初期結果は探索的検証として扱う。

複数試合が揃うまでは、ポゼッション単位のGroup Splitを使う。

## 16. 評価指標

主評価:

```text
Log Loss
PR-AUC
ROC-AUC
Brier Score
```

補助評価:

```text
Calibration plot
Precision / Recall
係数の符号と信頼区間
```

シュート発生は少数クラスになる可能性が高いため、Accuracyを主評価にしない。

## 17. ポゼッション集約

基礎文献はフレーム確率をポゼッション単位へ集約している。

初期候補:

```text
max CE-xS in possession
```

理由:

- ポゼッション中の最も危険な瞬間を表せる。
- 長いポゼッションほど機械的に値が大きくなる問題を抑えやすい。

将来候補:

```text
1 - product(1 - CE-xS_t)
```

第一段階ではフレーム予測を先に検証し、ポゼッション集約はその後に行う。

## 18. 可視化要件

最低限、以下を確認できるようにする。

```text
シュート前5秒のCE-xS時系列
高スコア局面のピッチ図
攻撃・守備重心の位置
攻撃EA・守備EA
実際のシュート時刻
```

モデル性能だけでなく、高スコア場面がサッカーとして妥当かを確認する。

## 19. データ保護・出力

読み取り専用:

```text
data/candidates/01_sorted/
data/processed/ の既存CSV
はるた/outputs/ の既存成果物
```

新規出力先:

```text
はるた/outputs/experiments/{dataset_name}/pre_shot_threat/
```

想定ファイル:

```text
features_centroid_ea_by_frame.csv
labels_future_shots_by_frame.csv
model_comparison.csv
predictions_by_frame.csv
predictions_by_possession.csv
```

既存ファイルを上書きしない。

## 20. 完了条件

### Phase 0

```text
シュートイベントの所在と時刻形式が分かる。
重心・EA・イベントを安全に結合できる。
shot_in_1sを作れるか判断できる。
```

### MVP

```text
M1〜M3をポゼッション単位分割で評価できる。
重心、EA、重心+EAの性能差が分かる。
高スコア場面を可視化できる。
```

### 研究成立の最低条件

```text
重心+EAが単独特徴量より一貫して改善する、
または特定の特徴量が複数の分析でシュート発生と関係する。
```

ボール特徴量を取得できた場合は、M0に重心・EAを加えることで性能が改善するかを追加条件とする。

## 21. 未決事項

実装前に確認する。

```text
1. シュートイベントが入っているファイル
2. イベント時刻とトラッキング時刻の同期方法
3. ポゼッションIDを既存データから作れるか
4. 攻撃方向を判定できる情報
5. ボール位置・速度を取得できるか
6. EAを現在の1/25サンプリングで使うか、全フレーム再計算するか
7. 守備EA=0の437行を有効値とするか、欠損・特殊状態として扱うか
8. 追加試合でEAを生成できるか
```

## 22. 初回実装の停止条件

以下が確認できるまで、モデル実装へ進まない。

```text
シュートイベントの時刻同期
攻撃方向
同一ポゼッション判定
正例数
重心とEAの結合整合性
```

最初の作業は、モデル作成ではなくデータ可用性確認とする。
