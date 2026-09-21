# 別PCへの引き継ぎ

## 合意済み

- UE 5.8、Blueprintのみ、GitHubはPublic。
- リポジトリ名: mrtksHub/codex_ue_sample。
- Opening → MainTitle → Game → Ending → Openingの4 Level。
- BP_GameInstanceを最初から用意し、指定の5つの共通変数を持たせる。
- 今回は仮画面。戦闘、グリッド、ユニット、セーブの永続化は対象外。

## 現状（2026-09-20）

- [x] .uproject、Config、Git除外設定、Contentフォルダーの準備
- [x] Blueprint実装仕様の作成
- [ ] Unreal Editorでのプロジェクト読み込み
- [ ] 型定義、Blueprint、Widget、4 Levelの作成（未完了）
- [ ] Maps & Modesと各LevelのGameModeの確認
- [ ] PIE / Standaloneでの一周確認
- [ ] Windowsパッケージでの一周確認
- [x] GitHubリポジトリ作成とpush

公開先: https://github.com/mrtksHub/codex_ue_sample （Public）

公開時の検証: 初回コミットの `git diff --cached --check` が成功し、`main` の初回pushが成功。UEでの起動、PIE、Standalone、パッケージングの検証は未実施。

### 今回の実装準備結果

- 作業ブランチ: `feat/ue58-mcp-implementation`
- Unreal MCP接続: 成功（UE 5.8.2、プロジェクト `codex_ue_sample`）
- 作成・保存・AssetToolsで確認済み: `Blueprints/Core/Types/E_ScreenState`、`Blueprints/Core/Types/ST_PlayerData`、`Blueprints/Core/Types/ST_Settings`
- `E_ScreenState`の列挙子: Opening、MainTitle、Game、Ending
- `ST_PlayerData`: PlayerName（String）、PlayerLevel（Integer）を作成済み。既定値の確認は次回再確認する
- `ST_Settings`: Structアセットのみ作成済み。メンバー定義は未完了
- 未作成: `ST_SaveData`、4つの指定Level、GameInstance、GameMode、Controller、Widget
- Editor上のLevel: `/Temp/Untitled_0`
- MCPで実施した検証: AssetToolsのアセット検索・保存・存在確認、SceneToolsの現在Level取得、LogsToolsetのログ確認
- 未実施: PIE、Standalone、入力、パッケージング、画面遷移、データ保持

### 基礎型・Level作業の続行結果（2026-09-20）

- 再確認した既存型: `E_ScreenState`、`ST_PlayerData`、`ST_Settings`
- `ST_Settings`を修正し、`MasterVolume`、`BGMVolume`、`SEVolume`をFloat型として追加した。既定値の保存操作まで実施した
- `ST_SaveData`をEditorで新規作成し、`SaveVersion`（Integer）、`SavedChapter`（Integer）、`HasSave`（Boolean）を追加した。既定値の保存操作まで実施した
- `ST_PlayerData`は`PlayerName`（String）、`PlayerLevel`（Integer）を確認した。既定値の最終確認はLevel作成後に再確認する
- 4つのLevelは未作成。`L_Opening`の保存ボタンをSlate経由で押した際、Editorの保存ダイアログがMCP呼び出しをブロックしたため、そこで停止した
- `/Game/Maps/L_Opening`、`L_MainTitle`、`L_Game`、`L_Ending`の存在確認は未実施
- 今回はGameInstance、GameMode、Controller、Widgetを変更していない
- 次回はEditor上の保存ダイアログで`Content/Maps`を選び、4つのEmpty Levelを指定名で保存してから、AssetToolsで各`.umap`の存在とSceneToolsでロード可能性を確認する

### LevelEditorSubsystem調査（2026-09-20 継続）

- SlateのSave Asダイアログは再使用しない方針に変更した
- `LevelEditorSubsystem.NewLevel`をMCPのProgrammatic / Editor Scripting経路から呼べるか調査を開始した
- 調査中、Unreal MCPの`describe_toolset`およびEditor状態取得が応答待ちになり、ProgrammaticToolsetのスキーマ確認まで進めなかった
- 4つの`.umap`はまだ作成されていない。`Content/Maps`には`.gitkeep`のみ存在する
- 公式API呼び出し、Level作成、Levelロード、Struct既定値の再取得、`NewUserDefinedStruct1`の参照調査は未実施
- 次回はMCP接続が復旧したことを確認してから、既存LevelのAssetTools確認、ProgrammaticToolsetの利用可否、`LevelEditorSubsystem.NewLevel`の安全な呼び出しを順に行う

### Level作成の再試行結果（2026-09-20 継続）

