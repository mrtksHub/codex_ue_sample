# Blueprint実装仕様

この文書は、`codex_ue_sample` の現在の実装仕様を示します。

## 概要

- Unreal Engine 5.8
- Blueprintのみで実装
- Opening / MainTitle / Game / Ending の4 Level構成
- ゲーム用Pawnは使用しないUI中心の構成
- Level Blueprintには画面遷移ロジックを持たせない
- ゲーム固有ロジックではなく、再利用可能なゲーム全体の骨格を対象とする
- Cursor / Pause / Exit / Confirm Dialogを共通UI機能として実装

基本の画面遷移は次のループです。

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

EndingのFinishはアプリ終了ではなくOpeningへ戻ります。

MainTitleおよびGameのPause Menuからは、確認ダイアログを経由してゲームを終了できます。

## Level

| Level | 役割 |
| --- | --- |
| `/Game/Maps/L_Opening` | 起動画面。任意のキー入力でMainTitleへ進む |
| `/Game/Maps/L_MainTitle` | Start / Exit Gameを表示するタイトル画面 |
| `/Game/Maps/L_Game` | 仮のゲーム画面。Game ClearとPause機能を提供 |
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

主な役割:

- `NotifyScreenReady(Opening)`
- `WBP_Opening` を生成してViewportへ追加
- マウスカーソルを表示
- Any Key入力で `GoToScreen(MainTitle)`

Openingでは画面遷移入力をPlayerController側で処理します。

### BP_TitleController

アセット:

`/Game/Blueprints/MainTitle/BP_TitleController`

主な役割:

- `NotifyScreenReady(MainTitle)`
- `WBP_MainTitle` を生成してViewportへ追加
- マウスカーソルを表示
- `Set Input Mode Game And UI` を使用
- FocusableなButtonをフォーカス対象として設定
- Esc入力でExit Confirm Dialogを表示
- Confirm Dialog表示中のEscはCancelとして処理
- Confirm Dialogの多重生成を防止
- ConfirmのYesでゲーム終了、No / CancelでTitleへ復帰

### BP_GameController

アセット:

`/Game/Blueprints/Game/BP_GameController`

主な役割:

- `NotifyScreenReady(Game)`
- `WBP_GameHUD` を生成してViewportへ追加
- マウスカーソルを表示
- `Set Input Mode Game And UI` を使用
- FocusableなButtonをフォーカス対象として設定
- Game通常状態のEscでPause Menuを表示
- Pause開始時に `Set Game Paused(true)` 相当の処理を実行
- Pause Menu表示中のEscでResume
- Confirm Dialog表示中のEscはCancelとして処理
- Return to Title / Exit Gameは共通Confirm Dialogを経由
- Resume / Return to TitleのYes / Exit GameのYesでPause状態を解除
- Title遷移時にPause関連UI参照を解放

Pause中でもEsc入力を処理できるよう、EscイベントはPause中の実行を許可しています。

### BP_EndingController

アセット:

`/Game/Blueprints/Ending/BP_EndingController`

主な役割:

- `NotifyScreenReady(Ending)`
- `WBP_Ending` を生成してViewportへ追加
- マウスカーソルを表示
- Focusableな `Btn_Finish` をフォーカス対象として設定

UIのFocus対象には、画面全体のUserWidgetではなくFocusableなButtonを使用します。

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

主なUI:

- `Btn_Start`
- `Btn_ExitGame`

Start:

```text
Btn_Start.OnClicked
  ↓
Get Game Instance
  ↓
Cast To BP_GameInstance
  ↓
GoToScreen(Game)
```

Exit Game:

```text
Btn_ExitGame
  ↓
Confirm Dialog: "Exit the game?"
  ├─ Yes → UIを閉じてゲーム終了
  └─ No  → Confirm Dialogを閉じてTitleへ復帰
```

Title通常状態でEscを押した場合も同じExit Confirm Dialogを表示します。

Confirm Dialog表示中のEscはNo / Cancel相当として扱います。

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

Pause Menuの生成・状態管理はGame HUDではなく `BP_GameController` 側を中心に行います。

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

Finishはゲーム終了ではなくOpeningへ戻る処理です。

### WBP_ConfirmDialog

アセット:

