# 独自ドメインでGithub Pagesを使う。

## tl;dr

・これを追っていただけると良い(10:03 PM - Jun 19, 2018の部分をクリック)

 [https://twitter.com/kitkatayama/status/1009059027819458560]

## やりたいこと

* GitHub Pagesが強そうなのを知ったので、どうせなので使ってみたい
  * 独自ドメインのresume的なサイトが欲しい
  * Amazon S3だと、小数点以下が降り積もって課金されるかもしれない

## やったこと

基本的には、公式サイトに記載の内容を実行しただけ。ドメインのCNAME設定のみ、Azureを使用したことが特別かもしれない。
[https://pages.github.com/]

### 1. GitHubでのリポジトリ作成、index.htmlの作成、push

割愛。GitHub Pagesに記載のそのままの操作で実行できるので、試してほしい。（できない場合は、コメントいただけると助かる。）

### 2. DNS設定をしない状態で、Custom Domainを設定する

ここからは、"Custom URLs"に記載の内容となる。Quick Startの記述が若干ややこしい気がする。  
[https://help.github.com/articles/quick-start-setting-up-a-custom-domain/]  

### 3. Azure DNSでCNAME設定する

### 4. HTTPS対応する

## 結果

* [独自ドメインで、HTTPSに対応しつつ、ハローワールドできた。](https://www.moffulab.com/)

## 今後

* Jekyllを使ってresume等の、発信したいページを書くための準備をする。

## 技術的補足

気づいたことを記載する。

### DNS設定とGitHubの連携について

[コミットを追っていただけるとわかる](https://github.com/kitkatayama/kitkatayama.github.io)が、直下に"CNAME"というファイルが作成されている。これは、リポジトリのGitHub Pagesを編集した際に勝手に作成、変更される。
推測だが、Settingsの画面を変更しなくとも、同等に編集してpushすることで反映されるのではないかと考えた。

### AzureのDNS設定について

今考えれば当然だったが、@にCNAMEが使えない。素直にwwwを登録する。

### GitHub Pages 独自ドメイン使用時の証明書について

Let's Encryptが使用されている。