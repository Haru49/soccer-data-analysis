# Centroid / Effective Area Validation

重心と Effective Area の計算が、現行コードの定義どおりに出ているかを確認したメモ。

## 確認対象

```text
DATASET_NAME = 2023-11-25_117093_tsukuba_vs_tsukuba-b
```

対象Notebook:

```text
はるた/notebooks/重心/00_data_check.ipynb
はるた/notebooks/effective_area/00_effective_area.ipynb
```

対象出力:

```text
data/processed/2023-11-25_117093_tsukuba_vs_tsukuba-b/team_centroids_by_frame.csv
data/processed/2023-11-25_117093_tsukuba_vs_tsukuba-b/team_centroids_by_sequence.csv
はるた/outputs/tables/2023-11-25_117093_tsukuba_vs_tsukuba-b/effective_area.csv
```

## 結論

現行コードの定義に対して、重心とEAの計算値はXMLからの手計算と一致した。

ただし、研究として使う前に以下は決める必要がある。

```text
1. 重み付き重心の重み値
2. EAの定義が参照論文と完全に一致しているか
3. EAが0になるフレームの扱い
```

## 重心チェック

現行の重心定義:

```text
GKを除いた10人
座標は 0〜1 から中央基準へ変換
x_centered = x - 0.5
y_centered = y - 0.5
```

確認結果:

```text
検証対象: 先頭200行のうちXMLを読み込んだフレーム
最大誤差: 9.7e-17
不一致: なし
```

つまり、現在の `team_centroids_by_frame.csv` は、少なくとも検証した範囲ではXMLの選手座標から正しく再計算できる。

## 重み付き重心の注意

metadata XML の `position` は以下のような細かい表記になっている。

```text
CB, LB, RB, CM, CAM, LM, RM, CF, LW, RW
```

以前の設定は `DF`, `MF`, `FW` だったため、実データのpositionと一致せず、重み付きにしても多くの選手がデフォルト重みになっていた。

現在はNotebook側の `POSITION_WEIGHTS` を実データのposition表記に合わせた。

```python
POSITION_WEIGHTS = {
    'CB': 1.0,
    'LB': 1.0,
    'RB': 1.0,
    'CM': 1.0,
    'CAM': 1.0,
    'LM': 1.0,
    'RM': 1.0,
    'CF': 1.0,
    'LW': 1.0,
    'RW': 1.0,
}
```

現時点では全て1.0なので、単純平均と同じ結果になる。重み付き重心として研究に使うには、仮説に合わせて値を決める必要がある。

## EAチェック

現行のEA定義:

```text
GKを含む11人
座標は 0〜1 からメートル単位へ変換
x_m = x_raw * pitch_width
y_m = y_raw * pitch_height
守備EA = 守備側三角形のうち、周囲長36m未満の三角形の和集合
攻撃EA = 守備EA・採用済み攻撃EAと面積を持って重ならない攻撃三角形の和集合
```

PDFの手書き疑似コードを確認した結果、守備EAは「全有効三角形」ではなく「周囲長36m未満の守備三角形」だけを採用する定義だった。
そのため、EA実装をPDF定義に合わせて修正した。

確認結果:

```text
frame 293: defensive_ea_m2=192.999692, attacking_ea_m2=1223.796, n_defensive_triangles=6, n_attacking_triangles=8
frame 318: defensive_ea_m2=164.934000, attacking_ea_m2=1044.582, n_defensive_triangles=4, n_attacking_triangles=8
frame 498: defensive_ea_m2=96.857913, attacking_ea_m2=1024.947, n_defensive_triangles=4, n_attacking_triangles=6
frame 523: defensive_ea_m2=61.047000, attacking_ea_m2=1052.793, n_defensive_triangles=2, n_attacking_triangles=6
frame 548: defensive_ea_m2=93.177000, attacking_ea_m2=1189.167, n_defensive_triangles=2, n_attacking_triangles=6
```

EA出力はPDF定義に合わせた現行コードで再生成済み。

```text
行数: 3790
サンプリング: 有効フレーム25枚に1枚
守備EA平均: 127.515 m2
攻撃EA平均: 1111.105 m2
```

## EAが0になるフレーム

`effective_area.csv` には、守備EAが0になる行がある。

```text
守備EA 0 の行数: 437
攻撃EA 0 の行数: 0
```

これは、そのフレームで周囲長36m未満の守備三角形が作れなかったことを意味する。

研究で使う場合は、以下のどちらで扱うかを決める。

```text
1. 0をそのまま「守備EAなし」として使う
2. 外れ値・アルゴリズム上の失敗ケースとして別扱いする
```

## 修正したこと

```text
1. active NotebookのPROJECT_ROOT判定を修正
2. はるた/outputs に出るように出力先を明確化
3. 00_data_check.ipynb の POSITION_WEIGHTS を実データのposition表記に合わせた
4. PDFのEA定義に合わせ、守備EAを周囲長36m未満の守備三角形だけで計算するように修正
5. 00_data_check.ipynb と 00_effective_area.ipynb を再実行してCSVを更新
```

## 次に決めること

```text
1. 重み付き重心で、どのポジションにどの重みを置くか
2. 守備EAの0行を分析対象に含めるか
3. 攻撃EAの三角形採用順を、論文通りの順番として固定してよいか
4. 重心とEAを組み合わせたゴール脅威度指標をどう定義するか
```
