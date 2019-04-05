# Challonge.comの英日翻訳を完了しました

## tl;dr

* 2019/03/01現在 1万3000 wordsくらいからなるChallonge.comの英語リソースの翻訳を一人でやった
* Challongeに出てくる日本語は99.9%私の日本語だからクソだったら教えてくれ！！
* 新登場したコミュニティ機能も是非使ってね！EVO Japanで配ってたアレ！！

EVO Japanで配ってたアレ※画像

## モチベーション

* Challongeというサービスが好きだから
* 格闘ゲームが好きだから
* Challongeが栄えれば格闘ゲームコミュニティがより良くなると信じているから
* Smash.ggが日本語化対応しっかりしてるから負けたくない

## やったこと

* Challonge.comの英語リソースを日本語に翻訳
  * 活動自体は[2016年5月23日(リンク)](https://twitter.com/kitkatayama/statuses/734739056098054144)から
    * [<s>OneKey</s>OneSkyの件でメール(リンク)](https://twitter.com/kitkatayama/statuses/734241465178591232)して日本語用のページも用意してもらいました
  * 最終的に、全体の99.9%のリソースを個人で担当
    * 勇気を出して翻訳してくれた有志も居るので後述
    * おおよそ12,500 words担当
  * 2019/02/28時点で最終的に全部デプロイしてもらった
    * [チェンジログ](https://challonge.com/ja/changelog)のこの部分

> 2019-02-28  
> Changed  
>・Added page title and open graph info (social sharing image, title, description) for communities  
>・Updated Japanese translations  
>Fixed  
>・Various minor changes to communities, some bigger additions coming soon

* 私以外に翻訳に参加された皆様、お疲れさまでした
  * [スパイクさん](https://twitter.com/spiq)
  * 0061dさん
  * River_The_Demonさん
  * Challonge LLCの中の方

Challongeの翻訳に貢献したサマリ 前述の合計よりも多いのは英語の訂正を再翻訳とかしたため※画像

## 気を付けたこと

* より日本人が読みやすく、かつ統一性のある翻訳へ
  * 訳間の言葉を統一
    * 翻訳済みリソースのリファクタリング
    * 似たようなリソースのうち、副詞(Successfullyなど)が英文中の場所が不定であっても、日本語では統一する
  * 直訳より自然な言葉へ
    * いわずもがな
  * 実際に操作する
    * 英語サイトと日本語サイトを比較しながら
      * Chromeのユーザー機能使うと同時に別ユーザーでログインできて良い
    * 各リソースにString Key（例: tournaments.labels.*）や、ファイル名（例: views__tournaments.yml）があるので、ある程度配置を類推する
    * 触ってわかりづらかったりするところは戻って修正
* 英語（第一リソース）が必ずしも合っているとは限らない
  * 例1: 言葉の意味が似ていて誤って配置している(Type, Format)
  * 例2: Typoしてる(Seach->Search, Udated->Updated)

## 今後

* 増えた英語リソースの対応
  * 2019/03/01時点、未翻訳のリソースが500wors増えてたので<s>対応する</s>2019/03/02時点で対応した
  * 新機能開発中なので追従する必要あり
    * 直近だと[Challongeコミュニティ(要ログイン)](https://challonge.com/communities.html)向けのリソースとか、一部の切り出されてなかったリソースの切り出しが完了したっぽい

* 宣伝する
  * いくらサービスが最高でも誰も使わなければそれはクソだ（とか偉い人が言ってた気がする）
  * 操作する上でのウィザード翻訳も質の良いものにしたつもりなので、それに従ってくれればほぼ問題ないようにはなったはずではあるけど・・・
  * How-toの日本語動画とか、IT業界的な手順書みたいなものとか、そういうものが必要な気がしてる

* 翻訳方法、ナリッジの整理
  * 万が一私がやる気を無くす/亡くなる等があったとき、誰も引き継げなくて廃れる
  * 最低限、OneSkyというプラットフォームの日本語情報は、ほぼない状況なのでどうにかしたい
    * [Launchpad](https://launchpad.net/)とか[Crowdin](https://crowdin.com/)とか強いもんね・・・
  * 一人で対応した方が衝突や齟齬が無いというのも考えるのが難しいところ

## おまけ

<s>EVO Japanのときにもおねだりしてたんですが、</s>中の人であるDavid氏から永年Challongeプレミアをいただきました（[ユーザーページにもアイコンが付きました](https://challonge.com/ja/users/moffumoffu)）。

> Hi MoffuMoffu, this is David at Challonge. Thank you so much for your help with translating Challonge to Japanese! We greatly appreciate it, and as a token of our appreciation, I upgraded your account to Challonge Premier indefinitely (at least until we replace Premier with something else in the coming year or so).

[https://twitter.com/kitkatayama/status/1101657960529326081]

## 余談: Challongeの分析とかポエム

この辺、ちょっと個人的な懸念があるので、誰か教えてくれると嬉しい。

* Challongeは、収益化という観点で日本展開に乗り気じゃない気がする
  * Challongeは有料トーナメントのチケット、商品販売窓口、プレイヤー間の寄付の分配など、お金が動くときの手数料で儲かることができる
  * だが、寄付や賞金制のトーナメントが行える仕組みはあるんだが、日本の法律的にNG
    * "トーナメント予想"はその最たるもの
      * 日本人としては「トーナメントの結果予想して何になるの？」という感じ
      * 寄付付き大会では、「トーナメント予想を当てた人が推すプレイヤーに賞金が渡される」という仕組みがある
  * EVO Japanなどは、スタッフ派遣やトーナメント開催のコンサルでお金貰っていると推測するが・・・
  * EVO 2019がSmash.ggを使うのを見るに、ちょっと立場が危うい気もする

* イマイチChallongeの主戦場が分からない
  * ChallongeサイトのオーナーがSplitmedia Labs(XSplitの開発)から子会社のBettercast Limitedになってるっぽいし、政治状況が気になる。