## tl;dr

* 起動オプションに `--disable-gpu-compositing` をつければ大体解決する
* アプリケーションのGPU支援機能を停止すると、基本的に**パフォーマンスは落ちる**ので注意

## モチベーション

* XSplitを使用して配信する際、ChromeやVS Codeを使いたいが映らないためモニターキャプチャをしていたが、放送事故が起きかねないので独占ウィンドウキャプチャしたかった
* GPUとレンダリング、その他の機構について興味がわいた

## 結果

* tl;drの通り、`--disable-gpu-compositing`を付けるだけでよい。
* アプリケーションごとにGPU支援機能を停止する方法は異なるので、もし映らないアプリケーションがあれば、全てのGPU支援機能をオフにするような設定を探すと良い
  * Discord: `オプション->テーマ->ハードウェアアクセラレーション`
  * VS Code: 恒久的な設定はないっぽい。issueになっている。 [https://github.com/Microsoft/vscode/issues/15211:title]

**画像**
chromeに --disable-gpu-compositing オプションを付ける

## パフォーマンス比較

※XSplitで非均等な負荷がかかっているため、参考程度に。

[https://youtu.be/vyp9JyRS-Bc:embed:cite]

* オプション無し [https://web.basemark.com/result/?4PTWFcmV]
* --disable-gpu-compositing [https://web.basemark.com/result/?4PTWPEtB]
* --disable-gpu 遅すぎて無理
  * "ハードウェア アクセラレーションが使用可能な場合は使用する"をオフ と等価の設定

## chrome://gpu の比較

* Chromeのオプションを見ることで、何がONで何がOFFなのか何となく知ることができる

**画像**
オプション無し 、--disable-gpu-compositing、--disable-gpu

## あとがき

* 最近Chrome重くなったなーと思ってたら、配信向けにハードウェア アクセラレーション外してから1か月以上経過していた。アホだった
* 配信用に`--disable-gpu-compositing`のオプションを付けたショートカットを準備していると便利
* 参照URLにChromeのレンダリング概要やオプション一覧が載っている。参考になれば。

## 参照

* [https://stackoverflow.com/questions/29966747/how-can-i-disable-gpu-rendering-in-visual-studio-code:title]
* [https://github.com/Microsoft/vscode/issues/52096:title]
* [https://superuser.com/questions/988929/how-to-launch-chrome-with-explicit-hardware-acceleration-setting:title]
* [https://www.chromium.org/developers/how-tos/run-chromium-with-flags:title]
* [https://peter.sh/experiments/chromium-command-line-switches/:title]
* [https://www.chromium.org/developers/design-documents/gpu-accelerated-compositing-in-chrome:title]
* [https://gist.github.com/Lange/a2e44b0097ee99538b7c]