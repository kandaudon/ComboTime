# Combo Time — Design Notes / 設計ノート

Target / 対象：`index.html`（Combo Time - Mystery Spot Tour v7.0.8）

This document records the design decisions made while implementing a falling-block puzzle in a single HTML file, and the reasons behind them.
この文書は、単一HTMLファイルで落ちものパズルを実装する際に決めたことと、その理由の記録です。

## 1. Overall structure: a single file, on purpose / 全体構成：あえて単一ファイル

- HTML / CSS / JavaScript all live in one `index.html` (about 2,950 lines) / HTML / CSS / JavaScript を `index.html` 1ファイル（約2,950行）に収めている
- Zero dependencies on external libraries, CDNs, or image/audio files. Open the file in a browser and it runs / 外部ライブラリ、CDN、画像・音声ファイルへの依存はゼロ。ファイルをブラウザで開けばそのまま動く
- Pros: trivial to distribute and share, no setup, deploys to GitHub Pages as-is / 利点：配布と共有が簡単、環境構築が不要、GitHub Pagesにそのまま置ける
- Cons: diffs become hard to review as the file grows. If splitting, CSS first, then JS, is the natural order / 限界：ファイルが大きくなると差分レビューがしづらい。分割するなら CSS → JS の順が自然

## 2. Board and pieces / 盤面とピース

- The board is **4 columns × 20 rows** (`COLS = 4`, `ROWS = 20`). With 4 columns instead of the usual 10, lines are short and combos rarely break, making the game about chaining clears / 盤面は **横4列 × 20段**（`COLS = 4`, `ROWS = 20`）。一般的な10列ではなく4列にしたことで、1ラインが短く、コンボが途切れにくい「連続消し」中心のゲーム性になっている
- Pieces are the standard 7 Tetriminos (I / J / L / O / S / T / Z), defined as arrays / ピースは標準的なテトリミノ7種（I / J / L / O / S / T / Z）を配列で定義
- Wall kicks use separate tables for JLSTZ and I pieces (SRS-style); candidates are tried in order for each rotation transition (e.g. `0>1`) / 回転の壁蹴りは、JLSTZ用とI用で別テーブルを用意（SRS風）。回転前後の状態（例 `0>1`）ごとに補正候補を順に試す

## 3. Lock delay / 接地とロックディレイ

- The delay before a grounded piece locks depends on difficulty / ピースが接地してから固定されるまでの猶予（ロックディレイ）を難易度で変える
  - Normal: 1.0s (max 15 lock resets) / Normal：1.0秒（リセット上限15回）
  - Hard: 0.5s (max 10) / Hard：0.5秒（同10回）
  - Hell: 0.25s (max 5) / Hell：0.25秒（同5回）
- Capping the number of move/rotate resets prevents stalling forever / 移動や回転でタイマーをリセットできる回数に上限を設け、無限に引き延ばせないようにしている

## 4. Game loop: fixed timestep / ゲームループ：固定タイムステップ

- Logic updates at a **fixed 60 FPS** (`FIXED_DT = 1/60`) with an accumulator; rendering is left to `requestAnimationFrame` / ロジックは **60 FPS固定**（`FIXED_DT = 1/60`）で更新し、描画は `requestAnimationFrame` に任せるアキュムレータ方式
- At most 6 logic steps per frame, to avoid a spiral of death after tab switches / 1フレームで進められるステップは最大6回に制限し、タブ復帰時などの暴走を防ぐ
- An on-screen performance monitor (FPS and render CPU time) lets you check input latency and load in place — measurement tooling built into the game itself / 画面には FPS と描画CPU時間のパフォーマンスモニタを表示でき、入力遅延や処理負荷をその場で確認できる（ノウハウ検証のための計測をゲーム側に内蔵した例）

## 5. Scoring and combos / スコアとコンボ

