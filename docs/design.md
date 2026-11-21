# Combo Time — Design Notes / 設計ノート

Target / 対象：`index.html`（Combo Time - Mystery Spot Tour v7.1.1）

This document records the design decisions made while implementing a falling-block puzzle in a single HTML file, and the reasons behind them.
この文書は、単一HTMLファイルで落ちものパズルを実装する際に決めたことと、その理由の記録です。

## 1. Overall structure: a single file, on purpose / 全体構成：あえて単一ファイル

- HTML / CSS / JavaScript all live in one `index.html` (about 3,500 lines) / HTML / CSS / JavaScript を `index.html` 1ファイル（約3,500行）に収めている
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
- Hard moves to the wall (see section 6) also register a lock reset when the piece lands on the ground, under the same cap / 壁までの一気移動（6章）でも、接地した場合は同じ上限の範囲でロックリセットを数える

## 4. Game loop: fixed timestep / ゲームループ：固定タイムステップ

- Logic updates at a **fixed 60 FPS** (`FIXED_DT = 1/60`) with an accumulator; rendering is left to `requestAnimationFrame` / ロジックは **60 FPS固定**（`FIXED_DT = 1/60`）で更新し、描画は `requestAnimationFrame` に任せるアキュムレータ方式
- At most 6 logic steps per frame, to avoid a spiral of death after tab switches / 1フレームで進められるステップは最大6回に制限し、タブ復帰時などの暴走を防ぐ
- An on-screen performance monitor (FPS and render CPU time) lets you check input latency and load in place — measurement tooling built into the game itself / 画面には FPS と描画CPU時間のパフォーマンスモニタを表示でき、入力遅延や処理負荷をその場で確認できる（ノウハウ検証のための計測をゲーム側に内蔵した例）
- Since v7.1.1 the monitor also shows the live state of every action input (see section 6) / v7.1.1からは、モニタに全アクションの入力状態も表示する（6章）

## 5. Scoring and combos / スコアとコンボ

- Base points per clear: `[0, 100, 300, 500, 800]` by lines cleared, × level / ライン消しの基礎点は `[0, 100, 300, 500, 800]`（同時消し数ごと）× レベル
- Combo bonus: add `(combo - 1) × 50`, then multiply the total by **combo × level** / コンボが続くほど加算：コンボボーナス `(コンボ数 - 1) × 50` を加えたうえで、**合計 × コンボ数 × レベル**
- A perfect clear (all clear) multiplies the result by **×15** / 全消し（パーフェクトクリア）はさらに **×15**
- Level = `floor(lines / 10) + 1`; higher levels raise drop speed and change the background / レベルは `消去ライン数 / 10 + 1`。レベルが上がると落下速度と背景が変わる
- Splitting scoring into "base → bonus → multiplier" steps keeps tuning points easy to find / スコア計算を「基礎点 → ボーナス加算 → 倍率」の順に分解してあるので、調整箇所が追いやすい

## 6. Input: three systems, one action set — and one timing rule / 入力：3系統を同じ操作に集約し、タイミングも統一

Keyboard, touch, and gamepad input all converge on the same game actions (move, rotate, drop, hold).
キーボード、タッチ、ゲームパッドの入力を、ゲーム側の操作（移動・回転・ドロップ・ホールド）に集約している。

v7.1.1 reworked this layer around a single timing constant, so the *feel* of a press is the same no matter which device produced it.
v7.1.1ではこの層を作り替え、1つの時間定数を中心に据えた。どのデバイスから入力しても、押した感覚が同じになるようにするためである。

### 6.1 One long-press threshold / 長押し閾値の統一

- `LONG_PRESS_MS = 250` is the only long-press threshold, shared by touch buttons, the flick area, and the keyboard / 長押しの閾値は `LONG_PRESS_MS = 250` の1つだけにし、タッチボタン・フリックエリア・キーボードで共有する
- Before v7.1.1 each path had its own numbers (e.g. flick was judged at release with ≥20 px within 350 ms). Unifying the constant makes behaviour predictable and tunable in one place / v7.1.1以前は経路ごとに数値がバラバラだった（例：フリックは離した時点で「20px以上かつ350ms以内」で判定）。定数を1か所に寄せると、挙動が予測しやすくなり、調整も1か所で済む

