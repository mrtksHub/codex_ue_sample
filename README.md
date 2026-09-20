# codex_ue_sample

Unreal Engine 5.8 / Blueprintのみで実装する、4画面の遷移サンプル。
公開先: https://github.com/mrtksHub/codex_ue_sample

## 現在の状態

**UE未インストール環境で作成した、実装前のプロジェクトひな形です。**
設定、フォルダー構成、実装仕様、引き継ぎ手順を収録しています。
`.umap` / `.uasset` はまだ存在せず、画面・Blueprint・変数・遷移は未実装です。
起動、PIE、パッケージングの動作確認も未実施です。
設定には完成時のアセットパスを記載しているため、作成前に開くと
既定Map / GameMode / GameInstanceが見つからない警告が出る可能性があります。
空ファイルをUEアセットとして扱うことはしません。

## 画面の流れ

```text
L_Opening -- Any Key --> L_MainTitle -- Start --> L_Game
    ^                                             |
    +--------------- Finish -- L_Ending <-- Game Clear
```

- Opening: 仮ロゴと「Press Any Key」
- MainTitle: 「Start」ボタン
- Game: 仮背景と「Game Clear」ボタン
- Ending: 「Ending」表示と「Finish」ボタン。アプリ終了ではなくOpeningへ戻る。
- GameInstance: 共通データと画面遷移の入口。終了後の永続保存は今回の対象外。
- Grid / Units / Battle / Camera: 将来ゲーム画面に追加。今回は実装しない。

## 別PCでの続き

1. UE 5.8とGitをインストールする。Blueprintのみなので、このサンプル用のC++ビルド環境は不要。
2. リポジトリをcloneする（未公開の場合は、このフォルダーをそのままコピーする）。
3. Codexでリポジトリを開き、`AGENTS.md` と `docs/HANDOFF.md` を読ませる。
4. `docs/BLUEPRINT_SPEC.md` に従い、Unreal Editorで実アセットを作成する。
5. `docs/HANDOFF.md` の確認項目を実施してからアセットをコミットする。

Contentの空ディレクトリは `.gitkeep` で保持します。
UEのバイナリアセットもGitに含めます。大型のArt/Audioを追加する場合は、追加前にGit LFS導入を検討してください。

## 初回コミットと公開

Gitのユーザー名とメールアドレスを設定済みの環境で実行します。

```powershell
git add .
git commit -m "chore: scaffold UE 5.8 Blueprint screen flow project"
```

GitHub CLIを利用する場合は、`mrtksHub` で認証したうえで実行します。
この操作でPublicリポジトリを作成し、コミットをpushします。

```powershell
gh auth login
gh repo create mrtksHub/codex_ue_sample --public --source=. --remote=origin --push
```

既にリポジトリが存在する場合は再作成せず、既存のremoteと内容を確認してください。
