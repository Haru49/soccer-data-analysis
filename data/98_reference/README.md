# Reference Data

プログラムの入力ではないが、分析の考え方やアルゴリズム確認に使う資料を置く。

## papers

```text
papers/
```

現在は Effective Area などの確認用PDFを置いている。

このフォルダの資料はNotebookから直接読み込まない。

## 研究テーマ用文献

研究テーマ:

```text
チーム重心とEffective Areaを用いたシュート前攻撃危険度指標の提案
```

この研究では、論文本体PDFではなく、リンクと使い道を管理する。
PDFをローカルに保存する場合も、GitHubには上げない。

| 優先 | 文献 | 種別 | リンク | この研究での使い道 |
| --- | --- | --- | --- | --- |
| 1 | Beyond Expected Goals: A Probabilistic Framework for Shot Occurrences in Soccer | arXiv | https://arxiv.org/abs/2512.00203 | xGの「シュート後評価」だけではなく、シュート発生前の危険度を見る土台 |
| 2 | A framework for the fine-grained evaluation of the instantaneous expected value of soccer possessions | arXiv | https://arxiv.org/abs/2011.09426 | EPV。フレームごとの攻撃状態価値としてEA・重心を考える土台 |
| 3 | Evaluation of creating scoring opportunities for teammates in soccer via trajectory prediction | arXiv | https://arxiv.org/abs/2206.01899 | オフザボールの得点機会創出。EA・重心をチーム全体の配置価値として説明する材料 |
| 4 | Actions Speak Louder Than Goals: Valuing Player Actions in Soccer | arXiv / KDD系 | https://arxiv.org/abs/1802.07127 | VAEP。ゴールやシュート以外の行動価値を見る考え方 |
| 5 | Space evaluation at the starting point of soccer transitions | arXiv | https://arxiv.org/abs/2505.14711 | トランジション時のスペース価値。重心・EAの変化を見る方向の補強 |
| 6 | A Neighbor-based Approach to Pitch Ownership Models in Soccer | arXiv | https://arxiv.org/abs/2501.05870 | Pitch Ownership。EAを簡易的な空間支配指標として位置づける材料 |
| 7 | Prediction-based evaluation of back-four defense with spatial control in soccer | arXiv | https://arxiv.org/abs/2511.06191 | 守備EA・守備重心・守備ライン高さなど、守備側の空間指標の補強 |

読み進める順番:

```text
1. Beyond Expected Goals
2. EPV
3. Off-ball scoring opportunity
4. Pitch Ownership
5. VAEP
6. Space evaluation / transitions
7. Defensive spatial indicators
```
