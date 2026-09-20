# Blueprint実装仕様

この文書は未実装の設計です。各項目をUE 5.8で作成して検証します。

## 必須アセット

| パス（Contentからの相対パス） | 種類 / 親クラス |
| --- | --- |
| Maps/L_Opening | Level |
| Maps/L_MainTitle | Level |
| Maps/L_Game | Level |
| Maps/L_Ending | Level |
| Blueprints/Core/BP_GameInstance | GameInstance |
| Blueprints/Core/BP_GameModeBase | GameModeBase |
| Blueprints/Opening/BP_OpeningController | PlayerController |
| Blueprints/MainTitle/BP_TitleController | PlayerController |
| Blueprints/Game/BP_GameController | PlayerController |
| Blueprints/Ending/BP_EndingController | PlayerController |
| UI/Opening/WBP_Opening | UserWidget |
| UI/MainTitle/WBP_MainTitle | UserWidget |
| UI/Game/WBP_GameHUD | UserWidget |
| UI/Ending/WBP_Ending | UserWidget |

型の定義用に `Blueprints/Core/Types/` を追加し、以下を作成します。

| 型 | 初期の定義 |
| --- | --- |
| E_ScreenState | Opening, MainTitle, Game, Ending |
| ST_PlayerData | PlayerName: String（空）、PlayerLevel: Integer（1） |
| ST_Settings | MasterVolume: Float（1.0）、BGMVolume: Float（1.0）、SEVolume: Float（1.0） |
| ST_SaveData | SaveVersion: Integer（1）、SavedChapter: Integer（0）、HasSave: Boolean（false） |

## BP_GameInstance

変数:

| 名前 | 型 | 初期値 |
| --- | --- | --- |
| GameState | E_ScreenState | Opening |
| CurrentChapter | Integer | 0 |
| PlayerData | ST_PlayerData | 上記の構造体初期値 |
| Settings | ST_Settings | 上記の構造体初期値 |
| SaveData | ST_SaveData | 上記の構造体初期値 |
| bTransitionInProgress | Boolean | false |

`GameState` はこのサンプル独自の画面状態です。UEのGameStateクラスとは別です。
SaveDataは将来用のメモリー上のデータであり、ディスク保存は実装しません。

関数 `GoToScreen(Target: E_ScreenState)`:

1. bTransitionInProgressがtrueならreturn。
2. trueにする。
3. TargetでSwitchし、対応Levelを `Open Level (by Object Reference)` で開く。
4. 4つのLevelの参照を明示的に設定する。文字列によるパスの組み立ては不要。

関数 `NotifyScreenReady(Screen: E_ScreenState)`:

1. GameStateをScreenにする。
2. bTransitionInProgressをfalseにする。

各ControllerのBeginPlayからNotifyScreenReadyを呼びます。
これにより各Levelを直接PIE起動した場合も画面状態が一致します。
LevelをまたいでもGameInstanceは同じインスタンスなので、共通データをBeginPlayで初期化し直しません。

## GameModeとControllerの選択

BP_GameModeBaseのDefault Pawn ClassはNoneにします。
`Get Player Controller Class to Spawn` をOverrideし、`Get Current Level Name`
（Remove Prefix String=true）の結果で以下のController Classを返します。
UE 5.8のエディターでOverrideと返り値の型を確認して実装してください。

| Level名 | Controller Class |
| --- | --- |
| L_Opening | BP_OpeningController |
| L_MainTitle | BP_TitleController |
| L_Game | BP_GameController |
| L_Ending | BP_EndingController |

未一致時はBP_OpeningControllerを返し、開発時の警告を出します。
各LevelのWorld Settingsは共通BP_GameModeBaseを使用します。
Level Blueprintには遷移処理を書きません。

## UIと入力

各ControllerのBeginPlay:

1. Get Game Instance → Cast to BP_GameInstance → NotifyScreenReadyで対応状態を通知。
2. Create Widget（対応WBP、Owning Player=self）→ Add to Viewport。
3. 入力モードとフォーカスを設定する。

Opening:

- WBP_Openingは仮ロゴ「CODEX UE SAMPLE」と「Press Any Key」を中央表示。
- Set Input Mode Game Only。カーソルは非表示。
- BP_OpeningControllerのAny KeyのPressed → GoToScreen(MainTitle)。
- マウス移動では遷移しない。キー押下・マウスボタン・ゲームパッドボタンを実機で確認する。
- PIEでは入力取得にビューポートのクリックが必要な場合があるため、Standaloneでも確認する。

その他の画面:

- Set Input Mode UI Only、Show Mouse Cursor=true。
- Widgetを表示後、対応ボタンへSet User Focus。ボタンのIs Focusable=true。
- WidgetのOnClicked → Get Game Instance → Cast → GoToScreen(次の画面)。
- MainTitle: Start → Game。
- Game: 色付き背景（UMG Border等）とGame Clear → Ending。
- Ending: EndingテキストとFinish → Opening。
- マウスとキーボードによるボタン操作を確認する。
- データ更新や遷移先のパス解決はWidget内へ分散させない。

## Levelと設定

Empty Levelを4つ作成し、指定パスで保存します。今回はUMGの仮背景でよく、ゲーム用のPawnは不要です。
すべてのBlueprintをCompile / Saveした後、Project Settingsで以下を確認します。

- Maps & Modes / Editor Startup Map: L_Opening
- Maps & Modes / Game Default Map: L_Opening
- Maps & Modes / Default GameMode: BP_GameModeBase
- Maps & Modes / Game Instance Class: BP_GameInstance
- Packaging / List of maps to include: 4つすべて

Configには上記の完成時のパスを先に設定済みです。実アセット作成後に一致を確認してください。
