---
title: Running scripts
description:
  A guide to using uv to run Python scripts, including support for inline dependency metadata,
  reproducible scripts, and more.
---

# スクリプトの実行

Python スクリプトは、`python <script>.py` などのスタンドアロン実行を目的としたファイルです。uv を使用してスクリプトを実行すると、環境を手動で管理しなくてもスクリプトの依存関係が管理されます。

!!! note

    Python 環境に慣れていない場合: すべての Python インストールには、パッケージをインストールできる環境があります。通常、各スクリプトに必要なパッケージを分離するには、[仮想環境](https://docs.python.org/3/library/venv.html) を作成することをお勧めします。uv は仮想環境を自動的に管理し、依存関係に対して [宣言型](#declaring-script-dependencies) アプローチを優先します。

## 依存関係のないスクリプトの実行

スクリプトに依存関係がない場合は、`uv run` で実行できます:

```python title="example.py"
print("Hello world")
```

```console
$ uv run example.py
Hello world
```

<!-- TODO(zanieb): Once we have a `python` shim, note you can execute it with `python` here -->

同様に、スクリプトが標準ライブラリのモジュールに依存している場合は、これ以上何もする必要はありません:

```python title="example.py"
import os

print(os.path.expanduser("~"))
```

```console
$ uv run example.py
/Users/astral
```

スクリプトに引数を与えることができる:

```python title="example.py"
import sys

print(" ".join(sys.argv[1:]))
```

```console
$ uv run example.py test
test

$ uv run example.py hello world!
hello world!
```

さらに、スクリプトは stdin から直接読み取ることもできます:

```console
$ echo 'print("hello world!")' | uv run -
```

または、シェルが[ヒアドキュメント](https://en.wikipedia.org/wiki/Here_document)をサポートしている場合:

```bash
uv run - <<EOF
print("hello world!")
EOF
```

`uv run` を _プロジェクト_、つまり `pyproject.toml` のあるディレクトリで使用すると、スクリプトを実行する前に現在のプロジェクトがインストールされることに注意してください。スクリプトがプロジェクトに依存しない場合は、`--no-project` フラグを使用してこれをスキップします。

```console
$ # Note: the `--no-project` flag must be provided _before_ the script name.
$ uv run --no-project example.py
```

プロジェクトでの作業の詳細については、[プロジェクトガイド](./projects.md)を参照してください。

## 依存関係のあるスクリプトを実行する

スクリプトに他のパッケージが必要な場合は、スクリプトが実行される環境にインストールする必要があります。uv は、手動で管理された依存関係を持つ長期仮想環境を使用するのではなく、オンデマンドでこれらの環境を作成することを好みます。これには、スクリプトに必要な依存関係の明示的な宣言が必要です。一般的に、依存関係を宣言するには [プロジェクト](./projects.md) または [インライン・メタデータ](#declaring-script-dependencies) を使用することをお勧めしますが、uv は呼び出しごとに依存関係を要求することもサポートしています。

たとえば、次のスクリプトでは `rich` が必要です。

```python title="example.py"
import time
from rich.progress import track

for i in track(range(20), description="For example:"):
    time.sleep(0.05)
```

依存関係を指定せずに実行すると、このスクリプトは失敗します:

```console
$ uv run --no-project example.py
Traceback (most recent call last):
  File "/Users/astral/example.py", line 2, in <module>
    from rich.progress import track
ModuleNotFoundError: No module named 'rich'
```

`--with` オプションを使用して依存関係を要求します:

```console
$ uv run --with rich example.py
For example: ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:01
```

特定のバージョンが必要な場合は、要求された依存関係に制約を追加できます:

```console
$ uv run --with 'rich>12,<13' example.py
```

`--with` オプションを繰り返すことで、複数の依存関係を要求できます。

`uv run` が _プロジェクト_ で使用されている場合、これらの依存関係はプロジェクトの依存関係 _に加えて_ 含まれることに注意してください。この動作をオプトアウトするには、`--no-project` フラグを使用します。

## Python スクリプトの作成

Python は最近、[インラインスクリプト・メタデータ](https://packaging.python.org/en/latest/specifications/inline-script-metadata/#inline-script-metadata) の標準フォーマットを追加しました。これにより、Python バージョンの選択と依存関係の定義が可能になります。インライン・メタデータを使用してスクリプトを初期化するには、`uv init --script` を使用します。

```console
$ uv init --script example.py --python 3.12
```

## スクリプトの依存関係の宣言

インライン・メタデータ形式を使用すると、スクリプトの依存関係をスクリプト自体で宣言できます。

uv はインラインスクリプト・メタデータの追加と更新をサポートしています。スクリプトの依存関係を宣言するには、`uv add --script` を使用します。

```console
$ uv add --script example.py 'requests<3' 'rich'
```

これにより、スクリプトの先頭に TOML を使用して依存関係を宣言する `script` セクションが追加されます。

```python title="example.py"
# /// script
# dependencies = [
#   "requests<3",
#   "rich",
# ]
# ///

import requests
from rich.pretty import pprint

resp = requests.get("https://peps.python.org/api/peps.json")
data = resp.json()
pprint([(k, v["title"]) for k, v in data.items()][:10])
```

uv はスクリプトを実行するために必要な依存関係を持つ環境を自動的に作成します。例:

```console
$ uv run example.py
[
│   ('1', 'PEP Purpose and Guidelines'),
│   ('2', 'Procedure for Adding New Modules'),
│   ('3', 'Guidelines for Handling Bug Reports'),
│   ('4', 'Deprecation of Standard Modules'),
│   ('5', 'Guidelines for Language Evolution'),
│   ('6', 'Bug Fix Releases'),
│   ('7', 'Style Guide for C Code'),
│   ('8', 'Style Guide for Python Code'),
│   ('9', 'Sample Plaintext PEP Template'),
│   ('10', 'Voting Guidelines')
]
```

!!! important

    インラインスクリプト・メタデータを使用する場合、`uv run` が [プロジェクトで使用されている](../concepts/projects/run.md) 場合でも、プロジェクトの依存関係は無視されます。`--no-project` フラグは必要ありません。

uv は Python のバージョン要件も尊重します:

```python title="example.py"
# /// script
# requires-python = ">=3.12"
# dependencies = []
# ///

# Use some syntax added in Python 3.12
type Point = tuple[float, float]
print(Point)
```

!!! note

    `dependencies` フィールドは空の場合でも提供する必要があります。

`uv run` は必要な Python バージョンを検索して使用します。Python バージョンがインストールされていない場合はダウンロードされます。詳細については、[Python バージョン](../concepts/python-versions.md) のドキュメントを参照してください。

## 代替パッケージインデックスの使用

依存関係を解決するために別の [パッケージインデックス](../configuration/indexes.md) を使用する場合は、`--index` オプションを使用してインデックスを指定できます。

```console
$ uv add --index "https://example.com/simple" --script example.py 'requests<3' 'rich'
```

これにより、インライン・メタデータにパッケージデータが含まれます。

```python
# [[tool.uv.index]]
# url = "https://example.com/simple"
```

パッケージ インデックスにアクセスするために認証が必要な場合は、[パッケージインデックス](../configuration/indexes.md) のドキュメントを参照してください。

## 依存関係のロック

uv は、`uv.lock` ファイル形式を使用して PEP 723 スクリプトの依存関係のロックをサポートします。プロジェクトとは異なり、スクリプトは `uv lock` を使用して明示的にロックする必要があります。

```console
$ uv lock --script example.py
```

`uv lock --script` を実行すると、スクリプトの隣に `.lock` ファイルが作成されます (例: `example.py.lock`)。

一度ロックされると、`uv run --script`、`uv add --script`、`uv export --script`、`uv tree --script` などの後続の操作では、ロックされた依存関係が再利用され、必要に応じてロックファイルが更新されます。

このようなロックファイルが存在しない場合は、`uv export --script` などのコマンドは期待どおりに機能しますが、ロックファイルは作成されません。

## 再現性の向上

依存関係をロックするだけでなく、uv はインラインスクリプト・メタデータの `tool.uv` セクションの `exclude-newer` フィールドをサポートし、特定の日付より前にリリースされたディストリビューションのみを考慮するように uv を制限します。これは、後で実行するときにスクリプトの再現性を向上させるのに役立ちます。

日付は [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339.html) タイムスタンプとして指定する必要があります (例: `2006-12-02T02:07:43Z`)。

```python title="example.py"
# /// script
# dependencies = [
#   "requests",
# ]
# [tool.uv]
# exclude-newer = "2023-10-16T00:00:00Z"
# ///

import requests

print(requests.__version__)
```

## 異なる Python バージョンの使用

uv を使用すると、スクリプトの呼び出しごとに任意の Python バージョンを要求できます。次に例を示します。

```python title="example.py"
import sys

print(".".join(map(str, sys.version_info[:3])))
```

```console
$ # Use the default Python version, may differ on your machine
$ uv run example.py
3.12.6
```

```console
$ # Use a specific Python version
$ uv run --python 3.10 example.py
3.10.15
```

Python バージョンのリクエストの詳細については、[Python バージョンのリクエスト](../concepts/python-versions.md#requesting-a-version) ドキュメントを参照してください。

## GUI スクリプトの使用

Windows では、`uv` は `pythonw` を使用して `.pyw` 拡張子で終わるスクリプトを実行します:

```python title="example.pyw"
from tkinter import Tk, ttk

root = Tk()
root.title("uv")
frm = ttk.Frame(root, padding=10)
frm.grid()
ttk.Label(frm, text="Hello World").grid(column=0, row=0)
root.mainloop()
```

```console
PS> uv run example.pyw
```

![Run Result](../assets/uv_gui_script_hello_world.png){: style="height:50px;width:150px"}

依存関係も同様に機能します:

```python title="example_pyqt.pyw"
import sys
from PyQt5.QtWidgets import QApplication, QWidget, QLabel, QGridLayout

app = QApplication(sys.argv)
widget = QWidget()
grid = QGridLayout()

text_label = QLabel()
text_label.setText("Hello World!")
grid.addWidget(text_label)

widget.setLayout(grid)
widget.setGeometry(100, 100, 200, 50)
widget.setWindowTitle("uv")
widget.show()
sys.exit(app.exec_())
```

```console
PS> uv run --with PyQt5 example_pyqt.pyw
```

![Run Result](../assets/uv_gui_script_hello_world_pyqt.png){: style="height:50px;width:150px"}

## 次のステップ

`uv run` の詳細については、[コマンドリファレンス](../reference/cli.md#uv-run) を参照してください。

または、uv を使用して [ツールを実行およびインストール](./tools.md) する方法については、以下をお読みください。