`/Game/UI/Common/WBP_ConfirmDialog`

再利用可能な共通確認ダイアログです。

主なUI:

- `Text_Message`
- `Btn_Yes`
- `Btn_No`

結果通知:

- `OnConfirmed`
- `OnCancelled`

Yes / No / Esc Cancelの結果をEvent Dispatcherで呼び出し側へ通知します。

TitleのExit Game、Pause MenuのReturn to Title、Pause MenuのExit Gameで共通利用します。

Confirm Dialog表示中は新しいConfirm Dialogを重複生成しません。

### WBP_PauseMenu

アセット:

`/Game/UI/Common/WBP_PauseMenu`

主なUI:

- `Btn_Resume`
- `Btn_ReturnToTitle`
- `Btn_ExitGame`

各ButtonはFocusableです。

動作:

```text
Resume
  → Pause Menuを閉じる
  → Pause解除
  → Gameへ復帰

Return to Title
  → Confirm Dialog
     ├─ Yes → Pause解除 → MainTitle
     └─ No / Esc → Pause Menu

Exit Game
  → Confirm Dialog
     ├─ Yes → Pause解除 → ゲーム終了
     └─ No / Esc → Pause Menu
```

Pause Menu表示中のEscはResumeと同じ動作です。

## Esc入力の優先順位

Esc入力は、現在最前面にあるUI状態に対して1段階だけ処理します。

```text
1. Confirm Dialog表示中
   Esc → Cancel / No

2. Pause Menu表示中
   Esc → Resume

3. Game通常状態
   Esc → Pause Menu表示

4. MainTitle通常状態
   Esc → Exit Confirm Dialog表示
```

1回のEsc入力で複数階層を同時に戻らないようにします。

例:

```text
Game
  ↓ Esc
Pause Menu
  ↓ Return to Title
Confirm Dialog
  ↓ Esc
Pause Menu
```

このEsc入力でさらにGameまで戻ることはありません。

## Pause状態管理

GameのPause Menu表示時にゲーム進行を停止します。

以下ではPause状態を解除します。

- Resume
- Return to Title の Yes
- Exit Game の Yes

Return to TitleではPause解除後にLevel遷移します。

No / Cancelの場合はPause状態を維持し、Pause Menuへ戻ります。

Level遷移後にPause状態やPause Menuの参照が残らないように管理します。

## Cursor / Input Mode / Focus

- Opening / MainTitle / Game / Endingの全画面でマウスカーソルを表示
- MainTitle / Game / Confirm Dialog / Pause Menuでは `Set Input Mode Game And UI` を利用
- UI操作時のFocus対象はFocusableなButton
- Non-Focusable UserWidgetを直接Focus対象にしない

これにより、過去に発生していた以下の警告を回避します。

```text
InputMode:UIOnly - Attempting to focus Non-Focusable widget
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
- Opening / MainTitle / Game / Endingでカーソル表示
- MainTitleでExit Game Confirmを表示
- ConfirmのNo / Esc CancelでTitleへ復帰
- Standalone GameでEscによるPause Menu表示を含むUI遷移を確認
- Pause MenuのResume / Return to Title / Exit Gameの一連の遷移を確認
- Confirm DialogからEscで1段階だけ戻ることを確認
- Pause解除後のGame復帰およびTitle遷移を確認
- StandaloneでExit Gameの終了動作を確認
- Confirm Dialogの多重生成防止を確認
- UI Focus設定によるNon-Focusable widget警告が新規発生していない
- Windowsパッケージ生成成功
- Windowsパッケージ版で基本画面遷移を確認

Windows PackagingではArchitectureに明示的なx64指定をせず、`Project Default` を使用した構成で動作確認しています。

## 現在のスコープ外

以下は現在実装していません。

- セーブデータのファイル永続化
- Settings画面
- 実ゲームロジック
- Pawn / Character
- 戦闘
- グリッド
- ユニット
- カメラ制御
- 本番用アート / オーディオ

このプロジェクトは、UE 5.8のBlueprintでLevel、GameMode、PlayerController、UMG Widget、GameInstanceを組み合わせ、画面遷移・Pause・Exit・Confirm Dialogなどのゲーム共通骨格を確認するための最小サンプルです。
