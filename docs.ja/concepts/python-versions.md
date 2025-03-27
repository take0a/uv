# Pythonのバージョン

Python バージョンは、Python インタープリター (つまり、`python` 実行可能ファイル)、標準ライブラリ、およびその他のサポート ファイルで構成されます。

## 管理された、そして、システムPythonのインストール

システムに既存の Python インストールが存在することはよくあるため、 uv は Python バージョンの [検出](#discovery-of-python-versions) をサポートしています。ただし、 uv は [Python バージョンのインストール](#installing-a-python-version) 自体もサポートしています。これら 2 種類の Python インストールを区別するために、 uv はインストールする Python バージョンを _managed_ Python インストールと呼び、その他のすべての Python インストールを _system_ Python インストールと呼びます。

!!! note

    uv は、オペレーティングシステムによってインストールされた Python バージョンと、他のツールによってインストールおよび管理された Python バージョンを区別しません。たとえば、Python インストールが `pyenv` で管理されている場合でも、uv では _system_ Python バージョンと見なされます。

## バージョンのリクエスト {#requesting-a-version}

ほとんどの uv コマンドでは、`--python` フラグを使用して特定の Python バージョンを要求できます。たとえば、仮想環境を作成する場合は次のようになります:

```console
$ uv venv --python 3.11.6
```

uv は Python 3.11.6 が利用可能であることを確認し (必要に応じてダウンロードしてインストールします)、それを使用して仮想環境を作成します。

次の Python バージョン リクエスト形式がサポートされています:

- `<version>` (e.g., `3`, `3.12`, `3.12.3`)
- `<version-specifier>` (e.g., `>=3.12,<3.13`)
- `<implementation>` (e.g., `cpython` or `cp`)
- `<implementation>@<version>` (e.g., `cpython@3.12`)
- `<implementation><version>` (e.g., `cpython3.12` or `cp312`)
- `<implementation><version-specifier>` (e.g., `cpython>=3.12,<3.13`)
- `<implementation>-<version>-<os>-<arch>-<libc>` (e.g., `cpython-3.12.3-macos-aarch64-none`)

さらに、特定のシステム Python インタープリターを次のように要求することもできます:

- `<executable-path>` (e.g., `/opt/homebrew/bin/python3`)
- `<executable-name>` (e.g., `mypython3`)
- `<install-dir>` (e.g., `/some/environment/`)

デフォルトでは、システム上に Python バージョンが見つからない場合、uv は自動的にそのバージョンをダウンロードします。この動作は [`python-downloads` オプションで無効にできます](#disabling-automatic-python-downloads)。

### Python バージョンファイル

`.python-version` ファイルを使用して、デフォルトの Python バージョン要求を作成できます。uv は、作業ディレクトリとその各親で `.python-version` ファイルを検索します。見つからない場合、uv はユーザーレベルの構成ディレクトリをチェックします。上記の要求形式はどれでも使用できますが、他のツールとの相互運用性のためにバージョン番号の使用が推奨されます。

[`uv python pin`](../reference/cli.md/#uv-python-pin) コマンドを使用すると、現在のディレクトリに `.python-version` ファイルを作成できます。

[`uv python pin --global`](../reference/cli.md/#uv-python-pin) コマンドを使用して、ユーザー構成ディレクトリにグローバル `.python-version` ファイルを作成できます。

`.python-version` ファイルの検出は `--no-config` で無効にできます。

uv は、プロジェクトまたはワークスペースの境界を超えて `.python-version` ファイルを検索しません (ユーザー構成ディレクトリを除く)。

## Pythonバージョンのインストール {#installing-a-python-version}

uv には、macOS、Linux、Windows 用のダウンロード可能な CPython および PyPy ディストリビューションのリストがバンドルされています。

!!! tip

    デフォルトでは、`uv python install` を使用せずに、必要に応じて Python バージョンが自動的にダウンロードされます。

特定のバージョンの Python をインストールするには:

```console
$ uv python install 3.12.3
```

最新のパッチバージョンをインストールするには:

```console
$ uv python install 3.12
```

制約を満たすバージョンをインストールするには:

```console
$ uv python install '>=3.8,<3.10'
```

複数のバージョンをインストールするには:

```console
$ uv python install 3.9 3.10 3.11
```

特定の実装をインストールするには:

```console
$ uv python install pypy
```

ファイル パスなどのローカル インタープリターを要求するために使用されるものを除き、すべての [Python バージョン要求](#requesting-a-version) 形式がサポートされています。

デフォルトでは、`uv python install` は管理された Python バージョンがインストールされていることを確認するか、最新バージョンをインストールします。`.python-version` ファイルが存在する場合、uv はファイルにリストされている Python バージョンをインストールします。複数の Python バージョンを必要とするプロジェクトでは、`.python-versions` ファイルを定義できます。存在する場合、uv はファイルにリストされているすべての Python バージョンをインストールします。

!!! important

    利用可能な Python バージョンは、uv リリースごとに固定されています。新しい Python バージョンをインストールするには、uv をアップグレードする必要がある場合があります。

### Python 実行ファイルのインストール {#installing-python-executables}

!!! important

    Python 実行ファイルのインストールのサポートは _プレビュー_ 段階です。つまり、動作は実験的であり、変更される可能性があります。

Python 実行ファイルを `PATH` にインストールするには、`--preview` オプションを指定します:

```console
$ uv python install 3.12 --preview
```

これにより、要求されたバージョンの Python 実行ファイルが `~/.local/bin` にインストールされます (例: `python3.12`)。

!!! tip

    `~/.local/bin` が `PATH` にない場合は、`uv tool update-shell` を使用して追加できます。

`python` および `python3` 実行可能ファイルをインストールするには、`--default` オプションを含めます:

```console
$ uv python install 3.12 --default --preview
```

Python 実行可能ファイルをインストールする場合、uv は既存の実行可能ファイルが uv によって管理されている場合にのみ、それを上書きします。たとえば、`~/.local/bin/python3.12` がすでに存在する場合、uv は `--force` フラグなしでそれを上書きしません。

uv は管理する実行可能ファイルを更新します。ただし、デフォルトでは各 Python マイナー バージョンの最新のパッチ バージョンが優先されます。例:

```console
$ uv python install 3.12.7 --preview  # Adds `python3.12` to `~/.local/bin`
$ uv python install 3.12.6 --preview  # Does not update `python3.12`
$ uv python install 3.12.8 --preview  # Updates `python3.12` to point to 3.12.8
```

## プロジェクト Python バージョン

uv は、プロジェクト コマンドの呼び出し時に、`pyproject.toml` ファイルの `requires-python` で定義された Python 要件を尊重します。`.python-version` ファイルや `--python` フラグなどによってバージョンが要求されない限り、要件と互換性のある最初の Python バージョンが使用されます。

## 利用可能な Python バージョンの表示 {#viewing-available-python-versions}

インストール済みと利用可能な Python バージョンを一覧表示するには:

```console
$ uv python list
```

Python バージョンをフィルタリングするには、リクエストを指定します。たとえば、すべての Python 3.13 インタープリターを表示するには、次のようにします。

```console
$ uv python list 3.13
```

また、すべての PyPy インタープリターを表示するには:

```console
$ uv python list pypy
```

デフォルトでは、他のプラットフォームのダウンロードと古いパッチ バージョンは非表示になっています。

すべてのバージョンを表示するには:

```console
$ uv python list --all-versions
```

他のプラットフォームの Python バージョンを表示するには:

```console
$ uv python list --all-platforms
```

ダウンロードを除外し、インストールされている Python バージョンのみを表示するには:

```console
$ uv python list --only-installed
```

詳細については、[`uv python list`](../reference/cli.md#uv-python-list) リファレンスを参照してください。

## Python 実行ファイルの検索

Python 実行可能ファイルを見つけるには、`uv python find` コマンドを使用します:

```console
$ uv python find
```

デフォルトでは、最初に利用可能な Python 実行可能ファイルへのパスが表示されます。実行可能ファイルを検出する方法の詳細については、[検出ルール](#discovery-of-python-versions)を参照してください。

このインターフェースは、多くの [リクエスト形式](#requesting-a-version) もサポートしています。たとえば、バージョン 3.11 以降の Python 実行可能ファイルを検索するには、次のようにします:

```console
$ uv python find '>=3.11'
```

デフォルトでは、`uv python find` には仮想環境の Python バージョンが含まれます。作業ディレクトリまたは親ディレクトリのいずれかに `.venv` ディレクトリが見つかった場合、または `VIRTUAL_ENV` 環境変数が設定されている場合は、`PATH` 上の Python 実行可能ファイルよりも優先されます。

仮想環境を無視するには、`--system` フラグを使用します:

```console
$ uv python find --system
```

## Pythonバージョンの検出 {#discovery-of-python-versions}

Python バージョンを検索する場合、次の場所がチェックされます:

- `UV_PYTHON_INSTALL_DIR` 内の管理された Python インストール
- `PATH` 上の Python インタープリター (macOS および Linux では `python`、`python3`、または `python3.x`、Windows では `python.exe`)。
- Windows では、要求されたバージョンに一致する Windows レジストリ内の Python インタープリターと Microsoft Store Python インタープリター (「py --list-paths」を参照)。

場合によっては、 uv は仮想環境の Python バージョンの使用を許可します。この場合、上記のようにインストールを検索する前に、仮想環境のインタープリターがリクエストと互換性があるかどうかがチェックされます。詳細については、[pip 互換の仮想環境の検出](../pip/environments.md#discovery-of-python-environments) ドキュメントを参照してください。

検出を実行する際、実行可能ファイル以外のファイルは無視されます。検出された各実行可能ファイルは、メタデータが照会され、[要求された Python バージョン](#requesting-a-version) を満たしているかどうかが確認されます。照会が失敗した場合、実行可能ファイルはスキップされます。実行可能ファイルが要求を満たしている場合は、追加の実行可能ファイルを検査せずに使用されます。

管理された Python バージョンを検索する場合、 uv は新しいバージョンを優先します。システム Python バージョンを検索する場合、 uv は最新バージョンではなく、最初の互換性のあるバージョンを使用します。

システム上で Python バージョンが見つからない場合、uv は互換性のある管理対象 Python バージョンのダウンロードをチェックします。

### Python プレリリース

Python プレリリースはデフォルトでは選択されません。リクエストに一致する他の利用可能なインストールがない場合、Python プレリリースが使用されます。たとえば、プレリリース バージョンのみが利用可能な場合はそれが使用されますが、そうでない場合は安定したリリース バージョンが使用されます。同様に、プレリリース Python 実行可能ファイルへのパスが指定されている場合は、リクエストに一致する他の Python バージョンがないため、プレリリース バージョンが使用されます。

プレリリース版の Python バージョンが利用可能で、リクエストに一致する場合、 uv は代わりに安定した Python バージョンをダウンロードしません。

## Python の自動ダウンロードを無効にする {#disabling-automatic-python-downloads}

デフォルトでは、 uv は必要に応じて Python バージョンを自動的にダウンロードします。

[`python-downloads`](../reference/settings.md#python-downloads) オプションを使用すると、この動作を無効にすることができます。デフォルトでは、`automatic` に設定されています。`uv python install` 中にのみ Python のダウンロードを許可するには、`manual` に設定します。

!!! tip

    `python-downloads` 設定を [永続的な設定ファイル](../configuration/files.md) で設定してデフォルトの動作を変更するか、`--no-python-downloads` フラグを任意の uv コマンドに渡すことができます。

## 管理された Python バージョンの要求または無効化 {#requiring-or-disabling-managed-python-versions}

デフォルトでは、uv はシステムで見つかった Python バージョンの使用を試み、必要な場合にのみ管理された Python バージョンをダウンロードします。システムの Python バージョンを無視し、管理された Python バージョンのみを使用するには、`--managed-python` フラグを使用します:

```console
$ uv python list --managed-python
```

同様に、管理対象 Python バージョンを無視し、システム Python バージョンのみを使用するには、`--no-managed-python` フラグを使用します:

```console
$ uv python list --no-managed-python
```

設定ファイルで uv のデフォルトの動作を変更するには、[`python-preference` 設定](#adjusting-python-version-preferences) を使用します。

## Python バージョン設定の調整 {#adjusting-python-version-preferences}

[`python-preference`](../reference/settings.md#python-preference) 設定は、システムにすでに存在する Python インストールを使用するか、uv によってダウンロードおよびインストールされた Python インストールを使用するかを決定します。

デフォルトでは、`python-preference` は `managed` に設定されており、システム Python インストールよりも管理された Python インストールが優先されます。ただし、システム Python インストールは、管理された Python バージョンのダウンロードよりも優先されます。

次の代替オプションが利用可能です:

- `only-managed`: 管理された Python インストールのみを使用し、システム Python インストールは使用しないでください。`--managed-python` と同等です。
- `system`: 管理された Python インストールよりもシステム Python インストールを優先します。
- `only-system`: システム Python インストールのみを使用し、管理された Python インストールは使用しないでください。`--no-managed-python` と同等です。

!!! note

    設定を変更しなくても、自動 Python バージョン ダウンロードを [無効にする](#disabling-automatic-python-downloads) ことができます。

## Python 実装サポート

uv は、CPython、PyPy、および GraalPy Python 実装をサポートしています。Python 実装がサポートされていない場合、uv はそのインタープリターを検出できません。

実装は、長い名前または短い名前のいずれかでリクエストできます:

- CPython: `cpython`, `cp`
- PyPy: `pypy`, `pp`
- GraalPy: `graalpy`, `gp`

実装名リクエストでは大文字と小文字は区別されません。

サポートされている形式の詳細については、[Python バージョン リクエスト](#requesting-a-version) ドキュメントを参照してください。

## マネージド Python ディストリビューション {#managed-python-distributions}

uv は、CPython および PyPy ディストリビューションのダウンロードとインストールをサポートしています。

### CPython ディストリビューション

Python は公式に配布可能な CPython バイナリを公開していないため、uv は代わりに Astral [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) プロジェクトのビルド済み配布を使用します。`python-build-standalone` は、[Rye](https://github.com/astral-sh/rye)、[Mise](https://mise.jdx.dev/lang/python.html)、[bazelbuild/rules_python](https://github.com/bazelbuild/rules_python) など、他の多くの Python プロジェクトでも使用されています。

uv Python ディストリビューションは自己完結型で、移植性が高く、パフォーマンスに優れています。Python は `pyenv` などのツールのようにソースからビルドできますが、そのためには事前にインストールされたシステム依存関係が必要であり、最適化されたパフォーマンスの高いビルド (PGO や LTO が有効になっているものなど) の作成には非常に時間がかかります。

これらのディストリビューションには、一般的には移植性の結果として、いくつかの動作上の癖があります。また、現時点では、uv は Alpine Linux などの musl ベースの Linux ディストリビューションへのインストールをサポートしていません。詳細については、[`python-build-standalone` quirks](https://gregoryszorc.com/docs/python-build-standalone/main/quirks.html) のドキュメントを参照してください。

### PyPy ディストリビューション

PyPy ディストリビューションは PyPy プロジェクトによって提供されます。
