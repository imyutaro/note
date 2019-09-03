##### 実行環境
```
macOS High Sierra
$ sw_vers
ProductName:    Mac OS X
ProductVersion: 10.13.5
BuildVersion:   17F77
```

---

### C++導入ノート
クラスタリングの授業の際にC++とgnuplotの環境をインストールしたのでメモ．

#### C++の環境構築
- reference
  - [Mac OS X 環境で C 言語を書くために](http://ecei-tohoku.github.io/ppa/docs/gcc_for_osx.html)

コンパイルできるようにするためにgccをインストール．
すでにインストールされているかを確認．
```
$ which g++
/usr/bin/g++
```
上記のように出てきたらすでにインストールされている．
出てこなかった場合，App StoreでXcodeをインストールし，command line toolsをインストールする．
Terminalで
```
$ xcode-select --install
```
を入力してインストール．
インストールされているかを
```
$ which g++
/usr/bin/g++
```
で確認．

#### C++のライブラリ，Boostをインストール
- reference
  - [【C++】Mac に Boost を インストールした](https://chomado.com/programming/c-plus-plus/cpp-boost-install-on-mac/)
  - [Installing boost and boost-python on OSX with Homebrew](https://www.pyimagesearch.com/2015/04/27/installing-boost-and-boost-python-on-osx-with-homebrew/)
  - [Boostのバージョンの確認法](http://e-kwsm.hatenablog.com/entry/2015/01/20/155650)


boostについてはreferenceまたはboostのページを参照．インストールは以下のコマンドをTerminalで実行．
```
$ brew install boost
```
boostのversionの確認は以下のコードを実行するとわかる．
```c++
#include <iostream>
#include <boost/version.hpp>

int main()
{
    int major = BOOST_VERSION / 100000;
    int minor = BOOST_VERSION / 100 % 1000;
    int patch = BOOST_VERSION % 100;
    std::cout << "boost version " << major << "." << minor << "." << patch
        << " or " << BOOST_LIB_VERSION << std::endl;
    return 0;
}
```

---

### gnuplotのインストール
- reference
  - [gnuplotをHomebrewからインストールするときの手順
](https://qiita.com/noanoa07/items/a20dccff0902947d3e0c)
  - [Gnuplot をインストールする( Mac ElCapitan/Sierra)](http://mashiroyuya.hatenablog.com/entry/installgnuplot)

普通に`$ brew install gnuplot`をTerminalで実行するだけでもインストールできるがgnuplotを起動して`gnuplot> plot sin(x)`としても表示されない．これは出力先(Terminal type)が指定されていないから起こっている．以下のように`Terminal type set to 'unknown'`となっている．
```
Terminal type set to 'unknown'</boost/version.hpp>
gnuplot> plot sin(x)
```

#### AquaTerm, X11のインストール(どちらか一つで良い)

##### AquaTermのインストール
[AquaTermのダウンロードページ](https://sourceforge.net/projects/aquaterm/)からAquaTerm-1.1.1.dmgをダウンロードしてインストール．Macintosh HD>アプリケーション フォルダに AquaTerm.appが出来ていればOK．

##### X11のインストール
[Mac用の X11ライブラリ XQuartz のダウンロードページ](https://www.xquartz.org/)からXQuartz-2.7.11.dmgをダウンロードしてインストール．Macintosh HD>アプリケーション＞ユーティリティ フォルダに X11.appが出来ていればOK．インストールする際にセキュリティの設定によってはインストールできないと出るのでその場合は，`システム環境設定>セキュリティとプライバシー>一般`のところの`ダウンロードしたアプリケーションの実行許可`のところから実行の許可をしてインストール．

#### gnuplotインストール
インストールオプションを確認
```
$ brew info gnuplot
==> Options
--with-aquaterm
    Build with AquaTerm support
--with-cairo
    Build the Cairo based terminals
--with-qt
    Build with qt support
--with-wxmac
    Build wxmac support. Need with-cairo to build wxt terminal
--with-x11
    Build with x11 support
--without-lua
    Build without the lua/TikZ terminal
--HEAD
    Install HEAD version
```
出力先をインストールの際に指定しないといけないのでオプションを付けてインストール．
```
$ brew uninstall gnuplot  #一旦 gnuplotをアンインストールする
$ brew install gnuplot --with-aquaterm --with-x11
```
このオプションではaquaterm，x11両方を指定している．

#### gnuplotを起動して確認
`$ gnuplot`で起動．
Terminal type set to 'aqua' とメッセージが表示されていれば，AquaTermが出力先に設定されている．

`gnuplot> plot sin(x)`でAquaTermのウィンドウが開いて，サインカーブのグラフが表示されるはず．

#### 出力先の変更

指定可能な出力先のリストを表示させる．
```
Available terminal types:
             aqua  Interface to graphics terminal server for Mac OS X
           canvas  HTML Canvas object
              cgm  Computer Graphics Metafile
              ・
              ・
              ・
        vttek  VT-like tek40xx terminal emulator
          x11  X11 Window System interactive terminal
         xlib  X11 Window System (dump of gnuplot_x11 command stream)
        xterm  Xterm Tektronix 4014 Mode
```
x11に変更するには以下のコマンドをgnuplotで実行する．
```
gnuplot> set terminal x11
Terminal type set to 'x11'
Options are ' nopersist'
```
`gnuplot> plot sin(x)`とかしてX11ウィンドウが開いて，グラフが表示されればOK．
AquaTermへの変更は同様に
```
gnuplot> set terminal aqua
Terminal type set to 'aqua'
Options are '0 title "Figure 0" size 846,594 font "Times-Roman,14" noenhanced solid'
```
で`gnuplot> plot sin(x)`とかしてAquaTermのウィンドウが開いて，グラフが表示されればOK．