- MCP接続は復旧した
- ProgrammaticToolsetの実行環境を確認した。登録済みMCPツールの呼び出し専用で、任意のPythonから`LevelEditorSubsystem.NewLevel`を直接呼ぶAPIは公開されていなかった
- SlateのSave Asは使用していない
- Editorが作成した`/Game/Maps/NewMap`（World）をAssetToolsのEditorネイティブなDuplicateで複製し、以下4つを作成・保存した
  - `/Game/Maps/L_Opening`
  - `/Game/Maps/L_MainTitle`
  - `/Game/Maps/L_Game`
  - `/Game/Maps/L_Ending`
- 各LevelはAssetToolsで存在を確認し、SceneToolsで4件すべて順番にロードできることを確認した。最終ロード状態は`/Game/Maps/L_Ending`
- `bIsPartitionedWorld=false`の個別プロパティはObjectToolsから公開されず、値の直接取得は未実施。元の`NewMap`がEmpty Levelとして作成されたWorldを複製している
- `ST_Settings`は`MasterVolume`、`BGMVolume`、`SEVolume`をFloat型として確認した。Structアセットの既定値はObjectToolsから直接取得できなかった
- `ST_SaveData`は`SaveVersion`（Integer）、`SavedChapter`（Integer）、`HasSave`（Boolean）を確認した。既定値はObjectToolsから直接取得できなかった
- `NewUserDefinedStruct1`はUserDefinedStructで、`memberVar_0`（Boolean）が存在する。参照元は空で、未使用の一時アセットと判断できるが、削除はしていない
- Output Logには既存のAutomationTest、GameFeatures設定、KismetScriptErrorが残っている。今回のLevel複製・ロード処理に固有のエラーかは切り分け未完了

### 次にやる作業

1. Unreal Editorで `ST_PlayerData` の既定値（PlayerName=""、PlayerLevel=1）を再確認する。
2. `ST_Settings` の3メンバー（各Float=1.0）を追加する。
3. `ST_SaveData`（SaveVersion=1、SavedChapter=0、HasSave=false）を作成する。
4. Empty Levelを `Maps/L_Opening`、`Maps/L_MainTitle`、`Maps/L_Game`、`Maps/L_Ending` として作成・保存する。
5. その後、仕様書の順序でGameInstance、Controller、Widget、GameModeを実装する。
6. 各段階でCompile / Saveし、PIE、Standalone、パッケージングを検証する。

Enum/Structの作成とLevel新規作成は、現行MCPの専用Toolだけでは完結せず、SlateによるEditor UI操作が必要になる。動的なSlate参照が安定しない場合は、Editorでの手作業を行い、実施内容をこのファイルへ追記する。空ファイルや独自バイナリで代用しない。

指定した基礎型の一部は作成済みですが、画面やBlueprintの実装は未完了です。エディターを用意しただけでこのひな形が動作するわけではありません。

## 作成順

1. UE 5.8でuprojectを開く。未作成アセット参照の警告が出る場合は閉じて、Empty Levelから作業する。
2. Blueprints/Core/TypesにEnumとStructを作成する。
3. 4つの空Levelを作成・保存する。
4. BP_GameInstanceを作成し、変数と遷移関数を実装する。
5. 各WBPと各Controllerを作成する。
6. BP_GameModeBaseを作成し、Levelに応じたController選択を実装する。
7. Project SettingsとWorld Settingsを確認し、全アセットをCompile / Saveする。
8. 次の検証を実施する。

## 動作確認

- Openingを起動すると仮ロゴとPress Any Keyが表示される。
- Any Key → Start → Game Clear → Finishで一周できる。
- 2周以上繰り返してもWidgetの重複や入力不能が起きない。
- ボタン連打で複数遷移しない。
- キーボード・マウス操作を確認する。利用可能ならゲームパッドも確認する。
- 各Levelから直接PIEを起動しても対応するControllerとWidgetになる。
- GameStateが現在のLevelと一致する。
- デバッグでCurrentChapterとPlayerDataを変更し、Levelをまたいでも保持されることを確認する。
- Output LogにBlueprint実行エラーがない。
- Standaloneで起動時にOpeningになる。
- Windows向けにパッケージし、エディター外でも4画面を一周できる。
- Content配下の必要な.umap / .uassetがgit statusに表示され、生成キャッシュは除外される。

## Codexへの引き継ぎ文