- Base points per clear: `[0, 100, 300, 500, 800]` by lines cleared, × level / ライン消しの基礎点は `[0, 100, 300, 500, 800]`（同時消し数ごと）× レベル
- Combo bonus: add `(combo - 1) × 50`, then multiply the total by **combo × level** / コンボが続くほど加算：コンボボーナス `(コンボ数 - 1) × 50` を加えたうえで、**合計 × コンボ数 × レベル**
- A perfect clear (all clear) multiplies the result by **×15** / 全消し（パーフェクトクリア）はさらに **×15**
- Level = `floor(lines / 10) + 1`; higher levels raise drop speed and change the background / レベルは `消去ライン数 / 10 + 1`。レベルが上がると落下速度と背景が変わる
- Splitting scoring into "base → bonus → multiplier" steps keeps tuning points easy to find / スコア計算を「基礎点 → ボーナス加算 → 倍率」の順に分解してあるので、調整箇所が追いやすい

## 6. Input: three systems, one action set / 入力：3系統を同じ操作に集約

Keyboard, touch, and gamepad input all converge on the same game actions (move, rotate, drop, hold).
キーボード、タッチ、ゲームパッドの入力を、ゲーム側の操作（移動・回転・ドロップ・ホールド）に集約している。

- Keyboard: arrows + Z / X (rotate), Space (hard drop), C / Shift (hold), P (pause) / キーボード：矢印キー＋ Z / X（回転）、Space（ハードドロップ）、C / Shift（ホールド）、P（ポーズ）
- Touch: 3 switchable modes, including button layouts and flick input. Flick detection uses explicit constants (e.g. ≥20px horizontal movement within 350ms). Long-pressing the hard-drop button turns it into a soft drop / タッチ：3モードを切り替え可能。画面ボタン配置モード、フリック併用モードなど。フリック判定は「横移動20px以上かつ350ms以内」のような閾値を定数として明示している。ハードドロップボタンは長押しでソフトドロップに切り替わる
- Gamepad: polled input, with remappable buttons stored in localStorage / ゲームパッド：ポーリングで読み取り、ボタン割り当て（マッピング）を設定画面で変更でき、localStorageに保存する

The comparison study behind these choices lives in the Touch Input Lab (see [touch-input-lab.md](touch-input-lab.md)).
方式の比較検討そのものは [touch-input-lab.md](touch-input-lab.md) 側の実験で行っている。

## 7. Presentation and audio / 演出と音

- Backgrounds are drawn on Canvas: 20 world mystery spots (Nazca, Moai, Pyramids, ...) cycling by level, with 10 color palettes / 背景はレベルごとに世界のミステリースポット20種（ナスカ、モアイ、ピラミッド等）を Canvas で描画し、10種のカラーパレットを周回させる
- BGM and SFX are PSG-style sounds synthesized with the Web Audio API — no audio files. Tempo is adjusted with a multiplier (`TEMPO_SCALE`) / BGMと効果音は Web Audio API で波形を生成する PSG 風サウンド。音声ファイルを持たない。テンポには倍率（`TEMPO_SCALE`）をかけて調整している

## 8. Storage: safe localStorage wrappers / 保存：localStorageの安全ラッパ

- High scores (per difficulty) and gamepad settings are stored in localStorage / ハイスコア（難易度別）、ゲームパッド設定などを localStorage に保存
- All access goes through exception-safe wrappers (`safeGetStorage` / `safeSetStorage`), so storage failures (e.g. private mode) never stop the game / 読み書きは `safeGetStorage` / `safeSetStorage` という例外処理付きラッパ経由に統一し、プライベートモード等で保存に失敗してもゲームが止まらないようにしている
- Score names are entered by selecting initials, so free-form text is never rendered back as-is / ハイスコアの名前入力はイニシャル選択式で、自由入力テキストをそのまま表示に使わない

## 9. Built-in test support / テスト支援機能

- The game records the score history during play and shows a graph plus a board replay after game over / プレイ中のスコア推移を記録し、ゲームオーバー後にグラフと盤面リプレイで振り返れる機能を内蔵
- Building "verification features" into the game itself, not just play features, makes balance tuning decisions much easier / 「遊ぶための機能」ではなく「挙動を検証するための機能」をゲーム本体に入れておくと、バランス調整の判断がしやすくなる、という実験でもある
