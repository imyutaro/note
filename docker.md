# Docker Note

実験用の`Dockerfile`は[自分のレポジトリ](https://github.com/imyutaro/dockerfiles)を参照．

基本的な流れは(脳死手順)

1. イメージをビルドしてコンテナを作成

    ```console
    docker build
    ```

2. コンテナを起動

    ```console
    docker run
    ```

3. コンテナの停止

    ```console
    docker stop
    ```

    * コンテナ内で実行とか

      ```console
      docker exec -it test bash
      ```

4. コンテナの削除

    ```console
    docker stop
    ```

5. イメージの削除

    ```console
    docker rmi image_id
    ```

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

なので，上記のコマンドは`Dockerfile.gpu`というDockerfileをもとに`test`というタグを付けてイメージをビルドすると言う意味．

### コンテナを起動してコマンドをコンテナ内で実行

イメージのビルドと同様に`Dockerfile`のあるディレクトリで

```console
docker run bash
```

を実行するとコンテナを起動してコマンド`bash`を起動したコンテナ内で実行できる．

オプション（フラグ？）はたくさんあるが基本的に自分が使うのは以下のオプション

```text
--gpus    : GPUを使うかどうか
--name    : コンテナに名前をつける
--init    : シグナルを送ったり，プロセスを管理するinitプロセスをコンテナ内で起動する
-p       : コンテナ内のポートとホストのポートをマップする
-v       : コンテナ内のディレクトリとホストのディレクトリをマップする
-d       : コンテナ内に入らない(コンテナからデタッチする)．
-t $NAME : 疑似ttyを$NAMEコンテナに割り当てる．つまり$NAMEという名前のコンテナに現在実行しているターミナルの標準入出力をつなげる(マップする)ということ．
-i       : アタッチしていなくてもSTDINを開いたままにする．
```

* [docker run description](https://docs.docker.com/engine/reference/commandline/run/)
* ttyについて
  * [ttyとかptsとかについて確認してみる](https://qiita.com/toshihirock/items/22de12f99b5c40365369)
  * 一言でいうと，terminalとかのコマンドライン(標準出力)
* [GPUオプションについて](https://docs.docker.com/config/containers/resource_constraints/#gpu)

```console
#!/usr/bin/zsh -e

Dockerfile=${1:-Dockerfile-pytorch}
NAME=${NAME:-tmptmp}
USER_NAME=docker
PORT1=9876
PORT2=8765
PATH1=$PWD/
PATH2=/home/$USER_NAME/$NAME/

sudo docker build --build-arg USER_ID=${UID} \
                  --build-arg USER_NAME=$USER_NAME \
                  --build-arg NAME=$NAME \
                  -f $Dockerfile \
                  -t $NAME .
docker run --gpus all \
           --name $NAME \
           --init \
           -p $PORT1:$PORT2 \
           -v $PATH1:$PATH2 \
           -d \
           -i \
           -t $NAME \
           jupyter lab --no-browser --port=$PORT2 --ip=0.0.0.0
```

上記のコマンドでは

* ホストのポート`$PORT1`とコンテナ内のポート`$PORT2`をマップする
* ホスト側のディレクトリ`$PATH1`からコンテナ内のディレクトリ`$PATH2`へマウントする

### コンテナ内で他のコマンド実行

起動したコンテナ内で他のコマンドを実行したいとき
(例えば，実験のためにGPUなどの環境をコンテナで構築してコンテナ内でファイルを変更しながら実験コードを実行したいとかのとき)
は以下のコマンドを実行することでコンテナ内に入れる．

```console
docker exec -it test bash
```

なのでコマンドは「`test`という名前のコンテナ内で`bash`というコマンドを実行する」
という意味．`-it`オプションをつけることでコンテナ内のttyと接続できる．

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

で起動しているコンテナを停止できる．

### コンテナ削除

```console
docker rm container_name
```

で停止しているコンテナを削除できる．

### イメージ削除

```console
docker rmi iamge_name
```

でイメージを削除できる．削除できるイメージは`docker ps -a`で表示されていないコンテナのイメージのみ．

### Docker Composeを利用する場合

[Docker Compose](https://docs.docker.com/compose/)を利用して一連のビルドなどを簡単に行うことができる．

docker composeは`docker build`, `docker run`時のオプションなどをまとめて`docker-compose.yml`に記述し，そのオプションを付けてイメージのビルド，コンテナの起動ができる．

gpu flagについては現在(2020/03/13)ではdocker-composeでは`/etc/docker/daemon.json`を追加してごちゃごちゃするしか方法がないっぽい

* [Support for NVIDIA GPUs under Docker Compose · Issue #6691 · docker/compose](https://github.com/docker/compose/issues/6691)

上記のissue内の[コメント](https://github.com/docker/compose/issues/6691#issuecomment-571309691)にある以下のPRを利用してもいいかも．マージされてないから公式じゃないけど...

* [Add device requests by Lucidiot · Pull Request #2471 · docker/docker-py](https://github.com/docker/docker-py/pull/2471)
* [WIP: Forward device requests by yoanisgil · Pull Request #7124 · docker/compose](https://github.com/docker/compose/pull/7124)

#### イメージのビルド，コンテナの起動

`docker-compose.yml`のあるディレクトリで下記のコマンドを実行することでイメージのビルド，コンテナの起動ができる．

```console
docker-compose up
```

#### コンテナの停止，コンテナの削除，イメージの削除

`docker-compose.yml`のあるディレクトリで下記のコマンドを実行することで起動しているコンテナの停止，削除，イメージの削除ができる．

```console
docker-compose down --rmi all
```

### HostOSの環境変数をdocker-compose.yml, Dockerfileに渡す方法

~~なんでかわからないが`docker-compose.yml`内の`user: ****:****`を`user: 1000:1000`以外のUID, GIDにするとdocker内のuserが`I have no name!`になってしまう．
しかし，UID, GIDを1000にしてしまうとHostOSのuserのUID, GIDが1000以外だとマウントしたvolumeにアクセスできなくなる．~~

`Dockerfile`内での`useradd`の際にHostOS側と同じUIDを指定して`useradd`すれば解決した．

しかし，HostOS側のUIDを`${UID}`とかで指定できないのがなんだかなぁ...

```bash
#!/usr/bin/bash
export UID=${UID}
export GID=${GID}
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

理由は

* [Dockerの--initフラグについて - Carpe Diem](https://christina04.hatenablog.com/entry/docker-init)

の記事中の

> Dockerケースを要約すると、親、子、孫の3プロセスが起動している状態で
> | ケース | 何が起きる | 問題点 |
> | --- | --- | --- |
> | 親が死ぬ | 子も孫も強制的に死ぬ | 処理中リクエストをハンドリングできない |
> | 子が死ぬ | 孫は親に紐付けられるが、親はそれを知らないので孫はゾンビになる | リソースリーク |
> という問題が起きます。
> 親、子だけのケースであれば前者が起きます。

の子が死んだケース．

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

なので`docker build` + `docker run`でやるしかない．

## init processについて

initプロセスを実行しないとdocker内で実行したプロセスが適切に処理できない．

* [Dockerの--initフラグについて - Carpe Diem](https://christina04.hatenablog.com/entry/docker-init)

### man docker run

`man docker run`すると...

```text
-d, --detach=true|false
        Detached mode: run the container in the background and print the new container ID. The default is false.
-i, --interactive=true|false
        Keep STDIN open even if not attached. The default is false.
-t, --tty=true|false
        Allocate a pseudo-TTY. The default is false.

-v|--volume[=[[HOST-DIR:]CONTAINER-DIR[:OPTIONS]]]
        Create a bind mount. If you specify, -v /HOST-DIR:/CONTAINER-DIR, Docker
        bind mounts /HOST-DIR in the host to /CONTAINER-DIR in the Docker
        container. If 'HOST-DIR' is omitted,  Docker automatically creates the new
        volume on the host.  The OPTIONS are a comma delimited list and can be:

            · [rw|ro]

            · [z|Z]

            · [[r]shared|[r]slave|[r]private]

            · [delegated|cached|consistent]

            · [nocopy]
    ...

Mapping Ports for External Usage
      The exposed port of an application can be mapped to a host port using the -p flag. For example, an httpd port
       80 can be mapped to the host port 8080 using the following:

            # docker run -p 8080:80 -d -i -t fedora/httpd

--name=""
          Assign a name to the container

      The operator can identify a container in three ways:

      ┌──────────────────────┬────────────────────────────────────────────────────────────────────┐
      │Identifier type       │ Example value                                                      │
      ├──────────────────────┼────────────────────────────────────────────────────────────────────┤
      │UUID long identifier  │ "f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778" │
      ├──────────────────────┼────────────────────────────────────────────────────────────────────┤
      │UUID short identifier │ "f78375b1c487"                                                     │
      ├──────────────────────┼────────────────────────────────────────────────────────────────────┤
      │Name                  │ "evil_ptolemy"                                                     │
      └──────────────────────┴────────────────────────────────────────────────────────────────────┘

      The UUID identifiers come from the Docker daemon, and if a name is not assigned to the container with --name
      then the daemon will also generate a random string name. The name is useful when defining links (see --link)
      (or any other place you need to identify a container). This works for both background and foreground Docker
      containers.

--init
      Run an init inside the container that forwards signals and reaps processes

--gpus
      GPU devices to add to the container (‘all’ to pass all GPUs)

```

* Reference
  * [docker run description](https://docs.docker.com/engine/reference/commandline/run/)

### docker-composeについて

#### build オプション

`build`オプションはビルド時に適用するオプションを指定できる．

* Reference
  * [build - Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/#build)
  * [build - Compose ファイル・リファレンス | Docker-docs-ja 17.06.Beta ドキュメント](http://docs.docker.jp/compose/compose-file.html#build)

#### context オプション

* Reference
  * [context - Compose file version 3 reference | Docker Documentation](https://docs.docker.com/compose/compose-file/#context)
  * [context - Compose ファイル・リファレンス | Docker-docs-ja 17.06.Beta ドキュメント](http://docs.docker.jp/compose/compose-file.html#context)

## コンテナのメモリ使用量とかCPU使用量とか見たい

```console
docker stats
```

* Ref
  * [docker stats | Docker Documentation](https://docs.docker.com/engine/reference/commandline/stats/)

## 特定のディレクトリをマウントしない

やり方は`volumesに定義したkey`を使って、サービス内にマウントしない みたいにする。

* Ref
  * [Dockerで特定のサブディレクトリだけホストと共有しない](http://www.denzow.me/entry/2018/03/18/104708)
  * [docker-composeを爆速にする](https://qiita.com/shotat/items/57d049793605ffc20135)
  * [3分くらいで分かるdockerのdata volume](https://qiita.com/damele0n/items/ac20e93749eb060fddfa)
    * `driver: local`について書いてある


