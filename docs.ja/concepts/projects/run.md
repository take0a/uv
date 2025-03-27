# プロジェクト内でのコマンドの実行

プロジェクトで作業する場合、`.venv` の仮想環境にインストールされます。この環境はデフォルトで現在のシェルから分離されているため、`python -c "import example"` などのプロジェクトを必要とする呼び出しは失敗します。代わりに、`uv run` を使用してプロジェクト環境でコマンドを実行します:

```console
$ uv run python -c "import example"
```

`run` を使用する場合、uv は指定されたコマンドを実行する前にプロジェクト環境が最新であることを確認します。

指定されたコマンドはプロジェクト環境によって提供されるか、その外部に存在する可能性があります。例:

```console
$ # Presuming the project provides `example-cli`
$ uv run example-cli foo

$ # Running a `bash` script that requires the project to be available
$ uv run bash scripts/foo.sh
```

## 追加の依存関係のリクエスト {#requesting-additional-dependencies}

呼び出しごとに追加の依存関係または異なるバージョンの依存関係を要求できます。

`--with` オプションは、呼び出しの依存関係を含めるために使用されます。たとえば、`httpx` の別のバージョンを要求する場合などです:

```console
$ uv run --with httpx==0.26.0 python -c "import httpx; print(httpx.__version__)"
0.26.0
$ uv run --with httpx==0.25.0 python -c "import httpx; print(httpx.__version__)"
0.25.0
```

プロジェクトの要件に関係なく、要求されたバージョンが尊重されます。たとえば、プロジェクトで `httpx==0.24.0` が必要な場合でも、上記の出力は同じになります。

## スクリプトの実行

インライン メタデータを宣言するスクリプトは、プロジェクトから分離された環境で自動的に実行されます。詳細については、[スクリプト ガイド](../../guides/scripts.md#declaring-script-dependencies) を参照してください。

たとえば、次のようなスクリプトがあるとします:

```python title="example.py"
# /// script
# dependencies = [
#   "httpx",
# ]
# ///

import httpx

resp = httpx.get("https://peps.python.org/api/peps.json")
data = resp.json()
print([(k, v["title"]) for k, v in data.items()][:10])
```

呼び出し `uv run example.py` は、指定された依存関係のみがリストされた状態で、プロジェクトから _分離_ して実行されます。

## レガシー Windows スクリプト

[レガシー setuptools スクリプト](https://packaging.python.org/en/latest/guides/distributing-packages-using-setuptools/#scripts)のサポートが提供されています。これらのタイプのスクリプトは、setuptools によって `.venv\Scripts` にインストールされる追加ファイルです。

現在、`.ps1`、`.cmd`、および `.bat` 拡張子を持つレガシー スクリプトのみがサポートされています。

たとえば、以下はコマンド プロンプト スクリプトを実行する例です。

```console
$ uv run --with nuitka==2.6.7 -- nuitka.cmd --version
```

また、拡張子を指定する必要もありません。`uv` は、`.ps1`、`.cmd`、`.bat` で終わるファイルを実行順に自動的に検索します。

```console
$ uv run --with nuitka==2.6.7 -- nuitka --version
```

## シグナルのハンドリング

uv は、失敗時に適切なエラー メッセージを提供するために、生成されたコマンドにプロセスの制御を譲りません。その結果、uv は、要求されたコマンドが実行される子プロセスにいくつかのシグナルを転送する役割を担います。

Unix システムでは、uv は SIGINT と SIGTERM を子プロセスに転送します。シェルは Ctrl-C でフォアグラウンド プロセス グループに SIGINT を送信するため、uv は SIGINT が複数回検出された場合、または子プロセス グループが uv と異なる場合にのみ、子プロセスに SIGINT を転送します。

Windows では、これらの概念は適用されず、uv は Ctrl-C イベントを無視し、子プロセスに処理を延期して、正常に終了できるようにします。
