# Shell script

## string1 in string2 like python

Pythonのように対象の文字列A内にある文字列Bが含まれているかどうかを調べる．

```bash
string='My long string'
if [[ $string == *"My long"* ]]; then
  echo "It's there!"
fi
```

* References
  * [How to check if a string contains a substring in Bash - Stack Overflow](https://stackoverflow.com/questions/229551/how-to-check-if-a-string-contains-a-substring-in-bash)
