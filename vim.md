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
