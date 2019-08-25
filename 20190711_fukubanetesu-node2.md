# #ふくばねてす node-2 にお邪魔してきました。

## イベントについて

[#ふくばねてす node-2](https://fukubernetes.connpass.com/event/135117/)

## まとめ（個人、運営問わず）

Toggeter: https://togetter.com/li/1375491
loftkun: https://github.com/loftkun/conference/blob/master/2019/0711-fukubernetes-02/README.md

## セッション

### Transnano さん「コンテナ移行におけるアレコレと使えるアレコレ(仮)」

* 資料
  * [2019/07/11 ふくばねてす node-2 コンテナ移行におけるアレコレと使えるアレコレ(仮)](https://speakerdeck.com/transnano/)11-hukubanetesu-node-2-kontenayi-xing-niokeruarekoretoshi-eruarekore-jia
* 参照
  * [日本企業のDockerコンテナー本番導入率は1割未満--IDC調べ - ZDNet Japan](https://japan.zdnet.com/article/35139430/)
  * [Ops meets NoOps](https://www.slideshare.net/ToruMakabe/ops-meets-noops)

#### Cloud Native Trail Map

https://github.com/cncf/landscape/blob/master/README.md

1. コンテナ化
    1. 一般的にDockerコンテナで行われる
    1. どのような規模のアプリケーションや依存関係(たとえエミュレーター下で実行されている[PDP-11](https://ja.wikipedia.org/wiki/PDP-11)のコードさえ)もコンテナ化可能
    1. 時間をかけて適切にアプリケーションを分割し、マイクロサービスとして新機能を記述する方向目指すべき
1. CI/CD(継続的インテグレーション/継続的デリバリー)
    1. 以下のようにCI/CDをセットアップする：ソースコードの変更が新しいコンテナをビルドし、テストし、ステージング環境にデプロイされ、最終的にはプロダクション環境にもなるように。
    1. 自動ロールアウト、ロールバック、テストをセットアップする
1. オーケストレーション&アプリケーション定義
    1. Kubernetesは業界をリードするオーケストレーションソリューション
    1. [cncf.io/ck](https://cncf.io/ck)から、認定済みのKubernetesディストリビューション、ホスト済みプラットフォーム、またはインストーラーを使用すべき
    1. Helm Chartsは、非常に複雑なKubernetesアプリケーションでも定義、インストール、アップグレードするあなたを助ける
1. 可観測性&分析(Observability & Analysis)
    1. モニタリング、ロギング、トレーシングのソリューションを決めましょう
    1. CNCFプロジェクトのPrometheusをモニタリングとして、Fluentdをロギングとして、Jaegerをトレーシングとしてオススメします
    1. トレーシングについては、JaegerのようなOpenTracing互換実装のものを探しましょう
1. サービスプロキシ、ディスカバリ&メッシュ
    1. CoreDNSは高速かつフレキシブルなツールで、サービスディスカバリに非常に有用です。
    1. EnvoyとLinkerdはサービスメッシュアーキテクチャを実現します
    1. このようなツール群は、ヘルスチェック、ルーティング、ロードバランシングを提供します
1. ネットワーキング&ポリシー
    1. さらにフレキシブルなネットワーキングを可能にするには、Calico、Flannel、Weave NetのようなCNI準拠のネットワークプロジェクトを使用しましょう。
    1. Open Policy Agent(OPA)は、認可やアドミッションコントロ－ルからデータフィルタリングまでこなすことのできる汎用的なポリシーエンジンです。
1. 

### Kta-M さん「わりとゴツいKubernetesハンズオンそのあとに」

* 資料: https://speakerdeck.com/ktam1219/waritogotuikuberneteshanzuon-sofalseatoni
* 参照:

### kyswtnbさん「kubernetesのアップデートに関するあれこれ(仮)」

* 資料: TBO

### loftkunさん「IstioによるTraffic Management」

* 資料: https://speakerdeck.com/loftkun/traffic-management-with-istio
* 参照:

### udzuraさん「CNDT2019 見所をご紹介...???」

* 資料: https://gist.github.com/udzura/616f792b1187d9110029f332893e17ae

### nwiizo さん「反脆弱なシステムは目指してもいいものか？」

* 参照: https://speakerdeck.com/nwiizo/can-you-aim-for-an-anti-fragile-system

## 締めの挨拶→撤収

手書きのメモについては、以下のフォトライフに上げています。

http://f.hatena.ne.jp/kitkatayama/20180408_fukubanetesu/
