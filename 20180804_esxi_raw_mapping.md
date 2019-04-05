# ESXiの物理HDDをRAWマッピングして、仮想マシンのWindows ServerからNTFSを読み取る

## tl;dr

* RAWデバイスマッピングにはCUI操作が必要、かもしれない

## 作業に当たって

### モチベーション

古いSATA接続のHDDからデータを取得したかったが、SATA-USBのアダプタが使用できなかった※1ため、仕方なくESXiからどうにかファイルアクセスできないか考えた。

### 作業前の予想

ESXiの物理ホストにHDDを追加し、RAWデバイス（vmdk等ではなく）として仮想マシンにマウントしてネットワーク共有をかければ良いのではないか

### 結果

概ね予想はあっていたが、CUIを通じてvmdkファイルを作成し、既存ハードディスクとしてマウントしなければならない。

---

## 手順

例によって、アップロード済みなので参考にしていただきたい。

VMwareのKBを参考にする。引っかかった点は、以下の2点。

### vmdkの作成

[https://kb.vmware.com/s/article/2093360]

### 仮想マシンへのマウント

TBO

### 仮想マシン上での設定

TBO

---

## 留意点

* VMware Docsに独立型の読み取り専用ディスクについての注意点が書いてあるので、一読いただきたい。
  * [https://docs.vmware.com/jp/VMware-vSphere/6.5/com.vmware.vsphere.security.doc/GUID-1E583D6D-77C7-402E-9907-80B7F478D3FC.html]
  * しかし、前述のとおり、HTML5クライアントでは、現状独立型の読み取り専用ディスクに設定できなさそうである。