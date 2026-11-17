# Touch Input Lab — Design Notes / 設計ノート

Target / 対象：`touch_input_lab_13.html`（Touch Input Lab - 13 Interfaces）

For mobile games, "which touch scheme is fast and error-resistant" is usually argued by feel alone. This experiment file lets you **switch between 13 touch input methods on one screen and measure them with the same metrics**. It was built as the preliminary study for Combo Time's touch modes.

モバイル向けゲームのタッチ操作は「どの方式が速く、誤操作が少ないか」が体感でしか語られがちです。この実験ファイルは、**13種類のタッチ入力方式を1つの画面で切り替えて試し、同じ指標で計測する**ためのものです。Combo Time のタッチモード設計の前段として作りました。

## The 13 methods / 比較している13方式

1. Fixed buttons / 固定ボタン
2. Virtual D-pad / 仮想十字キー
3. Fixed analog stick / 固定アナログスティック
4. Floating stick (origin follows your touch) / フローティングスティック（触れた位置が原点になる）
5. Flick / フリック
6. Swipe / スワイプ
7. Drag (relative movement) / ドラッグ（相対移動）
8. Absolute position / pointing / 絶対位置・ポイント
9. Trace / トレース
10. Long press / 長押し
11. Multi-touch / マルチタッチ
12. Pinch / ピンチ
13. Rotate / 回転

## Metrics / 計測項目

Whichever method is selected, the side panel shows the same metrics.
方式を切り替えても、右側のパネルで同じ指標が表示されます。

- Active pointers / 接触中の指の本数（ポインタ数）
- Input state / 入力状態（待機／接触中など）
- Position X / Y / 座標 X / Y
- Distance and speed / 距離と速度
- Direction and angle / 方向と角度
- Duration (ms) / 継続時間（ms）
- Event log (raw touch event flow) / イベントログ（生のタッチイベントの流れ）

The key point is separating "the method under test" from "the measuring instrument": if measurement code is rewritten per method, the numbers stop being comparable.
「方式の違い」と「計測の物差しを揃えること」を分けるのがポイントで、方式ごとに計測コードを書き直すと数値の比較ができなくなります。

## Applied to Combo Time / Combo Time への反映

- Based on this study, the game uses switchable modes combining button layouts and flick input / この実験での検討を経て、本体ではボタン配置とフリックを組み合わせた複数モードを切り替える設計にしました
- Expressing flick thresholds (distance and time) as constants also comes from this measurement work / フリック判定の閾値（距離・時間）を定数として切り出す考え方も、ここでの計測が元になっています
- Keeping the input experiment as a separate, standalone file means input methods can be verified without being dragged along by game rule changes / 入力方式の実験は本体から分離し、単体で開けるファイルにしておくと、ゲームのルール変更に引きずられず方式だけを検証できます
