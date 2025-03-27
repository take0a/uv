# uv

Rust で書かれた、非常に高速な Python パッケージおよびプロジェクトマネージャー。

<p align="center">
  <img alt="Shows a bar chart with benchmark results." src="https://github.com/astral-sh/uv/assets/1309177/629e59c0-9c6e-4013-9ad4-adb2bcf5080d#only-light">
</p>

<p align="center">
  <img alt="Shows a bar chart with benchmark results." src="https://github.com/astral-sh/uv/assets/1309177/03aa9163-1c79-4a87-a31d-7a9311ed9310#only-dark">
</p>

<p align="center">
  <i>ウォームキャッシュを利用して <a href="https://trio.readthedocs.io/">Trio</a> の依存関係をインストール</i>
</p>

## ハイライト

- 🚀 `pip`, `pip-tools`, `pipx`, `poetry`, `pyenv`, `twine`, `virtualenv` などを置き換える単一ツール
- ⚡️ `pip` よりも [10-100倍速い](https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md)
- 🗂️ [ユニバーサル・ロックファイル](./concepts/projects/layout.md#the-lockfile)を使用して、[包括的なプロジェクト管理](#projects)を提供
- ❇️ [インライン依存関係メタデータ](./guides/scripts.md#declaring-script-dependencies)をサポートして[スクリプトを実行](#scripts)
- 🐍 Python の各バージョンを[インストールおよび管理](#python-versions)
- 🛠️ Python パッケージとして公開されたツールの[実行とインストール](#tools)
- 🔩 使い慣れた CLI でパフォーマンスを維持できる [pip 互換インターフェース](#the-pip-interface)を含む
- 🏢 スケーラブルなプロジェクトのため Cargo スタイルの[ワークスペース](./concepts/projects/workspaces.md)をサポート
- 💾 依存関係の重複を排除する[グローバルキャッシュ](./concepts/cache.md)を備え、ディスクスペースを効率的に使用
- ⏬ `curl` または `pip` 経由で Rust や Python なしでインストール可能
- 🖥️ macOS, Linux, Windows をサポート

uv は [Ruff](https://github.com/astral-sh/ruff) の作者である [Astral](https://astral.sh) によってサポートされています。

## インストール

公式のスタンドアロンインストーラーで uv をインストールします:

=== "macOS and Linux"

    ```console
    $ curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

=== "Windows"

    ```console
    $ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    ```

次に、[最初の手順](./getting-started/first-steps.md)を確認するか、簡単な概要を読んでください。

!!! tip

    uv は pip, Homebrew などを使用してインストールすることもできます。[インストールのページ](./getting-started/installation.md)で全ての方法を確認しくて下さい。

## プロジェクト {#projects}

uv は `rye` や `poetry` と同様、ロックファイルやワークスペースなどをサポートし、プロジェクトの依存関係と環境を管理します:

```console
$ uv init example
Initialized project `example` at `/home/user/example`

$ cd example

$ uv add ruff
Creating virtual environment at: .venv
Resolved 2 packages in 170ms
   Built example @ file:///home/user/example
Prepared 2 packages in 627ms
Installed 2 packages in 1ms
 + example==0.1.0 (from file:///home/user/example)
 + ruff==0.5.4

$ uv run ruff check
All checks passed!

$ uv lock
Resolved 2 packages in 0.33ms

$ uv sync
Resolved 2 packages in 0.70ms
Audited 1 package in 0.02ms
```

開始するには、[プロジェクトガイド](./guides/projects.md)を参照してください。

uv は、uv で管理されていない場合でも、プロジェクトのビルドと公開をサポートします。詳細については、[パッケージングガイド](./guides/package.md)を参照してください。

## スクリプト {#scripts}

uv は単一ファイル スクリプトの依存関係と環境を管理します。

新しいスクリプトを作成し、その依存関係を宣言するインライン メタデータを追加します:

```console
$ echo 'import requests; print(requests.get("https://astral.sh"))' > example.py

$ uv add --script example.py requests
Updated `example.py`
```

次に、分離された仮想環境でスクリプトを実行します:

```console
$ uv run example.py
Reading inline script metadata from: example.py
Installed 5 packages in 12ms
<Response [200]>
```

開始するには、[スクリプトガイド](./guides/scripts.md)を参照してください。

## ツール {#tools}

uv は、`pipx` と同様に、Python パッケージによって提供されるコマンドラインツールの実行とインストールを行います。

`uvx` (`uv tool run` のエイリアス) を使用して一時的な環境でツールを実行します。

```console
$ uvx pycowsay 'hello world!'
Resolved 1 package in 167ms
Installed 1 package in 9ms
 + pycowsay==0.0.0.2
  """

  ------------
< hello world! >
  ------------
   \   ^__^
    \  (oo)\_______
       (__)\       )\/\
           ||----w |
           ||     ||
```

`uv tool install` でツールをインストールします:

```console
$ uv tool install ruff
Resolved 1 package in 6ms
Installed 1 package in 2ms
 + ruff==0.5.4
Installed 1 executable: ruff

$ ruff --version
ruff 0.5.4
```

開始するには、[ツールガイド](./guides/tools.md)を参照してください。

## Python の各種バージョン {#python-versions}

uv は Python をインストールし、バージョン間の切り替えを素早く行うことができます。

複数の Python バージョンをインストールする:

```console
$ uv python install 3.10 3.11 3.12
Searching for Python versions matching: Python 3.10
Searching for Python versions matching: Python 3.11
Searching for Python versions matching: Python 3.12
Installed 3 versions in 3.42s
 + cpython-3.10.14-macos-aarch64-none
 + cpython-3.11.9-macos-aarch64-none
 + cpython-3.12.4-macos-aarch64-none
```

必要に応じて Python のバージョンをダウンロードする:

```console
$ uv venv --python 3.12.0
Using CPython 3.12.0
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate

$ uv run --python pypy@3.8 -- python
Python 3.8.16 (a9dbdca6fc3286b0addd2240f11d97d8e8de187a, Dec 29 2022, 11:45:30)
[PyPy 7.3.11 with GCC Apple LLVM 13.1.6 (clang-1316.0.21.2.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>>
```

現在のディレクトリで特定の Python バージョンを使用する:

```console
$ uv python pin 3.11
Pinned `.python-version` to `3.11`
```

開始するには、[Python のインストールガイド](./guides/install-python.md)を参照してください。

## pip インターフェース {#the-pip-interface}

uv は、一般的な `pip`、`pip-tools`、および `virtualenv` コマンドを置き換えます。

uv は、依存関係バージョンのオーバーライド、プラットフォームに依存しない解決、再現可能な解決、代替解決戦略などの高度な機能でそれらのインターフェースを拡張します。

`uv pip` インターフェースを使用すると、既存のワークフローを変更せずに uv に移行し、10 ～ 100 倍の高速化を体験できます。

要件をプラットフォームに依存しない requirements ファイルにコンパイルします:

```console
$ uv pip compile docs/requirements.in \
   --universal \
   --output-file docs/requirements.txt
Resolved 43 packages in 12ms
```

仮想環境を作成します:

```console
$ uv venv
Using CPython 3.12.3
Creating virtual environment at: .venv
Activate with: source .venv/bin/activate
```

ロックされた要件をインストールする:

```console
$ uv pip sync docs/requirements.txt
Resolved 43 packages in 11ms
Installed 43 packages in 208ms
 + babel==2.15.0
 + black==24.4.2
 + certifi==2024.7.4
 ...
```

開始するには、[pip インターフェースのドキュメント](./pip/index.md)を参照してください。

## もっと詳しく知る

[最初の手順](./getting-started/first-steps.md)を確認するか、[ガイド](./guides/index.md)に直接移動して uv の使用を開始します。
