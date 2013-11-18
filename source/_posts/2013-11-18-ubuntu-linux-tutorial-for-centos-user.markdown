---
layout: post
title: CentOSユーザーのためのUbuntu Linux入門(1)
date: 2013-11-18 20:12
comments: true
categories: ubuntu, centos, linux, tutorial

---

## Ubuntuを使うシチュエーション

Ubuntuに入門したいCentOSユーザーというのは私自身のことです。

最近、Ubuntu Linuxを使いこなせるようになりたいと思う場面が多いです。
普段はCentOSまたはAmazon Linuxを使っていて、Debian系のUbuntuはおそらく全く使えないということはないのでしょうが、なんとなくハードルを感じてしまうところがあります。

前職では最初Ubuntuを使っているシステムがいくつかあったので、厳密にいうと私が最初にログインしてちょっと触ったLinuxはUbuntuでした。
ただそのときは実質的に何もいじらなかった(ログインしたことあるだけ)ので、ソーシャルゲームを開発することになって、CentOSを本気で触り始めたのが最初です。
それ以来、CentOSばかり使っています。

CentOSを勉強し始めたときは、自分でキューブ型PCを買ってきて、自分の部屋においてセットアップしました。
確かCentOSのイメージを落としてきてDVD-Rか何かに焼いて、それを使ってセットアップした記憶があります。

今Linuxのサーバーを新しく作ろうと思った場合、VirtualBox(Vagrant)やDockerがあるから楽です。
そもそもVagrantやDockerをより深く知りたいがためにUbuntuをやろうと思ったという動機もあります。
chefも（最近少なくなってきましたが）Ubuntuのほうがやりやすい部分があったり、Tizenも現状ではUbuntu前提ですよね。

というわけで、Ubuntuも抵抗感なく普通に使えるほうがよさげなのできちんとさわってみることにしました。　

## バージョンの違いなど

