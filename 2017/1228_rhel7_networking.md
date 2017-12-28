# RHEL7 系のネットワーク設定

6 系と変わった点、変わってない点を整理するために調査した。主に参照した情報ソースは[「RHEL7ドキュメント」](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/)と[「標準テキスト CentOS7」](http://www.sbcr.jp/products/4797382686.html)の2つ。

## NetworkManger とは
### ドキュメント

* [NETWORKMANAGER について](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-introduction_to_networkmanager)
* [NETWORKMANAGER とネットワークスクリプト](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-NetworkManager_and_the_Network_Scripts)
* [RHEL 7 における Network Initscript と NetworkManager の違い](https://access.redhat.com/ja/node/1441163)


### 概要

* RHEL6 ではネットワークスクリプト（`/etc/init.d/network`）によりネットワーク設定が行われていた
  * ここが `/etc/sysconfig/network-scripts` や `/etc/sysconfig/static-routes` などを読んでいた
* RHEL7 より `NetworkManager` を使ってネットワーク設定することが推奨されている
* ただしそのままスクリプトを継続して使用することもできる
* NetworkManager は現在、RHEL 7 でデフォルトで使用されている
* NetworkManager を無効すると、ネットワークインターフェイス管理には代わりに initscripts が使用される
* NetworkManager の使用が増えてきているため、今後のメジャーリリースで Network Initscript が廃止になる可能性がある
* RHEL7 では NetworkManager が最初に起動し、/etc/init.d/network が NetworkManager をチェックして NetworkManager 接続の改ざんを防ぐ
* NetworkManager は sysconfig 設定ファイルを使用するプライマリーアプリケーションとされており、/etc/init.d/network はフォールバックとなるセカンダリーとされている
* RHEL7 でのネットワーク設定の方法には具体的には以下の4つの方法がある
  * `nmcli`
  * `nmtui`
  * GNOME コントロールセンター
  * ファイルを直接編集する


## NetworkManager の制御
### ドキュメント

* [NETWORKMANAGER のインストール](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-installing_networkmanager#sec-Interacting_with_NetworkManager)


### 概要

* `NetworkManger` は `systemd` によって管理されていて、デフォルトでは起動時に立ち上がる
* 制御は `systemctl` コマンドを利用して以下のように行う

```
systemctl start NetworkManager
systemctl stop NetworkManager
systemctl restart NetworkManager
systemctl status NetworkManager
```

* NetworkManager がデーモンで起動していることは次のように確認できる

```
# systemctl status NetworkManager
● NetworkManager.service - Network Manager
   Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2017-12-26 05:04:04 JST; 1 day 22h ago
     Docs: man:NetworkManager(8)
 Main PID: 746 (NetworkManager)
   CGroup: /system.slice/NetworkManager.service
           └─746 /usr/sbin/NetworkManager --no-daemon
```

## ネットワーク設定と設定ファイル

ネットワーク設定ファイルの位置はかわらずだが、ファイルを直接編集せず nmcli, nmtui などを利用することが推奨されている。サーバー構築時に主に設定する内容と対応する設定ファイルは以下のとおり。

* ホスト名とIPアドレスの対応関係: `/etc/hosts`
* 問い合わせ先DNSサーバ設定: `/etc/resolve.conf`
  * nmcli で DNS サーバーを指定すると自動的に更新される
* ネットワーク名とネットワークアドレスの対応関係: `/etc/networks`
* インターフェースの設定: `/etc/syscnofig/network-scripts/ifcfg-*`
  * NIC の命名が eth0, eth1 のような形ではなく、`udev` によりデバイスに応じて命名されている
* インターフェースごとのルーティング設定: `/etc/sysconfig/network-scripts/route-*`
  * ip コマンドを使って設定した静的ルートは、システムが終了したり再起動すると失われる

重要な設定を一部抜粋して確認する


### /etc/sysconfig/network-scripts/ifcfg-*
#### ドキュメント

* [ifcfg ファイルを使用したネットワークインターフェースの設定](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_command_line_interface#sec-Configuring_a_Network_Interface_Using_ifcg_Files)
* [nmcli を使用したネットワーク接続](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_networkmanager_command_line_tool_nmcli#sec-Connecting_to_a_Network_Using_nmcli)


#### 概要

* ip コマンドを使ったネットワークインターフェースの設定は、システムが終了したり再起動すると失われる
* システム再起動後も維持されるようにネットワークインターフェースを設定するには、`/etc/sysconfig/network-scripts/` ディレクトリー内のインターフェースごとの設定ファイルに格納する必要がある
* ファイル名は、ifcfg-<ifname>
* 設定項目はざっくりと以下のような具合
  * TYPE(Ehternet, Wireless, Bridge): 種類
  * BOOTPROTO(none, bootp, dhcp): 起動方法
  * DEFROUTE(yes, no): デフォルトルートとして使用されるか
  * IPV4_FAILURE_FATAL(yes, no): IPV4 で初期化に失敗したときにインターフェースを起動失敗扱いにするか
  * NAME: インターフェースの名前
  * UUID: そのまま
  * DEVICE: 物理デバイス名
  * ONBOOT(yes, no): 起動時にインターフェースを起動するか
  * IPADDR: IPアドレス
  * PREFIX: ネットマスク

```
# 動的ネットワーク設定の例
DEVICE=em1
BOOTPROTO=dhcp
ONBOOT=yes

# 静的ネットワーク設定の例
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
PREFIX=24
IPADDR=10.0.1.27
```

* ネットワークインターフェースの設定には以下の方法がある
  * nmcli コマンドを実行する
  * nmcli エディターを利用する
  * nmtui を利用する

コネクション名が扱いづらい形なので修正したほうが作業がしやすい

```
$ nmcli connection show
NAME           UUID                 TYPE            DEVICE
System ens192  *******************  802-3-ethernet  ens192

$ nmcli connection modify "System ens192" connection.id eth0
NAME  UUID                 TYPE            DEVICE
eth0  *******************  802-3-ethernet  ens192
```

設定はおおむね以下のような形

```
$ nmcli connection modify \
  connection.autoconnect yes \
  ipv4.dns 8.8.8.8 \
  ipv4.method manual ipv4.addresses 10.*.*.*/24 \
  +ipv4.routes "192.168.0.0/16  10.*.*.254"
  ipv4.ignore-auto-routes no
  ipv4.ignore-auto-dns yes
```


### /etc/sysconfig/network-scripts/route-*
#### ドキュメント

* [ifcfg ファイルでの静的ルートの設定](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_command_line_interface#sec-Configuring_Static_Routes_in_ifcfg_files)
* [nmcli を使った静的ルートの設定](https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_networkmanager_command_line_tool_nmcli#sec-Configuring_Static_Routes_Using_nmcli)


#### 概要

* ip コマンドを使って設定した静的ルートは、システムが終了したり再起動すると失われる
* システム再起動後も維持される静的ルートを設定するには、`/etc/sysconfig/network-scripts/` ディレクトリー内のインターフェースごとの設定ファイルに格納する必要がある
* ファイル名は、route-<ifname>
* このファイルへの記述は次の2つの形式が利用できる
  * IP コマンド引数形式を使用した静的ルート
  * ネットワーク/ネットマスクディレクティブの形式

```
# IPコマンド引数形式の例
default via 192.168.0.1 dev eth0
10.10.10.0/24 via 192.168.0.10 dev eth0
172.16.1.10/32 via 192.168.0.10 dev eth0

# ネットワークネットマスクディレクティブ形式の例
ADDRESS0=10.10.10.0
NETMASK0=255.255.255.0
GATEWAY0=192.168.0.10
ADDRESS1=172.16.1.10
NETMASK1=255.255.255.0
GATEWAY1=192.168.0.10
```

* 静的ルートを設定するには、以下の方法がある
  * nmcli コマンドを実行する
  * nmcli エディターを利用する

```
# nmcli コマンドで静的ルート設定を追加
nmcli connection modify eth0 +ipv4.routes "192.168.122.0/24 10.10.10.1"

# nmcli エディタ
nmcli connection edit type ethernet con-name eth0
nmcli> set ipv4.routes 192.168.122.0/24 10.10.10.1
```


### 非推奨: `/etc/sysconfig/network`

個別のデバイス設定を用いることを推奨、グローバルのnetworkファイルは非推奨になった。

> Red Hat Enterprise Linux ではグローバルの /etc/sysconfig/network ファイルの使用は非推奨となっており、ゲートウェイの指定はインターフェースごとの設定ファイルでのみ行なってください。
> https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_command_line_interface


## nmcli コマンドについて

公式ドキュメント: https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/7/html/networking_guide/sec-using_the_networkmanager_command_line_tool_nmcli

* nmcli は各種ネットワーク設定をコマンドラインから実行できるツール
* 各パラメータは全方一致で省略が可能

```
nmcli [OPTIONS...] {help | general | networking | radio | connection | device | agent | monitor} [COMMAND] [ARGUMENTS...]
```


### networking

ネットワークへの接続性を確認/制御できる

* `on`, `off`, `connectivity` の三種類

```
$ nmcli network connectivity
full
```

connectivityの状態は、none（切断）, portal（認証前）, limited（インターネットに出れない）, full（インターネットに出れる）, unknown（不明） の5種類


### general

* NetworkManager の状態や権限などを表示できる
* ホスト名, ログレベル, ドメインを取得できる

```
# Usage: nmcli general { COMMAND | help }
# COMMAND := { status | hostname | permissions | logging }

# hostname 表示/変更 => hostnamectl でも操作可能
$ nmcli general hostname
test01
$ nmcli general hostname prod01
$ nmcli general hostname
prod01
$ cat /etc/hostname
prod01

# status 表示
$ nmcli general status
STATE      CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN
connected  full          enabled  enabled  enabled  enabled
```


## ip コマンドについて

* RHEL7 系では route, ifconfig コマンド(net-tools)に代わり、ip コマンド(iproute2)の利用が推奨されている
* ip コマンドはルーティング、デバイスなどを表示・設定できるコマンド

```
SYNOPSIS
       ip [ OPTIONS ] OBJECT { COMMAND | help }

       ip [ -force ] -batch filename

       OBJECT := { link | address | addrlabel | route | rule | neigh | ntable | tunnel | tuntap | maddress | mroute | mrule | monitor | xfrm | netns | l2tp | tcp_metrics | token }

       OPTIONS := { -V[ersion] | -h[uman-readable] | -s[tatistics] | -d[etails] | -r[esolve] | -iec | -f[amily] { inet | inet6 | ipx | dnet | link } | -4 | -6 | -I | -D | -B | -0 | -l[oops] { maxi‐
               mum-addr-flush-attempts } | -o[neline] | -rc[vbuf] [size] | -t[imestamp] | -ts[hort] | -n[etns] name | -a[ll] }
```
