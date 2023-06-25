# podmanを試してみる

Docs : https://docs.podman.io/en/latest/

> Most users can simply alias Docker to Podman (alias docker=podman) without any problems.

らしい、docker commandをpodmanに置き換えるだけで、いいらしい。

> Podman is a daemonless, open source, Linux native tool

デーモンレスらしい、dockerはデーモンを走らせておく必要があったはず。（要確認）

Install on Max : https://podman.io/getting-started/installation#macos


- Other seems to be useful docs
  - https://www.slideshare.net/AkihiroSuda/dockerpodman

## docker-composeを使えるようにする

standaloneのdocker composeをインストール or `brew install docker-compose` して、以下のsudoを使わずに以下のコマンドを実行。

standaloneのdocker composeはdocker-composeコマンドになるので、利用する際はdocker composeをdocker-composeに置き換えてコマンドを実行する。

```
$ export DOCKER_HOST="unix:///tmp/podman.sock"

# podmanのVMが開けているportとAPIのURIを確認する。
$ podman system connection ls
# Outpput:
#    Name                         URI                                                                   Identity                      Default
#    podman-machine-default       ssh://core@127.0.0.1:<port番号>/run/user/<XXXX>/podman/podman.sock    <podmanのVMのsshキー>         true
#    podman-machine-default-root  ssh://root@127.0.0.1:<port番号>/run/podman/podman.sock                <podmanのVMのsshキー>         false

# 上記で表示されている<>で囲まれている部分をポート番号とURIに置き換える。
# ssh -fnNT -L/tmp/podman.sock:<上記コマンドで得られたURI> -i ~/.ssh/podman-machine-default ssh://core@localhost:<port番号> -o StreamLocalBindUnlink=yes
$ ssh -fnNT -L/tmp/podman.sock:/run/user/<XXXX>/podman/podman.sock -i ~/.ssh/podman-machine-default ssh://core@localhost:<port番号> -o StreamLocalBindUnlink=yes

# docker-composeが利用できるようになっている
$ cat << EOF > docker-compose.yml
version: "3.9"
services:
  test:
    image: "alpine:3.16"
    tty: true
EOF

# brewでインストールしたdocker-composeはdocke composeコマンドではなくdocker-composeコマンド
$ docker-compose up -d
```

**メモ**

以下のようなエラーが出た場合、URIやポートが適切に設定できていない可能性があるので、間違っていないか確認する。
> error during connect: Get "http://%2Ftmp%2Fpodman.sock ... EOF

**リンク**

- standaloneのdocker compose
  - https://docs.docker.com/compose/install/standalone/
- docker-composeでpodmanを利用する設定
  - Linuxの場合
    - https://github.com/containers/podman/discussions/16338#discussioncomment-3998663
    - https://medium.com/nttlabs/docker-podman-28ced4f7cb90
  - Macの場合
    - https://gist.github.com/kaaquist/dab64aeb52a815b935b11c86202761a3
- その他
   - https://zokibayashi.hatenablog.com/entry/2022/10/02/225120
   - https://rheb.hatenablog.com/entry/podman3-rootless-docker-compose