> README.md、AGENTS.md、docs/BLUEPRINT_SPEC.md、docs/HANDOFF.mdを読んで続きを進めてください。
> UE 5.8のBlueprintのみで、指定パスの4 Level、GameInstance、GameMode、Controller、Widgetを作成したいです。
> 現在は実アセット未作成のひな形です。まずUEの場所とエディター操作手段を確認し、
> 実アセットを作成・保存して、画面遷移とデータ保持を検証してください。
> C++に置き換えないでください。実行できなかった検証は明記してください。
### BP_GameInstance 実装試行（2026-09-20）

- NewUserDefinedStruct1 と /Game/Maps/NewMap は、AssetTools の参照確認で存在せず、Unreal Editor 側で削除済みであることを再確認した。
- /Game/Maps/L_Opening、L_MainTitle、L_Game、L_Ending、および E_ScreenState、ST_PlayerData、ST_Settings、ST_SaveData の存在を確認した。
- /Game/Blueprints/Core/BP_GameInstance を GameInstance 親で作成し、CurrentChapter (Integer)、PlayerData (ST_PlayerData)、Settings (ST_Settings)、SaveData (ST_SaveData)、bTransitionInProgress (Boolean) を追加した。CDO の既定値は ObjectTools で確認でき、指定値（0、各Structの指定値、false）と一致した。Compile は成功した。
- BlueprintTools の add_variable はユーザー定義Enumを受け付けず、GameState : E_ScreenState を専用APIだけで追加できなかった。Slate UIでの追加も試行したが、追加メニューを安定して操作できず、GameStateおよび関数グラフ（NotifyScreenReady / GoToScreen）は未実装のまま停止した。
- したがって、今回のBP_GameInstanceは部分作成状態であり、Widget実装へはまだ進めない。人間がBlueprint EditorでGameState変数をE_ScreenState型として追加し、2関数を実装・Compile・保存する必要がある。
### BP_GameInstance 関数グラフ実装（2026-09-21）

- 人間が追加した GameState : E_ScreenState = Opening、NotifyScreenReady(Screen : E_ScreenState)、GoToScreen(Target : E_ScreenState) を BlueprintTools で確認した。既存5変数（CurrentChapter、PlayerData、Settings、SaveData、bTransitionInProgress）も確認した。
- NotifyScreenReady は Graph DSL と Pin 接続で実装した。Function Entry の then → SetGameState（Screen入力）→ SetTransitioninProgress（false）の実行線、および Screen データ線を確認した。
- GoToScreen は、Graph DSL が if 用の Branch ノードを解決できず、ガード条件を含む実装を完了できなかった。仕様を変えた無条件遷移は作成していない。
- BP_GameInstance は Compile 成功、保存済み。LogBlueprint で当該BlueprintのCompile記録を確認し、関連Errorは検出されなかった。
- Widget実装には進んでいない。GoToScreenのBranchとSwitch/OpenLevelの実装を完了してから次フェーズへ進む。
### BP_GameInstance 完成確認（2026-09-21）

- 人間が追加した GoToScreen のグラフをBlueprintToolsで読み戻し、BranchのConditionに bTransitionInProgress のGet、False側に Set bTransitionInProgress=true、SetからSwitch on E_ScreenState、TargetからSelectionが接続されていることを確認した。True側は終端で何もしない。
- Switchの4実行Pinはそれぞれ Open Level (by Object Reference) に接続され、Level参照は L_Opening、L_MainTitle、L_Game、L_Ending の指定パスと一致した。
- NotifyScreenReady も既存の Screen → SetGameState、実行線 → Set bTransitionInProgress=false を再確認した。
- Compile成功、Save成功。LogBlueprintで今回のCompile記録を確認し、関連Errorは検出されなかった。
- BP_GameInstanceは完成状態と判断する。次フェーズはWidget設計・作成であり、Controller/GameModeはWidget連携方針を確定後に進める。今回Widget実装は行っていない。
### Widget実装試行（2026-09-21）

- `WBP_Opening` を作成し、CanvasRoot配下に `Text_Logo`（CODEX UE SAMPLE）と `Text_PressAnyKey`（Press Any Key）を追加した。中央配置用のCanvasスロット設定、変数公開、Compile、Save、Widget Tree確認まで完了した。遷移グラフは追加していない。
- `WBP_MainTitle` を作成し、CanvasRoot配下に `Btn_Start`（IsFocusable=true）と `Text_Start`（Start）を追加した。`OnClicked`イベントバインドまでは完了した。
- MainTitleのOnClicked処理で、BlueprintToolsからBP_GameInstanceのカスタム関数 `GoToScreen` 呼び出しノードを生成する経路を調査したが、関数ノードの有効なTypeIdを解決できなかった。GetGameInstance/Castノードは試行後に削除し、仕様を変更した処理は残していない。
- `WBP_MainTitle` はCompile/SaveとWidget Tree確認まで完了したが、OnClicked → GoToScreen(Game)は未実装。ユーザー指定の「エラーが発生したWidgetで停止」に従い、WBP_GameHUDとWBP_Endingは未作成。
- 次回は人間がMainTitleのOnClickedからBP_GameInstance.GoToScreenを呼ぶノードをEditorで追加するか、BlueprintToolsでカスタムBlueprint関数呼び出しノードを生成できる経路を確認してから再開する。