### 6.2 Press = one cell, long press = to the wall / 短押しは1マス、長押しで壁まで

- New actions `hardMoveLeft()` / `hardMoveRight()` move the piece until it collides with a wall or a block / 新しい操作 `hardMoveLeft()` / `hardMoveRight()` は、壁かブロックにぶつかるまでピースを移動させる
- Touch buttons, keyboard arrows, and flick input all follow the same rule: a short press moves one cell, holding for 250 ms performs the hard move / タッチボタン・矢印キー・フリックのいずれでも規則は同じで、短押しは1マス移動、250ms保持すると一気移動になる
- In the flick area, the direction is decided during `touchmove` once the swipe passes **16 px** (`SWIPE_THRESHOLD`); the long-press timer starts at that moment, and reversing the swipe while holding switches the hard move to the other wall / フリックエリアでは、`touchmove` 中にスワイプが **16px**（`SWIPE_THRESHOLD`）を超えた時点で方向を確定し、そこから長押しタイマーを開始する。長押し中に逆方向へ振り直すと、一気移動の向きも切り替わる

### 6.3 DROP: short release = hard, long press = soft / DROP：短く離せばハード、長押しはソフト

- The DROP button (renamed from "HARD DROP"), the ↓ key, and a down-swipe in the flick area all share one behaviour / DROPボタン（旧「HARD DROP」から改名）、↓キー、下方向スワイプは、すべて同じ挙動を共有する
  - Pressing starts a soft drop immediately / 押した瞬間にソフトドロップが始まる
  - Releasing in under 250 ms fires a hard drop / 250ms未満で離すとハードドロップが発動する
  - Releasing after 250 ms or more does **not** fire a hard drop — the press was a soft drop / 250ms以上で離した場合はハードドロップを発動しない。その押下はソフトドロップとして完結する
- The piece that was current at press time is remembered (`targetPieceRef`), and the hard drop is only fired if that same piece is still current at release. This prevents a press that started on one piece from dropping the next piece / 押下時点のピースを覚えておき（`targetPieceRef`）、離した時点で同じピースが現役の場合にだけハードドロップする。1つ前のピースへの押下が、次のピースを落としてしまう誤発を防ぐためである
- Cancelling the gesture (`touchcancel` / `mouseleave`) never fires a hard drop / ジェスチャのキャンセル（`touchcancel` / `mouseleave`）ではハードドロップを発動しない

### 6.4 Keyboard / キーボード

- Arrows: ← / → move (long press = hard move, §6.2), ↓ is the DROP action (§6.3), ↑ / X rotate clockwise, Z rotates counter-clockwise / 矢印キー：←／→は移動（長押しで一気移動、6.2節）、↓はDROP操作（6.3節）、↑／Xは時計回り、Zは反時計回りに回転
- Space: hard drop. C / Shift: hold. Esc: pause / Space：ハードドロップ。C／Shift：ホールド。Esc：ポーズ

### 6.5 Touch: 4 modes / タッチ：4モード

Modes cycle 1 → 2 → 3 → 4 → 1. All modes share the DROP button (§6.3) and a HOLD button of the same position and size (105 × 110 px, icon + label), so switching modes does not move the controls the player's fingers have learned.
モードは 1 → 2 → 3 → 4 → 1 と循環する。全モードで DROPボタン（6.3節）と、同じ位置・同じ大きさ（105 × 110px、アイコン＋ラベル）の HOLDボタンを共有し、モードを切り替えても指が覚えた配置が動かないようにしている。

