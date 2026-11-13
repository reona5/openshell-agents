# openshell-agents

[NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell) の sandbox で Claude Code と Codex を動かすための一式です。
どちらもサブスクリプションの OAuth(Claude の Team / Max、ChatGPT)で使えます。

- sandbox の中には本物のトークンを置きません。環境変数には placeholder が入り、OpenShell の proxy が許可した宛先への通信でだけ本物に差し替えます。
- 最初に開いている宛先は、各エージェントの API、GitHub(clone と fetch のみ。push は不可)、PyPI と npm のレジストリ(読み取りのみ)だけです。
- それ以外の宛先は `osa allow` でその場で開けます。sandbox を作り直す必要はありません。

OpenShell v0.1.7、Claude Code 2.1.318、Codex 0.165.0 で確認しました(2026-11-05、Apple Silicon の macOS と Docker Desktop)。

## 前提

- OpenShell v0.1.7 以降と、動いているローカル gateway
- Docker
- ホストにある `claude`、`codex`、`gh`(`gh auth login` 済み)、`jq`

macOS では gateway の設定が 1 か所必要です。[macos/README.md](macos/README.md) を見てください。

## 使い方

```sh
bin/osa setup                    # provider profile の登録とイメージのビルド
bin/osa login claude             # claude setup-token の長期トークンを貼る(表示されない)
bin/osa login codex              # gateway 専用に codex login し、更新を gateway に任せる
bin/osa login github             # gh auth token を登録する

bin/osa new work owner/repo      # sandbox を作り、中で owner/repo を clone する
bin/osa claude work              # Claude Code を起動する
bin/osa codex work               # Codex を起動する
bin/osa shell work               # シェルを開く
bin/osa allow work mcp.example.com   # 宛先を 1 つ開ける
bin/osa rm work
```

`login` で登録しなかった provider は sandbox に付きません。Claude Code だけ使う、といった構成もできます。

Claude Code を Team プランで使っている場合、初回の起動時に組織の managed settings の承認画面が出ます。

## 中身

| パス | 内容 |
|---|---|
| `bin/osa` | 上のコマンド |
| `image/` | OpenShell-Community の base イメージに、版を固定した Claude Code と Codex、Codex 用の起動スクリプトを足したもの |
| `policy/sandbox.yaml` | sandbox の policy。ファイルシステムの設定と、認証情報なしのレジストリだけを書いている |
| `profiles/` | Claude Code と Codex を OAuth で使うための provider profile。認証情報を付ける宛先と、通信してよいバイナリはここで決める |
| `macos/` | Docker Desktop for Mac で必要な gateway の設定 |

GitHub の provider profile は OpenShell 公式の [providers/github.yaml](https://github.com/NVIDIA/OpenShell/blob/v0.1.7/providers/github.yaml) をそのまま使います。

## 設計

認証情報を付ける宛先は provider profile にだけ書きます。profile に書いた宛先とバイナリは、provider を付けた sandbox で `_provider_<name>` というルールになります。そのため policy に Anthropic や OpenAI の宛先を書く必要はありません。

base イメージに同梱されている policy は、次の理由で使っていません。

- `tls: terminate` という項目が残っていて、v0.1.x のスキーマでは無効になる
- Codex 用のルールが L4 だけで書かれている。認証情報を付ける宛先は L7 で検査しないと拒否される
- Claude Code 用と Codex 用のルールが `/usr/bin/node` を許可しているので、任意の node スクリプトが認証情報付きでその宛先に出られる

## 制約

- Codex の refresh token は gateway が持ち、access_token の期限の前に更新します。ホストの `~/.codex` とは別にログインするので、ホストの Codex のログインには影響しません。更新に失敗したら(`openshell provider refresh status codex`)、`bin/osa login codex` をもう一度実行してください。
- Codex の `account_id` は、Codex がローカルで比較するので placeholder にできません。平文の環境変数で渡しています。
- `osa allow` に渡すバイナリのパスは、このイメージの配置を前提にしています。

## ライセンス

Apache-2.0。[LICENSE](LICENSE) を見てください。
