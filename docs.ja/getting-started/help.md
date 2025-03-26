# ヘルプを得る

## ヘルプメニュー

`--help` フラグを使用すると、コマンドのヘルプ メニューを表示できます (例: `uv`):

```console
$ uv --help
```

特定のコマンド（例：`uv init`）のヘルプ メニューを表示するには、次のようにします:

```console
$ uv init --help
```

`--help` フラグを使用すると、uv は簡略化されたヘルプ メニューを表示します。コマンドのより長いヘルプ メニューを表示するには、`uv help` を使用します:

```console
$ uv help
```

特定のコマンド（例：`uv init`）の長いヘルプ メニューを表示するには、次のようにします:

```console
$ uv help init
```

長いヘルプ メニューを使用する場合、uv は `less` または `more` を使用して出力を「ページング」し、一度にすべてが表示されないようにします。ページャーを終了するには、`q` を押します。

## バージョンの表示

ヘルプを求めるときは、使用している uv のバージョンを確認することが重要です。新しいバージョンで問題がすでに解決されている場合もあります。

インストールされているバージョンを確認するには:

```console
$ uv version
```

以下も有効です:

```console
$ uv --version      # Same output as `uv version`
$ uv -V             # Will not include the build commit and date
$ uv pip --version  # Can be used with a subcommand
```

## 問題のトラブルシューティング

リファレンスドキュメントには、一般的な問題に関する[トラブルシューティング ガイド](../reference/troubleshooting/index.md)が含まれています。

## GitHubで問題を開く

GitHub の [問題追跡システム](https://github.com/astral-sh/uv/issues) は、バグを報告したり機能をリクエストしたりするのに適した場所です。他の人も同じ問題に遭遇することが多いので、まずは類似の問題を検索してください。

## Discordでチャット

Astral には [Discord サーバー](https://discord.com/invite/astral-sh) があり、質問したり、UV について詳しく学んだり、他のコミュニティメンバーと交流したりするのに最適な場所です。
