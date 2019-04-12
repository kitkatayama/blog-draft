# #ふくばねてす node-1 にお邪魔してきました。

## tl;dr

* 
* 
* 

-----

## まとめ

Toggeter: https://togetter.com/li/1336069  
loftkun: https://github.com/loftkun/conference/blob/master/2019/0408-fukubernetes-01/README.md  
kitkatayama: TBO

## 開場～開催前

## セッション

### 本当の"""負荷検知"""をお見せしますよ - cgroup v2 feat. PSI by Uchio Kondoさん (@udzura)

* 資料: https://speakerdeck.com/udzura/cgroup-v2-and-psi
* 参照:
  * https://www.linuxplumbersconf.org/event/2/contributions/204/attachments/143/378/LPC2018-cgroup-v2.pdf
  * うづらさんのブログ: https://udzura.hatenablog.jp/
    * https://udzura.hatenablog.jp/entry/2019/02/12/191458
    * https://udzura.hatenablog.jp/entry/2019/02/14/194244
    * https://udzura.hatenablog.jp/entry/2019/02/15/170221

cgroup v2とPSIがサポートされ、今までホストごとだったパフォーマンス情報取得がコンテナ単位になり、今後のCloud Nativeで利用されるかもしれないという話。

### 何と読む？謎のk3s by Shindo Yosukeさん (@sindoy)

* 資料: TBO
* 参照:
  * https://qiita.com/cyberblack28/items/7ae275f77183024accc1
  * https://qiita.com/ishida330/items/dfff18362ea16aa92f88
  * https://qiita.com/inductor/items/1463cefb72296369b49f 

Rancherの中の人。とにかく話が面白かった。

### kokotapでPodのパケットキャプチャ by Takayuki Konishiさん (@leather_sole)

* 資料: https://www.slideshare.net/leathersole/kokotappod

Kubernetesのpodにパケットキャプチャ用のmirrorデバイスが作成できるという話。Nodeから出る前の生のパケットを拾えるので、これは楽しそう。ネットワーク系のセッションにも使えそうで後ほど試したい。

### telepresenceにキャッチアップ - 後藤さんを倒すには - （仮題） by kunitさん (@kunit)

* 資料: TBO
* 参照: https://speakerdeck.com/shiro16/telepresence-deshi-meru-k8s-shi-dai-falserokarukai-fa

telepresenceはKubernetesの一部のpodをローカルのサーバやDockerに置き換えることができる。そのため、ビルド毎のpod更新の手間なんかを省くことができる。

### いっぱい kustomize する by iwaさん (@mananyuki)

* 資料: TBO

kubectlにapplyするYAMLをレイヤー管理できるようにするツール。YAML同士でinclude、overrideできるようになるイメージ。

似たようなツールとしてHelmがあるが、kustomizeでは生のYAMLを編集し、レイヤーを重ねるイメージなのに対し、Helmの場合はカタログが用意されるが、`$変数`のようなテンプレートを提供するため、テンプレートを元新しいYAMLを生成するイメージ。

### Secure your K8s cluster from multi-layers by Transnanoさん (@transnano)

* 参照: https://www.slideshare.net/JiantangHao1/secure-your-k8s-cluster-from-multilayers

YahooでK8sを実運用した際のノウハウ。元のスライドは30分だったが5分でザッと読むLT。その場でざっくり読んで何となくそうだろうなーぐらいしかわからなったので、後ほど読んだ試訳を置いておきます。

#### Secure your K8s cluster from multi-layers 試訳

-----

* 何故Kubernetesはセキュリティを困難にするのか
  * どこにでもあるトラフィック。コンテナは、ホスト間、もしくはクラウド間に動的に配置できます。
  * 攻撃面の増加。各コンテナには、それぞれ悪用可能な異なる攻撃面や脆弱性が存在します。
  * 古いセキュリティツール。セキュリティのための古いモデルやツールは変わりづつけるコンテナ環境に対応できません。
* Infrastructureレイヤー
  * 監査ログ(audit log)の有効化
  * 不要なポートをEXPOSEしない
  * 可能であればクラスタをプライベートサブネットかVPC内でホストする
  * KubernetesノードへのSSHを制限し、”kubectl”を使用する
  * kube-apiへのアクセスを制限する
