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
```

## how to check diff between local and remote
```bash
$ git diff HEAD remote_name/branch_name

# example
$ git diff HEAD origin/master
```

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

## how to git pull force
```bash
$ git fetch
$ git reset --hard origin/branch_name

# example
$ git fetch
$ git reset --hard origin/master
```
