# MaxMind GeoLite2 の取得方法が変わったので対応する。

かたやまです。あけましておめでとうございます。転職エントリより先にこちらを書くとは思いませんでした。MaxMindのGeoLite2を使われている方は、年を越せたでしょうか。

[MaxMind社から提供方針の変更がアナウンスされてました](
https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/
)ね。今までFreeにURLにアクセスすればダウンロードできていたGeoLite2ですが、2019/12/30からダウンロードができなくなりました。

その対応を行っていきましょう。

## TL;DR

* EULAが変わったのでよく読むこと
* ライセンスキーを含んだwgetを行うことで、今まで通りスクリプトで取得できる

## TOC

* EULAに同意する
* アカウントの登録
* ライセンスキーの入手
* URLの修正

## EULAに同意する

https://www.maxmind.com/en/geolite2/eula

割と長いが頑張って読む。独自解釈を書くのはご法度なので自分で読んでほしい。

## アカウントの登録

https://www.maxmind.com/en/geolite2/signup

EULAに同意することは強制されるので諦めてほしい。チェックボックスを外しても先に進まない。

## ライセンスキーの入手

アカウント左メニューの **My License Key** から入手できる。

尚、警告にある通りライセンスキーが表示されるのはこの瞬間のみのため、必ずメモを取ること。

また、ライセンスキーを入手した時点で、登録したメールアドレスに一報来る。万が一MaxMindのアカウントが不正アクセスされてライセンスを入手されたとしても連絡が行くかもしれない。

## URLの修正

[このページの最後に書かれている](https://dev.maxmind.com/geoip/geoipupdate/#For_Free_GeoLite2_Databases)通り、

* `/geoip_download_by_token` を `/geoip_download` に書き換える
* `token=XXXX` を `license_key=YOUR_LICENSE_KEY` に書き換える。 `YOUR_LICENSE_KEY` は MaxMind のアカウントページから取得できる。
* 必要であれば `date` パラメータを削除する。[^1]
* `wget` や `curl` を使う場合は、必ずクオートでURLを囲むこと。[^2]

例えば、GeoLite2-Countryのgzipで、本日(1/9)から最新の物を取得したいURLに書き換えたい場合、

> https://download.maxmind.com/app/**geoip_download_by_token**?edition_id=GeoLite2-Country&**date=20200107**&**token=XXXXXXXXXXXXXX**&suffix=tar.gz

を

> https://download.maxmind.com/app/**geoip_download**?edition_id=GeoLite2-Country&**license_key=YOUR_LICENSE_KEY**&suffix=tar.gz

とする。

あとはwgetやcurlで叩けば入手できる。

```bash
$ wget "https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=xxxxxxxxxxxx&suffix=tar.gz" -O geolite2-c.tar.gz
--2020-01-09 22:20:36--  https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=xxxxxxxxxxxx&suffix=tar.gz
Resolving download.maxmind.com (download.maxmind.com)... 2606:4700::6810:252f, 2606:4700::6810:262f, 104.16.38.47, ...
Connecting to download.maxmind.com (download.maxmind.com)|2606:4700::6810:252f|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2050306 (2.0M) [application/gzip]
Saving to: ‘geolite2-c.tar.gz’

geolite2-c.tar.gz             100%[=================================================>]   1.96M  2.85MB/s    in 0.7s

2020-01-09 22:20:37 (2.85 MB/s) - ‘geolite2-c.tar.gz’ saved [2050306/2050306]

$ tar xvf geolite2-c.tar.gz
GeoLite2-Country_20200107/
GeoLite2-Country_20200107/LICENSE.txt
GeoLite2-Country_20200107/COPYRIGHT.txt
GeoLite2-Country_20200107/GeoLite2-Country.mmdb
```

## 番外: GeoIP Updateの利用

[GeoIP Update](https://github.com/maxmind/geoipupdate)を使うこともできる。様々な理由が無ければ、利用を検討してみることをおすすめしたい。

## おわりに

TwitterとGitHubはそろそろ私の使っているIPをドイツから日本にしてほしい。

[^1]: EULA的には「最新のものをリリースしたら30日以内に削除して新しいのに差し替えろ」って言ってるがよくわからない

[^2]: http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.mmdb.gz のような文字列から単純に置き換えると、シェルスクリプトは & でしぬ