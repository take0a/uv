# プロジェクトの設定

## Python バージョン要件 {#python-version-requirement}

プロジェクトでは、`pyproject.toml` の `project.requires-python` フィールドで、プロジェクトでサポートされている Python バージョンを宣言できます。

`requires-python` 値を設定することが推奨されます:

```toml title="pyproject.toml" hl_lines="4"
[project]
name = "example"
version = "0.1.0"
requires-python = ">=3.12"
```

Python バージョン要件により、プロジェクトで許可される Python 構文が決定され、依存関係バージョンの選択に影響します (同じ Python バージョン範囲をサポートする必要があります)。

## エントリーポイント {#entry-points}

[エントリポイント](https://packaging.python.org/en/latest/specifications/entry-points/#entry-points) は、インストールされたパッケージがインターフェースをアドバタイズするための正式な用語です。これには次のものが含まれます:

- [コマンドラインインターフェース](#command-line-interfaces)
- [グラフィカルユーザーインターフェース](#graphical-user-interfaces)
- [プラグインのエントリポイント](#plugin-entry-points)

!!! important

    エントリポイントテーブルを使用するには、[ビルドシステム](#build-systems) を定義する必要があります。

### コマンドラインインターフェース {#command-line-interfaces}

プロジェクトは、`pyproject.toml` の `[project.scripts]` テーブルでプロジェクトのコマンドラインインターフェイス (CLI) を定義できます。

たとえば、`example` モジュールの `hello` 関数を呼び出す `hello` というコマンドを宣言するには、次のようにします:

```toml title="pyproject.toml"
[project.scripts]
hello = "example:hello"
```

次に、コンソールからコマンドを実行できます:

```console
$ uv run hello
```

### グラフィカルユーザーインターフェース {#graphical-user-interfaces}

プロジェクトでは、`pyproject.toml` の `[project.gui-scripts]` テーブルでプロジェクトのグラフィカルユーザーインターフェイス (GUI) を定義できます。

!!! important

    これらは、Windows の [コマンドラインインターフェイス](#command-line-interfaces) とのみ異なります。Windows では、これらは GUI 実行可能ファイルでラップされているため、コンソールなしで起動できます。他のプラットフォームでも、同じように動作します。

たとえば、`example` モジュールの `app` 関数を呼び出す `hello` というコマンドを宣言するには、次のようにします:

```toml title="pyproject.toml"
[project.gui-scripts]
hello = "example:app"
```

### プラグインのエントリポイント {#plugin-entry-points}

プロジェクトは、`pyproject.toml` の [`[project.entry-points]`](https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/#using-package-metadata) テーブルでプラグイン検出のエントリ ポイントを定義できます。

たとえば、`example-plugin-a` パッケージを `example` のプラグインとして登録するには、次のようにします。

```toml title="pyproject.toml"
[project.entry-points.'example.plugins']
a = "example_plugin_a"
```

次に、`example` では、プラグインは次のようにロードされます:

```python title="example/__init__.py"
from importlib.metadata import entry_points

for plugin in entry_points(group='example.plugins'):
    plugin.load()
```

!!! note

    `group` キーは任意の値にすることができ、パッケージ名や「プラグイン」を含める必要はありません。ただし、他のパッケージとの衝突を避けるために、キーをパッケージ名で名前空間化することをお勧めします。

## ビルドシステム {#build-systems}

ビルド システムは、プロジェクトをパッケージ化してインストールする方法を決定します。プロジェクトは、`pyproject.toml` の `[build-system]` テーブルでビルド システムを宣言および構成できます。

uv は、ビルド システムの存在に基づいて、プロジェクトにプロジェクト仮想環境にインストールする必要があるパッケージが含まれているかどうかを判断します。ビルド システムが定義されていない場合、uv はプロジェクト自体のビルドやインストールを試行せず、依存関係のみを試行します。ビルド システムが定義されている場合、uv はプロジェクトをビルドしてプロジェクト環境にインストールします。

`--build-backend` オプションを `uv init` に指定すると、適切なレイアウトでパッケージ化されたプロジェクトを作成できます。`--package` オプションを `uv init` に指定すると、デフォルトのビルド システムでパッケージ化されたプロジェクトを作成できます。

!!! note

    uv はビルド システム定義なしでは現在のプロジェクトをビルドおよびインストールしませんが、他のパッケージでは `[build-system]` テーブルの存在は必須ではありません。レガシーな理由から、ビルド システムが定義されていない場合は、パッケージのビルドに `setuptools.build_meta:__legacy__` が使用されます。依存するパッケージはビルド システムを明示的に宣言していない可能性がありますが、それでもインストール可能です。同様に、ローカル パッケージに依存関係を追加したり、`uv pip` を使用してインストールしたりすると、uv は常にそのパッケージをビルドしてインストールしようとします。

### ビルドシステムのオプション

ビルドシステムは、次の機能を実現するために使用されます:

- 配布物にファイルを含めるか除外するか
- 編集可能なインストール時の動作
- 動的プロジェクトメタデータ
- ネイティブコードのコンパイル
- 共有ライブラリのベンダー化

これらの機能を構成するには、選択したビルド システムのドキュメントを参照してください。

## プロジェクトのパッケージ化 {#project-packaging}

[ビルド システム](#build-systems) で説明したように、Python プロジェクトをインストールするにはビルドする必要があります。このプロセスは一般に「パッケージ化」と呼ばれます。

以下のことを行う場合は、おそらくパッケージが必要になります:

- プロジェクトにコマンドを追加する
- プロジェクトを他の人に配布する
- `src` と `test` レイアウトを使用する
- ライブラリを書く

次のような場合は、おそらくパッケージは必要ありません:

- スクリプトの作成
- シンプルなアプリケーションの構築
- フラットレイアウトの使用

uv は通常、[ビルド システム](#build-systems) の宣言を使用してプロジェクトをパッケージ化するかどうかを決定しますが、uv では [`tool.uv.package`](../../reference/settings.md#package) 設定でこの動作をオーバーライドすることもできます。

`tool.uv.package = true` を設定すると、プロジェクトが強制的にビルドされ、プロジェクト環境にインストールされます。ビルド システムが定義されていない場合、uv は setuptools レガシー バックエンドを使用します。

`tool.uv.package = false` を設定すると、プロジェクト パッケージはビルドされず、プロジェクト環境にインストールされません。uv は、プロジェクトと対話するときに宣言されたビルド システムを無視します。

## プロジェクト環境パス {#project-environment-path}

`UV_PROJECT_ENVIRONMENT` 環境変数を使用して、プロジェクトの仮想環境パス (デフォルトでは `.venv`) を設定できます。

相対パスが指定されている場合は、ワークスペース ルートを基準として解決されます。絶対パスが指定されている場合は、そのまま使用されます。つまり、環境の子ディレクトリは作成されません。指定されたパスに環境が存在しない場合は、uv によって作成されます。

このオプションはシステムの Python 環境に書き込むために使用できますが、推奨されません。`uv sync` はデフォルトで環境から不要なパッケージを削除するため、システムが壊れた状態になる可能性があります。

システム環境をターゲットにするには、`UV_PROJECT_ENVIRONMENT` を Python インストールのプレフィックスに設定します。たとえば、Debian ベースのシステムでは、これは通常 `/usr/local` です:

```console
$ python -c "import sysconfig; print(sysconfig.get_config_var('prefix'))"
/usr/local
```

この環境をターゲットにするには、`UV_PROJECT_ENVIRONMENT=/usr/local` をエクスポートします。

!!! important

    絶対パスが指定され、設定が複数のプロジェクトにわたって使用されている場合、環境は各プロジェクトでの呼び出しによって上書きされます。この設定は、CI または Docker イメージ内の単一のプロジェクトにのみ使用することをお勧めします。

!!! note

    デフォルトでは、uv はプロジェクト操作中に `VIRTUAL_ENV` 環境変数を読み取りません。`VIRTUAL_ENV` がプロジェクトの環境とは異なるパスに設定されている場合、警告が表示されます。`--active` フラグを使用すると、`VIRTUAL_ENV` を尊重するようにオプトインできます。`--no-active` フラグを使用すると、警告を黙らせることができます。

## 解決が限定された環境 {#limited-resolution-environments}

プロジェクトがサポートするプラットフォームや Python バージョンの数が限られている場合は、PEP 508 環境マーカーのリストを受け入れる `environments` 設定を使用して、解決するプラットフォームのセットを制限できます。たとえば、ロックファイルを macOS と Linux に制限し、Windows を除外するには、次のようにします。

```toml title="pyproject.toml"
[tool.uv]
environments = [
    "sys_platform == 'darwin'",
    "sys_platform == 'linux'",
]
```

詳細については、[解決のドキュメント](../resolution.md#limited-resolution-environments)を参照してください。

## 必要な環境

プロジェクトが特定のプラットフォームまたは Python バージョンをサポートしなければならない場合は、`required-environments` 設定でそのプラットフォームを必須としてマークできます。たとえば、プロジェクトが Intel macOS をサポートすることを必須にするには、次のようにします。

```toml title="pyproject.toml"
[tool.uv]
required-environments = [
    "sys_platform == 'darwin' and platform_machine == 'x86_64'",
]
```

`required-environments` 設定は、ソースディストリビューションを公開しないパッケージ (PyTorch など) にのみ関連します。このようなパッケージは、そのパッケージによって公開されるビルド済みバイナリディストリビューション (wheels) のセットによってカバーされる環境にのみインストールできるためです。

詳細については、[解決のドキュメント](../resolution.md#required-environments)を参照してください。

## ビルドの分離 {#build-isolation}

デフォルトでは、uv は [PEP 517](https://peps.python.org/pep-0517/) に従って、すべてのパッケージを分離された仮想環境でビルドします。一部のパッケージは、意図的 (たとえば、重いビルド依存関係 (ほとんどの場合 PyTorch が一般的) または意図的でない (たとえば、レガシー パッケージ設定の使用)) かにかかわらず、ビルド分離と互換性がありません。

特定の依存関係のビルド分離を無効にするには、`pyproject.toml` の `no-build-isolation-package` リストに追加します:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["cchardet"]

[tool.uv]
no-build-isolation-package = ["cchardet"]
```

ビルド分離なしでパッケージをインストールするには、パッケージ自体をインストールする前に、パッケージのビルド依存関係をプロジェクト環境にインストールする必要があります。これは、ビルド依存関係とそれを必要とするパッケージを個別の追加項目に分離することで実現できます。例:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = []

[project.optional-dependencies]
build = ["setuptools", "cython"]
compile = ["cchardet"]

[tool.uv]
no-build-isolation-package = ["cchardet"]
```

上記の場合、ユーザーはまず `build` 依存関係を同期します:

```console
$ uv sync --extra build
 + cython==3.0.11
 + foo==0.1.0 (from file:///Users/crmarsh/workspace/uv/foo)
 + setuptools==73.0.1
```

`compile` 依存関係が続きます:

```console
$ uv sync --extra compile
 + cchardet==2.1.7
 - cython==3.0.11
 - setuptools==73.0.1
```

`uv sync --extra compile` は、デフォルトでは `cython` および `setuptools` パッケージをアンインストールすることに注意してください。代わりにビルドの依存関係を保持するには、2 回目の `uv sync` 呼び出しで両方の追加パッケージを含めます。

```console
$ uv sync --extra build
$ uv sync --extra build --extra compile
```

上記の `cchardet` のような一部のパッケージでは、`uv sync` の _installation_ フェーズでのみビルド依存関係が必要です。`flash-attn` のような他のパッケージでは、_resolution_ フェーズでプロジェクトのロックファイルを解決するためにもビルド依存関係が存在する必要があります。

このような場合、`uv lock` または `uv sync` コマンドを実行する前に、下位レベルの `uv pip` API を使用してビルド依存関係をインストールする必要があります。たとえば、次のようになります:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = ["flash-attn"]

[tool.uv]
no-build-isolation-package = ["flash-attn"]
```

`flash-attn` を同期するには、次のコマンド シーケンスを実行できます:

```console
$ uv venv
$ uv pip install torch setuptools
$ uv sync
```

あるいは、[`dependency-metadata`](../../reference/settings.md#dependency-metadata) 設定を介して `flash-attn` メタデータを事前に提供することもできます。これにより、依存関係解決フェーズでパッケージをビルドする必要がなくなります。たとえば、`flash-attn` メタデータを事前に提供するには、`pyproject.toml` に次の内容を含めます:

```toml title="pyproject.toml"
[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

!!! tip

    `flash-attn` のようなパッケージのパッケージ メタデータを確認するには、適切な Git リポジトリに移動する、または [PyPI](https://pypi.org/project/flash-attn) で検索してパッケージのソース配布をダウンロードします。パッケージ要件は通常、`setup.py` または `setup.cfg` ファイルにあります。

    (パッケージにビルド済みのディストリビューションが含まれている場合は、それを解凍して `METADATA` ファイルを見つけることができます。ただし、ビルド済みのディストリビューションが存在すると、メタデータがすでに uv で使用できるため、事前にメタデータを提供する必要がなくなります。)

一度含めると、2 段階の `uv sync` プロセスを使用してビルド依存関係をインストールできます。次の `pyproject.toml` があるとします:

```toml title="pyproject.toml"
[project]
name = "project"
version = "0.1.0"
description = "..."
readme = "README.md"
requires-python = ">=3.12"
dependencies = []

[project.optional-dependencies]
build = ["torch", "setuptools", "packaging"]
compile = ["flash-attn"]

[tool.uv]
no-build-isolation-package = ["flash-attn"]

[[tool.uv.dependency-metadata]]
name = "flash-attn"
version = "2.6.3"
requires-dist = ["torch", "einops"]
```

`flash-attn` を同期するには、次のコマンド シーケンスを実行できます:

```console
$ uv sync --extra build
$ uv sync --extra build --extra compile
```

!!! note

    `tool.uv.dependency-metadata` の `version` フィールドは、レジストリベースの依存関係ではオプションです (省略すると、uv はメタデータがパッケージのすべてのバージョンに適用されると想定します) が、直接 URL 依存関係 (Git 依存関係など) では必須です。

## 編集可能モード

デフォルトでは、プロジェクトは編集可能モードでインストールされるため、ソース コードへの変更はすぐに環境に反映されます。`uv sync` と `uv run` はどちらも `--no-editable` フラグを受け入れます。このフラグは、uv にプロジェクトを編集不可モードでインストールするように指示します。`--no-editable` は、Docker コンテナの構築など、プロジェクトを元のソース コードに依存せずにデプロイされた環境に含める必要があるデプロイメント ユースケースを対象としています。

## 競合する依存関係 {#conflicting-dependencies}

uv では、プロジェクトによって宣言されたすべてのオプションの依存関係 (「extras」) が相互に互換性があることが要求され、ロックファイルを作成するときにすべてのオプションの依存関係が一緒に解決されます。

あるエクストラで宣言されたオプションの依存関係が別のエクストラの依存関係と互換性がない場合、uv はエラーを出してプロジェクトの要件を解決できません。

これを回避するために、uv は競合する追加要素の宣言をサポートしています。たとえば、互いに競合する 2 つのオプション依存関係セットを考えてみましょう:

```toml title="pyproject.toml"
[project.optional-dependencies]
extra1 = ["numpy==2.1.2"]
extra2 = ["numpy==2.0.0"]
```

上記の依存関係で `uv lock` を実行すると、解決は失敗します:

```console
$ uv lock
  x No solution found when resolving dependencies:
  `-> Because myproject[extra2] depends on numpy==2.0.0 and myproject[extra1] depends on numpy==2.1.2, we can conclude that myproject[extra1] and
      myproject[extra2] are incompatible.
      And because your project requires myproject[extra1] and myproject[extra2], we can conclude that your projects's requirements are unsatisfiable.
```

しかし、`extra1` と `extra2` が競合していると指定した場合、uv はそれらを個別に解決します。`tool.uv` セクションで競合を指定します:

```toml title="pyproject.toml"
[tool.uv]
conflicts = [
    [
      { extra = "extra1" },
      { extra = "extra2" },
    ],
]
```

これで、`uv lock` の実行は成功します。ただし、`extra1` と `extra2` の両方を同時にインストールすることはできないことに注意してください。

```console
$ uv sync --extra extra1 --extra extra2
Resolved 3 packages in 14ms
error: extra `extra1`, extra `extra2` are incompatible with the declared conflicts: {`myproject[extra1]`, `myproject[extra2]`}
```

このエラーは、`extra1` と `extra2` の両方をインストールすると、同じ環境に 2 つの異なるバージョンのパッケージがインストールされるため発生します。

競合する追加機能に対処するための上記の戦略は、依存関係グループでも機能します:

```toml title="pyproject.toml"
[dependency-groups]
group1 = ["numpy==2.1.2"]
group2 = ["numpy==2.0.0"]

[tool.uv]
conflicts = [
    [
      { group = "group1" },
      { group = "group2" },
    ],
]
```

競合する追加機能との唯一の違いは、`extra` ではなく `group` を使用する必要があることです。
