# uv のインストール

## インストール方法

スタンドアロン・インストーラーまたは選択したパッケージ・マネージャーを使用して uv をインストールします。

### スタンドアロン・インストーラー

uv は uv をダウンロードしてインストールするためのスタンドアロン・インストーラーを提供します:

=== "macOS and Linux"

    `curl` を使用してスクリプトをダウンロードし、`sh` で実行します:

    ```console
    $ curl -LsSf https://astral.sh/uv/install.sh | sh
    ```

    システムに `curl` がない場合は `wget` を使うことができます:

    ```console
    $ wget -qO- https://astral.sh/uv/install.sh | sh
    ```

    URL に特定のバージョンを含めることで、そのバージョンをリクエストします:

    ```console
    $ curl -LsSf https://astral.sh/uv/0.6.10/install.sh | sh
    ```

=== "Windows"

    `irm` を使用してスクリプトをダウンロードし、`iex` で実行します:

    ```console
    $ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
    ```

    [実行ポリシー](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4#powershell-execution-policies)を変更すると、インターネットからのスクリプトを実行できるようになります。

    URL に特定のバージョンを含めることで、そのバージョンをリクエストします:

    ```console
    $ powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/0.6.10/install.ps1 | iex"
    ```

!!! tip

    インストールスクリプトは使用前に検査できます:

    === "macOS and Linux"

        ```console
        $ curl -LsSf https://astral.sh/uv/install.sh | less
        ```

    === "Windows"

        ```console
        $ powershell -c "irm https://astral.sh/uv/install.ps1 | more"
        ```

    あるいは、インストーラーまたはバイナリを [GitHub](#github-releases) から直接ダウンロードすることもできます。

uv のインストールのカスタマイズの詳細については、[インストーラーの構成](../configuration/installer.md) のドキュメントを参照してください。

### PyPI

便宜上、uv は [PyPI](https://pypi.org/project/uv/) に公開されています。

PyPI からインストールする場合は、`pipx` などを使用して、隔離された環境に uv をインストールすることをお勧めします:

```console
$ pipx install uv
```

ただし、`pip` も使用できます:

```console
$ pip install uv
```

!!! note

    uv には、多くのプラットフォーム向けのビルド済みディストリビューション (wheels) が付属しています。特定のプラットフォームで wheel が利用できない場合は、uv はソースからビルドされます。これには Rust ツールチェーンが必要です。ソースから uv をビルドする方法の詳細については、[コントリビューティング・セットアップガイド](https://github.com/astral-sh/uv/blob/main/CONTRIBUTING.md#setup) を参照してください。

### Cargo

uv は Cargo 経由で利用可能ですが、未公開の crates に依存しているため、[crates.io](https://crates.io) ではなく Git からビルドする必要があります。

```console
$ cargo install --git https://github.com/astral-sh/uv uv
```

### Homebrew

uv はコア Homebrew パッケージで利用できます。

```console
$ brew install uv
```

### WinGet

uv は [WinGet](https://winstall.app/apps/astral-sh.uv) から入手できます。

```console
$ winget install --id=astral-sh.uv  -e
```

### Scoop

uv は [Scoop](https://scoop.sh/#/apps?q=uv) から入手できます。

```console
$ scoop install main/uv
```

### Docker

uv は [`ghcr.io/astral-sh/uv`](https://github.com/astral-sh/uv/pkgs/container/uv) で Docker イメージを提供します。

詳細については、[Docker での uv の使用](../guides/integration/docker.md) に関するガイドを参照してください。

### GitHub リリース

uv のリリース成果物は、[GitHub リリース](https://github.com/astral-sh/uv/releases)から直接ダウンロードできます。

各リリースページには、サポートされているすべてのプラットフォームのバイナリと、`astral.sh` の代わりに `github.com` 経由でスタンドアロン・インストーラーを使用する手順が含まれています。

## uv のアップグレード

uv をスタンドアロン・インストーラーでインストールすると、オンデマンドで更新できます:

```console
$ uv self update
```

!!! tip

    uv を更新するとインストーラーが再実行され、シェルのプロファイルが変更される可能性があります。この動作を無効にするには、`INSTALLER_NO_MODIFY_PATH=1` を設定します。

別のインストール方法を使用すると、自己更新は無効になります。代わりにパッケージ・マネージャーのアップグレード方法を使用してください。たとえば、`pip` の場合:

```console
$ pip install --upgrade uv
```

## シェルの自動補完

!!! tip

    シェルを確認するには、`echo $SHELL` を実行します。

uv コマンドのシェル自動補完を有効にするには、次のいずれかを実行します:

=== "Bash"

    ```bash
    echo 'eval "$(uv generate-shell-completion bash)"' >> ~/.bashrc
    ```

=== "Zsh"

    ```bash
    echo 'eval "$(uv generate-shell-completion zsh)"' >> ~/.zshrc
    ```

=== "fish"

    ```bash
    echo 'uv generate-shell-completion fish | source' >> ~/.config/fish/config.fish
    ```

=== "Elvish"

    ```bash
    echo 'eval (uv generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
    ```

=== "PowerShell / pwsh"

    ```powershell
    if (!(Test-Path -Path $PROFILE)) {
      New-Item -ItemType File -Path $PROFILE -Force
    }
    Add-Content -Path $PROFILE -Value '(& uv generate-shell-completion powershell) | Out-String | Invoke-Expression'
    ```

uvx のシェルの自動補完を有効にするには、次のいずれかを実行します:

=== "Bash"

    ```bash
    echo 'eval "$(uvx --generate-shell-completion bash)"' >> ~/.bashrc
    ```

=== "Zsh"

    ```bash
    echo 'eval "$(uvx --generate-shell-completion zsh)"' >> ~/.zshrc
    ```

=== "fish"

    ```bash
    echo 'uvx --generate-shell-completion fish | source' >> ~/.config/fish/config.fish
    ```

=== "Elvish"

    ```bash
    echo 'eval (uvx --generate-shell-completion elvish | slurp)' >> ~/.elvish/rc.elv
    ```

=== "PowerShell / pwsh"

    ```powershell
    if (!(Test-Path -Path $PROFILE)) {
      New-Item -ItemType File -Path $PROFILE -Force
    }
    Add-Content -Path $PROFILE -Value '(& uvx --generate-shell-completion powershell) | Out-String | Invoke-Expression'
    ```

次に、シェルを再起動するか、シェル構成ファイルを読み込みます。

## アンインストール

システムから uv を削除する必要がある場合は、次の手順に従ってください:

1.  保存されたデータをクリーンアップする（オプション）:

    ```console
    $ uv cache clean
    $ rm -r "$(uv python dir)"
    $ rm -r "$(uv tool dir)"
    ```

    !!! tip

        バイナリを削除する前に、uv が保存したデータをすべて削除することをお勧めします。

2.  uv と uvx バイナリを削除します:

    === "macOS and Linux"

        ```console
        $ rm ~/.local/bin/uv ~/.local/bin/uvx
        ```

    === "Windows"

        ```powershell
        $ rm $HOME\.local\bin\uv.exe
        $ rm $HOME\.local\bin\uvx.exe
        ```

    !!! note

        0.5.0 より前では、uv は `~/.cargo/bin` にインストールされていました。アンインストールするには、そこからバイナリを削除できます。古いバージョンからアップグレードしても、`~/.cargo/bin` からバイナリは自動的に削除されません。

## 次のステップ

[最初のステップ](./first-steps.md)を参照するか、[ガイド](../guides/index.md)に直接移動して uv の使用を開始してください。
