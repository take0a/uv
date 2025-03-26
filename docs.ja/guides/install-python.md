---
title: Installing and managing Python
description:
  A guide to using uv to install Python, including requesting specific versions, automatic
  installation, viewing installed versions, and more.
---

# Python のインストール

システムに Python がすでにインストールされている場合、 uv は設定なしでそれを [検出して使用](#using-existing-python-versions) します。ただし、 uv は Python のバージョンをインストールして管理することもできます。 uv は必要に応じて不足している Python のバージョンを [自動的にインストール](#automatic-python-downloads) します。開始するために Python をインストールする必要はありません。

## はじめる

最新の Python のバージョンをインストールするには:

```console
$ uv python install
```

!!! note

    Python は公式の配布可能なバイナリを公開していません。そのため、uv は Astral [`python-build-standalone`](https://github.com/astral-sh/python-build-standalone) プロジェクトの配布を使用します。詳細については、[Python 配布](../concepts/python-versions.md#managed-python-distributions) のドキュメントを参照してください。

Python がインストールされると、`uv` コマンドによって自動的に使用されるようになります。

!!! important

    Python が uv によってインストールされると、グローバルには利用できなくなります (つまり、`python` コマンド経由では利用できなくなります)。この機能のサポートは _preview_ 段階です。詳細については、[Python 実行ファイルのインストール](../concepts/python-versions.md#installing-python-executables) を参照してください。

    [`uv run`](../guides/scripts.md#using-different-python-versions) または [仮想環境を作成してアクティブ化](../pip/environments.md) を使用して、`python` を直接使用することもできます。

## 特定のバージョンのインストール

特定の Python バージョンをインストールするには:

```console
$ uv python install 3.12
```

複数の Python バージョンをインストールするには:

```console
$ uv python install 3.11 3.12
```

代替の Python 実装をインストールするには、例えば PyPy:

```console
$ uv python install pypy@3.10
```

詳細については、[`python install`](../concepts/python-versions.md#installing-a-python-version) のドキュメントを参照してください。

## Python の再インストール

UV 管理の Python バージョンを再インストールするには、`--reinstall` を使用します。例:

```console
$ uv python install --reinstall
```

これにより、以前にインストールされたすべての Python バージョンが再インストールされます。Python ディストリビューションには常に改良が加えられているため、再インストールすると、Python バージョンが変更されていなくてもバグが解決される可能性があります。

## Python インストールの表示

利用可能なインストール済みの Python のバージョンを表示するには:

```console
$ uv python list
```

詳細については、[`python list`](../concepts/python-versions.md#viewing-available-python-versions) のドキュメントを参照してください。

## Python の自動ダウンロード

uv を使用するために Python を明示的にインストールする必要はありません。デフォルトでは、uv は必要なときに Python バージョンを自動的にダウンロードします。たとえば、次の例では、Python 3.12 がインストールされていない場合はダウンロードされます:

```console
$ uvx python@3.12 -c "print('hello world')"
```

特定の Python バージョンが要求されていない場合でも、uv は要求に応じて最新バージョンをダウンロードします。たとえば、システムに Python バージョンがない場合、次のコマンドは新しい仮想環境を作成する前に Python をインストールします:

```console
$ uv venv
```

!!! tip

    Python のダウンロードのタイミングをより細かく制御したい場合は、Python の自動ダウンロードを [簡単に無効にする](../concepts/python-versions.md#disabling-automatic-python-downloads) ことができます。

<!-- TODO(zanieb): Restore when Python shim management is added
Note that when an automatic Python installation occurs, the `python` command will not be added to the shell. Use `uv python install-shim` to ensure the `python` shim is installed.
-->

## 既存の Python バージョンの使用

uv は、システムに既存の Python インストールが存在する場合はそれを使用します。この動作に必要な設定はありません。uv は、コマンド呼び出しの要件を満たす場合、システムの Python を使用します。詳細については、[Python の検出](../concepts/python-versions.md#discovery-of-python-versions) ドキュメントを参照してください。

uv にシステム Python の使用を強制するには、`--no-managed-python` フラグを指定します。詳細については、[Python バージョンの設定](../concepts/python-versions.md#requiring-or-disabling-managed-python-versions) ドキュメントを参照してください。

## 次のステップ

`uv python` の詳細については、[Python バージョンのコンセプト](../concepts/python-versions.md) ページと [コマンドリファレンス](../reference/cli.md#uv-python) を参照してください。

または、[スクリプトを実行](./scripts.md)して uv で Python を呼び出す方法については、以下をお読みください。
