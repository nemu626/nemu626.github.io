---
title: "Windows11にWSLベースの開発環境をセットアップした記録"
date: 2023-03-10T16:42:55+09:00
draft: false
---

この度、Windows11のノートPCを購入したので、開発環境をセットアップしました。ここ数年はLinux、Macを開発用に使っていたので、Windowsで快適な環境構築ができることに驚きつつ、記録を残しておきます。

ざっと以下のような作業を行います。

1. Windows Terminalの設定
2. WSLの導入
3. VsCodeの導入
4. Ubuntuのセットアップ(In WSL)

## Windows Terminalの設定

最新のWindows11には、コマンドベースのパッケージマネージャーである`winget`やモダンなターミナル環境`windows terminal`がデフォルトで入っており、CLI環境がかなり進化していてびっくりです。特にこだわらなくてもいい感じのCLI環境が手に入りますね。 Windows Terminalをより使いやすくするために設定していきます。

### nerd fontの設定

[`nerd font`](https://www.nerdfonts.com/#home) とは、各種アイコン類を内包した開発者向けのフォント群のことです。ターミナルをおしゃれにするには必須。英語のみのNerdFontは、上記リンクからダウンロードできます。日本語対応しているNerd Fontをいくつかリストアップしておきます。

* [GitHub - yuru7/HackGen: Hack と源柔ゴシックを合成したプログラミングフォント 白源 (はくげん／HackGen)](https://github.com/yuru7/HackGen)
*

私は`Hack`フォントに日本語`mgen+`を合成した[cica](https://github.com/miiton/Cica)を愛用しています。

### quake modeの設定

`quake mode`とは、ホットキー（通常は`F12`）を押すと、いつでも画面の上部にターミナルがニュッと現れてくるモードのことです。いつでもターミナルにアクセスできるのがかなり便利です。`Windows terminal` 1.9からquake modeに対応しているみたいです。デフォルトのホットキーは`Win` + `~` になっていますが、お好きなキーに設定しましょう。私はLinuxやMacで使っていたのと同じく`F12`にしました。

### 外観の設定

`windows terminal`

## WSLの導入

Windows上でLinuxを使えるようになる`Windows Subsystem for Linux`はコマンド一行で導入できました。

```powershell
wsl --install
```

`wsl --list --online`でInstallするディストロを確認することができます。Arch系があれば使いたかったのですが、Debian・Ubuntu系しかなかったので、おとなしくUbuntuにします。
インストール後再起動が必要なので再起動してください。

## vscodeの導入

みんな大好きVscodeもPowershellでWingetコマンドでインストールできるのが便利ですね。もちろん公式HPからインストーラをダウンロードしてインストーすることもできます。

```powershell
winget install vscode
```

WSLのディレクトリをWindowsのVscodeで開くには、LinuxやMacと同じように、WSLのシェル上で以下のように開けばOKっぽいです。感動です。

```shell
code .
```

## WSL上のUbuntuのセットアップ

Windowsのセットアップとか書いておいて、実はこのターミナル環境の整備がメインタスクですね。

### install essential tools

```shell
$ sudo apt update && sudo apt upgrade -y
sudo apt install build-essential
```

### install docker

WSL2上から普通にインストールしても良いのですが、MS公式が勧めているのはDocker DesktopをWindowsにインストール後、BackendにWSL2をIntegrationして使えとのことです。

```shell
$ 

### install zsh

UbuntuのデフォルトShellは`Bash`ですが、より高機能な`zsh`をインストールし、既定のShellにします。`fish`も使いやすいですが、POSIX互換を重視するならZshのほうが無難です。

```shell
$ sudo apt install zsh
$ chsh -s /bin/zsh
```

`zsh`の初回起動時に初期設定ウィザードが出てくるので、設定します。基本的にすべてデフォルトで良いと思います。

### install starship

[starship](https://starship.rs/ja-jp/)はどのシェルでもいい感じのPromptにしてくれるRust製cross-shell promptです。

```shell
curl -sS https://starship.rs/install.sh | sh

echo 'eval "$(starship init zsh)"' >> ~/.zshrc
```

### install rust CLI applications

既存のCLIコマンドを代替するRust製コマンドをインストールしておくとかなり便利なのでおすすめです。

* <https://github.com/TaKO8Ki/awesome-alternatives-in-rust>
  
私の場合、以下のコマンド群をインストールしました。

* `bat` : ページングやシンタクスハイライトなどがついている`cat`コマンド
* `lsd` : カラフルな`ls`コマンド
* `skim` : rust製`fzf`。あいまいな単語でファイルを検索してくれます。
* `navi` : CLIコマンドの使い方をカタログのように表示してくれるチートシートです。困ったらこれ
* `zoxide` : かなり便利な`cd`コマンドの代替
* `delta` : 見やすいdiff tool

```shell
# install rust
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

$ cargo install bat lsd skim navi zoxide git-delta
$ echo 'eval "$(zoxide init zsh)"
alias ls=lsd
alias ll="lsd -lah"
alias cat="bat"
alias fzf="sk"' >> ~/.zshrc

```
