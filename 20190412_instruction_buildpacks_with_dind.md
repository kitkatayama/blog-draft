# Docker in Docker で手軽にbuildpacksを試す

## tl;dr

* https://gist.github.com/kitkatayama/258f8eb74bf7b090667cd87fbca3e75b でとりあえず動く
* buildpacksすごい

## この記事のゴール

* dockerのある環境でbuildpacksが実行できるようになる
* buildpacksをほんのり理解している

## 下準備

dockerの動くホストを用意すること。この記事に記載する内容は、以下の環境で実行を確認している。

* OS: RancherOS v1.5.1
* Docker: version 18.06.1-ce, build e68fc7a

## 概要

大きく分けて、以下の内容を実行する。

1. Docker公式のDocker Hubより、Docker in Docker用(以下、dind)のコンテナを実行
1. packをインストールし、buildpacks公式のサンプルJavaプログラムをクローン
1. buildpacksでビルド、動作確認

```
docker run --privileged --rm --name instruction_buildpacks -p 3000:8080 -d docker:dind
docker exec -it `docker ps -f name=instruction_buildpacks -q` /bin/sh
cd tmp
wget https://github.com/buildpack/pack/releases/download/v0.1.0/pack-v0.1.0-linux.tgz
tar xvf pack-v0.1.0-linux.tgz && rm pack-v0.1.0-linux.tgz && mv ./pack /usr/local/bin/
pack --help
apk add git
git clone https://github.com/buildpack/sample-java-app.git
cd sample-java-app/
pack set-default-builder cloudfoundry/cnb:bionic
docker images
pack build myapp
docker images
docker run -d --rm -p 8080:8080 myapp

## ブラウザで以下のURLにアクセス
http://<実行したホストのIP:3000/

## 後処理
exit
docker stop instruction_buildpacks
```

## 手順

### 1 Docker公式のDocker Hubより、Docker in Docker用(以下、dind)のコンテナを実行

```
docker run --privileged --rm --name instruction_buildpacks -p 3000:8080 -d docker:dind
docker exec -it `docker ps -f name=instruction_buildpacks -q` /bin/sh
```

