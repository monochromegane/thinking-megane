---
title: "Vagrantのネットワークが起動しないときは"
date: 2013-04-06
comments: true
tags: [ "vagrant" ]
---

Vagrantでネットワーク構成を変更した場合など、こういうエラーが出てネットワークが起動しないことがあります。

```
[default] Configuring and enabling network interfaces...
The following SSH command responded with a non-zero exit status.
Vagrant assumes that this means the command failed!

/sbin/ifup eth1 2> /dev/null
```

このエラーが出ると後続のマウント処理もキャンセルされて不便したので、対策をまとめておきます。

# 調査した環境

- Mac
- Vagrant 1.1.5
- VirtualBox
- CentOS 6.4

# ブリッジネットワークの場合

ブリッジネットワーク構成にした場合、2回目以降の起動時にエラーが発生します。

## 原因

Vagrantはブリッジ構成にした場合に以下の処理を行います。

- ネットワークインターフェース設定ファイルの作成(/etc/sysconfig/network-script/ifcfg-eth1)
- ネットワークインターフェースの起動(DHCP取得)

2回目以降の起動時、作成された設定ファイルをもとにOS側でネットワークインターフェースが起動(DHCP起動)、その後、さらにVagrant側での起動処理が走るためDHCP取得済みエラーとなってしまっています。

```
[vagrant@localhost ~]$ sudo ifup eth1

eth1 のIP情報を検出中...dhclient(1040) is already running - exiting.

This version of ISC DHCP is based on the release available
on ftp.isc.org.  Features have been added and other changes
have been made to the base software release in order to make
it work better with this distribution.

Please report for this software via the Red Hat Bugzilla site:
    http://bugzilla.redhat.com

exiting.
 失敗
```

## 対策

対策方法は2つです。

- Vagrant側でネットワークインターフェース起動時のエラーを無視する
- OS側でネットワークインターフェース(DHCP取得)を自動起動しない

どちらもVagrantのソースを修正して対応します。

### Vagrant側でネットワークインターフェース起動時のエラーを無視する

**/Applications/Vagrant/embedded/gems/gems/vagrant-1.1.5/plugins/guests/redhat/guest.rb**

以下の記述を

`vm.communicate.sudo("/sbin/ifup eth#{interface} 2> /dev/null")`

以下のように変更

`vm.communicate.sudo("/sbin/ifup eth#{interface} 2> /dev/null", :error_check => false)`

この場合、DHCP取得処理を2回試みることには変わりないため、もうひとつの対策のほうがいいかと思います。

### OS側でネットワークインターフェース(DHCP取得)を自動起動しない

**/Applications/Vagrant/embedded/gems/gems/vagrant-1.1.5/templates/guests/redhat/network_dhcp.erb**

以下の記述を

`ONBOOT=yes`

以下のように変更

`ONBOOT=no`

手動でOS側のネットワークインターフェース設定ファイル内の該当箇所の記載を変更しても起動のたびにこのテンプレートファイルで上書きされてしまうので注意してください。

- - -

# ブリッジ、もしくはホストオンリーネットワークでパッケージングした場合

ブリッジ、もしくはホストオンリーネットワーク構成で`vagrant package`した場合、作成したボックスで、ブリッジ、もしくはホストオンリーネットワークを利用する場合もやはりエラーが発生します。

## 原因

この現象はVagrantというよりも、仮想マシン作成時にMACアドレスが変更されることが原因となっています。

ブリッジ、もしくはホストオンリーネットワーク構成で起動すると、ネットワークインターフェース設定ファイルがVagrantにより作成されるのは前述しましたが、OS側ではネットワークインターフェースが追加された際に、ネットワークインターフェースとMACアドレスをマッピングしています。

```
[vagrant@localhost ~]$ cat /etc/udev/rules.d/70-persistent-net.rules
# This file was automatically generated by the /lib/udev/write_net_rules
# program, run by the persistent-net-generator.rules rules file.
#
# You can modify it, as long as you keep each rule on a single
# line, and change only the value of the NAME= key.

# PCI device 0x8086:0x100e (e1000)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:f7:f0:48", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

# PCI device 0x8086:0x100e (e1000)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:b8:8a:d9", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"
```

パッケージングしたボックスで新しく仮想マシンを起動すると別のMACアドレスが割り振られるため、OS側は新しいネットワークインターフェースeth2としてマッピングしてしまうようです。
にもかかわらず、ネットワークインターフェース設定ファイルはeth1用しか準備されておらず、eth1はOS側ではすでに存在しないMACアドレスのため、起動に失敗します。

```
[vagrant@localhost ~]$ sudo ifup eth1
デバイス eth1 は存在しないようですので、初期化を遅らせます。
```

## 対策

パッケージングする前のゲストOS側でMACアドレスとのマッピングを無効にしておきます。

`[vagrant@localhost ~]$ sudo ln -s -f /dev/null /etc/udev/rules.d/70-persistent-net.rules`

この対策後は、パッケージングしたボックスで新しい仮想マシンを作成してMACアドレスが変更されてもエラーにならず起動することができます。

開発環境などの用途であれば、上記の対策で十分と思われます。

機能を無効にしたくない場合は、該当ファイルを削除してネットワークを再起動することで再度マッピングされますので、用途に応じて対策を行なってみてください。

VagrantとChef-Soloを使って開発環境のローカル化を進めていますが、とても快適です。

今後は、Chef-SoloのCookbooksなども紹介していきたいなーと思ってます。



