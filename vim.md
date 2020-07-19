# Vim note

## Homebrewでインストールしたvimをデフォルトにする（Mac）

まず，vimのpathの確認．

```console
$ type -a vim
vim is /usr/bin/vim
```

`brew link vim`でリンクする．

```console
$ brew link vim
Linking /usr/local/Cellar/vim/8.2.0600...
Error: Could not symlink share/man/de/man1/ex.1
/usr/local/share/man/de/man1 is not writable.
```

書き換えができるように権限を変える．

```console
$ sudo chown -R `whoami`:admin /usr/local/share/man/de/man1
```

それでもう一度 `brew link vim`

```console
$ brew link vim
Linking /usr/local/Cellar/vim/8.2.0600... 163 symlinks created
```

```console
$ type -a vim
vim is /usr/local/bin/vim
vim is /usr/local/bin/vim
vim is /usr/bin/vim
```

* Ref
  * [Make Homebrew installed Vim override system installed one
](https://apple.stackexchange.com/questions/362833/make-homebrew-installed-vim-override-system-installed-one)

## Vim for beginner

### Vim Tutorial / Cheatsheet

Vimを使い始めた時に、常時手元に置いていたチートシート。

* [Vim チートシート](https://namaraii.com/files/vim-cheatsheet.pdf)

使い始めた時は、やっぱり矢印を使って移動してしまっていたので
[矢印キーを無効にして](###矢印キーを無効にする)のようにして、矢印キーを無効にしていました。

ちなみに、下記のように`vimtutor`コマンドをターミナルで実行すると、Vimのチュートリアルが始まります。

```console
vimtutor
```

`vimtutor`は基本的なことしか、取り扱ってくれていないので、個人的には下記の動画がおすすめです。

* [Vim Diesel's OFFICIAL Vimtutor Let's Play/Commentary! (1 HOUR+ Special)](https://youtu.be/d8XtNXutVto)

↓ の記事も良さそう。

* [はじめてのVim 〜 Vimはいいぞ！ゴリラと学ぶVim講座](https://knowledge.sakura.ad.jp/21687/)

その他Tips Link

* [vimを使うとき十字キーで移動して、vimmerに殺されるその前に](https://qiita.com/fasahina/items/2767891134028648f288)
### 矢印キーを無効にする

Vimを使い始めた時は、やっぱり矢印キーを使ってしまっていました。
なので、矢印キーを無効にして、強制的に h j k l で移動するように強制してました。

以下をホームディレクトリ直下の`.vimrc`に書けば矢印キーを無効にできます。

```
" disable arrow key
map <Up> <Nop>
map <Down> <Nop>
map <Left> <Nop>
map <Right> <Nop>
inoremap <Up> <Nop>
inoremap <Down> <Nop>
inoremap <Left> <Nop>
inoremap <Right> <Nop>
```

## Escを押したら英数に切り替える

Karabiner-Elementsを使う。

```
brew cask install karabiner-elements
```

1. インストールしたらアプリを開いて
2. システム環境設定 > セキュリティとプライバシー > 入力監視 のところで`karabiner_grabber`と`karabiner_observer`にチェックを入れる \
   (アプリを起動したらチェックを入れろって警告が出てくる)
3. Preferences > Complex Modifications > Add rule > Import more rules from the Internetで
  日本語用のルール`For Japanese （日本語環境向けの設定） (rev 5)`をインポートする
4. インポートしたら`escキーを押したときに、英数キーも送信する（vim用）`をenableにする

下記のようにキーボードを`Apple Internal Keyboard / Trackpad (Apple Inc.)`のものでなくて
`No product name (No manufacturer name)`にしないと正常に動作しなかった。
(Escを押した時に英数キーが同時に押される設定だけ)
(macOS Catalina, version 10.15.4)

![キーボード設定](https://github.com/imyutaro/note/blob/img/vim_karabiner.png, キーボード詳細設定)

* Ref
  * [MacのVimでESCまたは<C-[>押した際に日本語入力をOFFにしたい（JISキーボード編）](https://yukidarake.hateblo.jp/entry/2018/06/17/231345)

## \_(アンダーバー)で止まるようにする

`.vimrc`に`set iskeyword-=_`を書く。

* Ref
  * [How do I change until the next underscore in Vim?](https://superuser.com/a/244070)

## 全角空白を色付けする

`.vimrc`に以下を記述。

```
" ハイライトの色を設定
function! ZenkakuSpace()
  highlight ZenkakuSpace ctermbg=203
endfunction

" ハイライトの設定を全角スペースに適用
if has('syntax')
  augroup ZenkakuSpace
    autocmd!
    " ZenkakuSpaceをカラーファイルで設定するなら次の行は削除
    autocmd ColorScheme       * call ZenkakuSpace()
    " 全角スペースのハイライト指定
    autocmd VimEnter,WinEnter * match ZenkakuSpace /　/
    autocmd VimEnter,WinEnter * match ZenkakuSpace '\%u3000'
  augroup END
  call ZenkakuSpace()
endif
```

* Ref
  * [全角スペースを強調表示する](https://sites.google.com/site/fudist/Home/vim-nihongo-ban/vim-color#color-zenkaku)

## 「」, （）をマッチペアとして設定する

`.vimrc`に以下を記述。

```
set matchpairs+=「:」,（:）
```

* Ref
  * [日本語のカーソル移動 - matchpairs](https://sites.google.com/site/fudist/Home/vim-nihongo-ban/vim-japanese#TOC-matchpairs)

## 良さげなvimrcの例

* https://github.com/flatusv/dotfiles/blob/master/.vimrc



 (shift + o) の遅延をなくす

O(shift + o)を押した時に遅れが起きる。それの対処法。ちなみにOは「上の新しい行にinsert mode」。
`set ttimeoutlen=100`とする。

* Ref
  * [Why does vim delay for a second whenever I use the 'O' command (open a new line above and insert)?](https://superuser.com/questions/161178/why-does-vim-delay-for-a-second-whenever-i-use-the-o-command-open-a-new-line)

## Vim + tmux で ペアプロ

* https://www.hamvocke.com/blog/remote-pair-programming-with-tmux/

## Insert Current date

```
:pu=strftime('%c')
```

or

```
:put =strftime(\"%c\")
:put =strftime('%c')
```

etc...

* Ref
  * [Insert current date or time](https://vim.fandom.com/wiki/Insert_current_date_or_time)

## 検索

| Key | action |
| -- | -- |
| /検索文字列 | 下方向に、検索文字列を検索する。 |
| ?検索文字列 | 上方向に、検索文字列を検索する。 |
| n | /や?で検索を行った後に、順方向に次の検索結果にジャンプする。 |
| N | /や?で検索を行った後に、逆方向に次の検索結果にジャンプする。 |
| * | カーソル位置の文字列を下方向に検索する。 |
| # | カーソル位置の文字列を上方向に検索する。 |

* Ref
  * [vim文字列の検索](http://www.webhtm.net/vim/search.htm)

## Markdownのsyntax highlightを有効にする

```
" Enable markdown highlight
set syntax=markdown
au BufRead,BufNewFile,BufFilePre *.md set filetype=markdown
```

を`~/.vimrc`に書く。

* Ref
  * [Enabling markdown highlighting in Vim - Stack Overflow](https://stackoverflow.com/a/30113820)

