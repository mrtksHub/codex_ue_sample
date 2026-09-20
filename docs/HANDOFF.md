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
