---
title: プロジェクトで作業する
description:
  A guide to using uv to create and manage Python projects, including adding dependencies, running
  commands, and building publishable distributions.
---

# プロジェクトで作業する

uv は、`pyproject.toml` ファイルで依存関係を定義する Python プロジェクトの管理をサポートします。

## 新しいプロジェクトの作成

`uv init` コマンドを使用して新しい Python プロジェクトを作成できます:

```console
$ uv init hello-world
$ cd hello-world
```

あるいは、作業ディレクトリでプロジェクトを初期化することもできます:

```console
$ mkdir hello-world
$ cd hello-world
$ uv init
```

uv は次のファイルを作成します:

```text
.
├── .python-version
├── README.md
├── main.py
└── pyproject.toml
```

`main.py` ファイルには、単純な「Hello world」プログラムが含まれています。`uv run` で試してみましょう:

```console
$ uv run main.py
Hello from hello-world!
```

## プロジェクト構造

プロジェクトは、連携して動作し、uv がプロジェクトを管理できるようにするいくつかの重要な部分で構成されています。`uv init` によって作成されたファイルに加えて、uv は、プロジェクト コマンド (`uv run`、`uv sync`、または `uv lock`) を初めて実行したときに、プロジェクトのルートに仮想環境と `uv.lock` ファイルを作成します。

完全なリストは次のようになります:

```text
.
├── .venv
│   ├── bin
│   ├── lib
│   └── pyvenv.cfg
├── .python-version
├── README.md
├── main.py
├── pyproject.toml
└── uv.lock
```

### `pyproject.toml`

`pyproject.toml` にはプロジェクトに関するメタデータが含まれています:

```toml title="pyproject.toml"
[project]
name = "hello-world"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
dependencies = []
```

このファイルを使用して、依存関係や、プロジェクトの説明やライセンスなどのプロジェクトの詳細を指定します。このファイルを手動で編集することも、`uv add` や `uv remove` などのコマンドを使用してターミナルからプロジェクトを管理することもできます。

!!! tip

    `pyproject.toml` 形式の使用開始に関する詳細については、公式の [`pyproject.toml` ガイド](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) を参照してください。

このファイルを使用して、[`[tool.uv]`](../reference/settings.md) セクションで uv [構成オプション](../configuration/files.md) を指定します。

### `.python-version`

`.python-version` ファイルには、プロジェクトのデフォルトの Python バージョンが含まれています。このファイルは、プロジェクトの仮想環境を作成するときに使用する Python バージョンを uv に指示します。

### `.venv`

`.venv` フォルダには、プロジェクトの仮想環境、つまりシステムの他の部分から分離された Python 環境が含まれています。ここに、uv がプロジェクトの依存関係をインストールします。

詳細については、[プロジェクト環境](../concepts/projects/layout.md#the-project-environment)のドキュメントを参照してください。

### `uv.lock`

`uv.lock` は、プロジェクトの依存関係に関する正確な情報を含むクロスプラットフォーム ロックファイルです。プロジェクトの広範な要件を指定するために使用される `pyproject.toml` とは異なり、ロックファイルには、プロジェクト環境にインストールされている解決済みの正確なバージョンが含まれています。このファイルはバージョン管理にチェックインして、マシン間で一貫性があり再現可能なインストールを可能にする必要があります。

`uv.lock` は人間が読める TOML ファイルですが、uv によって管理されており、手動で編集しないでください。

詳細については、[ロックファイル](../concepts/projects/layout.md#the-lockfile)のドキュメントを参照してください。

## 依存関係の管理

`uv add` コマンドを使用して、`pyproject.toml` に依存関係を追加できます。これにより、ロックファイルとプロジェクト環境も更新されます:

```console
$ uv add requests
```

バージョン制約や代替ソースを指定することもできます:

```console
$ # Specify a version constraint
$ uv add 'requests==2.31.0'

$ # Add a git dependency
$ uv add git+https://github.com/psf/requests
```

`requirements.txt` ファイルから移行する場合は、`-r` フラグを指定した `uv add` を使用して、ファイルからすべての依存関係を追加できます。

```console
$ # Add all dependencies from `requirements.txt`.
$ uv add -r requirements.txt -c constraints.txt
```

パッケージを削除するには、`uv remove` を使用します:

```console
$ uv remove requests
```

パッケージをアップグレードするには、`--upgrade-package` フラグを指定して `uv lock` を実行します:

```console
$ uv lock --upgrade-package requests
```

`--upgrade-package` フラグは、ロックファイルの残りの部分をそのままにしながら、指定されたパッケージを最新の互換性のあるバージョンに更新しようとします。

詳細については、[依存関係の管理](../concepts/projects/dependencies.md)に関するドキュメントを参照してください。

## コマンドの実行 {#running-commands}

`uv run` を使用すると、プロジェクト環境で任意のスクリプトまたはコマンドを実行できます。

すべての `uv run` 呼び出しの前に、uv はロックファイルが `pyproject.toml` で最新であること、および環境がロックファイルで最新であることを確認し、手動介入を必要とせずにプロジェクトを同期させます。`uv run` は、コマンドが一貫したロックされた環境で実行されることを保証します。

たとえば、`flask` を使うには:

```console
$ uv add flask
$ uv run -- flask run -p 3000
```

または、スクリプトを実行するには:

```python title="example.py"
# Require a project dependency
import flask

print("hello world")
```

```console
$ uv run example.py
```

あるいは、`uv sync` を使用して環境を手動で更新し、コマンドを実行する前にアクティブ化することもできます:

=== "macOS and Linux"

    ```console
    $ uv sync
    $ source .venv/bin/activate
    $ flask run -p 3000
    $ python example.py
    ```

=== "Windows"

    ```powershell
    uv sync
    source .venv\Scripts\activate
    flask run -p 3000
    python example.py
    ```

!!! note

    `uv run` を使用せずにプロジェクト内のスクリプトやコマンドを実行するには、仮想環境をアクティブにする必要があります。仮想環境のアクティブ化は、シェルとプラットフォームによって異なります。

詳細については、プロジェクトでの[コマンドとスクリプトの実行](../concepts/projects/run.md)に関するドキュメントを参照してください。

## ディストリビューションの構築

`uv build` は、プロジェクトのソース ディストリビューションとバイナリ ディストリビューション (wheel) をビルドするために使用できます。

デフォルトでは、`uv build` は現在のディレクトリにプロジェクトをビルドし、ビルドされた成果物を `dist/` サブディレクトリに配置します:

```console
$ uv build
$ ls dist/
hello-world-0.1.0-py3-none-any.whl
hello-world-0.1.0.tar.gz
```

詳細については、[プロジェクトのビルド](../concepts/projects/build.md)に関するドキュメントを参照してください。

## 次のステップ

UV を使用したプロジェクトの作業の詳細については、[プロジェクトの概念](../concepts/projects/index.md) ページと [コマンド リファレンス](../reference/cli.md#uv) を参照してください。

または、[プロジェクトをビルドしてパッケージインデックスに公開する](./package.md) 方法については、以下をお読みください。
