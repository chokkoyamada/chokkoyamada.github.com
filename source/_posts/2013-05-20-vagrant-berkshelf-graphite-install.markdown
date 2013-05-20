---
layout: post
title: BerkshelfとVagrantの連携(2) 
date: 2013-05-20 22:02
comments: true
categories: chef vagrant berkshelf graphite
---

## Berkshelf+Vagrantで簡単にGraphiteをインストール

昨日はBerkshelfの導入をやりました。とりあえず簡単なレシピがインストールできることは分かったので、今回は実用的なところで、インストールしてみたかったけど一見面倒そうで敬遠していた、Graphiteをchefのcookbookを使ってインストールしてみます。

* Graphite

http://graphite.wikidot.com

http://graphite.readthedocs.org/en/0.9.10/

Graphiteはデータのリアルタイム集積・グラフ可視化ツールです。フロントのGUIはDjangoで、そこでデータを集計加工してみたりグラフを描画したりができます。バックエンドはcarbonというデーモンで、ここに向かってネットワーク経由でデータを送信します。・・・というところまでがドキュメントを読んだ限りの概要で、まだ使ったことはありません。とりあえずインストールします。


## GitHub上のcookbookをFork

ウェブ上にGraphiteをインストールするレシピが無いかと探してみました。Ubuntu向けのものは公式であるのですが、CentOS/Amazon Linuxで使えるものではなかったので、GitHubを探したところ、ありました。

https://github.com/taos/graphite-chef-centos

しかし、このcookbookのままだとepelのrpmが404になってしまって動作しませんでした。
そこで自分のところにforkして、修正して使います。

https://github.com/chokkoyamada/graphite-chef-centos

修正したのはepelのRPMのURLだけで動きました。


## metadataのバージョン番号がポイント


Berksfileを次のように書きます。

```
cookbook 'graphite-chef-centos', git: 'git://github.com/chokkoyamada/graphite-chef-centos.git'
```

ここではまりどころだったのは、その前にオリジナルの作者であるtaos氏のレポジトリのURLを指定していたのを自分のレポジトリのURLに書き換えたのですが、その後vagrant up/reloadしてもキャッシュが効いていたのか、cookbookがうまく更新できなくて悩みました。

metadataがポイントなんですね。[バージョン番号を更新](https://github.com/chokkoyamada/graphite-chef-centos/blob/master/metadata.rb#L5) したら新しいものが入りました。gemと同じ要領ですね。

今回のVagrantfileはこちらです。ローカル(Mac)からウェブを見るためにポートフォワーディングを設定している以外は前回と同じです。

``` ruby
Vagrant.configure("2") do |config|
  config.berkshelf.enabled = true
  config.berkshelf.berksfile_path = "Berksfile"
  config.vm.box = "centos63chef"

  config.vm.network :forwarded_port, guest: 80, host: 8080

  config.vm.synced_folder "data", "/vagrant_data"

  config.vm.provision :chef_solo do |chef|
    chef.add_recipe "graphite-chef-centos"
  end
end
```

あとはvagrantを走らせるだけ。

```
vagrant up
```

```
vagrant ssh
sudo service iptables stop
```
でiptablesを止めてから、[http://localhost:8080/] にアクセスします。

{% img /images/2013-05-20-graphite.png %}

Graphiteがこんなに簡単にインストールできました! コマンド1個1個打っていくよりはるかに楽です。

[Pull Request](https://github.com/taos/graphite-chef-centos/pull/1)もしておきました。

次はGraphiteを使い倒していきます。