### Widget milestone update (2026-09-21)
- WBP_MainTitle was read back with BlueprintTools. Btn_Start OnClicked executes Get Game Instance, Cast To BP_GameInstance, then GoToScreen. Cast Object and execution pins are connected, Target is E_ScreenState::Game (NewEnumerator2), and compile succeeded.
- WBP_GameHUD was created at /Game/UI/Game/WBP_GameHUD. Widget tree: CanvasRoot, Background_Border (colored full-screen background), Btn_GameClear (IsFocusable=true), and Text_GameClear (Game Clear). OnClicked is bound and connected to Get Game Instance and Cast To BP_GameInstance. GoToScreen(Ending) is intentionally omitted because the known custom-function TypeId limitation remains.
- WBP_Ending was created at /Game/UI/Ending/WBP_Ending. Widget tree: CanvasRoot, Text_Ending (Ending), Btn_Finish (IsFocusable=true), and Text_Finish (Finish). OnClicked is bound and connected to Get Game Instance and Cast To BP_GameInstance. GoToScreen(Opening) is intentionally omitted for the same known limitation.
- WBP_GameHUD and WBP_Ending each compiled successfully, were saved, and their Widget Trees and relevant properties were read back with UMGToolSet/ObjectTools. No Controller or GameMode work was started.
- LogBlueprint and LogUMG queries found no Error entries related to these widgets. Human follow-up: add GoToScreen(Ending) after the successful cast in WBP_GameHUD, and GoToScreen(Opening) after the successful cast in WBP_Ending.

### Widget verification and Controller phase (2026-09-21)
- BlueprintTools/UMGToolSet verification completed for all four widgets. WBP_Opening contains Text_Logo = CODEX UE SAMPLE and Text_PressAnyKey = Press Any Key with no transition graph. WBP_MainTitle Btn_Start is focusable and its OnClicked chain is Get Game Instance -> Cast To BP_GameInstance -> GoToScreen(Target=Game). WBP_GameHUD Btn_GameClear is focusable and chains to GoToScreen(Target=Ending). WBP_Ending Btn_Finish is focusable and chains to GoToScreen(Target=Opening). Execution pins, cast Object references, enum values, and CompileWidgetBlueprint all verified.
- Controller assets were created for Opening, MainTitle, Game, and Ending. BP_OpeningController was the only controller saved before the required stop condition. Its ReceiveBeginPlay -> Get Game Instance -> Cast To BP_GameInstance scaffold is connected and saved.
- MCP could not create a NotifyScreenReady call node: BlueprintTools reported that type `|NotifyScreenReady` does not exist. Per instruction, no workaround or specification change was attempted, and implementation stopped at BP_OpeningController. The other Controller assets were not implemented or saved.
- Human Editor work still required: in each Controller, complete BeginPlay with NotifyScreenReady, Create Widget, Add to Viewport, input mode/cursor/focus settings, and Opening Any Key -> GoToScreen(MainTitle). Then compile/save/verify each Controller. Do not start GameMode until these are complete.
- Output Log showed compile records for the four widgets and BP_OpeningController; no related Error entries were returned by LogBlueprint or LogUMG.

### Controller standard-node implementation update (2026-09-21)
- Four PlayerController Blueprints were implemented with standard nodes and saved: BP_OpeningController, BP_TitleController, BP_GameController, and BP_EndingController.
- Each BeginPlay now executes Get Game Instance -> Cast To BP_GameInstance -> Create Widget (the corresponding WBP) -> Add to Viewport. Create Widget Class and OwningPlayer pins were verified.
- BP_OpeningController additionally executes Set Input Mode Game Only with the controller reference and Set Show Mouse Cursor(false). An `Any Key` input event (`KeyboardEvents|いずれかのキー`, Pressed pin) was created. Its GoToScreen(MainTitle) call remains intentionally absent.
- BP_TitleController, BP_GameController, and BP_EndingController execute Set Input Mode UI Only with the created widget as Widget To Focus, Show Mouse Cursor(true), and Set User Focus using Btn_Start, Btn_GameClear, and Btn_Finish respectively.
- NotifyScreenReady and GoToScreen calls remain intentionally unimplemented because their custom function TypeIds cannot be generated by BlueprintTools. This limitation was not re-investigated.
- All four Controllers were compiled and saved. Latest compile calls returned no error payloads; earlier transient target errors from the initial getter wiring were corrected by connecting each Create Widget typed return to the widget getter self pin. Output Log has no new related errors; historical errors remain in the session log.
- No GameMode or Level Blueprint work was performed.

