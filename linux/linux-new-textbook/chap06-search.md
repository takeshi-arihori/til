# 探す、調べる

## ファイルを探す
ファイルを探すコマンド。
- `find <検索開始ディレクトリ> <検索条件> <アクション>`
引数で指定した<検索開始ディレクトリ>を起点として、<検索条件>を満たすファイルを探し、<アクション>を実行。(検索条件に何も指定しない場合は全てのファイルとディレクトリが対象になる)

例: `find . -name file-1.txt -print`: -printは省略可

### findコマンドで利用可能な検索条件
- `-name` `-iname`: ファイル名を指定(`-iname`は大文字小文字は区別しない)
  - ファイル名の指定にはワイルドカードとして`*`と`?`が利用できる。(`’’`や`""`で囲まないとパス名展開が行われるため)
  - 例: `find . -name '*.txt' -print`
- `-type`: 検索条件を絞り込む
  - `-type f`: 通常ファイル
  - `-type d`: ディレクトリ
  - `-type l`: シンボリックリンク
  - `-a`: 複数の検索条件指定 例: `find . -type f -a -name '*.txt' -print`

### locate コマンド
- `locate`: パス名の一部を指定してファイル名データベースからファイルを探す(findコマンドに比べて非常に高速に検索できる)
  - `sudo apt-get install plocate`: locateのインストール
  - `sudo updatedb`: install後に行うコマンド。updatedbコマンドを実行して、データベースを作成する
  - `locate bash`: `bash`という文字列を含むパス名を検索
 
### findとlocateの違い
- find: 実行するたびにディレクトリツリーを下って全てのファイルを探す
- locate: 事前にファイルパスのデータベースが作られており、このデータベースだけを検索するため、高速に動作する。  
  locateコマンドはインストール時に、ファイルパスのデータベースを一日一回作成するように設定されている。
  更新される前のファイルは見つからないので注意。

## コマンドの使い方を調べる
### --helpオプション
コマンド自身のヘルプメッセージを表示  
例: `cat --help`    
**結果**  
```
使用法: cat [オプション]... [ファイル]...
ファイル (複数可) の内容を結合して標準出力に出力します。

ファイルの指定がない場合や FILE が - の場合, 標準入力から読み込みを行います。

  -A, --show-all           -vET と同じ
  -b, --number-nonblank    空行以外に行番号を付ける。-n より優先される
  -e                       -vE と同じ
  -E, --show-ends          行の最後に $ を付ける
  -n, --number             全ての行に行番号を付ける
  -s, --squeeze-blank      連続した空行の出力を行わない
  -t                       -vT と同じ
  -T, --show-tabs          TAB 文字を ^I で表示
  -u                       (無視される)
  -v, --show-nonprinting   ^ や M- 表記を使用する (LFD と TAB は除く)
      --help        display this help and exit
      --version     output version information and exit

```

### manコマンド
指定したコマンドのオンラインマニュアルを表示するコマンド 
- `man <調べたいコマンド名>`: lessコマンドを使ってマニュアルが表示される
- `man -k <キーワード>`: キーワードからコマンドを探す

例: `man man`: manコマンドのマニュアル

### コマンドのフルパス
- `echo $PATH`: コマンドを探す場所の確認(コマンドを探すディレクトリを`:`で連結したものが表示される)
本来、`ls -lF /bin/cat`のようにフルパスで入力する必要があるが、シェルがコマンドを実行する際に、`$PATH` という環境変数に設定された場所から自動的にコマンドを探してくれるため `cat` のみで実行できる。


### whichコマンド
- `which [オプション] <コマンド名>`: 指定されたコマンド名をサーチパスから探して、見つかった実行ファイルのフルパスを表示
例: `which cat` -> `/bin/cat`

## 日本語ドキュメントと英語ドキュメント
- `LANG=ja_JP.UTF-8 cat --help`: 日本語を明示的に指定してヘルプ表示
- `LANG=C cat --help`: 英語を明示的に指定