- **MODE 1** — Buttons only: left / right buttons, clockwise rotate, HOLD, DROP. The centre is a LANE display; there is no flick area and no counter-clockwise button / ボタンのみ：左右ボタン、時計回り回転、HOLD、DROP。中央は LANE 表示で、フリックエリアも反時計回りボタンもない
- **MODE 2** — Left / right buttons combined with a flick area, with both clockwise and counter-clockwise rotate buttons / 左右ボタンとフリックエリアの併用。回転ボタンは時計回り・反時計回りの両方がある
- **MODE 3** — Flick area only for movement (no left / right buttons), clockwise rotate only. The counter-clockwise button is rendered but disabled, keeping the layout identical to MODE 4 / 移動はフリックエリアのみ（左右ボタンなし）、回転は時計回りのみ。反時計回りボタンは無効化した状態で表示され、MODE 4とレイアウトが同一に保たれる
- **MODE 4** — The MODE 3 layout with counter-clockwise rotate enabled / MODE 3のレイアウトで、反時計回り回転を有効化したもの

MODE 3 exists to test a one-rotation play style without changing the layout: keeping the disabled button visible means the comparison between MODE 3 and MODE 4 isolates "having CCW" as the only difference.
MODE 3は、レイアウトを変えずに片回転のプレイ感を検証するためにある。無効ボタンを表示したままにすることで、MODE 3とMODE 4の比較では「反時計回りがあるかどうか」だけが変数になる。

### 6.6 Binding structure / バインドの構成

The old generic `bindTouch()` was replaced by three binders, one per press semantics. Each binder tracks the active touch identifier, so multi-touch presses on different buttons do not interfere, and guards against the touch + mouse double-fire on the same press.
旧来の汎用 `bindTouch()` は廃止し、押下の意味ごとに3つのバインダへ分けた。各バインダは接触中のタッチ識別子を保持するため、別ボタンへのマルチタッチと干渉せず、同じ押下に対するタッチ＋マウスの二重発火も防ぐ。

- `bindInstantPress()` — single-fire actions (rotate, hold, and the HOLD panel, which now also fires on press instead of click) / 単発操作用（回転、ホールド。HOLDパネルも click ではなく押下で発動するようになった）
- `bindPressAndHoldMove()` — left / right buttons (§6.2) / 左右ボタン用（6.2節）
- `setupDropButton()` — the DROP button (§6.3) / DROPボタン用（6.3節）

### 6.7 Gamepad / ゲームパッド

- Gamepad: polled input, with remappable buttons stored in localStorage / ゲームパッド：ポーリングで読み取り、ボタン割り当て（マッピング）を設定画面で変更でき、localStorageに保存する
- Soft drop on the gamepad follows the same press/release tracking as the other paths, and also reports into the action-state monitor (§6.8) / ゲームパッドのソフトドロップも他の経路と同じ押下／解放の追跡を行い、アクション状態モニタ（6.8節）にも報告する

### 6.8 Action-state monitor / アクション状態モニタ

- The performance monitor (§4) now has rows for LEFT / RIGHT / DROP / ROT CW / ROT CCW / HOLD, each showing one of three states — none / pressed / long-pressed — with distinct colours / パフォーマンスモニタ（4章）に LEFT／RIGHT／DROP／ROT CW／ROT CCW／HOLD の行が増え、それぞれ「なし／あり／長押し」の3状態を色分けで表示する
- Every input path reports through a single function, `setActionMonitorState()`; states are reset on game start, pause, and window blur / 全入力経路が単一の関数 `setActionMonitorState()` 経由で報告する。状態はゲーム開始時・ポーズ時・ウィンドウのフォーカス喪失時にリセットされる
- This turns "did my long press actually register?" from a feeling into something visible on screen, which is what made the §6.1–6.3 unification practical to tune / 「長押しが本当に認識されたか」を体感ではなく画面で確認できるようにしたことで、6.1〜6.3節の統一を実際に調整できるようになった

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
- The action-state monitor (§6.8) extends the same idea to input: the timing rules of §6 can be verified on screen while playing, instead of only by reading the code / アクション状態モニタ（6.8節）は、同じ考え方を入力にも広げたもの。6章のタイミング規則を、コードを読まなくてもプレイ画面で検証できる
- Building "verification features" into the game itself, not just play features, makes balance tuning decisions much easier / 「遊ぶための機能」ではなく「挙動を検証するための機能」をゲーム本体に入れておくと、バランス調整の判断がしやすくなる、という実験でもある
