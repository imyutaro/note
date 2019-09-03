## addしたあとのcommit前に差分を確認する方法
```bash
$ git diff --cached filename
or
$ git dif --cached .
```

## addを取り消す
```bash
$ git reset HEAD filename
```

## how to do git pull force
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
