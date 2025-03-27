# ワークスペースの使用

同じ名前の [Cargo](https://doc.rust-lang.org/cargo/reference/workspaces.html) コンセプトに触発されたワークスペースは、「一緒に管理される、_ワークスペース メンバー_ と呼ばれる 1 つ以上のパッケージのコレクション」です。

ワークスペースは、大規模なコードベースを共通の依存関係を持つ複数のパッケージに分割して整理します。FastAPI ベースの Web アプリケーションと、バージョン管理され、個別の Python パッケージとして維持される一連のライブラリがすべて同じ Git リポジトリ内にあると考えてください。

ワークスペースでは、各パッケージが独自の `pyproject.toml` を定義しますが、ワークスペースは単一のロックファイルを共有し、ワークスペースが一貫した依存関係セットで動作することを保証します。

そのため、`uv lock` はワークスペース全体を一度に操作しますが、`uv run` と `uv sync` はデフォルトでワークスペースのルートを操作します。ただし、どちらも `--package` 引数を受け入れるため、任意のワークスペース ディレクトリから特定のワークスペース メンバーでコマンドを実行できます。

## はじめる

ワークスペースを作成するには、`pyproject.toml` に `tool.uv.workspace` テーブルを追加します。これにより、そのパッケージをルートとするワークスペースが暗黙的に作成されます。

!!! tip

    デフォルトでは、既存のパッケージ内で `uv init` を実行すると、新しく作成されたメンバーがワークスペースに追加され、ワー​​クスペース ルートに `tool.uv.workspace` テーブルが作成されます (まだ存在しない場合)。

ワークスペースを定義するときは、`members` (必須) キーと `exclude` (オプション) キーを指定する必要があります。これらのキーは、ワークスペースに特定のディレクトリをメンバーとしてそれぞれ含めるか除外するかを指示し、glob のリストを受け入れます:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]
exclude = ["packages/seeds"]
```

`members` グロブによって含まれる (そして `exclude` グロブによって除外されない) すべてのディレクトリには、`pyproject.toml` ファイルが含まれている必要があります。ただし、ワークスペース メンバーは [applications](./init.md#applications) または [libraries](./init.md#libraries) のいずれかになります。どちらもワークスペース コンテキストでサポートされます。

すべてのワークスペースにはルートが必要ですが、これはワークスペースのメンバーでもあります。上記の例では、`albatross` がワークスペースのルートであり、ワークスペースのメンバーには `seeds` を除く `packages` ディレクトリの下にあるすべてのプロジェクトが含まれます。

デフォルトでは、`uv run` と `uv sync` はワークスペースのルートで動作します。たとえば、上記の例では、`uv run` と `uv run --package albatross` は同等ですが、`uv run --package bird-feeder` は `bird-feeder` パッケージ内のコマンドを実行します。

## ワークスペースでのソース

ワークスペース内では、ワークスペース メンバーへの依存関係は、次のように [`tool.uv.sources`](./dependencies.md) を介して促進されます:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }

[tool.uv.workspace]
members = ["packages/*"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

この例では、`albatross` プロジェクトは、ワークスペースのメンバーである `bird-feeder` プロジェクトに依存しています。`tool.uv.sources` テーブルの `workspace = true` キーと値のペアは、`bird-feeder` 依存関係が PyPI または別のレジストリから取得されるのではなく、ワークスペースによって提供される必要があることを示しています。

!!! note

    ワークスペース メンバー間の依存関係は編集可能です。

ワークスペース ルート内の `tool.uv.sources` 定義は、特定のメンバーの `tool.uv.sources` で上書きされない限り、すべてのメンバーに適用されます。たとえば、次の `pyproject.toml` があるとします:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { workspace = true }
tqdm = { git = "https://github.com/tqdm/tqdm" }

[tool.uv.workspace]
members = ["packages/*"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

特定のメンバーが独自の `tool.uv.sources` テーブル内の `tqdm` エントリを上書きしない限り、すべてのワークスペース メンバーはデフォルトで GitHub から `tqdm` をインストールします。

## ワークスペースのレイアウト

最も一般的なワークスペースレイアウトは、一連の付属ライブラリを備えたルートプロジェクトと考えることができます。

たとえば、上記の例を続けると、このワークスペースには `albatross` に明示的なルートがあり、`packages` ディレクトリに 2 つのライブラリ (`bird-feeder` と `seeds`) があります:

```text
albatross
├── packages
│   ├── bird-feeder
│   │   ├── pyproject.toml
│   │   └── src
│   │       └── bird_feeder
│   │           ├── __init__.py
│   │           └── foo.py
│   └── seeds
│       ├── pyproject.toml
│       └── src
│           └── seeds
│               ├── __init__.py
│               └── bar.py
├── pyproject.toml
├── README.md
├── uv.lock
└── src
    └── albatross
        └── main.py
```

`seeds` は `pyproject.toml` で除外されているため、ワークスペースには合計 2 つのメンバー (`albatross` (ルート) と `bird-feeder`) が含まれます。

## ワークスペースを使用する場合（使用しない場合）

ワークスペースは、単一のリポジトリ内で相互接続された複数のパッケージの開発を容易にすることを目的としています。コードベースの複雑さが増すにつれて、コードベースを、それぞれ独自の依存関係とバージョン制約を持つ、より小さく構成可能なパッケージに分割すると便利です。

ワークスペースは、関心の分離と分離を強制するのに役立ちます。たとえば、uv では、コア ライブラリとコマンド ライン インターフェイスに別々のパッケージがあり、コア ライブラリを CLI から独立してテストできます。その逆も同様です。

ワークスペースのその他の一般的な使用例は次のとおりです:

- 拡張モジュール (Rust、C++ など) に実装されたパフォーマンスクリティカルなサブルーチンを持つライブラリ。
- プラグイン システムを備えたライブラリ。各プラグインは、ルートに依存する個別のワークスペース パッケージです。

ワークスペースは、メンバーの要件が競合する場合や、メンバーごとに別の仮想環境が必要な場合には適していません。この場合、パス依存関係が参照可能であることがよくあります。たとえば、`albatross` とそのメンバーをワークスペースにグループ化するのではなく、各パッケージを独自の独立したプロジェクトとして定義し、パッケージ間の依存関係を `tool.uv.sources` のパス依存関係として定義することができます:

```toml title="pyproject.toml"
[project]
name = "albatross"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = ["bird-feeder", "tqdm>=4,<5"]

[tool.uv.sources]
bird-feeder = { path = "packages/bird-feeder" }

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

このアプローチは、同じ利点の多くをもたらしますが、依存関係の解決と仮想環境の管理をより細かく制御できます (欠点は、`uv run --package` が使用できなくなり、代わりに関連するパッケージディレクトリからコマンドを実行する必要があることです)。

最後に、uv のワークスペースは、すべてのメンバーの `requires-python` 値の共通部分を取得して、ワークスペース全体に単一の `requires-python` を適用します。ワークスペースの残りの部分でサポートされていない Python バージョンで特定のメンバーのテストをサポートする必要がある場合は、`uv pip` を使用してそのメンバーを別の仮想環境にインストールする必要があります。

!!! note

    Python は依存関係の分離を提供しないため、uv はパッケージが宣言された依存関係のみを使用し、それ以外は使用しないことを保証できません。特にワークスペースの場合、uv はパッケージが別のワークスペース メンバーによって宣言された依存関係をインポートしないことを保証できません。