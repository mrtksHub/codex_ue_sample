# Blueprint実装仕様

この文書は、`codex_ue_sample` の現在の実装仕様を示します。

## 概要

- Unreal Engine 5.8
- Blueprintのみで実装
- 4つのLevelを順番に遷移する最小サンプル
- ゲーム用Pawnは使用しないUI中心の構成
- Level Blueprintには画面遷移ロジックを持たせない

画面遷移は次のループです。

```text
L_Opening
  ↓ Any Key
L_MainTitle
  ↓ Start
L_Game
  ↓ Game Clear
L_Ending
  ↓ Finish
L_Opening
```

Quit機能は実装していません。EndingのFinishはアプリ終了ではなくOpeningへ戻ります。

## Level

| Level | 役割 |
| --- | --- |
| `/Game/Maps/L_Opening` | 起動画面。任意のキー入力でMainTitleへ進む |
| `/Game/Maps/L_MainTitle` | Startボタンを表示するタイトル画面 |
| `/Game/Maps/L_Game` | 仮のゲーム画面。Game Clearボタンを表示 |
| `/Game/Maps/L_Ending` | Ending表示とFinishボタンを表示 |

4 Levelとも画面固有のPlayerControllerをGameMode経由で使用します。

## 共通型

`/Game/Blueprints/Core/Types/` に以下の型を定義しています。

### E_ScreenState

- Opening
- MainTitle
- Game
- Ending

### ST_PlayerData

| メンバー | 型 | 初期値 |
| --- | --- | --- |
| PlayerName | String | 空文字 |
| PlayerLevel | Integer | 1 |

### ST_Settings

| メンバー | 型 | 初期値 |
| --- | --- | --- |
| MasterVolume | Float | 1.0 |
| BGMVolume | Float | 1.0 |
| SEVolume | Float | 1.0 |

### ST_SaveData

| メンバー | 型 | 初期値 |
| --- | --- | --- |
| SaveVersion | Integer | 1 |
| SavedChapter | Integer | 0 |
| HasSave | Boolean | false |

`ST_SaveData` は将来拡張用のメモリー上のデータ構造です。ファイルへの永続保存処理は現在実装していません。

## BP_GameInstance

アセット:

`/Game/Blueprints/Core/BP_GameInstance`

親クラス:

`GameInstance`

Levelをまたいで保持する共通状態と、画面遷移の入口を担当します。

### 変数

| 名前 | 型 | 初期値 |
| --- | --- | --- |
| GameState | E_ScreenState | Opening |
| CurrentChapter | Integer | 0 |
| PlayerData | ST_PlayerData | 構造体初期値 |
| Settings | ST_Settings | 構造体初期値 |
| SaveData | ST_SaveData | 構造体初期値 |
| bTransitionInProgress | Boolean | false |

`GameState` はこのサンプル独自の画面状態であり、Unreal EngineのGameStateクラスとは別です。

### NotifyScreenReady

```text
NotifyScreenReady(Screen)
  ↓
GameState = Screen
  ↓
bTransitionInProgress = false
```

各PlayerControllerのBeginPlayから呼び出します。

Level遷移後に `bTransitionInProgress` を解除し、現在のLevelと `GameState` を同期します。

### GoToScreen

```text
GoToScreen(Target)
  ↓
bTransitionInProgress ?
  ├─ true  → 何もしない
  └─ false
       ↓
     bTransitionInProgress = true
       ↓
     Switch on E_ScreenState
       ├─ Opening   → L_Opening
       ├─ MainTitle → L_MainTitle
       ├─ Game      → L_Game
       └─ Ending    → L_Ending
```

Level遷移には `Open Level (by Object Reference)` を使用します。

`bTransitionInProgress` により、連打などによる重複した画面遷移を防ぎます。

## GameMode構成

共通GameMode:

`/Game/Blueprints/Core/BP_GameModeBase`

- 親クラス: `GameModeBase`
- Default Pawn Class: `None`

画面ごとに子GameModeを用意し、それぞれ異なるPlayerControllerを使用します。

| Level | GameMode | PlayerController |
| --- | --- | --- |
| L_Opening | BP_OpeningGameMode | BP_OpeningController |
| L_MainTitle | BP_MainTitleGameMode | BP_TitleController |
| L_Game | BP_GameGameMode | BP_GameController |
| L_Ending | BP_EndingGameMode | BP_EndingController |

アセットパス:

- `/Game/Blueprints/Opening/BP_OpeningGameMode`
- `/Game/Blueprints/MainTitle/BP_MainTitleGameMode`
- `/Game/Blueprints/Game/BP_GameGameMode`
- `/Game/Blueprints/Ending/BP_EndingGameMode`

各LevelのWorld Settingsに対応するGameMode Overrideを設定しています。

単一GameMode内でLevel名を判定してControllerを切り替える方式は使用していません。

## PlayerController

### BP_OpeningController

