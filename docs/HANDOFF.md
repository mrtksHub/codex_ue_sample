# 別PCへの引き継ぎ

## 合意済み

- UE 5.8、Blueprintのみ、GitHubはPublic。
- リポジトリ名: mrtksHub/codex_ue_sample。
- Opening → MainTitle → Game → Ending → Openingの4 Level。
- BP_GameInstanceを最初から用意し、指定の5つの共通変数を持たせる。
- 今回は仮画面。戦闘、グリッド、ユニット、セーブの永続化は対象外。

## 現状

- [x] .uproject、Config、Git除外設定、Contentフォルダーの準備
- [x] Blueprint実装仕様の作成
- [ ] Unreal Editorでのプロジェクト読み込み
- [ ] 型定義、Blueprint、Widget、4 Levelの作成
- [ ] Maps & Modesと各LevelのGameModeの確認
- [ ] PIE / Standaloneでの一周確認
- [ ] Windowsパッケージでの一周確認
- [x] GitHubリポジトリ作成とpush

公開先: https://github.com/mrtksHub/codex_ue_sample （Public）

公開時の検証: 初回コミットの `git diff --cached --check` が成功し、`main` の初回pushが成功。UEでの起動、PIE、Standalone、パッケージングの検証は未実施。

UEアセットは未作成です。エディターを用意しただけでこのひな形が動作するわけではありません。

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