### Controller verification and GameMode investigation (2026-09-21)
- BlueprintTools readback verified the human additions in all four Controllers. NotifyScreenReady is connected after the BeginPlay Cast success path with enum values Opening (NewEnumerator0), MainTitle (NewEnumerator1), Game (NewEnumerator2), and Ending (NewEnumerator3).
- BP_OpeningController Any Key Pressed is connected to a second Get Game Instance -> Cast To BP_GameInstance -> GoToScreen node with Target MainTitle (NewEnumerator1). Its BeginPlay chain reaches Create Widget(WBP_Opening), Add to Viewport, SetInputMode_GameOnly, and SetShowMouseCursor(false).
- BP_TitleController, BP_GameController, and BP_EndingController have the corresponding Create Widget, Add to Viewport, SetInputModeUIOnly, ShowMouseCursor(true), and SetUserFocus chains. Focus targets are Btn_Start, Btn_GameClear, and Btn_Finish; Widget To Focus is connected to the created widget. Widget class and OwningPlayer pins were verified.
- Output Log contains historical wiring errors from an earlier compile, but no new related errors after the human additions. The latest compile records are present for all four Controllers. No changes were made during this verification.
- GameMode implementation was not started. UE 5.8 requires a BP_GameModeBase derived from GameModeBase with Default Pawn Class set to None for this UI-only flow. A GameMode's PlayerControllerClass property is a single class; one class cannot select four controllers per level by itself.
- BlueprintTools node discovery exposes GameModeBase PlayerControllerClass get/set-related class data but no `Get Player Controller Class to Spawn` Blueprint node or override entry. Epic's UE 5.8 AGameModeBase API documents PlayerControllerClass and C++ spawn/virtual functions, but the requested per-level override is not exposed through the available BlueprintTools node catalog.
- Recommended Blueprint-only design: create per-level GameMode child Blueprints (or equivalent Editor-created overrides) with PlayerControllerClass set to BP_OpeningController, BP_TitleController, BP_GameController, and BP_EndingController, then assign each in the corresponding Level World Settings GameMode Override. If the project must retain one BP_GameModeBase only, human Editor verification of an override hook is required before proceeding.
- Required World Settings: assign the appropriate GameMode Override for each Level and keep Level Blueprints empty. Required Project Settings: Game Default Map and Editor Startup Map L_Opening, Game Instance Class BP_GameInstance, and Default GameMode reference to the selected base/default GameMode. Packaging should include all four maps.

### GameMode実装結果 (2026-09-21)

Widget + Controllerフェーズは`631e670`（`Implement UI screens and controllers`）として`origin/feat/ue58-mcp-implementation`へpush済み。以後のGameMode変更はこの時点では未コミット。

作成・Compile・Save済み:
- `/Game/Blueprints/Core/BP_GameModeBase`（GameModeBase、Default Pawn Class=None）
- `/Game/Blueprints/Opening/BP_OpeningGameMode` -> `BP_OpeningController`
- `/Game/Blueprints/MainTitle/BP_MainTitleGameMode` -> `BP_TitleController`
- `/Game/Blueprints/Game/BP_GameGameMode` -> `BP_GameController`
- `/Game/Blueprints/Ending/BP_EndingGameMode` -> `BP_EndingController`

ObjectToolsで各CDOの`defaultPawnClass`と`playerControllerClass`を確認した。SceneToolsで各Levelを順にロードし、WorldSettingsの`defaultGameMode`をObjectToolsで設定・再読込し、AssetToolsで保存した。設定結果はL_Opening->BP_OpeningGameMode、L_MainTitle->BP_MainTitleGameMode、L_Game->BP_GameGameMode、L_Ending->BP_EndingGameMode。

Config確認結果: EditorStartupMap/GameDefaultMapはL_Opening、GlobalDefaultGameModeはBP_GameModeBase、GameInstanceClassはBP_GameInstance。DefaultGame.iniのMapsToCookには4 Levelが既に含まれている。DefaultEngine.iniの既存Android File Server設定と認証値を含む差分は今回のコミット対象外とした。.codexとDefaultInput.iniも対象外。

Output Logの今回のGameMode Compile記録にErrorはない。過去のController配線エラー記録は履歴として残るが、GameMode関連ではない。PIEはまだ開始していない。
