## tl;dr

* GROWIという新進気鋭のWikiシステムで、ドキュメントの和英翻訳を募集していた
* [インフラ勉強会 Wiki](https://wiki.infra-workshop.tech/)でもお世話になっているシステムなので、contributeしようと思った
* Windowsでも、簡単に開発環境構築してプルリクまで投げられるいい時代だ

## インフラ勉強会でのセッション動画

* インフラ勉強会はこちら
  * 公式サイト: [https://wp.infra-workshop.tech/]
  * 当日のカレンダー: [https://wp.infra-workshop.tech/event/how-to-contribute-to-oss-through-growi-docs/]

[https://www.youtube.com/watch?v=uv3K5lBZx7s:embed:cite]

## 先に書くあとがき

* ある程度の台本を考えて、2回リハーサルを行ったが、それでも本番は難しい
* とはいえ、自分のモチベーションのアップ、知識の定着、プレゼンテーション力の向上など、さまざまな方面で刺激や学びがあった
* 皆も [weseek / growi-docs](https://github.com/weseek/growi-docs) にPRして充実させていこう！

以下、動画内の手順（前後している部分もあるがご了承を）

## 環境構築

* 素のWindows 10から Visual Studio Code + git for windows + Node.js for Windowsでweseek/growi-docsをビルドできる環境を構築する

### Visual Studio Code

* 公式サイト: [https://azure.microsoft.com/ja-jp/products/visual-studio-code/]

#### インストール、動作確認

* 特に無し。流れに任せて。

#### 日本語化

* "Japanese Language Pack for Visual Studio Code" をインストール
  * 英語に慣れたい人はそのままでもOK!!

### git for windows

* 公式サイト: [https://gitforwindows.org/]

#### インストール、動作確認

* 基本的には流れの通り
  * 改行の扱いのみ、個人的には **Checkout as-is, commit as-is** を指定している
    * チェックアウトする際も、コミットする際も改行コードを保持するということ
    * 新規ファイルを作成する際は、Visual Studio Codeの設定に従われ、そのままの状態となる。必要に応じて変更すること。
    * Linux用のリソースしか扱わないという方は **Checkout as-is, commit Unix-style line endings** にするとトラブルが無くなるかもしれない

#### ユーザー設定

* SSH鍵の作成、GitHubに登録
  * 鍵の生成
    * `cd ~/.ssh/`
    * `ssh-keygen -t rsa`
    * `cat id_rsa.pub | clip`
  * 鍵の登録
    * GitHubにログインしてこちら [https://github.com/settings/emails]
* gitの設定
  * `git config --global user.name "Your Name"`
  * `git config --global user.email "protected-mail@example.com"`
    * gitのコミットを行うと、gitの編集履歴には両者が記録される。GitHub上や普段の利用では気にしないが、`git log`で確認することができる
    * ~/.gitconfigに設定は書かれている
  * GitHubでは、このコミットされるメールアドレスと、アカウントのVerify済みメールアドレスをもとにユーザーを識別している
    * [https://help.github.com/en/articles/setting-your-commit-email-address-in-git:title]

### Node.js for Windows

* [https://nodejs.org/ja/]

#### インストール、動作確認

* 特に無し。流れに任せて。

## OSSを動かす

* 今回の対象OSS: [https://github.com/weseek/growi-docs]
  * [https://docs.growi.org/] の元となるリポジトリ

### クローン、ビルド、確認

* `git clone git@github.com:weseek/growi-docs.git`
* 公式READMEより [https://github.com/weseek/growi-docs]
  * `yarn`
  * `yarn start`
    * しかしコマンドが無いと怒られる
* `yarn`が無いのでインストール
  * 公式サイトの手順 [https://yarnpkg.com/lang/ja/] と言いたいところだが、若干ズルをして・・・
  * `npm install -g yarn`
* ビルド、確認
  * `yarn`
  * `yarn start`

## OSSを修正、動作確認する

### フォーク、クローン、ビルド、確認

* [https://github.com/weseek/growi-docs] にアクセスして、「Fork」をクリック
  * Fork中のフォークアイコンかわいい
* Fork後、自アカウントにgrowi-docsができるので、そちらをクローンする
  * kitkatayamaの場合: [https://github.com/kitkatayama/growi-docs]
    * フォークしたものでも、リポジトリ名は、リポジトリ設定から変更することができる
* 先ほどビルドした公式リポジトリからのcloneはフォルダごと削除する
  * gitからcloneした内容や、Node.jsが実行した結果はフォルダ内に閉じているので便利
* `git clone git@github.com:<アカウント名>/growi-docs.git`
  * kitkatayamaの場合: `git clone git@github.com:kitkatayama/growi-docs.git`
* Visual Studio Codeでcloneしたディレクトリを開く
* コンソールを開き、ビルド確認
  * `yarn`
  * `yarn start`
    * 問題が無ければそのままビルドできる
* [https://localhost:8080/] にアクセス
  * Visual Studio Codeのコンソールに表示されているURLはCtrlを押しながらクリックで開くことができる

### ブランチ作成、修正、動作確認、コミット

* 本家masterと差分を作るため、ブランチを作成する
  * `git checkout -b "ブランチ名"`
    * ブランチ名に命名ルールはないが、一目で何を行ったかわかるようなブランチ名にするようにしている
    * issueを発行している場合、＃番号を含めておくことで連携することができる
* ファイルを修正、動作確認
  * `yarn start` している状態でファイルを変更すると、即座にビルドされ [https://localhost:8080/] にも変更が反映される
  * 望んだとおりの変更となっているか確認する
    * Visual Studio Code上でもMarkdownをプレビューすることはできるが、微妙にレイアウトや構文解析のロジックが違う可能性があるので、必ずビルドを通して確認する
* コミットする
  * `git add ...`, `git commit ...` してもいいが、せっかくなのでIDEの力を借りる
  * Visual Studio Code上でソース管理を開く
    * "+"ボタンでadd
    * コメント欄にコメントを記入
       * なるべく簡単な英語で、ブランチの変更内容をコメントする
    * "✓"ボタンでコミット

### プッシュ

* 上記までの操作は、あくまでローカルPC上での操作
  * これをGitHubにも教えてあげる
* プッシュする
  * `git push origin ブランチ名`
  * gui上でプッシュでも可

## OSSにプルリクエストを送る

* 以上で自分が望んだ変更を要求するための準備が完了した

### プルリクエストの作成

* プッシュによってGitHubが差分を検知するので、"Compare & pull request"する
* メッセージにはコミット時に使用したコミットメッセージが使用される
  * 補足や、対日本人のリポジトリにPRする場合は、日本語で補足を書いてもいいかもしれない
