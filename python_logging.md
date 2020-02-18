# About Python Logging

[Pythonのドキュメント](https://docs.python.org/ja/3/howto/logging.html)をもとに書く．

以下のレベルがあり，

| レベル | いつ使うか |
| --- | --- |
| `DEBUG` | おもに問題を診断するときにのみ関心があるような、詳細な情報。 |
| `INFO` | 想定された通りのことが起こったことの確認。 |
| `WARNING` | 想定外のことが起こった、または問題が近く起こりそうである (例えば、'disk space low') ことの表示。 |
| `ERROR` | より重大な問題により、ソフトウェアがある機能を実行できないこと。 |
| `CRITICAL` | プログラム自体が実行を続けられないことを表す、重大なエラー。 |

以下のプログラムを実行すると

```python
import logging
logging.debug('debugggg')  # will not print anything
logging.info('I told you so')  # will not print anything
logging.warning('Watch out!')  # will print a message to the console
logging.error('errorrrrr')  # will print a message to the console
logging.critical('crit')  # will print a message to the console
```

で`WARNING`以下のレベルのものが表示される．デフォルトでは`WARNING`以上のレベルのメッセージが出力される．

```text
WARNING:root:Watch out!
ERROR:root:errorrrrr
CRITICAL:root:crit
```

`DEBUG`などのレベルのメッセージを表示させるには`basicConfig(level=logging.DEBUG)`で表示レベルを変えることで表示させることができる．

```python
import logging
logging.basicConfig(level=logging.DEBUG) # print all message over debug level
logging.debug('debugggg')
logging.info('I told you so')
logging.warning('Watch out!')
logging.error('errorrrrr')
logging.critical('crit')
```

```text
DEBUG:root:debugggg
INFO:root:I told you so
WARNING:root:Watch out!
ERROR:root:errorrrrr
CRITICAL:root:crit
```

## ファイルへの出力

ファイルへの出力は`basicConfig(filename="example.log")`で`./example.log`にログの出力が表示される．

## 出力フォーマットの変更

`basicConfig(format='%(levelname)s:%(message)s')`で変更できる．
`basicConfig(format='%(message)s')`だとメッセージのみ出力される．

## Loggerに名前をつけた場合

Loggerに名前をつけた場合，`debug`などのメッセージを送るメソッドはつけた名前にチェーンすればよいが`basicConfig`などは`logging`にチェーンする．

```python
import logging

tmp_log = logging.getLogger(__name__)
# basicConfigとlevelはlogging.にする．
logging.basicConfig(format="%(message)s", level=logging.DEBUG, filename="example.log")

tmp_log.debug('debugggg')
tmp_log.info('I told you so')
tmp_log.warning('Watch out!')
tmp_log.error('errorrrrr')
tmp_log.critical('crit')
```