Docker公式( https://hub.docker.com/_/docker )でdocker:dindを用意されているため、これを使用する。dindにする意味は、buildpacksを試すにあたって現在の環境を汚さないためなので、「試すためVMでUbuntuを立てるよ」といった人はこの手順は不要。

#### Tips: 何故 `docker run ... /bin/sh` ではいけないのか

尚、上記のように一旦コンテナを作成後にexecでコンテナ内シェルに入らないと、下記のようにDockerデーモンに接続できない旨のエラーが出力される。

```
[rancher@rancher ~]$ docker run --privileged --rm --name instruction_buildpacks -p 3000:8080 -it docker:dind /bin/sh
/ # docker --version
Docker version 18.09.5, build e8ff056dbc
/ # docker run -it alpine:latest /bin/sh
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.
```

なぜこうなるかというと、下記のように、PID 1を `docker run ... -it /bin/sh` で実行したプロセスが使用し、dockerdプロセスがコンテナ上で起動されていないためである。

```
[rancher@rancher ~]$ docker run --privileged --rm --name instruction_buildpacks -p 3000:8080 -it docker:dind /bin/sh
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    6 root      0:00 ps aux
```

一方、`docker run ... -d`を行った場合は、以下のようになる。

```
[rancher@rancher ~]$ docker run --privileged --rm --name instruction_buildpacks -p 3000:8080 -d docker:dind
3e3901f0c479956514085f977b1e756eaa49a60e917ca28af996c4e9a48a87c5
[rancher@rancher ~]$ docker exec `docker ps -f name=instruction_buildpacks -q` ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 dockerd --host=unix:///var/run/docker.sock --host=tcp://0.0.0.0:2375
   21 root      0:00 containerd --config /var/run/docker/containerd/containerd.toml --log-level info
  139 root      0:00 ps aux
```

そのため、コンテナ内でdokcerdを起動すると、同じような動作は得られる。この時、`--hostオプション`はデフォルトを使用されることに留意する。(参照: http://docs.docker.jp/engine/reference/commandline/dockerd.html )

```
[rancher@rancher ~]$ docker run --privileged --rm --name instruction_buildpacks -p 3000:8080 -it docker:dind /bin/sh
/ # dockerd &
<省略>
/ # docker run -it alpine:latest /bin/sh
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
bdf0201b3a05: Pull complete
Digest: sha256:28ef97b8686a0b5399129e9b763d5b7e5ff03576aa5580d6f4182a49c5fe1913
/ # exit
<省略>
/ # ps aux
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    7 root      0:00 dockerd
   14 root      0:00 containerd --config /var/run/docker/containerd/containerd.toml --log-level info
  239 root      0:00 ps aux
```

### 2 packをインストールし、buildpacks公式のサンプルJavaプログラムをクローン

https://buildpacks.io/docs/install-pack/ に記載の手順通り、Linux向けの`pack`を`/usr/local/bin`にインストールする。`pack --help`が実行できれば問題ない。

ところで、 https://buildpacks.io/docs/app-journey/ に記載のサンプルプログラムをcloneするにはgitが必要である。docker:dindはAlpine Linuxを採用しており、`git`コマンドが存在しないため、`git`コマンドを`apk add`にてインストールを行う。

`git`のインストール後、サンプルを準備できれば完了である。

```
cd tmp
wget https://github.com/buildpack/pack/releases/download/v0.1.0/pack-v0.1.0-linux.tgz
tar xvf pack-v0.1.0-linux.tgz && rm pack-v0.1.0-linux.tgz && mv ./pack /usr/local/bin/
pack --help
apk add git
git clone https://github.com/buildpack/sample-java-app.git
cd sample-java-app/
```

#### メモ: サンプルプログラムの内容について

https://github.com/buildpack/sample-java-app を見るに、このプロジェクトはApache Maven ( https://maven.apache.org/ )というものを使用しているらしい。Javaに詳しくないのでよくわからないが、どうやらJavaのビルド管理をしてくれるツールらしい。

* https://www.slideshare.net/taktos/maven-28516988

buildpacksは、このリポジトリの内容を理解し、実行可能なコンテナに落とし込んでくれるということになる。

### 3 buildpacksでビルド、動作確認

`pack`及びサンプルプログラムが準備できたところで、実際に動かしてみる。Hello, Buildpacker!という画面が出てきたら完了である。

```
pack set-default-builder cloudfoundry/cnb:bionic
docker images
pack build myapp
docker images
docker run -d --rm -p 8080:8080 myapp

## ブラウザで以下のURLにアクセス
http://<実行したホストのIP:3000/

## 後処理
exit
docker stop instruction_buildpacks
```

#### メモ: builderについて

`pack set-default-builder cloudfoundry/cnb:bionic`の前に`pack build myapp`を実行すると、以下のようにbuilderの設定に関して警告が表示される。

```
/tmp/sample-java-app # pack build myapp
Please select a default builder with:

        pack set-default-builder <builder image>

Suggested builders:

        Cloud Foundry:     cloudfoundry/cnb:bionic         small base image with Java & Node.js
        Cloud Foundry:     cloudfoundry/cnb:cflinuxfs3     larger base image with Java, Node.js & Python
        Heroku:            heroku/buildpacks               heroku-18 base image with official Heroku buildpacks


Tip: Learn more about a specific builder with:

        pack inspect-builder [builder image]
```

builderとアプリケーションの関係は https://buildpacks.io/docs/using-pack/building-app/ に記載されている。が、実際のところ今イチよくわかっていない。説明文にある通り、以下のようにinspectすることで何をサポートしているかおぼろげにわかるので、興味ある人は実行してほしい。

```
# pack inspect-builder cloudfoundry/cnb:bionic
# pack inspect-builder cloudfoundry/cnb:cflinuxfs3
# pack inspect-builder heroku/buildpacks
```

### おわりに

以上より、割と手軽に、Dockerのある環境で任意のgitリポジトリをbuildpackでビルドできることが理解されたら幸いである。

また、buildpacksの仕組みについても、参照URLを辿ってもらったり、buildを実行した際のログや、docker imagesで表示されるdockerコンテナの内容によって、buildpacksがソースコードを理解している、builderとappでレイヤが分かれているといった仕組みも何となく理解いただけたらと思う。

実際のところ、私自身はまだまだbuildpacksをほとんど理解できていない上、buildpacks自体が比較的新しいプロジェクトなので、ドキュメントもまだ未整備の状態であるため、学習も難しい。

ただ、「リポジトリのソースコードを理解して、実行可能なコンテナを生成してくれる」という仕組みは非常に素晴らしいことだというのは直感的に理解できたので、今後の動向を楽しみにしている。