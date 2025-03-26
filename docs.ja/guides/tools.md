---
title: ツールの使用
description:
  A guide to using uv to run tools published as Python packages, including one-off invocations with
  uvx, requesting specific tool versions, installing tools, upgrading tools, and more.
---

# ツールの使用

多くの Python パッケージは、ツールとして使用できるアプリケーションを提供します。uv は、ツールを簡単に呼び出してインストールするための特別なサポートを備えています。

## ツールの実行

`uvx` コマンドはツールをインストールせずに呼び出します。

たとえば、`ruff` を実行するには:

```console
$ uvx ruff
```

!!! note

    これは次のものとまったく同じです:

    ```console
    $ uv tool run ruff
    ```

    `uvx` は便宜上エイリアスとして提供されています。

ツール名の後に引数を指定できます:

```console
$ uvx pycowsay hello from uv

  -------------
< hello from uv >
  -------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||

```

`uvx` を使用する場合、ツールは一時的な分離された環境にインストールされます。

!!! note

    [_プロジェクト_](../concepts/projects/index.md) でツールを実行していて、そのツールがプロジェクトのインストールを必要とする場合 (たとえば、`pytest` または `mypy` を使用する場合)、`uvx` ではなく [`uv run`](./projects.md#running-commands) を使用する必要があります。そうしないと、ツールはプロジェクトから分離された仮想環境で実行されます。

    プロジェクトがフラットな構造である場合、たとえば、モジュールに `src` ディレクトリを使用するのではない場合、プロジェクト自体をインストールする必要はなく、`uvx` で十分です。この場合、`uv run` の使用は、プロジェクトの依存関係にツールのバージョンを固定する場合にのみ有益です。

## 異なるパッケージ名のコマンド

`uvx ruff` が呼び出されると、uv は `ruff` コマンドを提供する `ruff` パッケージをインストールします。ただし、パッケージ名とコマンド名が異なる場合もあります。

`--from` オプションを使用すると、特定のパッケージ (たとえば、`httpie` によって提供される `http`) からコマンドを呼び出すことができます。

```console
$ uvx --from httpie http
```

## 特定のバージョンのリクエスト

特定のバージョンでツールを実行するには、`command@<version>` を使用します:

```console
$ uvx ruff@0.3.0 check
```

ツールを最新バージョンで実行するには、`command@latest` を使用します:

```console
$ uvx ruff@latest check
```

上記のように、`--from` オプションを使用してパッケージのバージョンを指定することもできます:

```console
$ uvx --from 'ruff==0.3.0' ruff check
```

または、バージョンの範囲を制限するには、次のようにします:

```console
$ uvx --from 'ruff>0.2.0,<0.3.0' ruff check
```

`@` 構文は、正確なバージョン以外には使用できません。

## 追加リクエスト

`--from` オプションを使用すると、追加機能付きのツールを実行できます:

```console
$ uvx --from 'mypy[faster-cache,reports]' mypy --xml-report mypy_report
```

これはバージョン選択と組み合わせることもできます:

```console
$ uvx --from 'mypy[faster-cache,reports]==1.13.0' mypy --xml-report mypy_report
```

## さまざまな情報源のリクエスト

`--from` オプションを使用して、代替ソースからインストールすることもできます。

たとえば、git からプルするには:

```console
$ uvx --from git+https://github.com/httpie/cli httpie
```

特定の名前付きブランチから最新のコミットをプルすることもできます:

```console
$ uvx --from git+https://github.com/httpie/cli@master httpie
```

または、特定のタグを取得します:

```console
$ uvx --from git+https://github.com/httpie/cli@3.2.4 httpie
```

あるいは特定のコミットでも:

```console
$ uvx --from git+https://github.com/httpie/cli@2843b87 httpie
```

## プラグインを使用したコマンド

追加の依存関係を含めることもできます。たとえば、`mkdocs` を実行するときに `mkdocs-material` を含めるなどです:

```console
$ uvx --with mkdocs-material mkdocs --help
```

## ツールのインストール

ツールを頻繁に使用する場合は、`uvx` を繰り返し呼び出すのではなく、ツールを永続的な環境にインストールして `PATH` に追加すると便利です。

!!! tip

    `uvx` は `uv tool run` の便利なエイリアスです。ツールを操作するための他のすべてのコマンドには、完全な `uv tool` プレフィックスが必要です。

`ruff` をインストールするには:

```console
$ uv tool install ruff
```

ツールがインストールされると、その実行ファイルは `PATH` の `bin` ディレクトリに配置され、uv なしでツールを実行できるようになります。`PATH` 上にない場合は警告が表示され、`uv tool update-shell` を使用して `PATH` に追加できます。

`ruff` をインストールすると、利用できるようになります:

```console
$ ruff --version
```

`uv pip install` とは異なり、ツールをインストールしても、そのモジュールは現在の環境で使用できません。たとえば、次のコマンドは失敗します:

```console
$ python -c "import ruff"
```

この分離は、ツール、スクリプト、プロジェクトの依存関係間の相互作用と競合を減らすために重要です。

`uvx` とは異なり、`uv tool install` は _パッケージ_ 上で動作し、ツールによって提供されるすべての実行可能ファイルをインストールします。

たとえば、次の例では、`http`、`https`、および `httpie` 実行可能ファイルがインストールされます:

```console
$ uv tool install httpie
```

さらに、パッケージのバージョンは `--from` なしでも含めることができます:

```console
$ uv tool install 'httpie>0.1.0'
```

同様に、パッケージ ソースの場合も次のようになります:

```console
$ uv tool install git+https://github.com/httpie/cli
```

`uvx` と同様に、インストールには追加のパッケージを含めることができます:

```console
$ uv tool install mkdocs --with mkdocs-material
```

## ツールのアップグレード

ツールをアップグレードするには、`uv tool upgrade` を使用します:

```console
$ uv tool upgrade ruff
```

ツールのアップグレードでは、ツールのインストール時に指定されたバージョン制約が尊重されます。たとえば、`uv tool install ruff >=0.3,<0.4` の後に `uv tool upgrade ruff` を実行すると、Ruff が `>=0.3,<0.4` の範囲内の最新バージョンにアップグレードされます。

代わりにバージョン制約を置き換えるには、`uv tool install` を使用してツールを再インストールします:

```console
$ uv tool install ruff>=0.4
```

代わりにすべてのツールをアップグレードするには:

```console
$ uv tool upgrade --all
```

## Python バージョンのリクエスト

デフォルトでは、uv はツールを実行、インストール、またはアップグレードするときに、デフォルトの Python インタープリター (最初に見つかったもの) を使用します。`--python` オプションを使用して、使用する Python インタープリターを指定できます。

たとえば、ツールを実行するときに特定の Python バージョンを要求するには、次のようにします:

```console
$ uvx --python 3.10 ruff
```

または、ツールをインストールする場合:

```console
$ uv tool install --python 3.10 ruff
```

または、ツールをアップグレードする場合:

```console
$ uv tool upgrade --python 3.10 ruff
```

Python バージョンのリクエストの詳細については、[Python バージョン](../concepts/python-versions.md#requesting-a-version) コンセプト ページを参照してください。

## レガシー Windows スクリプト

ツールは、[レガシー setuptools スクリプト](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#scripts) の実行もサポートしています。これらのスクリプトは、インストール時に `$(uv tool dir)\<tool-name>\Scripts` から利用できます。

現在、`.ps1`、`.cmd`、および `.bat` 拡張子を持つレガシー スクリプトのみがサポートされています。

たとえば、以下はコマンドプロンプト・スクリプトを実行する例です。

```console
$ uv tool run --from nuitka==2.6.7 nuitka.cmd --version
```

また、拡張子を指定する必要もありません。`uvx` は、`.ps1`、`.cmd`、`.bat` で終わるファイルを実行順に自動的に検索します。

```console
$ uv tool run --from nuitka==2.6.7 nuitka --version
```

## 次のステップ

uv を使用したツールの管理の詳細については、[ツールの概念](../concepts/tools.md)ページと[コマンドリファレンス](../reference/cli.md#uv-tool) を参照してください。

または、[プロジェクトに取り組む方法](./projects.md) を学習するには、以下をお読みください。
