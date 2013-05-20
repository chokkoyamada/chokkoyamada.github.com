---
layout: post
title: "BerkshelfとVagrantの連携(1)"
date: 2013-05-19 23:14
comments: true
categories: berkshelf vagrant chef
---

## 他所にあるchefのcookbookを気軽にインポートしたい
chefのcookbookを依存管理してくれるBerkshelfと、Mac上のVM実行補助のライブラリであるVagrantとの連携を試してみます。

私はAWSでchef-soloを使ったインフラの管理をしているのですが、開発の全てをEC2上でやろうとするとどうしてもインスタンスの起動・削除にお金がかさんでしまうので、開発はローカル環境で進めたいと思っています。

AWS上にproduction環境を置く場合、どうしてもAWS上でのテスト環境やステージング環境のようなものは必要になってきますが、その前段階の開発作業はローカルのLinux VM上でできないかと試行錯誤してきました。

その解決策として使ってきたのがVagrantです。VagrantはVirtual Boxをコマンドライン＋設定ファイルで扱えるようにしたツールで、chefやpuppetとの連携機能も備えているため、chefのcookbookのテストを気軽に試すことができます。Vagrantは起動も速いので、開発とテストにもってこいです。

さらに最近やりたいと感じていたこととして、「GitHub上にある他ユーザーのcookbookをインポートして気軽に使いたい」というのがありました。

例えばGraphiteやZabbixやHadoopなどといったような、セットアップが多少面倒くさいツールをセットアップしたいときに、他のユーザーのcookbookを使ってさくっとprovisionできたら非常に楽だと思ったのです。cookbookにはセットアップ手順が整理されて書いてあるので、必要なものを素早く把握することもでき一石二鳥です。

そのニーズを満たしてくれるライブラリとして、Berkshelfというのが便利、と同僚の神から教えてもらったので試してみました。今回は必要最低限の設定を行います。

## VagrantとBerkshelfのセットアップ

BerkshelfとVagrantの連携機能を使うためには公式サイトにある最新版を使う必要がありました。現時点で私が設定した環境のバージョンは次の通りです。

```
ᐅ ruby -v
ruby 1.9.3p194 (2012-04-20 revision 35410) [x86_64-darwin11.4.0]

ᐅ vagrant --version
Vagrant version 1.2.2

ᐅ berks --version
Berkshelf (1.4.4)
```

RubyはRVMを使っています。
Vagrantはgemだと古いバージョンだったので、公式サイトのdmgファイルからインストールしました。Berkshelfは`gem install berkshelf`でインストールしました。

Vagrantのboxは[CentOS 6.3 x86_64 + Chef 10.14.2 + VirtualBox 4.1.22 (with guest additions)](http://www.vagrantbox.es)を使いました。

まずBerksfileを設定します。[Berkshelfの公式サイトの例](http://berkshelf.com)にあるサンプルを試してみます。

``` ruby 
site :opscode

cookbook 'mysql'
cookbook 'nginx', '~> 0.101.5'
```

この場合、[MySQLはこれ](https://github.com/opscode-cookbooks/mysql)と[Nginxはこれ](https://github.com/opscode-cookbooks/nginx)が使われるみたいです。

次にVagrantfileを書きます。こちらは`vagrant init centos63chef`で自動生成してそこからカスタマイズしていきます。

``` ruby
Vagrant.configure("2") do |config|
  config.vm.box = "centos63chef"  
  config.vm.synced_folder "data", "/vagrant_data"

  config.berkshelf.enabled = true 
  config.berkshelf.berksfile_path = "Berksfile"

  config.vm.provision :chef_solo do |chef|
     chef.add_recipe "mysql"
     chef.add_recipe "nginx"
  end
end
```

これで動きました。

あとは`vagrnat up`でVMの作成とプロビジョンが行われます。今回は、mysqlクライアントとnginxパッケージとデーモンがインストールされたはずです。

##serverspecで確認

serverspecを使ってテストしてみます。serverspecは今回の主題ではないので詳細はすっ飛ばしますがmysqlとnginxがvagrantサーバーにきちんと入っているかどうかを調べています。

``` ruby spec/vagrant/nginx_spec.rb
require 'spec_helper'          
                               
describe package('nginx') do   
  it { should be_installed }
end

describe service('nginx') do   
  it { should be_running }     
end
  
describe port(80) do
  it { should be_listening }
end

describe file('/etc/nginx/nginx.conf') do
  it { should be_file }
  it { should contain "pid" }
end

```

``` ruby spec/vagrant/mysql_spec.rb
require 'spec_helper'          
                               
describe package('mysql') do   
  it { should be_installed }
end
```

```
ᐅ rake spec
/Users/xxxx/.rvm/rubies/ruby-1.9.3-p194/bin/ruby -S rspec spec/vagrant/mysql_spec.rb spec/vagrant/nginx_spec.rb
......

Finished in 0.18744 seconds
6 examples, 0 failures
```

serverspecのテストが通りました。

これで外部のcookbookを使う準備ができました。次回は他のもうちょっと複雑なcookbookを試したり、パラメータを渡したりといったことが問題なくできるかどうかをやっていきます。　


