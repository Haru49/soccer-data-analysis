# Analysis Summary

FC徳島 重心分析プロジェクトの現状まとめ。

## 目的

11対11のトラッキングデータを使い、攻撃中・守備中でチーム重心位置が統計的に異なるかを確認する。

## 重心の定義

各フレームで、GKを除いたフィールドプレーヤー10人の平均座標をチーム重心とする。

現在は、トラッキングXMLの `0〜1` 座標から `0.5` を引き、フィールド中央を原点にしてから重心を計算する。

```text
centroid_x = フィールドプレーヤー10人のx座標平均
centroid_y = フィールドプレーヤー10人のy座標平均
```

```text
x_centered = x - 0.5
y_centered = y - 0.5
```

## 攻撃/守備の定義

トラッキングXMLの `ballStatus` を使う。

```text
ballStatus = HOME
  homeチーム: attack
  awayチーム: defense

ballStatus = AWAY
  awayチーム: attack
  homeチーム: defense

ballStatus = BALLOUT / NEUTRAL
  分析対象外
```

## 比較グループ

```text
A_attack
A_defense
B_attack
B_defense
```

`mean_centroid_x` と `mean_centroid_y` は別々に分析する。

## 検定手順

```text
1. Shapiro-Wilk検定
   正規性の確認
   各グループのn数も確認し、nが3未満なら検定しない

2. Levene検定
   等分散性の確認

3. 一元配置分散分析
   4群の平均に差があるか確認

4. Tukey HSD
   ANOVAで有意差が出た場合のみ、どの群間に差があるか確認
```

## 現在の分析対象

現在の標準分析対象は `117093`。

```text
2023-11-25_117093_tsukuba_vs_tsukuba-b
```

入力:

```text
data/candidates/01_sorted/2023-11-25_117093_tsukuba_vs_tsukuba-b/01_tracking_core/117093_tracker_box_data.xml
data/candidates/01_sorted/2023-11-25_117093_tsukuba_vs_tsukuba-b/03_player_master/117093_tracker_box_metadata.xml
```

## 現在の結果

`117093` のANOVA結果:

```text
mean_centroid_x:
  p = 0.829025
  有意差なし
  注意: A_defenseで正規性に注意

mean_centroid_y:
  p = 0.964798
  有意差なし
```

## 解釈

現在の試合ID `117093`、現在の攻撃/守備判定、現在の局面集約方法では、攻撃中・守備中を含む4群の平均重心位置に統計的に有意な差があるとは言えない。

ただし、「差がない」と断定はしない。

別の試合、複数試合、別の攻撃/守備判定、別の集約方法では結果が変わる可能性がある。

## 見るべきファイル

```text
README.md
data/README.md
notebooks/README.md
outputs/README.md
reports/01_ANALYSIS_SUMMARY.md
```
