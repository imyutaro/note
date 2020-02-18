# How to use Hydra (hyper params config tool)

Example is in [this repo](https://github.com/omry/hydra-article-code). \
Article about Hydra is [this](https://medium.com/pytorch/hydra-a-fresh-look-at-configuration-for-machine-learning-projects-50583186b710).

## Hydra使い方

基本的には[このmediumの記事の日本語訳](https://medium.com/pytorch/hydra-a-fresh-look-at-configuration-for-machine-learning-projects-50583186b710)と[hydraのドキュメント](https://hydra.cc/docs/intro)をもとに書く．

### インストール

```bash
pip install hydra-core
```

でインストール．

### 使い方

この[repo](https://github.com/omry/hydra-article-code)のbasicフォルダ下をもとに説明する．\
configファイルを利用したい関数(例えば`my_app()`)に`@hydra.main(config_path="path/to/config/")`のデコレータをつけて，デコレータの第一引数にconfigファイルのpathを指定する．関数(`my_app()`)の引数に仮引数名を書く(この仮引数名は何でもいい)．\
そうすると辞書型(正確には`omegaconf.dictconfig.DictConfig`クラス)でconfigファイルの内容がその関数(`my_app()`)に渡る．

```yaml
# config.yaml
dataset:
  name: imagenet
  path: /datasets/imagenet
```

```python
# my_app.py
import hydra
from omegaconf import DictConfig


@hydra.main(config_path="config.yaml")
def my_app(cfg: DictConfig) -> None:
    print(cfg.pretty())
    print(cfg["dataset"])


if __name__ == "__main__":
    my_app()
```

`python my_app.py`を実行すると

```console
$ python my_app.py
dataset:
  name: imagenet
  path: /datasets/imagenet

{'name': 'cifar10', 'path': '/datasets/cifar10'}
```

configファイルの内容 ↑ が出力される．\
このconfigファイルの内容をconfigファイルを編集せずにコマンドラインから変更できる．\
例えば，`python my_app.py dataset.path=/datasets/imagenet20k`を実行したら

```text
dataset:
  name: imagenet
  path: /datasets/imagenet20k
```

datasetのpathが変わってるけど，configファイルの中身は変わってない．

実行したら実行したディレクトリ以下に`outputs`ディレクトリができて，`outputs`ディレクトリ以下に実行した日付，時刻でディレクトリができ，実行したときに参照したyamlファイルとかが`.hydra`ディレクトリ以下にある．

```text
.
├── config.yaml
├── my_app.py
└── outputs
    └── 2020-02-16
        └── 19-06-56
            ├── .hydra
            │   ├── config.yaml
            │   ├── hydra.yaml
            │   └── overrides.yaml
            └── my_app.log
```

#### 複数のconfigファイルを利用する，他のyamlファイルの読み込み

同様に[repo](https://github.com/omry/hydra-article-code)のcompositionフォルダ下をもとに説明する．\
複数のデータセットに対してそれぞれの別のconfigファイルを使いたい場合は

```text
.
├── conf
│   ├── config.yaml
│   └── dataset
│       ├── cifar10.yaml
│       └── imagenet.yaml
└── my_app.py
```

のようなディレクトリ構造で`my_app.py`で`/conf/config.yaml`を指定して，`python my_app.py`を実行すると

```text
dataset:
  name: cifar10
  path: /datasets/cifar10
```

が出力される．↑は`config.yaml`で指定したdefaultのconfigファイルである`cifar10.yaml`の内容．

```yaml
# config.yaml
defaults:
  - dataset: cifar10
```

```yaml
# cifar10.yaml
dataset:
  name: cifar10
  path: /datasets/cifar10
```

<!--
TODO: Write how to use multiple yaml config
-->
基本的にはディレクトリごとにグルーピングしていき，読み込んでいく．例えば，

```yaml
```

グルーピングしたくない場合は単純に

```yaml
# config.yaml
defaults:
  - hydra_basic # グルーピングしないで単純に読み込み
dataset:
  name: imagenet
  path: /datasets/imagenet
params:
  k: 32
  n: 40
  m: 100
```

`hydra_basic.yaml`は`config.yaml`と同じディレクトリ内に置いとく．
`hydra_basic.yaml`の中身は以下．

```yaml
# hydra_basic.yaml
hydra:
  run:
    dir:
      ../test/${now:%Y-%m-%d}/${now:%H-%M-%S}
  job_logging:
    formatters:
      simple:
        format: '[%(levelname)s] - %(message)s'
```

- [Non-config group defaults | hydra docs](https://hydra.cc/docs/tutorial/defaults#non-config-group-defaults)

#### 複数の設定を組み合わせて実行

`python my_app.py --multirun dataset=imagenet,cifar10 optimizer=adam,nesterov`
`python my_app.py -m dataset=imagenet,cifar10 optimizer=adam,nesterov`

#### 出力のディレクトリの変更

```yaml
hydra:
  run:
    dir: ../outputs/${hydra.job.name}/${now:%Y-%m-%d_%H-%M-%S}
```

で出力先のディレクトリを変更できる．

- [Customizing working directory pattern | hydra docs](https://hydra.cc/docs/configure_hydra/workdir)

#### Loggingの利用

```python
import hydra
import logging
from omegaconf import DictConfig
tmp_log = logging.getLogger(__name__) # loggingする

@hydra.main(config_path="config.yaml")
def my_app(cfg: DictConfig) -> None:
    print(cfg.pretty())

    # 下に行くほどlog levelが高い
    tmp_log.debug('debugggg')     # DEBUGレベル
    tmp_log.info('I told you so') # INFOレベル
    tmp_log.warning('Watch out!') # WARNINGレベル
    tmp_log.error('errorrrrr')    # ERRORレベル
    tmp_log.critical('crit')      # CRITICALレベル


if __name__ == "__main__":
    my_app()
```

`my_app.log`↓にlogが出力される．

```log
[2020-02-18 18:59:20,123][__main__][INFO] - I told you so
[2020-02-18 18:59:20,123][__main__][WARNING] - Watch out!
[2020-02-18 18:59:20,123][__main__][ERROR] - errorrrrr
[2020-02-18 18:59:20,123][__main__][CRITICAL] - crit
```

loggingのレベルに関しては[pythonのドキュメント](https://docs.python.org/ja/3/howto/logging.html)を参照．
hydraではloggingのレベルはデフォルトでは`INFO`レベル．`hydra.verbose=true`とすると`DEBUG`レベルにできる．
関数単位でレベルを変えたりもできる．詳しくはドキュメント参照．

- [Logging | hydra docs](https://hydra.cc/docs/tutorial/logging)

##### Loggingのカスタマイズ

以下の`hydra/job_logging/formatters/simple/format`で指定できる．

```yaml
# hydra_basic.yaml
hydra:
  run:
    dir:
      ../test/${now:%Y-%m-%d}/${now:%H-%M-%S}
  job_logging:
    formatters:
      simple:
        format: '[%(levelname)s] - %(message)s' # ここで指定できる
```

- [Customizing logging | hydra docs](https://hydra.cc/docs/configure_hydra/logging)

#### Workingディレクトリについて

データの読み込みとかモデルの保存時とか相対パスを普段使ってる場合は注意が必要．
実行ディレクトリ(`__file__`とか`os.getcwd()`とかで取得できるパス)はhydraの出力先のディレクトリに変わってるので，
もとのパスを指定したい場合は`hydra.utils`を使って以下のようにして取得する必要がある．

```python
import os
import hydra
from hydra import utils
@hydra.main()
def my_app(_cfg):
    print("Current working directory  : {}".format(os.getcwd()))
    print("Original working directory : {}".format(utils.get_original_cwd()))
    print("to_absolute_path('foo')    : {}".format(utils.to_absolute_path("foo")))
    print("to_absolute_path('/foo')   : {}".format(utils.to_absolute_path("/foo")))
```

```console
$ python examples/tutorial/8_working_directory/original_cwd.py
Current working directory  : /Users/omry/dev/hydra/outputs/2019-10-23/10-53-03
Original working directory : /Users/omry/dev/hydra
to_absolute_path('foo')    : /Users/omry/dev/hydra/foo
to_absolute_path('/foo')   : /foo
```

- [Output/Working directory | hydra docs](https://hydra.cc/docs/tutorial/working_directory)

#### Optimizerによってconfigを変える

<!--
TODO: Write how to chage config depending on optimizer
-->
