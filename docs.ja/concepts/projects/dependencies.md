# 依存関係の管理

## 依存関係のフィールド

プロジェクトの依存関係はいくつかのフィールドで定義されます:

- [`project.dependencies`](#project-dependencies): 公開された依存関係。
- [`project.optional-dependencies`](#optional-dependencies): 公開されたオプションの依存関係、つまり「追加機能」。
- [`dependency-groups`](#dependency-groups): 開発のためのローカル依存関係。
- [`tool.uv.sources`](#dependency-sources): 開発中の依存関係の代替ソース。

!!! note

    `project.dependencies` フィールドと `project.optional-dependencies` フィールドは、プロジェクトが公開されない場合でも使用できます。`dependency-groups` は最近標準化された機能であり、まだすべてのツールでサポートされているわけではない可能性があります。

uv は `uv add` と `uv remove` を使用してプロジェクトの依存関係を変更することをサポートしていますが、依存関係のメタデータは `pyproject.toml` を直接編集して更新することもできます。

## 依存関係の追加

依存関係を追加するには:

```console
$ uv add httpx
```

`project.dependencies` フィールドにエントリが追加されます:

```toml title="pyproject.toml" hl_lines="4"
[project]
name = "example"
version = "0.1.0"
dependencies = ["httpx>=0.27.2"]
```

[`--dev`](#development-dependencies)、[`--group`](#dependency-groups)、または [`--optional`](#optional-dependencies) フラグを使用して、代替フィールドに依存関係を追加できます。

依存関係には、パッケージの最新の互換性のあるバージョンに対する制約（例：`>=0.27.2`）が含まれます。別の制約を指定することもできます:

```console
$ uv add "httpx>=0.20"
```

パッケージ レジストリ以外のソースから依存関係を追加する場合、uv はソース フィールドにエントリを追加します。たとえば、GitHub から `httpx` を追加する場合:

```console
$ uv add "httpx @ git+https://github.com/encode/httpx"
```

`pyproject.toml` には [Git ソースエントリ](#git) が含まれます。

```toml title="pyproject.toml" hl_lines="8-9"
[project]
name = "example"
version = "0.1.0"
dependencies = [
    "httpx",
]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx" }
```

依存関係が使用できない場合、uv はエラーを表示します:

```console
$ uv add "httpx>9999"
  × No solution found when resolving dependencies:
  ╰─▶ Because only httpx<=1.0.0b0 is available and your project depends on httpx>9999,
      we can conclude that your project's requirements are unsatisfiable.
```

### 依存関係のインポート

`requirements.txt` ファイルで宣言された依存関係は、`-r` オプションを使用してプロジェクトに追加できます。

```
uv add -r requirements.txt
```

## 依存関係の削除

依存関係を削除するには:

```console
$ uv remove httpx
```

`--dev`、`--group`、または `--optional` フラグを使用して、特定のテーブルから依存関係を削除できます。

削除された依存関係に対して [source](#dependency-sources) が定義されており、依存関係への他の参照がない場合、その依存関係も削除されます。

## 依存関係の変更

既存の依存関係を変更するには、たとえば、`httpx` に別の制約を使用するには、次のようにします:

```console
$ uv add "httpx>0.1.0"
```

!!! note

    この例では、`pyproject.toml` 内の依存関係の制約を変更しています。依存関係のロックされたバージョンは、新しい制約を満たすために必要な場合にのみ変更されます。制約内でパッケージ バージョンを最新のものに強制的に更新するには、`--upgrade-package <name>` を使用します。例:

    ```console
    $ uv add "httpx>0.1.0" --upgrade-package httpx
    ```

    パッケージのアップグレードの詳細については、[ロックファイル](./sync.md#upgrading-locked-package-versions) のドキュメントを参照してください。

別の依存関係ソースを要求すると、`tool.uv.sources` テーブルが更新されます。たとえば、開発中にローカル パスから `httpx` を使用するなどです:

```console
$ uv add "httpx @ ../httpx"
```

## プラットフォーム固有の依存関係

依存関係が特定のプラットフォームまたは特定の Python バージョンにのみインストールされるようにするには、[環境マーカー](https://peps.python.org/pep-0508/#environment-markers) を使用します。

たとえば、Linux に `jax` をインストールし、Windows や macOS にはインストールしない場合は、次のようにします:

```console
$ uv add "jax; sys_platform == 'linux'"
```

結果として得られる `pyproject.toml` には、依存関係の定義に環境マーカーが含まれます:

```toml title="pyproject.toml" hl_lines="6"
[project]
name = "project"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = ["jax; sys_platform == 'linux'"]
```

同様に、Python 3.11 以降で `numpy` を含めるには、次のようにします:

```console
$ uv add "numpy; python_version >= '3.11'"
```

使用可能なマーカーと演算子の完全な列挙については、Python の [環境マーカー](https://peps.python.org/pep-0508/#environment-markers) ドキュメントを参照してください。

!!! tip

    依存関係ソースは [プラットフォームごとに変更](#platform-specific-sources) することもできます。

## プロジェクトの依存関係 {#project-dependencies}

`project.dependencies` テーブルは、PyPI にアップロードするときや wheel をビルドするときに使用される依存関係を表します。個々の依存関係は [依存関係指定子](https://packaging.python.org/en/latest/specifications/dependency-specifiers/) 構文を使用して指定され、テーブルは [PEP 621](https://packaging.python.org/en/latest/specifications/pyproject-toml/) 標準に従います。

`project.dependencies` は、プロジェクトに必要なパッケージのリストと、それらのインストール時に使用するバージョン制約を定義します。各エントリには、依存関係の名前とバージョンが含まれます。エントリには、プラットフォーム固有のパッケージの追加情報または環境マーカーが含まれる場合があります。
例:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
dependencies = [
  # Any version in this range
  "tqdm >=4.66.2,<5",
  # Exactly this version of torch
  "torch ==2.2.2",
  # Install transformers with the torch extra
  "transformers[torch] >=4.39.3,<5",
  # Only install this package on older python versions
  # See "Environment Markers" for more information
  "importlib_metadata >=7.1.0,<8; python_version < '3.10'",
  "mollymawk ==0.1.0"
]
```

## 依存元 {#dependency-sources}

`tool.uv.sources` テーブルは、開発中に使用される代替依存関係ソースで標準依存関係テーブルを拡張します。

依存関係ソースは、編集可能なインストールや相対パスなど、`project.dependencies` 標準ではサポートされていない一般的なパターンのサポートを追加します。たとえば、プロジェクト ルートを基準としたディレクトリから `foo` をインストールするには、次のようにします:

```toml title="pyproject.toml" hl_lines="7"
[project]
name = "example"
version = "0.1.0"
dependencies = ["foo"]

[tool.uv.sources]
foo = { path = "./packages/foo" }
```

uv では次の依存関係ソースがサポートされています:

- [Index](#index):特定のパッケージ インデックスから解決されたパッケージ。
- [Git](#git): Git リポジトリ。
- [URL](#url): リモートホイールまたはソース配布。
- [Path](#path): ローカル wheel、ソースディストリビューション、またはプロジェクトディレクトリ。
- [Workspace](#workspace-member): 現在のワークスペースのメンバー。

!!! important

    ソースは uv によってのみ尊重されます。別のツールが使用されている場合は、標準プロジェクトテーブルの定義のみが使用されます。開発に別のツールが使用されている場合は、ソーステーブルで提供されるメタデータを他のツールの形式で再指定する必要があります。

### Index

特定のインデックスから Python パッケージを追加するには、`--index` オプションを使用します:

```console
$ uv add torch --index pytorch=https://download.pytorch.org/whl/cpu
```

uv はインデックスを `[[tool.uv.index]]` に保存し、 `[tool.uv.sources]` エントリを追加します:

```toml title="pyproject.toml"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = { index = "pytorch" }

[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
```

!!! tip

    上記の例は、PyTorch インデックスの特性により、x86-64 Linux でのみ動作します。PyTorch の設定の詳細については、[PyTorch ガイド](../../guides/integration/pytorch.md) を参照してください。

`index` ソースを使用すると、パッケージが指定されたインデックスに _固定_ され、他のインデックスからはダウンロードされなくなります。

インデックスを定義するときに、`explicit` フラグを含めることができます。これは、インデックスが `tool.uv.sources` で明示的に指定されているパッケージに対してのみ使用されることを示します。`explicit` が設定されていない場合、他の場所で見つからない場合は、他のパッケージがインデックスから解決される可能性があります。

```toml title="pyproject.toml" hl_lines="3"
[[tool.uv.index]]
name = "pytorch"
url = "https://download.pytorch.org/whl/cpu"
explicit = true
```

### Git

Git 依存関係ソースを追加するには、Git 互換の URL (つまり、`git clone` で使用する URL) の前に `git+` を付けます。

例:

```console
$ uv add git+https://github.com/encode/httpx
```

```toml title="pyproject.toml" hl_lines="5"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx" }
```

特定の Git 参照 (タグなど) をリクエストできます:

```console
$ uv add git+https://github.com/encode/httpx --tag 0.27.0
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.0" }
```

または、ブランチ:

```console
$ uv add git+https://github.com/encode/httpx --branch main
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", branch = "main" }
```

または、リビジョン（コミット）:

```console
$ uv add git+https://github.com/encode/httpx --rev 326b9431c761e1ef1e00b9f760d1f654c8db48c6
```

```toml title="pyproject.toml" hl_lines="7"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", rev = "326b9431c761e1ef1e00b9f760d1f654c8db48c6" }
```

パッケージがリポジトリのルートにない場合は、`subdirectory` を指定できます:

```console
$ uv add git+https://github.com/langchain-ai/langchain#subdirectory=libs/langchain
```

```toml title="pyproject.toml"
[project]
dependencies = ["langchain"]

[tool.uv.sources]
langchain = { git = "https://github.com/langchain-ai/langchain", subdirectory = "libs/langchain" }
```

### URL

URL ソースを追加するには、wheel  (`.whl` で終わる) またはソース配布 (通常は `.tar.gz` または `.zip` で終わる。サポートされているすべての形式については、[こちら](../../concepts/resolution.md#source-distribution) を参照してください) への `https://` URL を指定します。

例:

```console
$ uv add "https://files.pythonhosted.org/packages/5c/2d/3da5bdf4408b8b2800061c339f240c1802f2e82d55e50bd39c5a881f47f0/httpx-0.27.0.tar.gz"
```

`pyproject.toml` は次のようになります:

```toml title="pyproject.toml" hl_lines="5"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { url = "https://files.pythonhosted.org/packages/5c/2d/3da5bdf4408b8b2800061c339f240c1802f2e82d55e50bd39c5a881f47f0/httpx-0.27.0.tar.gz" }
```

URL 依存関係は、`{ url = <url> }` 構文を使用して `pyproject.toml` に手動で追加または編集することもできます。ソース配布がアーカイブ ルートにない場合は、`subdirectory` を指定できます。

### Path

path ソースを追加するには、wheel のパス (`.whl` で終わる)、ソースディストリビューション (通常は `.tar.gz` または `.zip` で終わる。サポートされているすべての形式については、[こちら](../../concepts/resolution.md#source-distribution) を参照)、または `pyproject.toml` を含むディレクトリを指定します。

例:

```console
$ uv add /example/foo-0.1.0-py3-none-any.whl
```

`pyproject.toml` は次のようになります:

```toml title="pyproject.toml"
[project]
dependencies = ["foo"]

[tool.uv.sources]
foo = { path = "/example/foo-0.1.0-py3-none-any.whl" }
```

パスは相対パスでも構いません:

```console
$ uv add ./foo-0.1.0-py3-none-any.whl
```

または、プロジェクト ディレクトリへのパス:

```console
$ uv add ~/projects/bar/
```

!!! important

    [編集可能なインストール](#editable-dependencies)は、デフォルトではパス依存関係には使用されません。編集可能なインストールは、プロジェクト ディレクトリに対して要求される場合があります:

    ```console
    $ uv add --editable ../projects/bar/
    ```

    結果は次の内容の `pyproject.toml` になります:

    ```toml title="pyproject.toml"
    [project]
    dependencies = ["bar"]

    [tool.uv.sources]
    bar = { path = "../projects/bar", editable = true }
    ```

    同様に、プロジェクトが [非パッケージ](./config.md#build-systems) としてマークされているが、環境にパッケージとしてインストールしたい場合は、ソースで `package = true` を設定します:

    ```toml title="pyproject.toml"
    [project]
    dependencies = ["bar"]

    [tool.uv.sources]
    bar = { path = "../projects/bar", package = true }
    ```

    同じリポジトリ内に複数のパッケージがある場合は、[_workspaces_](./workspaces.md) の方が適している可能性があります。

### ワークスペースメンバー {#workspace-member}

ワークスペース メンバーへの依存関係を宣言するには、`{workspace = true }` を使用してメンバー名を追加します。すべてのワークスペース メンバーを明示的に指定する必要があります。ワークスペース メンバーは常に [編集可能](#editable-dependencies) です。ワークスペースの詳細については、[ワークスペース](./workspaces.md) のドキュメントを参照してください。

```toml title="pyproject.toml"
[project]
dependencies = ["foo==0.1.0"]

[tool.uv.sources]
foo = { workspace = true }

[tool.uv.workspace]
members = [
  "packages/foo"
]
```

### プラットフォーム固有のソース {#platform-specific-sources}

ソースに [依存関係指定子](https://packaging.python.org/en/latest/specifications/dependency-specifiers/) 互換の環境マーカーを指定することで、ソースを特定のプラットフォームまたは Python バージョンに制限できます。

たとえば、macOS でのみ GitHub から `httpx` をプルするには、次のようにします:

```toml title="pyproject.toml" hl_lines="8"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = { git = "https://github.com/encode/httpx", tag = "0.27.2", marker = "sys_platform == 'darwin'" }
```

ソースにマーカーを指定すると、uv はすべてのプラットフォームで `httpx` を含めますが、macOS では GitHub からソースをダウンロードし、他のすべてのプラットフォームでは PyPI にフォールバックします。

### 複数のソース

[PEP 508](https://peps.python.org/pep-0508/#environment-markers) 互換の環境マーカーによって曖昧さが解消されたソースのリストを提供することで、単一の依存関係に対して複数のソースを指定できます。

たとえば、macOS と Linux で異なる `httpx` タグを取得するには、次のようにします:

```toml title="pyproject.toml" hl_lines="8-9 13-14"
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = [
  { git = "https://github.com/encode/httpx", tag = "0.27.2", marker = "sys_platform == 'darwin'" },
  { git = "https://github.com/encode/httpx", tag = "0.24.1", marker = "sys_platform == 'linux'" },
]
```

この戦略は、環境マーカーに基づいて異なるインデックスを使用することにまで拡張されます。たとえば、プラットフォームに基づいて異なる PyTorch インデックスから `torch` をインストールするには、次のようにします:

```toml title="pyproject.toml" hl_lines="6-7"
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", marker = "platform_system == 'Darwin'"},
  { index = "torch-gpu", marker = "platform_system == 'Linux'"},
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu124"
```

### ソースを無効にする

uv に `tool.uv.sources` テーブルを無視するように指示するには (たとえば、パッケージの公開メタデータによる解決をシミュレートするため)、 `--no-sources` フラグを使用します:

```console
$ uv lock --no-sources
```

`--no-sources` を使用すると、uv が特定の依存関係を満たす可能性のある [ワークスペース メンバー](#workspace-member) を発見することもできなくなります。

## オプションの依存関係 {#optional-dependencies}

ライブラリとして公開されるプロジェクトでは、デフォルトの依存関係ツリーを減らすために、一部の機能をオプションにすることが一般的です。たとえば、Pandas には [`excel` エクストラ](https://pandas.pydata.org/docs/getting_started/install.html#excel-files) と [`plot` エクストラ](https://pandas.pydata.org/docs/getting_started/install.html#visualization) があり、明示的に要求されない限り、Excel パーサーと `matplotlib` のインストールを回避します。エクストラは、`package[<extra>]` 構文を使用して要求されます (例: `pandas[plot, excel]`)。

オプションの依存関係は、[依存関係指定子](#dependency-specifiers-pep-508)構文に従って、追加の名前からその依存関係にマッピングする TOML テーブルである `[project.optional-dependencies]` で指定されます。

オプションの依存関係は、通常の依存関係と同じように `tool.uv.sources` にエントリを持つことができます。

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "1.0.0"

[project.optional-dependencies]
plot = [
  "matplotlib>=3.6.3"
]
excel = [
  "odfpy>=1.4.1",
  "openpyxl>=3.1.0",
  "python-calamine>=0.1.7",
  "pyxlsb>=1.0.10",
  "xlrd>=2.0.1",
  "xlsxwriter>=3.0.5"
]
```

オプションの依存関係を追加するには、`--optional <extra>` オプションを使用します:

```console
$ uv add httpx --optional network
```

!!! note

    互いに競合するオプションの依存関係がある場合、明示的に [競合していると宣言](./config.md#conflicting-dependencies) しない限り、解決は失敗します。

ソースは、特定のオプションの依存関係にのみ適用されるように宣言することもできます。たとえば、オプションの `cpu` または `gpu` エクストラに基づいて、異なる PyTorch インデックスから `torch` をプルするには、次のようにします:

```toml title="pyproject.toml"
[project]
dependencies = []

[project.optional-dependencies]
cpu = [
  "torch",
]
gpu = [
  "torch",
]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", extra = "cpu" },
  { index = "torch-gpu", extra = "gpu" },
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu124"
```

## 開発依存関係 {#development-dependencies}

オプションの依存関係とは異なり、開発依存関係はローカルのみであり、PyPI または他のインデックスに公開されるときにプロジェクト要件に含まれません。そのため、開発依存関係は `[project]` テーブルに含まれません。

開発依存関係は、通常の依存関係と同じように `tool.uv.sources` にエントリを持つことができます。

開発依存関係を追加するには、`--dev` フラグを使用します:

```console
$ uv add --dev pytest
```

uv は開発依存関係の宣言に `[dependency-groups]` テーブル ([PEP 735](https://peps.python.org/pep-0735/) で定義) を使用します。上記のコマンドは `dev` グループを作成します:

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  "pytest >=8.1.1,<9"
]
```

`dev` グループは特別です。`--dev`、`--only-dev`、および `--no-dev` フラグを使用して、依存関係を含めるか除外するかを切り替えることができます。代わりにすべてのデフォルト グループを無効にするには、`--no-default-groups` を参照してください。また、`dev` グループは [デフォルトで同期されます](#default-groups)。

### 依存関係グループ {#dependency-groups}

開発依存関係は、`--group` フラグを使用して複数のグループに分割できます。

たとえば、`lint` グループに開発依存関係を追加するには、次のようにします:

```console
$ uv add --group lint ruff
```

その結果、次の `[dependency-groups]` 定義が生成されます:

```toml title="pyproject.toml"
[dependency-groups]
dev = [
  "pytest"
]
lint = [
  "ruff"
]
```

グループが定義されたら、`--all-groups`、`--no-default-groups`、`--group`、`--only-group`、および `--no-group` オプションを使用して、依存関係を含めたり除外したりできます。

!!! tip

    `--dev`、`--only-dev`、および `--no-dev` フラグは、それぞれ `--group dev`、`--only-group dev`、および `--no-group dev` と同等です。

uv では、すべての依存関係グループが相互に互換性があることが要求され、ロックファイルを作成するときにすべてのグループが一緒に解決されます。

あるグループで宣言された依存関係が別のグループの依存関係と互換性がない場合、uv はエラーを出してプロジェクトの要件を解決できません。

!!! note

    相互に競合する依存関係グループがある場合、明示的に [競合していると宣言](./config.md#conflicting-dependencies) しない限り、解決は失敗します。

### デフォルトグループ {#default-groups}

デフォルトでは、uv は環境に `dev` 依存グループを含めます (例: `uv run` または `uv sync` 中)。含めるデフォルトのグループは、`tool.uv.default-groups` 設定を使用して変更できます。

```toml title="pyproject.toml"
[tool.uv]
default-groups = ["dev", "foo"]
```

すべての依存関係グループをデフォルトで有効にするには、グループ名をリストする代わりに `"all"` を使用します:

```toml title="pyproject.toml"
[tool.uv]
default-groups = "all"
```

!!! tip

    `uv run` または `uv sync` 中にこの動作を無効にするには、`--no-default-groups` を使用します。特定のデフォルト グループを除外するには、`--no-group <name>` を使用します。

### レガシー `dev-dependencies`

`[dependency-groups]` が標準化される前は、uv は開発依存関係を指定するために `tool.uv.dev-dependencies` フィールドを使用していました。例:

```toml title="pyproject.toml"
[tool.uv]
dev-dependencies = [
  "pytest"
]
```

このセクションで宣言された依存関係は、`dependency-groups.dev` の内容と結合されます。最終的には、`dev-dependencies` フィールドは非推奨となり、削除されます。

!!! note

    `tool.uv.dev-dependencies` フィールドが存在する場合、`uv add --dev` は新しい `dependency-groups.dev` セクションを追加する代わりに既存のセクションを使用します。

## ビルドの依存関係

プロジェクトが [Python パッケージ](./config.md#build-systems) として構造化されている場合、プロジェクトのビルドに必要な依存関係を宣言できますが、実行には必要ありません。これらの依存関係は、[PEP 518](https://peps.python.org/pep-0518/) に従って、`build-system.requires` の下の `[build-system]` テーブルで指定されます。

たとえば、プロジェクトがビルド バックエンドとして `setuptools` を使用する場合、ビルド依存関係として `setuptools` を宣言する必要があります:

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"
```

デフォルトでは、uv はビルドの依存関係を解決するときに `tool.uv.sources` を尊重します。たとえば、ビルドに `setuptools` のローカル バージョンを使用するには、ソースを `tool.uv.sources` に追加します:

```toml title="pyproject.toml"
[project]
name = "pandas"
version = "0.1.0"

[build-system]
requires = ["setuptools>=42"]
build-backend = "setuptools.build_meta"

[tool.uv.sources]
setuptools = { path = "./packages/setuptools" }
```

パッケージを公開するときは、[`pypa/build`](https://github.com/pypa/build) などの他のビルドツールを使用する場合と同様に、`tool.uv.sources` が無効になっているときにパッケージが正しくビルドされるように、`uv build --no-sources` を実行することをお勧めします。

## 編集可能な依存関係 {#editable-dependencies}

Python パッケージを含むディレクトリの通常のインストールでは、最初に wheel が構築され、次にその wheel が仮想環境にインストールされ、すべてのソースファイルがコピーされます。パッケージソースファイルが編集されると、仮想環境に古いバージョンが含まれることになります。

編集可能なインストールでは、仮想環境内のプロジェクトへのリンク (`.pth` ファイル) を追加することでこの問題を解決し、インタープリターにソースファイルを直接含めるように指示します。

編集可能ファイルにはいくつかの制限があります (主に、ビルド バックエンドが編集可能ファイルをサポートする必要があり、ネイティブ モジュールはインポート前に再コンパイルされない) が、仮想環境では常にパッケージの最新の変更が使用されるため、開発には便利です。

uv は、デフォルトでワークスペース パッケージに編集可能なインストールを使用します。

編集可能な依存関係を追加するには、`--editable` フラグを使用します:

```console
$ uv add --editable ./path/foo
```

または、ワークスペースで編集可能な依存関係の使用をオプトアウトするには、次の手順を実行します:

```console
$ uv add --no-editable ./path/foo
```

## 依存関係指定子 (PEP 508) {#dependency-specifiers-pep-508}

uv は [依存関係指定子](https://packaging.python.org/en/latest/specifications/dependency-specifiers/) を使用します。これは以前は [PEP 508](https://peps.python.org/pep-0508/) と呼ばれていました。依存関係指定子は、次の順序で構成されます:

- 依存関係名
- ご希望の追加機能（オプション）
- バージョン指定子
- 環境マーカー（オプション）

バージョン指定子はコンマで区切られ、一緒に追加されます。たとえば、`foo >=1.2.3,<2,!=1.4.0` は、「`foo` のバージョンが少なくとも 1.2.3 だが 2 未満で、1.4.0 ではない」と解釈されます。

必要に応じて指定子の末尾にゼロが埋め込まれるため、`foo ==2` は foo 2.0.0 にも一致します。

最後の桁にアスタリスク (*) を等号と共に使用できます。たとえば、`foo ==2.1.*` は 2.1 シリーズのリリースをすべて受け入れます。同様に、`~=` は最後の桁が等しいかそれ以上の桁と一致します。たとえば、`foo ~=1.2` は `foo >=1.2,<2` と等しく、`foo ~=1.2.3` は `foo >=1.2.3,<1.3` と等しくなります。

追加情報は、名前とバージョンの間に角括弧でコンマで区切られます (例: `pandas[excel,plot] ==2.2`)。追加情報の名前間の空白は無視されます。

一部の依存関係は、特定の Python バージョンやオペレーティング システムなど、特定の環境でのみ必要です。たとえば、`importlib.metadata` モジュールの `importlib-metadata` バックポートをインストールするには、`importlib-metadata >=7.1.0,<8; python_version < '3.10'` を使用します。Windows に `colorama` をインストールするには (他のプラットフォームでは省略します)、`colorama >=0.4.6,<5; platform_system == "Windows"` を使用します。

マーカーは、`and`、`or`、括弧で結合されます。例: `aiohttp >=3.7.4,<4; (sys_platform != 'win32' or implementation_name != 'pypy') and python_version >= '3.10'`。マーカー内のバージョンは引用符で囲む必要がありますが、マーカーの _外側_ のバージョンは引用符で囲んではならないことに注意してください。
