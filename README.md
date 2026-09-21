# codex_ue_sample

Unreal Engine 5.8 / Blueprintのみで作成した、ゲーム全体の基本的な画面遷移とUI制御を確認するためのサンプルプロジェクトです。

## 現在の画面構成

```text
Opening
  ↓ Any Key
MainTitle
  ↓ Start
Game
  ↓ Game Clear
Ending
  ↓ Finish
Opening
```

ゲーム固有の戦闘やステージロジックではなく、ジャンルを問わず利用できる「ゲームの骨格」を主な対象としています。

## 実装済みの主な機能

- Opening / MainTitle / Game / Ending の4 Level構成
- 画面ごとのGameMode / PlayerController
- `BP_GameInstance` を介した共通画面遷移
- 全画面でのマウスカーソル表示
- MainTitleのExit Game
- 共通Confirm Dialog
- Game中のEscによるPause Menu
- Pause中のResume / Return to Title / Exit Game
- `Set Game Paused` を用いたPause状態管理
- Confirm Dialog → Pause Menu → Game / Title のEsc優先順位制御
- Confirm Dialogの多重生成防止
- Level遷移前のPause解除
- FocusableなButtonを対象としたInput Mode / Focus管理

## Pause / Exitの基本動作

```text
MainTitle
  ├─ Start → Game
  ├─ Exit Game → Confirm → Yes: Quit / No: MainTitle
  └─ Esc       → Confirm → Yes: Quit / Esc・No: MainTitle

Game
  └─ Esc → Pause Menu
            ├─ Resume → Game
            ├─ Return to Title → Confirm → Yes: MainTitle / No・Esc: Pause Menu
            └─ Exit Game       → Confirm → Yes: Quit / No・Esc: Pause Menu
```

Confirm Dialog表示中のEscはCancelとして扱い、1回のEsc入力で複数階層を同時に戻らないようにしています。

## 技術方針

- Unreal Engine 5.8
- Blueprintのみ
- Level Blueprintには画面遷移ロジックを持たせない
- `.uasset` / `.umap` はUnreal Editorで生成・編集・保存する
- ゲーム固有ロジックはまだ実装しない
- ローカル環境依存設定や認証情報はGitへ含めない

詳細なBlueprint構成と現在の仕様は [docs/BLUEPRINT_SPEC.md](docs/BLUEPRINT_SPEC.md) を参照してください。
