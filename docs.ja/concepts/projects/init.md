# プロジェクトの作成

uv は `uv init` を使用したプロジェクトの作成をサポートしています。

プロジェクトを作成するとき、uv は 2 つの基本テンプレート [**アプリケーション**](#applications) と [**ライブラリ**](#libraries) をサポートします。デフォルトでは、uv はアプリケーションのプロジェクトを作成します。代わりに `--lib` フラグを使用してライブラリのプロジェクトを作成することもできます。

## 対象ディレクトリ

uv は作業ディレクトリにプロジェクトを作成するか、または名前を指定してターゲット ディレクトリにプロジェクトを作成します (例: `uv init foo`)。ターゲット ディレクトリにすでにプロジェクトがある場合 (つまり、`pyproject.toml` がある場合)、uv はエラーで終了します。

## アプリケーション {#applications}

アプリケーションのプロジェクトは、Web サーバー、スクリプト、コマンド ライン インターフェイスに適しています。

アプリケーションは `uv init` のデフォルトのターゲットですが、 `--app` フラグで指定することもできます。

```console
$ uv init example-app
```

プロジェクトには、`pyproject.toml`、サンプル ファイル (`main.py`)、readme、Python バージョン ピン ファイル (`.python-version`) が含まれています。

```console
$ tree example-app
example-app
├── .python-version
├── README.md
├── main.py
└── pyproject.toml
```

!!! note

    v0.6.0 より前では、uv は `main.py` ではなく `hello.py` という名前のファイルを作成しました。

`pyproject.toml` には基本的なメタデータが含まれています。ビルドシステムは含まれておらず、[パッケージ](./config.md#project-packaging) ではないため、環境にインストールされません。

```toml title="pyproject.toml"
[project]
name = "example-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []
```

サンプルファイルでは、標準的な定型句を使用して `main` 関数を定義しています:

```python title="main.py"
def main():
    print("Hello from example-app!")


if __name__ == "__main__":
    main()
```

Python ファイルは `uv run` で実行できます:

```console
$ cd example-app
$ uv run main.py
Hello from example-project!
```

## パッケージ・アプリケーション {#packaged-applications}

多くのユースケースでは [パッケージ](./config.md#project-packaging) が必要です。たとえば、PyPI に公開されるコマンドライン インターフェイスを作成する場合や、専用のディレクトリでテストを定義する場合などです。

`--package` フラグを使用して、パッケージ化されたアプリケーションを作成できます:

```console
$ uv init --package example-pkg
```

ソースコードは、モジュールディレクトリと `__init__.py` ファイルとともに `src` ディレクトリに移動されます。

```console
$ tree example-pkg
example-pkg
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── example_pkg
        └── __init__.py
```

[ビルドシステム](./config.md#build-systems) が定義されているため、プロジェクトは環境にインストールされます:

```toml title="pyproject.toml" hl_lines="12-14"
[project]
name = "example-pkg"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
example-pkg = "example_pkg:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

!!! tip

    `--build-backend` オプションを使用すると、代替ビルド システムを要求できます。

[コマンド](./config.md#entry-points)の定義が含まれています:

```toml title="pyproject.toml" hl_lines="9 10"
[project]
name = "example-pkg"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[project.scripts]
example-pkg = "example_pkg:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

このコマンドは `uv run` で実行できます:

```console
$ cd example-pkg
$ uv run example-pkg
Hello from example-pkg!
```

## ライブラリ {#libraries}

ライブラリは、他のプロジェクトが使用できる関数とオブジェクトを提供します。ライブラリは、PyPI にアップロードするなどして構築および配布することを目的としています。

ライブラリは `--lib` フラグを使用して作成できます:

```console
$ uv init --lib example-lib
```

!!! note

    `--lib` を使用すると `--package` も暗黙的に使用されます。ライブラリには常にパッケージ化されたプロジェクトが必要です。

[パッケージ化されたアプリケーション](#packaged-applications)と同様に、`src`レイアウトが使用されます。`py.typed`マーカーは、ライブラリから型を読み取ることができることを消費者に示すために含まれています:

```console
$ tree example-lib
example-lib
├── .python-version
├── README.md
├── pyproject.toml
└── src
    └── example_lib
        ├── py.typed
        └── __init__.py
```

!!! note

    `src` レイアウトは、ライブラリを開発するときに特に役立ちます。これにより、ライブラリがプロジェクト ルート内の `python` 呼び出しから分離され、分散ライブラリ コードがプロジェクト ソースの残りの部分から適切に分離されます。

[ビルド システム](./config.md#build-systems) が定義されているため、プロジェクトは環境にインストールされます。

```toml title="pyproject.toml" hl_lines="12-14"
[project]
name = "example-lib"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.11"
dependencies = []

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

!!! tip

    `--build-backend` を `hatchling`、`flit-core`、`pdm-backend`、`setuptools`、`maturin`、または `scikit-build-core` とともに使用することで、別のビルドバックエンドテンプレートを選択できます。[拡張モジュールを含むライブラリ](#projects-with-extension-modules) を作成する場合は、代替のバックエンドが必要です。

作成されたモジュールは単純な API 関数を定義します:

```python title="__init__.py"
def hello() -> str:
    return "Hello from example-lib!"
```

そして、`uv run` を使用してインポートして実行できます:

```console
$ cd example-lib
$ uv run python -c "import example_lib; print(example_lib.hello())"
Hello from example-lib!
```

## 拡張モジュールを使用したプロジェクト {#projects-with-extension-modules}

ほとんどの Python プロジェクトは「純粋な Python」です。つまり、C、C++、FORTRAN、Rust などの他の言語でモジュールを定義しません。ただし、拡張モジュールを含むプロジェクトは、パフォーマンスが重要となるコードによく使用されます。

拡張モジュールを使用してプロジェクトを作成するには、別のビルド システムを選択する必要があります。uv は、拡張モジュールのビルドをサポートする次のビルド システムを使用したプロジェクトの作成をサポートしています。

- [`maturin`](https://www.maturin.rs) Rust のプロジェクト用
- [`scikit-build-core`](https://github.com/scikit-build/scikit-build-core) C, C++, FORTRAN, Cython のプロジェクト用

`--build-backend` フラグを使用してビルドシステムを指定します:

```console
$ uv init --build-backend maturin example-ext
```

!!! note

    `--build-backend` を使用すると `--package` も暗黙的に使用されます。

プロジェクトには、一般的な Python プロジェクトファイルに加えて、`Cargo.toml` ファイルと `lib.rs` ファイルが含まれています。

```console
$ tree example-ext
example-ext
├── .python-version
├── Cargo.toml
├── README.md
├── pyproject.toml
└── src
    ├── lib.rs
    └── example_ext
        ├── __init__.py
        └── _core.pyi
```

!!! note

    `scikit-build-core` を使用する場合は、代わりに CMake 構成と `main.cpp` ファイルが表示されます。

Rust ライブラリは単純な関数を定義します:

```rust title="src/lib.rs"
use pyo3::prelude::*;

#[pyfunction]
fn hello_from_bin() -> String {
    "Hello from example-ext!".to_string()
}

#[pymodule]
fn _core(m: &Bound<'_, PyModule>) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(hello_from_bin, m)?)?;
    Ok(())
}
```

そして、Python モジュールはそれをインポートします:

```python title="src/example_ext/__init__.py"
from example_ext._core import hello_from_bin


def main() -> None:
    print(hello_from_bin())
```

このコマンドは `uv run` で実行できます:

```console
$ cd example-ext
$ uv run example-ext
Hello from example-ext!
```

!!! important

    `lib.rs` または `main.cpp` の拡張コードを変更するには、`--reinstall` を実行して再構築する必要があります。

## 最小限のプロジェクトを作成する

`pyproject.toml` のみを作成する場合は、`--bare` オプションを使用します:

```console
$ uv init example --bare
```

uv は、Python バージョンのピンファイル、README、およびソースディレクトリまたはファイルの作成をスキップします。さらに、uv はバージョンコントロールシステム (つまり、`git`) を初期化しません。

```console
$ tree example-bare
example-bare
└── pyproject.toml
```

uv は、`description` や `authors` などの追加のメタデータを `pyproject.toml` に追加しません。

```toml
[project]
name = "example"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = []
```

`--bare` オプションは、`--lib` や `--build-backend` などの他のオプションと一緒に使用できます。これらの場合、uv はビルドシステムを構成しますが、期待されるファイル構造は作成されません。

`--bare` を使用する場合でも、追加機能はオプトインで使用できます:

```console
$ uv init example --bare --description "Hello world" --author-from git --vcs git --python-pin
```
