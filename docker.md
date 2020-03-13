
# Docker Note

## 基本的な使い方

### イメージの確認

```console
docker images
```

でそのPC内にあるdockerイメージを確認できる．

### イメージのビルド

`Dockerfile`のあるディレクトリで

```console
docker build
```

を実行すると`Dockerfile`をもとにコンテナをビルドする．
`fオプション`を使うとDockerfileの指定ができる．
`tオプション`を使うとイメージのタグの名前を指定できる．

```console
docker build -f Dockerfile.gpu -t test
```

なんで，上記のコマンドは`Dockerfile.gpu`というDockerfileをもとに`test`というタグを付けてイメージをビルドすると言う意味．

### コンテナを起動してコマンドをコンテナ内で実行

イメージのビルドと同様に`Dockerfile`のあるディレクトリで

```console
docker run bash
```

を実行するとコンテナを起動してコマンド`bash`を起動したコンテナ内で実行できる．

オプション(フラグ？)はたくさんあるが基本的に自分が使うのは以下のコマンド

```text
gpus  : GPUを使うかどうか
name  : $NAME
init  :
p     : $PORT:$PORT
v     : $PWD/:/home/$USER_NAME/$NAME/
tid   : $NAME
```

* [GPUオプションについて](https://docs.docker.com/config/containers/resource_constraints/#gpu)

```console
docker run --gpus all \
           --name $NAME \
           --init \
           -p $PORT:$PORT \
           -v $PWD/:/home/$USER_NAME/$NAME/ \
           -tid $NAME \
           jupyter lab --no-browser --port=$PORT --ip=0.0.0.0
```

### コンテナ内で他のコマンド実行

起動したコンテナ内で他のコマンドを実行したいとき
(例えば，実験のためにGPUなどの環境をコンテナで構築してコンテナ内でファイルを変更しながら実験コードを実行したいとかのとき)
は以下のコマンドを実行することでできる．

```console
docker exec -it test bash
```

### コンテナの確認

```console
docker ps
```

で起動しているdockerコンテナを確認できる．

```console
docker ps -a
```

で停止しているdockerコンテナも確認できる．

### コンテナ停止

```console
docker stop container_name
```

### コンテナ削除

```console
docker rm container_name
```

### イメージ削除

```console
docker rmi iamge_name
```

### Docker Composeを利用する場合

[Docker Compose](https://docs.docker.com/compose/)を利用して一連のビルドなどを簡単に行うことができる．

docker composeは`docker build`, `docker run`時のオプションなどをまとめて`docker-compose.yml`に記述し，そのオプションを付けてイメージのビルド，コンテナの起動ができる．

gpu flagについては現在(2020/03/13)ではdocker-composeでは`/etc/docker/daemon.json`を追加してごちゃごちゃするしか方法がないっぽい

* [Support for NVIDIA GPUs under Docker Compose · Issue #6691 · docker/compose](https://github.com/docker/compose/issues/6691)

上記のissue内の[コメント](https://github.com/docker/compose/issues/6691#issuecomment-571309691)にある以下のPRを利用してもいいかも．マージされてないから公式じゃないけど...

* [Add device requests by Lucidiot · Pull Request #2471 · docker/docker-py](https://github.com/docker/docker-py/pull/2471)
* [WIP: Forward device requests by yoanisgil · Pull Request #7124 · docker/compose](https://github.com/docker/compose/pull/7124)

#### イメージのビルド，コンテナの起動

```console
docker-compose up
```

#### コンテナの停止，コンテナの削除，イメージの削除

```console
docker-compose down --rmi all
```

### HostOSの環境変数をdocker-compose.yml, Dockerfileに渡す方法

なんでかわからないが`docker-compose.yml`内の`user: ****:****`を`user: 1000:1000`以外のUID, GIDにするとdocker内のuserが`I have no name!`になってしまう．
しかし，UID, GIDを1000にしてしまうとHostOSのuserのUID, GIDが1000以外だとマウントしたvolumeにアクセスできなくなる．

`Dockerfile`内での`useradd`の際にHostOS側と同じUIDを指定して`useradd`すれば解決した．

しかし，HostOS側のUIDを`${UID}`とかで指定できないのがなんだかなぁ...

```bash
#!/usr/bin/bash
export UID=$UID
export GID=$GID
sudo -E docker-compose up -d --build
```

ってしないとだめ... \
だからscriptとして上記のコマンドを実行するようにするしか方法がないみたい...

### docker-composeからDockerfileへ環境変数を渡す方法

`docker-compose.yml`には下記のように渡す．

```bash
#!/usr/bin/bash
export $USER_ID=${UID}
export $USER_NAME=docker
sudo -E docker-compose up -d --build
```

`docker-compose.yml`内で`args`として指定して`Dockerfile`に渡す．

```yml
build:
  context: .
  args:
    - USER_ID=${USER_ID}
    - USER_NAME=${USER_NAME}
```

`Dockerfile`内で`ARG`で変数を取得し，`$変数`で利用できる．

```Dockerfile
ARG USER_NAME
ARG USER_ID
RUN useradd -u $USER_ID -m $USER_NAME
```

* References
  * [How to use an environment variable from a docker-compose.yml in a Dockerfile? - Stack Overflow](https://stackoverflow.com/questions/48523346/how-to-use-an-environment-variable-from-a-docker-compose-yml-in-a-dockerfile)
  * [Docker-Compose の変数定義について - SIerだけど技術やりたいブログ](https://www.kimullaa.com/entry/2019/12/01/132115)

## DockerfileのENVとdocker-compose.ymlのenvironment

Dockerfile内のENVの環境変数はcontainer内でも利用できるが，docker-compose.yml内のenvironmentの環境変数は`docker run`にしか利用できない．

* References
  * [Define environment variable in Dockerfile or docker-compose? - Stack Overflow](https://stackoverflow.com/questions/57454581/define-environment-variable-in-dockerfile-or-docker-compose)
  * [Environment variables in Compose | Docker Documentation](https://docs.docker.com/compose/environment-variables/)

## \<none\> imageの一括削除

```bash
sudo docker rmi $(sudo docker images -f dangling=true -q)
```

## Docker内で実行したプロセスがゾンビプロセスになる

Docker container内で実行したプロセスがcontainerをkillしない限りずっとゾンビプロセスとして残ってしまう．
container内でkillできなく，ホストOS側からもkillできない．

### どんなときに起こったか

pythonをconatiner内でnohupをつけないでbackground実行してcontainerから抜けたときpythonのプロセスがゾンビプロセスになってしまった．

### 対処法

`docker-compose.yml`内で`init: true`を記述する．`docker-compose.yml`のformatのversion3.7から利用できるので，`docker-compose.yml`内の`version`の項目は`3.7`以降にする． \
`docker run`の場合は`docker run`する際に`--init`をつける． \
そうするとcontainer内にdocker-initプロセスが実行されていてsignalを適切に処理してくれるのでゾンビプロセスなどをkillしてくれる．

* References
  * [Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/#init)
  * [Docker and the PID 1 zombie reaping problem](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/)
  * [initのお仕事〜tiniのコードを読んでみた - ローファイ日記](https://udzura.hatenablog.jp/entry/2018/04/16/162024)
  * [Dockerでsupervisorを使う時によくハマる点まとめ](https://techracho.bpsinc.jp/morimorihoge/2017_06_05/40936)

## docker-composeを利用してGPUを使う

結論から言うとできない．`docker-compose`が`gpus`オプションをサポートしたらできる．

### 対処法

なので`docker build`+`docker run`でやるしかない．