アセット:

`/Game/Blueprints/Opening/BP_OpeningController`

BeginPlay:

```text
Get Game Instance
  ↓
Cast To BP_GameInstance
  ↓
NotifyScreenReady(Opening)
  ↓
Create Widget(WBP_Opening)
  ↓
Add to Viewport
  ↓
Set Input Mode Game Only
  ↓
Show Mouse Cursor = false
```

入力:

```text
Any Key Pressed
  ↓
Get Game Instance
  ↓
Cast To BP_GameInstance
  ↓
GoToScreen(MainTitle)
```

### BP_TitleController

アセット:

`/Game/Blueprints/MainTitle/BP_TitleController`

- `NotifyScreenReady(MainTitle)`
- `WBP_MainTitle` を生成してViewportへ追加
- Set Input Mode UI Only
- Show Mouse Cursor = true
- Widget To Focus = `Btn_Start`
- `Btn_Start` にSet User Focus

### BP_GameController

アセット:

`/Game/Blueprints/Game/BP_GameController`

- `NotifyScreenReady(Game)`
- `WBP_GameHUD` を生成してViewportへ追加
- Set Input Mode UI Only
- Show Mouse Cursor = true
- Widget To Focus = `Btn_GameClear`
- `Btn_GameClear` にSet User Focus

### BP_EndingController

アセット:

`/Game/Blueprints/Ending/BP_EndingController`

- `NotifyScreenReady(Ending)`
- `WBP_Ending` を生成してViewportへ追加
- Set Input Mode UI Only
- Show Mouse Cursor = true
- Widget To Focus = `Btn_Finish`
- `Btn_Finish` にSet User Focus

UI Only画面では、画面全体のUserWidgetではなくフォーカス可能なButtonを `Widget To Focus` に指定します。

## Widget

### WBP_Opening

アセット:

`/Game/UI/Opening/WBP_Opening`

表示:

- `CODEX UE SAMPLE`
- `Press Any Key`

画面遷移処理はWidget側には持たず、`BP_OpeningController` のAny Key入力で処理します。

### WBP_MainTitle

アセット:

`/Game/UI/MainTitle/WBP_MainTitle`

- `Btn_Start`
- 表示文字: `Start`
- Is Focusable = true

```text
Btn_Start.OnClicked
  ↓
Get Game Instance
  ↓
Cast To BP_GameInstance
  ↓
GoToScreen(Game)
```

### WBP_GameHUD

アセット:

`/Game/UI/Game/WBP_GameHUD`

- 仮の色付き背景
- `Btn_GameClear`
- 表示文字: `Game Clear`
- Is Focusable = true

```text
Btn_GameClear.OnClicked
  ↓
Get Game Instance
  ↓
Cast To BP_GameInstance
  ↓
GoToScreen(Ending)
```

### WBP_Ending

アセット:

`/Game/UI/Ending/WBP_Ending`

- `Ending` テキスト
- `Btn_Finish`
- 表示文字: `Finish`
- Is Focusable = true

```text
Btn_Finish.OnClicked
  ↓
Get Game Instance
  ↓
Cast To BP_GameInstance
  ↓
GoToScreen(Opening)
```

## Project Settings

現在の基本設定は次のとおりです。

| 項目 | 設定 |
| --- | --- |
| Editor Startup Map | `L_Opening` |
| Game Default Map | `L_Opening` |
| Default GameMode | `BP_GameModeBase` |
| Game Instance Class | `BP_GameInstance` |

PackagingのMapsToCookには以下4 Levelを含めています。

- `/Game/Maps/L_Opening`
- `/Game/Maps/L_MainTitle`
- `/Game/Maps/L_Game`
- `/Game/Maps/L_Ending`

各LevelではProject SettingsのDefault GameModeよりもWorld SettingsのGameMode Overrideが優先され、対応する画面用GameModeが使用されます。

## 動作確認済み範囲

以下を確認済みです。

- PIEで Opening → MainTitle → Game → Ending → Opening を一周
- Openingへ戻った後、再度MainTitleへ遷移可能
- Standalone Gameで同じ画面遷移を確認
- Windowsパッケージ生成成功
- Windowsパッケージ版で同じ画面遷移を確認
- UI Only画面のフォーカス設定によるNon-Focusable widget警告を解消

Windows PackagingではArchitectureに明示的なx64指定をせず、`Project Default` を使用した構成で動作確認しています。

## 現在のスコープ外

以下は現在実装していません。

- Quitボタン / Quit Game
- セーブデータのファイル永続化
- Settings画面
- 実ゲームロジック
- Pawn / Character
- 戦闘
- グリッド
- ユニット
- カメラ制御
- 本番用アート / オーディオ

このプロジェクトは、UE 5.8のBlueprintでLevel、GameMode、PlayerController、UMG Widget、GameInstanceを組み合わせた画面遷移構成を確認するための最小サンプルです。