* K8s Control Planeレイヤー
  * RBACを有効に、最低限--anonymous-auth falseに。
  * コンポーネント間TLS通信を有効に
  * 保管しているデータの暗号化
  * K8s監査ログの有効化
  * システムデーモン用コンピューターリソースの予約
  * Admission Controllers
* Admission Controllers
  * Kubernetes API サーバ内のフラグ設定によって有効に
  * Admissino Controllersを “validating”,”mutating”,もしくは両方に
* K8s Workloadレイヤー
  * ルートユーザー以外でのコンテナ実行
  * クラスター単位でのポッドセキュリティポリシーの実行
  * クラスタネットワークポリシーの作成と定義
  * 名前空間による孤立化
  * ポッドがアクセスできるノードの制御
  * リソース割り当て設定による機能制限
  * セキュリティコンテキスト
* K8s Container Runtimeレイヤー
  * Kata ContainerはLightweight VMでKernelが独立している
  * その分パフォーマンスは低い
* User Misconfigurationレイヤー
  * 最近の調査で70-75%の記号は最低でも1つの深刻なセキュリティ設定ミスを行っている
  * 不要なデフォルト設定を指定しない
  * シンプル、最小限の設定がエラーを少なくする
  * よりよい改善のため、Annotatinoにオブジェクトの説明を付け加える
  * 最新のAPIバージョンを指定する
  * CI/CDパイプライン設定の確認
  * https://kubesec.io/

-----

### Enjoying k8s clusuer with Minikube and Helm by loftkunさん (@loftkun)

* 資料: https://speakerdeck.com/loftkun/enjoying-k8s-cluster-with-minikube-and-helm

minikubeを使った開発環境の紹介とHelm chartに自作アプリケーションを載せた話。後述するが、RancherでHelmについては何となくわかった。

### Falcoを触ってみた by toenobuさん (@toenobu)

* 資料: TBO

https://falco.org/ CNCF謹製のセキュリティツール。K8sの侵入・異常検知のためのOSS PJ(認識間違ってたらごめんなさい）。

今回はデモとして、`1. docker exec -itした際の標準出力`と`2. delete podした際の障害通知`をメッセージアプリにjsonで通知するというもの。死活監視だけでも使えるので、まずはそこから導入するのもあり。

### buildpack、そう、そういうことなんだ俺たちのこれからは by P山さん (@pyama86)

* 資料: https://speakerdeck.com/pyama86/hukuhanetesu-1-cnbuildpack

P山さんのいつも通りのタイトル。Cloud Nativeはモテる、婚活であるとの持論を展開しつつ、本編のBuildpacksへ。

Herokuで採用されている「プログラムのコードを解析して、必要なプラットフォームを用意し、実行可能なコンテナを作成してくれるツール」をCNCF下でOSS化したもの。

例えばphpの場合。ベースを`ubuntu:latest`として、nginxとphp、php-fpm、mysqlをインストールし、`/var/www/`等にプログラムを配置すれば実行ができると予想されるわけだ。

それらの実行環境をDockerで実現しようとする場合、それらのコンテナの配置、関係をDockerfileやdocker-compose.ymlに記述し、CI/CDパイプラインに載せてビルドするわけだが、Buildpacksは、リポジトリ直下で`pack build myapp`とするだけで`myapp`というコンテナを用意してくれる。

この思想に従えば、アプリケーションコードを書く以外は全てをBuildpacksが賄ってくれるわけだ。これは、インフラ整備の大幅なコストを解決する。依存関係を整理されたアプリケーション実行基盤はビルド毎に最新に置き換えられるわけで、脆弱性対応なども意識しなくてよくなり、スケールメリットも大きい。つまり、インフラエンジニアが設計する大部分を自動化しようという話だと解釈した。

## 懇親会

## 終わりに

手書きのメモについては、以下のフォトライフに上げています。


## その後

触発されて以下2件のブログを書きました。

* [ベアメタルサーバにRancherOSをインストールする手順（といくつか気づいたこと）。](https://kitkatayama.hatenablog.com/entry/2019/04/09/230012)
* [Docker in Dockerで手軽にBuildpacksを試す。](https://kitkatayama.hatenablog.com/entry/2019/04/12/174340)

参照:
https://kubernetes.io/docs/tasks/tools/install-minikube/
https://help.ubuntu.com/community/KVM/Installation
18.04でもgroupがlibvertの模様。
https://github.com/dhiltgen/docker-machine-kvm/releases
18.04にdriverが無い
