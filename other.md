# その他

## sshのデバッグ

`-vT`オプションを使いましょう．

> ssh -T とは？
>
> * sshで接続テストをする際のTオプションの説明
>
> ```text
>  -T      Disable pseudo-tty allocation.
> ```
>
> * Disable pseudo-tty allocation = 疑似端末の割当を無効化する
> * どういうときに使うか？
>   * shell accessが許可されていない端末へsshアクセスする場合
>     * e.g. githubへの共通鍵を登録ができているかのチェック
> * 何が嬉しいのか？
>   * githubはshell accessを許可していないので、PTY(pseudo-tty) allocation requestが通らずconnectionが閉じてしまう
>
> ```bash
> > ssh github-kz-engineer
> PTY allocation request failed on channel 0
> Hi kz-engineer! You've successfully authenticated, but GitHub does not provide shell access.
> Connection to github.com closed.
>
> > ssh -T github-kz-engineer
> Hi kz-engineer! You've successfully authenticated, but GitHub does not provide shell access.
> ```
>
> * -tオプションの方が使われている？
>
> ```text
> -t Force pseudo-tty allocation. This can be used to execute
>    arbitrary screen-based programs on a remote machine, which can
>    be very useful, ...
> ```

ref: [\`pseudo-tty\` とは？ - kz-engineer -SCRAP-](http://kz-engineer-scrap.hatenablog.com/entry/2016/03/03/040625)

-vは

```text
-v      Verbose mode.  Causes ssh to print debugging messages about
        its progress.  This is helpful in debugging connection,
        authentication, and configuration problems.  Multiple -v
        options increase the verbosity.  The maximum is 3.
```

debug情報を表示してくれる．

```bash
ssh -vT hostname
```

## ls, cd command のdirの色を変える

`.zshrc`に

```bash
export LS_COLORS="${LS_COLORS}:di=01;36:ln=00;35:mi=di::ex=01;32"
source $ZSH/oh-my-zsh.sh
```

と追記．~~oh-my-zshを使っている場合(zshを使っている場合?)`source $ZSH/oh-my-zsh.sh`を最後に実行する必要があるので`.zshrc`には最後の行に書く．~~

`LS_COLORS`は`source $ZSH/oh-my-zsh.sh`前に書かなくちゃだめ．PROMPTOの設定は後に書かないとだめっぽい．ここらへんのルールはよくわからないから一回勉強しないとだめ...

それぞれの文字の意味は

```text
キーワード
di : ディレクトリ
ln : シンボリックリンク
mi : シンボリックリンクが指す存在しないファイル
ex : アクセス許可に「x」が設定されている実行可能ファイル

エフェクト
00 : デフォルトカラーを復元
01 : より明るい色

文字色
32 : 緑
35 : 紫
36 : シアン
```

256色拡張の色を使うには

```text
foregroudの場合は di=38;05;色番号
backgroudの場合は di=48;05;色番号
```

でできる．

他にもキーワード，エフェクト，文字色，背景色の指定ができるのでrefを参照．

* References
  * [LS_COLORSで設定するファイルの種類と色コード | Never catch a cold.](https://alliance7.blogspot.com/2017/04/lscolors.html)
  * [LS_COLORS : MinGW,MSYSで、lsのディレクトリ名を見やすくする - 銀の弾丸](https://takamints.hatenablog.jp/entry/2014/11/02/173631)
  * [サーバ環境セットアップ｜lsコマンドで表示される文字カラーを変更](https://www.fulldigit.co.jp/server_env/ls_colors.html)
  * [lsコマンドのカラー出力で256色拡張の色を使用する - 試験運用中なLinux備忘録・旧記事](https://kakurasan.hatenadiary.jp/entry/20080707/p1)
