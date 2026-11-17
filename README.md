# Combo Time — HTML Game Design Notes / HTMLゲーム設計ノート

This repository is not primarily about publishing a game. Its purpose is to **collect and share design know-how gained from building games in HTML / JavaScript**.

このリポジトリの目的は、ゲームそのものの公開ではなく、**HTML / JavaScriptでゲームを作る過程で得た設計ノウハウを蓄積し、共有すること**です。

As a testbed, it contains a falling-block puzzle game, "Combo Time - Mystery Spot Tour", implemented as a single HTML file. The game is a sample; the design records are the main content.

検証用の題材として、落ちものパズルゲーム「Combo Time - Mystery Spot Tour」を単一HTMLファイルで実装しています。ゲームはあくまでサンプルであり、中心にあるのは「どう設計したか」の記録です。

## Design documents / 設計ドキュメント

- [docs/design.md](docs/design.md) — Game design notes: board, rotation and wall kicks, fixed timestep, scoring and combos, input, audio, and storage / ゲーム本体の設計：盤面、回転と壁蹴り、固定タイムステップ、スコアとコンボ、入力、音、保存まわり
- [docs/touch-input-lab.md](docs/touch-input-lab.md) — A comparison experiment of 13 touch input methods (Touch Input Lab) / タッチ入力13方式の比較実験（Touch Input Lab）の設計と計測項目

## Sample game / サンプルゲーム

**Combo Time - Mystery Spot Tour** (`index.html`, v7.0.8)

- A 4-column × 20-row board: drop Tetriminos and complete horizontal lines / 横4列 × 20段の盤面にテトリミノを落とし、横一列を揃えて消す
- Chaining line clears builds combos with large score multipliers / 連続でラインを消すとコンボが伸び、スコアに大きく倍率がかかる
- Each level changes the background to another world mystery spot (20 spots in total) / レベルが上がるごとに、世界のミステリースポットを巡る背景（全20スポット）に切り替わる

### How to play / 遊び方

Just open `index.html` in a browser (no web server required). Also playable on GitHub Pages:

ブラウザで `index.html` を開くだけで遊べます（Webサーバ不要）。GitHub Pagesでも公開しています：

https://kandaudon.github.io/ComboTime/

| Input / 入力 | Action / 操作 |
| --- | --- |
| ← / → | Move / 移動 |
| ↓ | Soft drop / ソフトドロップ |
| ↑ / Z / X | Rotate / 回転 |
| Space | Hard drop / ハードドロップ |
| C / Shift | Hold / ホールド |
| P | Pause / ポーズ |
| Touch / タッチ | On-screen buttons and flick input, 3 switchable modes / 画面ボタン／フリックなど3つのモードを切り替え可能 |
| Gamepad / ゲームパッド | Supported; button mapping can be changed in settings / 対応。ボタン割り当ては設定画面で変更可能 |

## Experiment file / 実験ファイル

- `touch_input_lab_13.html` — Touch input method comparison experiment. See [docs/touch-input-lab.md](docs/touch-input-lab.md) for the design intent / タッチ入力の方式比較実験。設計意図は docs/touch-input-lab.md を参照

## Tech stack / 技術構成

- HTML5 / CSS3 / JavaScript (ES6+) only. No external libraries, CDN, or build step / HTML5 / CSS3 / JavaScript（ES6+）のみ。外部ライブラリ・CDN・ビルド工程なし
- Rendering on Canvas; audio generated with the Web Audio API (PSG-style) / 描画は Canvas、音は Web Audio API による PSG 風サウンド生成
- High scores and settings are stored in localStorage / ハイスコアと設定は localStorage に保存

## License / ライセンス

[MIT License](LICENSE)
