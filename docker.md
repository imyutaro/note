
# Docker

## HostOSの環境変数をdocker-compose.yml, Dockerfileに渡す方法

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
