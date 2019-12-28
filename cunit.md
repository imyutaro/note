# テストについて（大学の授業の課題チェックのため）

## CUnit C言語の単体テストについて

### CUintのインストール

大学のWindowsにインストールするためlocalにインストールする．

まず`$HOME`以下に`local/src`フォルダを作る．
そして以下のコマンドを実行してCUnitをダウンロード，解凍．

```bash
curl -k -L https://sourceforge.net/projects/cunit/files/latest/download -o ${HOME}/local/src/CUnit-2.1-3.tar.bz2
tar xvf ${HOME}/local/src/CUnit-2.1-3.tar.bz2 -C ${HOME}/local/src/
```

ダウンロードしたらconfigure scriptを生成して，実行，CUnitのインストール．

```bash
autoreconf -i                         # configure scriptの生成
./configure --prefix=${HOME}/local/   # インストール先を指定してmakefileの生成
make && make install                  # インストール
```

シェルに合わせて以下を追加しておく．bashなら`.bashrc`，zshなら`.zshrc`．自分のシェルが何なのか確認するには`echo $SHELL`．

```bash
export DIR="$HOME/local"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DIR/lib
export PATH=$HOME/local/bin:$PATH
```

追加したら読み込みを忘れずに．

```bash
source .bashrc
```

### 使い方

プロ入2のコードはscanfで入力を要求しているのでCUnitを使ったテストができない...main関数の引数部分をscanfに沿って自分で書き換えるコードを書くか，shell scriptでテストを書くのが良さそう... \
プログラムがちゃんと引数取って動くものを課題としていればCUnitでテストコード書いてある程度，自動化できたのに... \
CUnitの使い方はReferenceの1の記事が一番参考になる．

