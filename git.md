# Git note

## how to check diff between add and commit

how to check diff after adding before committing \
addした後，commit前に差分を確認する方法

```bash
git diff --cached filename
or
git diff --cached .
```

## how to cancel git add

```bash
git reset HEAD filename
```

## how to git pull force

```bash
git fetch
git reset --hard origin/branch_name

# example
git fetch
git reset --hard origin/master
```

## how to check diff between local and remote

how to check diff before pull. \
After doing a git fetch, do a `git log HEAD..origin/master` to show the log entries between your last common commit and the origin's master branch.
To show the diffs, use either `git log -p HEAD..origin/master` to show each patch,
or `git diff HEAD...origin/master` (three dots not two) to show a single diff.

```bash
git fetch
git diff HEAD remote_name/branch_name # to show diff between local and remote
git log -p HEAD..remote_name/branch_name # to show log
git log HEAD...remote__name/branch_name # to show a single diff

# example
git fetch
git diff HEAD origin/master
```

- [version control - How to preview git-pull without doing fetch? - Stack Overflow](https://stackoverflow.com/questions/180272/how-to-preview-git-pull-without-doing-fetch)

## how to check diff after commit

```bash
git diff HEAD^
```

## how to check diff before pull

```bash
git diff remote_name/branch_name HEAD

# example
git diff origin/master HEAD
```

## how to change commit message after commit

```bash
git commit --amend -m "commit message"

# example
git commit --amend -m "change commit message"
```

- [Gitのコミットメッセージを後から変更する方法をわかりやすく書いてみた | 株式会社グランフェアズ](https://www.granfairs.com/blog/staff/git-commit-fix)

## how to change commit message after push

If you want change last commit message, then you can do

```bash
# Change commit message
git commit --amend
or
git commit --amend -m "new message"

# Push it
git push --force-with-lease <repository> <branch>

# example
git commit --amend -m "change commit message"
git push --force-with-lease origin master
```

If the repository you want change commit message is not used by only you, you have to care about force push. \
See reference Stack Overflow.

- [Changing git commit message after push (given that no one pulled from remote) - Stack Overflow](https://stackoverflow.com/questions/8981194/changing-git-commit-message-after-push-given-that-no-one-pulled-from-remote)

## How to download specific files from GitHub

I don't know if it works because I haven't try it...

```
git archive --remote=ssh://host/pathto/repo.git HEAD README.md
```

Just use curl or wget from Raw code page.

```
curl https://raw.githubusercontent.com/pytorch/examples/master/vae/main.py
```

* Ref
  * [Retrieve a single file from a repository](https://stackoverflow.com/a/18331440)

## Make empty branch

下記のコマンドで履歴のないブランチを作成する。

```
git checkout --orphan empty-branch
```

ファイルを削除して、

```
git rm -rf .
```

コミットして、

```
git commit --allow-empty -m "root commit"
```

push。

```
git push origin empty-branch
```

* Ref
  * [Create empty branch on GitHub](https://stackoverflow.com/a/34100189)

## Make ssh key for github

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

* Ref
  * [新しい SSH キーを生成して ssh-agent に追加する](https://docs.github.com/ja/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

## 複数SSH鍵を利用する

```
git config --local user.name "username"
git config --local user.email "your_email@example.com"
```

* Ref
  * [複数のgitアカウントを使い分ける](https://qiita.com/0084ken/items/f4a8b0fbff135a987fea)
  * [gitconfig の基本を理解する](https://qiita.com/shionit/items/fb4a1a30538f8d335b35)

