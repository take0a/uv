---
title: パッケージの構築と公開
description: A guide to using uv to build and publish Python packages to a package index, like PyPI.
---

# パッケージの構築と公開

uv は、`uv build` を介して Python パッケージをソースおよびバイナリディストリビューションにビルドし、`uv publish` を使用してレジストリにアップロードすることをサポートしています。

## プロジェクトのパッケージ化の準備

プロジェクトを公開する前に、配布用にパッケージ化する準備ができていることを確認する必要があります。

プロジェクトの `pyproject.toml` に `[build-system]` 定義が含まれていない場合、uv はデフォルトでビルドを行いません。つまり、プロジェクトは配布の準備ができていない可能性があります。ビルド システムを宣言することによる影響の詳細については、[プロジェクトのコンセプト](../concepts/projects/config.md#build-systems) ドキュメントを参照してください。

!!! note

    公開したくない内部パッケージがある場合は、それを非公開としてマークできます:

    ```toml
    [project]
    classifiers = ["Private :: Do Not Upload"]
    ```

    この設定により、PyPI はアップロードされたパッケージの公開を拒否します。代替レジストリのセキュリティやプライバシー設定には影響しません。

    また、プロジェクトごとのトークンのみを生成することをお勧めします。プロジェクトに一致する PyPI トークンがないと、誤って公開されることはありません。

## パッケージの構築

`uv build` を使用してパッケージをビルドします:

```console
$ uv build
```

デフォルトでは、`uv build` は現在のディレクトリにプロジェクトをビルドし、ビルドされた成果物を `dist/` サブディレクトリに配置します。

あるいは、`uv build <SRC>` は指定されたディレクトリにパッケージをビルドし、`uv build --package <PACKAGE>` は現在のワークスペース内に指定されたパッケージをビルドします。

!!! info

    デフォルトでは、`uv build` は、`pyproject.toml` の `build-system.requires` セクションからビルド依存関係を解決するときに `tool.uv.sources` を尊重します。パッケージを公開するときは、[`pypa/build`](https://github.com/pypa/build) などの他のビルドツールを使用する場合と同様に、`tool.uv.sources` が無効になっているときにパッケージが正しくビルドされるように、`uv build --no-sources` を実行することをお勧めします。

## パッケージを公開する

`uv publish`でパッケージを公開します:

```console
$ uv publish
```

`--token` または `UV_PUBLISH_TOKEN` を使用して PyPI トークンを設定するか、`--username` または `UV_PUBLISH_USERNAME` を使用してユーザー名を設定し、`--password` または `UV_PUBLISH_PASSWORD` を使用してパスワードを設定します。GitHub Actions から PyPI に公開する場合、資格情報を設定する必要はありません。代わりに、[信頼できるパブリッシャーを PyPI プロジェクトに追加](https://docs.pypi.org/trusted-publishers/adding-a-publisher/)します。

!!! note

    PyPI はユーザー名とパスワードによる公開をサポートしなくなったため、代わりにトークンを生成する必要があります。トークンを使用することは、`--username __token__` を設定し、トークンをパスワードとして使用することと同じです。

`[[tool.uv.index]]` を通じてカスタム インデックスを使用している場合は、`publish-url` を追加し、`uv publish --index <name>` を使用します。例:

```toml
[[tool.uv.index]]
name = "testpypi"
url = "https://test.pypi.org/simple/"
publish-url = "https://test.pypi.org/legacy/"
explicit = true
```

!!! note

    `uv publish --index <name>` を使用する場合は、`pyproject.toml` が存在している必要があります。つまり、公開 CI ジョブにチェックアウト ステップが必要です。

`uv publish` は失敗したアップロードを再試行しますが、一部のファイルはアップロードされ、一部のファイルは依然として欠落しているため、途中で公開が失敗することがあります。PyPI を使用すると、まったく同じコマンドを再試行できます。既存の同一ファイルは無視されます。他のレジストリでは、パッケージが属するインデックス URL (公開 URL ではありません) とともに `--check-url <index url>` を使用します。`--index` を使用する場合、インデックス URL がチェック URL として使用されます。uv はレジストリ内のファイルと同一のファイルのアップロードをスキップし、競合する並列アップロードも処理します。既存のファイルは、以前にレジストリにアップロードされたものと完全に一致する必要があることに注意してください。これにより、同じバージョンの異なる内容のソース配布とホイールが誤って公開されることが回避されます。

## パッケージのインストール

`uv run` を使用してパッケージをインストールおよびインポートできることをテストします:

```console
$ uv run --with <PACKAGE> --no-project -- python -c "import <PACKAGE>"
```

`--no-project` フラグは、ローカルプロジェクトディレクトリからパッケージをインストールしないようにするために使用されます。

!!! tip

    パッケージを最近インストールした場合は、パッケージのキャッシュバージョンの使用を避けるために、`--refresh-package <PACKAGE>` オプションを含める必要がある場合があります。

## 次のステップ

パッケージの公開の詳細については、ビルドと公開に関する [PyPA ガイド](https://packaging.python.org/en/latest/guides/section-build-and-publish/) をご覧ください。

または、uv を他のソフトウェアと統合するための [ガイド](./integration/index.md) をお読みください。
