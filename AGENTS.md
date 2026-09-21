# 作業ルール

- UE 5.8、Blueprintのみ。C++モジュールやC++基盤クラスを追加しない。
- 最初にREADME.mdとdocs/BLUEPRINT_SPEC.mdを読む。
- 指定されたContent配下のパスとアセット名を維持する。
- .uasset / .umapはUnreal Editorで生成・保存する。空ファイルや独自バイナリで代用しない。
- 現在の実装仕様はdocs/BLUEPRINT_SPEC.mdを正とする。
- UEエディターを操作する手段がない場合は制約を伝える。必要な手作業を具体化する。
- Blueprintで実装可能な要件を、許可なくC++実装へ置き換えない。
- FinishはOpeningへ戻る。Quit Gameへ置き換えない。
- 変更後は実施した検証と未実施の検証を明確にする。
- Saved / Intermediate / DerivedDataCache / Binariesはコミットしない。
- リポジトリはPublic。秘密情報やローカルの認証情報を含めない。
