# #インフラ勉強会 にて「Ubuntu 19.10リリース！皆でリリースノート読もうぜ」のセッションを行いました。

## イベントについて

下記のURLに、詳細を記載しております。

https://wp.infra-workshop.tech/event/ubuntu-19-10-reading-release-note/

下記に、一部を抜粋いたします。

<pre>

■ 概要
10/18、Ubuntu 19.10がリリースされました🎉🎉🎉
https://ubuntu.com/

リリースノートはこちら！（英語）
https://wiki.ubuntu.com/EoanErmine/ReleaseNotes

本セッションでは、登壇者がUbuntu 19.10のリリースノートを元に資料を作成し、
それを参加される皆さんが見てマサカリ🪓を投げあい、それを眺めている皆さんが成長する場です💪

可能であれば、マイクをONにしていただいて、ワイワイと議論しましょう！
マイクは苦手というあなた！インフラ勉強会Wikiのこちらのページに、少しだけでもメモを書いてくれると嬉しいな👍
https://wiki.infra-workshop.tech/5da916eefe54f300584dd490

※尚、本セッションは下記のチャンネルにて、YouTubeで同時配信を行います。

・音声はDiscord、YouTube両方で配信いたしますので、ご視聴いただくアプリケーションをお選びください。
・YouTubeでは5～10秒程度の遅延が見込まれます。ご了承のほど、お願いいたします。
・両方に参加される場合は、片方のアプリケーションをお手元でミュートを行うなどの操作をお願いいたします。
・YouTubeで配信する関係上、マイクONで話される場合は、機密情報などが残らないようご配慮いただけると助かります（要望があれば、動画を一旦非公開後、編集して公開します）。

※以前行った「CentOS 8リリース記念！何が変わったのか抑えよう！」と同じクオリティは期待しないでください

CentOS 8リリース記念！何が変わったのか抑えよう！

■ 目的（ゴール）
Ubuntu 19.10について、わかった気になる。

</pre>

## 資料（スライド）

https://speakerdeck.com/kitkatayama/ubuntu-19-10-reading-release-note

## 配信アーカイブ

https://www.youtube.com/watch?v=D3ytBSs9YhQ

## あとがき

世の中ではラグビー日本代表がベスト8で勝利し、日本シリーズが開催されておりました。そんな中でも、10名を超える来場者を迎えることができ、大変光栄に思いました。

https://twitter.com/kitkatayama/status/1185904325312139264

前回の反省を活かし、DiscordのStreamkit + 独自のカスタムCSSを用意することでコメントの表示を改善することもできました。Streamkitの具体的な解説は避けますが、配信に使用したカスタムCSSは以下のようになります。参考にしていただけると幸いです。

<pre>

.chat-container{
 height:200px;
 width:1564px;
}
.chat-container .channel-name{
 line-height: 20px;
 font-size: 20px;
 padding: 6px;
}
.chat-container .messages{
 height: 152px;
 padding: 8px 5px;
}
.chat-container .messages .message .timestamp{
 font-family: monospace;
}

</pre>