CentOSだと、いま選ぶのはだいたい5か6系だと思います。
6.4が最新。5.10もStableバージョンで[2017年までメンテナンスされる予定](http://wiki.centos.org/Download)ですが、今なら6.4を選んで問題はないでしょう。

Ubuntuのほうですが、[このへんを見るに](http://www.ubuntulinux.jp/ubuntu)、12か13系あたりが選択肢になりそうです。
13系はまだ安定バージョンが無いようなので、2017年までサポート予定の12.04(コードネーム：Precise Pangolin)を選んでみることにします。

というわけで、それ用のvagrant boxを追加します。(http://www.vagrantbox.es より)

```
$ vagrant box add ubuntu12 http://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-amd64-vagrant-disk1.box
```

CentOSでは32bitのものをi386、64bitのものをx86_64と呼んでいましたが、Ubuntuでは64bitのほうをamd64と呼ぶようですね。　

vagrantでとりあえず起動してみます。

```
Welcome to Ubuntu 12.04.3 LTS (GNU/Linux 3.2.0-56-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  System information as of Mon Nov 18 11:44:43 UTC 2013

  System load:  0.17              Processes:           69
  Usage of /:   2.7% of 39.37GB   Users logged in:     0
  Memory usage: 30%               IP address for eth0: 10.0.2.15
  Swap usage:   0%                IP address for eth1: 192.168.33.11

  Graph this data and manage this system at https://landscape.canonical.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

  Use Juju to deploy your cloud instances and workloads:
    https://juju.ubuntu.com/#cloud-precise

0 packages can be updated.
0 updates are security updates.

_____________________________________________________________________
WARNING! Your environment specifies an invalid locale.
 This can affect your user experience significantly, including the
 ability to manage packages. You may install the locales by running:

   sudo apt-get install language-pack-ja
     or
   sudo locale-gen ja_JP.UTF-8

To see all available language packs, run:
   apt-cache search "^language-pack-[a-z][a-z]$"
To disable this message for all users, run:
   sudo touch /var/lib/cloud/instance/locale-check.skip
_____________________________________________________________________

vagrant@vagrant-ubuntu-precise-64:~$

```

いきなり親切なメッセージがいろいろ出てきました。

何かパッケージ入れてみようと思い、apacheはCentOSだとhttpdですがUbuntuはapache2というくらいは知っていたのですがあえてapt-get install httpdとやってみると、

```
root@vagrant-ubuntu-precise-64:~# apt-get install httpd
Reading package lists... Done
Building dependency tree
Reading state information... Done
Package httpd is a virtual package provided by:
  nginx-naxsi 1.1.19-1ubuntu0.4
  nginx-light 1.1.19-1ubuntu0.4
  nginx-full 1.1.19-1ubuntu0.4
  nginx-extras 1.1.19-1ubuntu0.4
  apache2-mpm-itk 2.2.22-1ubuntu1.4
  apache2-mpm-worker 2.2.22-1ubuntu1.4
  apache2-mpm-prefork 2.2.22-1ubuntu1.4
  apache2-mpm-event 2.2.22-1ubuntu1.4
  yaws 1.92-1
  webfs 1.21+ds1-8
  tntnet 2.0+dfsg1-2
  ocsigen 1.3.4-2build4
  monkey 0.9.3-1ubuntu1
  mini-httpd 1.19-9.2build1
  micro-httpd 20051212-13
  mathopd 1.5p6-1.1
  lighttpd 1.4.28-2ubuntu4
  ebhttpd 1:1.0.dfsg.1-4.2build1
  cherokee 1.2.101-1
  bozohttpd 20100920-1
  boa 0.94.14rc21-3.1
  aolserver4-daemon 4.5.1-15
  aolserver4-core 4.5.1-15
You should explicitly select one to install.

E: Package 'httpd' has no installation candidate
```

けっこう気が利く感じです。

apache2を入れてみます。

```
root@vagrant-ubuntu-precise-64:~# apt-get install apache2
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  apache2-mpm-worker apache2-utils apache2.2-bin apache2.2-common libapr1
  libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap ssl-cert
Suggested packages:
  apache2-doc apache2-suexec apache2-suexec-custom openssl-blacklist
The following NEW packages will be installed:
  apache2 apache2-mpm-worker apache2-utils apache2.2-bin apache2.2-common
  libapr1 libaprutil1 libaprutil1-dbd-sqlite3 libaprutil1-ldap ssl-cert
0 upgraded, 10 newly installed, 0 to remove and 0 not upgraded.
Need to get 1,855 kB of archives.
After this operation, 5,681 kB of additional disk space will be used.
Do you want to continue [Y/n]? y
Get:1 http://archive.ubuntu.com/ubuntu/ precise/main libapr1 amd64 1.4.6-1 [89.6 kB]
Get:2 http://archive.ubuntu.com/ubuntu/ precise/main libaprutil1 amd64 1.3.12+dfsg-3 [74.6 kB]
Get:3 http://archive.ubuntu.com/ubuntu/ precise/main libaprutil1-dbd-sqlite3 amd64 1.3.12+dfsg-3 [10.4 kB]
Get:4 http://archive.ubuntu.com/ubuntu/ precise/main libaprutil1-ldap amd64 1.3.12+dfsg-3 [8,044 B]
Get:5 http://archive.ubuntu.com/ubuntu/ precise-updates/main apache2.2-bin amd64 2.2.22-1ubuntu1.4 [1,340 kB]
Get:6 http://archive.ubuntu.com/ubuntu/ precise-updates/main apache2-utils amd64 2.2.22-1ubuntu1.4 [90.1 kB]
Get:7 http://archive.ubuntu.com/ubuntu/ precise-updates/main apache2.2-common amd64 2.2.22-1ubuntu1.4 [226 kB]
Get:8 http://archive.ubuntu.com/ubuntu/ precise-updates/main apache2-mpm-worker amd64 2.2.22-1ubuntu1.4 [2,284 B]
Get:9 http://archive.ubuntu.com/ubuntu/ precise-updates/main apache2 amd64 2.2.22-1ubuntu1.4 [1,492 B]
Get:10 http://archive.ubuntu.com/ubuntu/ precise-updates/main ssl-cert all 1.0.28ubuntu0.1 [12.3 kB]
Fetched 1,855 kB in 7s (236 kB/s)
Preconfiguring packages ...
Selecting previously unselected package libapr1.
(Reading database ... 61149 files and directories currently installed.)
Unpacking libapr1 (from .../libapr1_1.4.6-1_amd64.deb) ...
Selecting previously unselected package libaprutil1.
Unpacking libaprutil1 (from .../libaprutil1_1.3.12+dfsg-3_amd64.deb) ...
Selecting previously unselected package libaprutil1-dbd-sqlite3.
Unpacking libaprutil1-dbd-sqlite3 (from .../libaprutil1-dbd-sqlite3_1.3.12+dfsg-3_amd64.deb) ...
Selecting previously unselected package libaprutil1-ldap.
Unpacking libaprutil1-ldap (from .../libaprutil1-ldap_1.3.12+dfsg-3_amd64.deb) ...
Selecting previously unselected package apache2.2-bin.
Unpacking apache2.2-bin (from .../apache2.2-bin_2.2.22-1ubuntu1.4_amd64.deb) ...
Selecting previously unselected package apache2-utils.
Unpacking apache2-utils (from .../apache2-utils_2.2.22-1ubuntu1.4_amd64.deb) ...
Selecting previously unselected package apache2.2-common.
Unpacking apache2.2-common (from .../apache2.2-common_2.2.22-1ubuntu1.4_amd64.deb) ...
Selecting previously unselected package apache2-mpm-worker.
Unpacking apache2-mpm-worker (from .../apache2-mpm-worker_2.2.22-1ubuntu1.4_amd64.deb) ...
Selecting previously unselected package apache2.
Unpacking apache2 (from .../apache2_2.2.22-1ubuntu1.4_amd64.deb) ...
Selecting previously unselected package ssl-cert.
Unpacking ssl-cert (from .../ssl-cert_1.0.28ubuntu0.1_all.deb) ...
Processing triggers for man-db ...
Processing triggers for ureadahead ...
Processing triggers for ufw ...
Setting up libapr1 (1.4.6-1) ...
Setting up libaprutil1 (1.3.12+dfsg-3) ...
Setting up libaprutil1-dbd-sqlite3 (1.3.12+dfsg-3) ...
Setting up libaprutil1-ldap (1.3.12+dfsg-3) ...
Setting up apache2.2-bin (2.2.22-1ubuntu1.4) ...
Setting up apache2-utils (2.2.22-1ubuntu1.4) ...
Setting up apache2.2-common (2.2.22-1ubuntu1.4) ...
Enabling site default.
Enabling module alias.
Enabling module autoindex.
Enabling module dir.
Enabling module env.
Enabling module mime.
Enabling module negotiation.
Enabling module setenvif.
Enabling module status.
Enabling module auth_basic.
Enabling module deflate.
Enabling module authz_default.
Enabling module authz_user.
Enabling module authz_groupfile.
Enabling module authn_file.
Enabling module authz_host.
Enabling module reqtimeout.
Setting up apache2-mpm-worker (2.2.22-1ubuntu1.4) ...
 * Starting web server apache2                                                                                              apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName
                                                                                                                     [ OK ]
Setting up apache2 (2.2.22-1ubuntu1.4) ...
Setting up ssl-cert (1.0.28ubuntu0.1) ...
Processing triggers for libc-bin ...
ldconfig deferred processing now taking place

```

どうやらworkerのmpmだけがインストールされ、さらにapacheが勝手に起動しました(；´Д｀)
パッケージ入れただけなのに。。。

```
root@vagrant-ubuntu-precise-64:~# /etc/init.d/apache2 status
Apache2 is running (pid 2595).
```

いきなり勝手が違っててもやもやしてきたわけですが、続く。