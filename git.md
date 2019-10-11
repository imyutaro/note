## how to check diff between add and commit
how to check diff after adding before committing 
addしたあとのcommit前に差分を確認する方法
```bash
$ git diff --cached filename
or
$ git dif --cached .
```

## hoe to cancel git add
```bash
$ git reset HEAD filename
```

## how to git pull force
```bash
$ git fetch
$ git reset --hard origin/branch_name

# example
$ git fetch
$ git reset --hard origin/master
```

## how to check diff between local and remote
how to check diff before pull. \
After doing a git fetch, do a `git log HEAD..origin/master` to show the log entries between your last common commit and the origin's master branch.
To show the diffs, use either `git log -p HEAD..origin/master` to show each patch,
or `git diff HEAD...origin/master` (three dots not two) to show a single diff.
```bash
$ git fetch
$ git diff HEAD remote_name/branch_name # to show diff between local and remote
$ git log -p HEAD..remote_name/branch_name # to show log
$ git log HEAD...remote__name/branch_name # to show a single diff

# example
$ git fetch
$ git diff HEAD origin/master
```
ref:
- [version control - How to preview git-pull without doing fetch? - Stack Overflow](https://stackoverflow.com/questions/180272/how-to-preview-git-pull-without-doing-fetch)

## how to check diff after commit
```bash
$ git diff HEAD^
```

## how to check diff before pull
```bash
$ git diff remote_name/branch_name HEAD 

# example
$ git diff origin/master HEAD 
```

## how to change commit message after commit
```bash
$ git commit --amend -m "commit message"

# example
$ git commit --amend -m "change commit message"
```
ref: [Gitのコミットメッセージを後から変更する方法をわかりやすく書いてみた | 株式会社グランフェアズ](https://www.granfairs.com/blog/staff/git-commit-fix)
