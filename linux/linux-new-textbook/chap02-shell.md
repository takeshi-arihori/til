# シェルってなんだろう?

## シェルとコマンド

### Linux内で行われている処理
**例**: `date`  
1. キーボードから入力された、dateという文字列を受け取る
2. dateという名前のコマンドを探す
3. 見つかったコマンドを実行する <- Linuxカーネルが実行する
4. 実行した結果として得られた日時の文字列を画面に表示する

**カーネルの役割**: CPUやメモリなどのハードウェアを管理 + コマンド実行を処理するプロセス管理
![linux-Kernel drawio](https://github.com/user-attachments/assets/1ff881fe-edb0-4d8a-bcc3-5340aa558875)

**シェル**: カーネルのインターフェースになっているソフトウェア
![linux-Shell drawio](https://github.com/user-attachments/assets/6183e377-ef0e-4261-8e40-849a5f1d226a)

Linuxカーネルは直接人間が操作できるようにはなっていないため、ユーザーとカーネルの間に、「コマンドを実行する」「入出力を指定する」などのユーザインターフェースを持つソフトウェアが必要になる。(シェル)
シェル = 「殻」という意味を持っていることが由来

### カーネルとシェルを分離するメリット
- Linuxカーネルには手を入れずに、シェルだけを自分好みのものに変更できる
- Linux以外のOSを利用する際も、シェルが移植されていれば操作を同様に行うことができる
- シェルがエラーや高負荷状態になってクラッシュしたとしても、OS本体であるLinuxカーネルへの影響を最小限に抑えることができる

Linuxには「1つのプログラムには、1つのことをうまくやらせる」という考え方がある。

## プロンプト
- **ログインシェル**: ログイン時に最初に起動されるシェル

**自分の使用しているログインシェルの確認方法**  
```
echo $SHELL
```
bin というディレクトリにある bash がログインシェルである。Linuxは明示的に指定しない限り bash が利用される。

- **対話型(インタラクティブ)操作**: ユーザがキーボードと画面を利用して直接シェルを操作する方法
- **シェルスクリプト**: 一連の操作の流れを記述したファイル

## シェルの種類
- **sh**
  最も古くから存在するシェルで`Bornシェル`と呼ばれている。標準シェルとしての地位を確立しているため、シェルスクリプトを書く際には`sh`を利用するのが現在でも一般的。
  現在ではログインシェルで使われることはほとんどない。 

- **csh**:
  古くからあるシェルの一つで、`Cシェル` と呼ばれている。shに比べ、対話型操作が便利になる機能を実装したことで人気があった。
  シェルの文法がshと大きく異なり、特にシェルスクリプトを書く上では欠陥があるため適していない。
  
- **bash**:
  shを基本として、機能を拡張したシェル。shと後方互換性を持つため、単純にshを置き換える用途でも利用できる。
  対話型操作を行う上で十分な機能を持ち、多くのLinux環境でデフォルトのログインシェルとして使われている。
  
- **tcsh**:
  cshの後継として開発されたCシェル系のシェル。対話型操作で便利な機能を多く持っている。
  シェルスクリプトを書くのに向いていない。
  
- **zsh**:
  比較的新しいシェルで、bashやtcshをはじめとした他のシェルの機能を積極的に取り込み、さらに独自の拡張を加えたシェル。
  マニュアルだけでも17セクションあり、機能面では他のシェルを圧倒している。

**一時的にshに切り替え** 
```
sh
```
実際は他中に起動しているため、重ねている感じ   
![linux-shellの切り替え drawio](https://github.com/user-attachments/assets/4b5f6e47-b017-46a0-bf07-939d1074ddf7)

## ターミナルとは
シェルとは全く違うソフトウェア

現在ではハードウェアとしてのターミナルはほとんどない。-> ターミナルエミュレータ 
|  OS   |    ターミナルエミュレータ  |
| ----- | -------------------- |
| Windows | PuTTY, Tera Term |
| Mac OS X | ターミナル, iterm2 |
| Linux | GNOME端末(GNOME Terminal), Konsole|




