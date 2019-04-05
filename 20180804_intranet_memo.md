# 家イントラネット構築詰まり箇所のメモ

ADサーバー2号機がドメインに入れない  
⇒DNSサーバを外向き用にしていた。AD1号機のIPへ変更。

WSUSサーバーとの疎通が確認できない  
⇒WSUSサーバーのWindows Firewallをオフに。（多分オフにしないほうがいい。次考える。）

WSUSサーバーのWindows Updateに接続できない  
⇒WSUSのポートはIISマネージャー⇒サイト⇒WSUSの管理⇒バインドの編集で確認後、GPOで配布。  
　（確認のためならクライアント1台のローカルグループポリシーで変更）

[https://blogs.technet.microsoft.com/jpwsus/2014/04/02/wsus-4-0-windows-server-2012-2/]

そもそもWSUSを構成する上での設計をどうするか定まってない  
⇒KB

[https://technet.microsoft.com/ja-jp/library/cc720479(v=ws.10).aspx]

ついでにAD（AD DS)の設計も定まってない  
⇒KB

[https://technet.microsoft.com/ja-jp/library/cc984970.aspx]

収拾がつかないので、今回のイントラネット全体の設計の目的を定義。  
「10人を超えないユーザー、30台を超えないコンピューターで Office365をデバイス認証で使えるようにする」  
で、技術的興味からAD2号機を入れたが、同一HW内に存在するので意味がない。  
⇒クラウド上に2号機を仕込む検討。クラウド側から家イントラネットにVPNを接続する必要あり。  
　⇒AD1号機以外にVPNの制御を任せる必要あり。  

外と中のNWが同一。VPN構成やプロキシの構成が出来ない  
⇒見切り発車だったので、後ほど再構成する。

画面がロックされてわずらわしい  
⇒デスクトップの電源を切るを解除。

[http://azuread.net/2013/10/22/windows-server-2012-r2%E3%81%AE%E3%83%AD%E3%83%83%E3%82%AF%E8%A7%A3%E9%99%A4/]

ADサーバー2台もいらん。  
⇒2台入れたADのうち、1台を降格。

[https://blogs.technet.microsoft.com/jpntsblog/2013/11/17/windows-server-2012/]

ADFSサーバ構築。

[http://blog.o365mvp.com/2013/09/29/adfs_install_manual_win2k12r2/]

HDD容量がやばかったので拡張。外だけ変更の方法はわかったので、あとはディスクの管理からボリューム拡張ウィザードを。

[http://qiita.com/tada_infra/items/2f458832549c8006beb2]

その他、構成の上で参照したサイト。

Windows設計手順etc.

[http://jp.fujitsu.com/platform/server/primergy/technical/construct/]