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
