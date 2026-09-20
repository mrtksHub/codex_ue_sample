# 作業ルール

- UE 5.8、Blueprintのみ。C++モジュールやC++基盤クラスを追加しない。
- 最初にREADME.md、docs/HANDOFF.md、docs/BLUEPRINT_SPEC.mdを読む。
- 指定されたContent配下のパスとアセット名を維持する。
- .uasset / .umapはUnreal Editorで生成・保存する。空ファイルや独自バイナリで代用しない。
- このひな形には実アセットがない。設定ファイルだけを根拠に「実装済み」「動作済み」と報告しない。
- UEエディターを操作する手段がない場合は制約を伝える。必要な手作業を具体化する。
- Blueprintで実装可能な要件を、許可なくC++実装へ置き換えない。
- FinishはOpeningへ戻る。Quit Gameへ置き換えない。
- 変更後は実施した検証と未実施の検証をdocs/HANDOFF.mdに記録する。
- Saved / Intermediate / DerivedDataCache / Binariesはコミットしない。
- リポジトリはPublic公開をユーザーが指定済み。秘密情報やローカルの認証情報を含めない。
