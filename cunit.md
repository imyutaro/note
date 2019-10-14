# CUnit C言語の単体テストについて

## CUintのインストール

大学のWindowsにインストールするためlocalにインストールする．

まず`HOME`以下に`local/src`フォルダを作る．
そして以下のコマンドを実行してCUnitをダウンロード，解凍．

```bash
> curl -k -L https://sourceforge.net/projects/cunit/files/latest/download -o ${HOME}/local/src/CUnit-2.1-3.tar.bz2
> tar xvf ${HOME}/local/src/CUnit-2.1-3.tar.bz2 -C ${HOME}/local/src/
```

ダウンロードしたらconfigure scriptを生成して，実行，CUnitのインストール．

```bash
> autoreconf -i                         # configure scriptの生成
> ./configure --prefix=${HOME}/local/   # インストール先を指定してmakefileの生成
> make && make install                  # インストール
```

シェルに合わせて以下を追加しておく．bashなら`.bashrc`，zshなら`.zshrc`．自分のシェルが何なのか確認するには`echo $SHELL`．

```bash
export DIR="$HOME/local"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DIR/lib
export PATH=$HOME/local/bin:$PATH
```

追加したら読み込みを忘れずに．

```bash
> source .bashrc
```

## 使い方

プロ入2のコードはscanfで入力を要求しているのでCUnitを使ったテストができない...main関数の引数部分をscanfに沿って自分で書き換えるコードを書くか，shell scriptでテストを書くのが良さそう... \
プログラムがちゃんと引数撮って動くものを課題としていればCUnitでテストコード書いてある程度，自動化できたのに...

CUnitの使い方はReferenceの1の記事が一番参考になる．

References:
1. [CUnitについての備忘録 - Qiita](https://qiita.com/from_chc/items/db771bef1e83fc00783a)
2. [CUnitでCプログラムの単体テストをする - Qiita](https://qiita.com/muniere/items/ff9a984ed7e51eee7112)

## ちなみに

課題一括ダウンロードをwgetでする方法は以下のコマンドを実行すれば良い．`username`，`password`にはそれぞれ与えられてるユーザ名とパスワードを入れる．下記のやり方は`digest認証 wget`とかで調べれば出てくる．

```bash
> wget --http-user=username --http-passwd=password --no-check-certificate https://url/to/download
```

## シェルスクリプトでのテスト

ということでshell scriptでテストコードを書く方法をメモ．(テストって言ってもassertとかしないで入力とかを自動で入力させて結果を全部自動で出力させて目視で確認かも...)

```bash
> echo 1 | ./a.out
```

でscanfに入力できる．