* References
  * [CUnitについての備忘録 - Qiita](https://qiita.com/from_chc/items/db771bef1e83fc00783a)
  * [CUnitでCプログラムの単体テストをする - Qiita](https://qiita.com/muniere/items/ff9a984ed7e51eee7112)

### ちなみに

課題一括ダウンロードをwgetでする方法は以下のコマンドを実行すれば良い．`username`，`password`にはそれぞれ与えられてるユーザ名とパスワードを入れる．下記のやり方は`digest認証 wget`とかで調べれば出てくる．

```bash
wget --http-user=username --http-passwd=password --no-check-certificate https://url/to/download
```

## シェルスクリプトでプログラムを実行，入力する

ということでshell scriptでテストコードを書く方法をメモ．(テストって言ってもassertとかしないで入力とかを自動で入力させて結果を全部自動で出力させて目視で確認かも...)

```bash
echo -e "1\n" | ./a.out
```

echoで改行をしとくとscanfの入力が正確にできる．

assertとかで拾えたらいいけどprintfで標準出力させてるせいで表記ゆれの問題があって不可能．
目的の機能を持つ関数を別で定義して返す値をもっとspecificにしてくれたらテストコードかけるんだけど... \
~~そういうふうに課題書き換えたらいいと思うんだけど...~~

以下のようなシェルスクリプト実行して目視で確認するしかない...

下のテンプレートをベースにテストプログラムを作成する．
(外部ファイルを読み込むようにしてもいいかもしれない．外部ファイルとしてyamlファイルとかを用意してそこにテストケースを書いて，そのファイルを読み込んで実行とか．)

```bash
#!/bin/bash
. ../util.sh --source-only

shpath=$PWD
cd $(dirname $0)
path=${1:-../src}
cd $path

tfile=`basename $0`
n_file=`ls -1 $tfile*.c 2> /dev/null | wc -l`

if [ $n_file = 0 ]; then
  echo "There is no file starting with $tfile"
  exit 1
fi

# rm only exec files
efile=`ls $tfile* | grep -Ev "*.c"`
n_efile=`ls -1 $tfile* 2>/dev/null | grep -Ev "*.c" | wc -l`
if [ $n_efile -gt 0 ]; then
  echo "-- removed --------------"
  echo $efile
  echo "-------------------------"
  rm $efile
fi

echo -e "\n=============================" \
        "\n* \n" \
        "\n  \n" \
        "\n* " \
        "\n============================="

echo -e "\n==========Test case==========" \
        "\n " \
        "\n " \
        "\n " \
        "\n============================="

fail=()
for cfile in $tfile*.c
do
  filename=`basename $cfile .c`
  printf "\n%s\n" "$pad"
  echo "$filename"
  printf "%s\n" "$line"
  gcc "$PWD/$cfile" -o "$PWD/$filename" 2>> $shpath/err.log
  if [ $? -ne 0 ]; then
    # fail("${fail[@]}" $cfile)
    fail+=( $cfile )
    echo -e "Compile filed." \
            "\n============================="
    continue
  fi
  echo  | $PWD/$filename
  echo -e " Expected: \n"
  printf "%s\n" "$pad"
done

n_efile=`ls $tfile* | grep -Ev "*.c" | wc -l`
echo -e "\n========Compile check========" \
        "\nSrc File      : $n_file" \
        "\nExecute File  : $n_efile" \
        "\nCompile Error : $(($n_file-$n_efile))" \
        "\n============================="

print_files fail[@] "Failed files"
```

コンパイルエラーのファイルを表示するなど何回も利用する関数は`util.sh`にまとめておく．

```bash
#!/bin/bash

RED="\e[31m%s\e[m"
GREEN="\e[32m%s\e[m"
YELLOW="\e[33m%s\e[m"
pad=$(printf "%0.1s" "="{1..40})
line=$(printf "%0.1s" "-"{1..40})

pad_print(){
  padlength=40
  string=${1}
  len=$(( (padlength-${#string})/2 ))
  midlen=$(( len+${#sting}-1 ))
  printf "\n%*.*s" 0 $((len-1)) "$pad"
  printf " $RED " "${string}"
  printf "%*.*s\n" $midlen $((padlength-len-${#string}-1)) "$pad"
}

print_files(){
  arr=("${!1}")
  item="${2}"
  if [ ${#arr[@]} -gt 0 ]; then
    pad_print "${item}"
    for failfile in ${arr[@]}; do
      echo " ${failfile}"
    done
    printf "%s\n" "$pad"
  fi
}

extract_filename(){
  src=${1}
  mode=${2}
  filename=`cat $src | tr -d " " | pcregrep -o --no-filename "(?<=fopen\(\").*?(?=\",\"$mode\"\))"`
  if [ ${#filename} -gt 0 ]; then
    echo "$filename"
  else
    echo "tmp.dat"
  fi
}

null_check(){
  src=("${!1}")
  no_null_check=()

  for cfile in ${src[@]}; do
    vars=(`cat ${cfile} | tr -d " *" | pcregrep -o "(?<=FILE).*?(?=\;)" | tr "," " "`)
    # delete // comment out and spaces
    code=`cat ${cfile} | tr -d " " | grep -v "^\s*//"`
    # delete /**/ comment out
    code=`echo ${code} | sed -e "s/\/\*.*\*\// /"`

    for var in ${vars[@]}; do
      cond="${var}=fopen("
      # nc="if(${var}==NULL)"
      nc1="${var}==NULL"
      # nc2="${var}!=NULL"
      # nc3="(${var}=fopen(.*))==NULL"
      if [[ ${code} == *${cond}* ]]; then
        if [[ ${code} != *${nc1}* ]]; then
        # if ! { [[ ${code} != *${nc1}* ]] || [[ ${code} != *${nc2}* ]]; }; then
          no_null_check+=(`basename ${cfile}`)
          break
        fi
      fi
    done
  done

  if [ ${#no_null_check[@]} -gt 0 ]; then
    print_files no_null_check[@] "Not NULL check"
    return 1
  else
    printf "\n$pad\n"
    printf " $GREEN" "All files check NULL"
    printf "\n$pad\n"
    return 0
  fi
}
```

## シェルスクリプトのテスト

bashアプリケーションをテストできれば実質，C言語のテストもできるんじゃないか？
ということで調べた．

* References
  * [Bashアプリケーションをテストする | POSTD](https://postd.cc/bash%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E3%83%86%E3%82%B9%E3%83%88%E3%81%99%E3%82%8B/)


felicity official
