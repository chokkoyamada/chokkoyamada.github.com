---

layout: post
title: Web Api Hackathon Vol.3 というイベントで「Backlog Kanban」というウェブアプリを作成しました
date: 2013-11-18 16:23
comments: true
categories: hackathon backlog kanban rails ruby

---

## Web Api Hackathon Vol.3

2013/11/17に行われた、[Web Api Hackathon](http://connpass.com/series/533/)に行ってきました。私は初参加でしたが、もう3回目で、とりあえずWeb Api叩いて何か作ろうというざっくりしたテーマで行うハッカソン形式のイベントです。
[Mashup Award](http://ma9.mashupaward.jp/)のコンセプトにも近いですね。

場所はお台場にある[Membersさんのオフィス](http://www.members.co.jp/company/access.html)のラウンジ。机やお座敷やソファのあるかなり大きなスペースで、中央には大きなプロジェクタがあって、裏の隠れたところに昼寝スペースという、個人的にはかなりお気に入りのスペースでした。
リラックマの超大きなぬいぐるみがあったのも印象的でした(写真撮影NGだったのが残念・・・)。

## Backlog Kanban - カンバン形式でBacklogのプロジェクトを見る

[Backlog](http://www.backlog.jp) というプロジェクト管理ツールがあって、仕事でも利用しています。
1プロジェクト、10人までは無料で使えるので、個人プロジェクトにも利用しています。
このウェブサービスは、ポップなデザインと使いやすいインタフェース、メールに直接返信で課題が更新される、などなどでとても良いサービスなのですが、一点不満があって、それはプロジェクトの概観をしづらいということでした。

なのでこれまではBacklogとは別に、ホワイトボードにポストイットを貼って管理していました。（[こういうイメージ](http://www.infoq.com/jp/articles/agile-kanban-boards) ）
ただ、これをBacklogで直接やれたらいいのにとつねづね思っていました。

そこで、それを今回のハッカソンで作ってみようと思いたち、そうしてできたのがこちらです。

### Backlog Kanban(デモサイト)
[http://kanban.kirishikistudios.com/](http://kanban.kirishikistudios.com/)

### ソース(GitHub)
https://github.com/backlog-kanban/backlog-kanban

![backlog-kanban](/images/2013-11-17-backlog-kanban.png)

できたというか、まだ全くの開発途中ですが、一応カンバン形式で閲覧するツールとしては動作しています。

[Backlog API](http://www.backlog.jp/api/) を使って課題の一覧を取得し、新しいUIに落としただけ、という簡単なものですが、「これがやりたかったんだよ！！」というのができた気がします。

「スケジュールが過ぎたものは赤色で表示される」「課題の種別によって色分け」「課題をクリックするとBacklog本体の課題のぺーじに飛ぶ」などの機能があります。もちろん、Backlog本体側を更新すればその内容はこちらに即座に反映されます。

使った技術はこちら。

* Amazon EC2 (Amazon Linux, micro instance)
* Ruby 2.0
* Ruby on Rails 4.0.1
* rbenv
* apache + passenger

プレゼン資料はこちら。

<iframe src="http://www.slideshare.net/slideshow/embed_code/28331762" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC;border-width:1px 1px 0;margin-bottom:5px" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/mima_v/20131117-backlog-kanban" title="20131117 backlog kanban【第３回Web API ハッカソン】" target="_blank">20131117 backlog kanban【第３回Web API ハッカソン】</a> </strong> from <strong><a href="http://www.slideshare.net/mima_v" target="_blank">Marie Suenaga</a></strong> </div>

今後は、

* Backlogアカウントによるログインと閲覧の認証
* 課題の追加・編集機能
* カテゴリや種別の追加・変更への対応

などができるとよいと思います。

## デザイン・フロント・サーバーの3人で開発してみて

ハッカソンで、初対面の人たち同士でアプリを作るということをこれまで何回かやってきましたが、本当に難しいと思います。
各メンバーのスキルセットを理解して、何のツールなら熟知しているのか、どこの部分で境界線を引いて分担するのが良いのか、など。
そもそも作りたいものが一致しているのかもすり合わせないといけません。

今回チームを作ったのは3人ですが、みな初対面。
その場で自己紹介をして、基本的な開発手法について議論と相談をして、どう作業するかを決めていきました。

役割分担はこんな感じ。

* [@mima-v](https://github.com/mima-v) 全体デザイン、画像制作
* [@oogushitakuya](https://github.com/oogushitakuya) フロント部分(HTML,JavaScriptのコーディング)
* [@chokkoyamada](https://github.com/chokkoyamada) サーバーサイド(Railsを使ったBacklogAPIの操作、公開用サーバー設定)

アイデア段階から実際のアプリ作成まで8時間弱で、何かできるか不安でしたが、各メンバーがGitHubを使ったワークフローをわかっていたおかげでスムーズに作業が進んだと思います。

## 作ったアプリを公開する場所があったらいい

Railsに限らずですが、何かアプリを作ったときにそれをグローバルに無料で公開できる場所が少ないなと思いました。
Railsだと有名なのはHerokuですが、PostgreSQLなのがネックだし、他にも選択肢があると良いと思います。

今回は結局AWSでMicroインスタンスを立ててそこにapache入れてpassenger入れて・・・とやったのですが、Microインスタンスにしてもこの１アプリだけを置いておくにはもったいないです。
今後また別のRailsアプリを作ったらこのインスタンスにデプロイしていきたいと思います。

### 参考リンク

WebAPI Hackathon | デザイナー/コーダーとエンジニアが集まってWeb APIを使ってなにかを作る会  http://webapi-hackathon.info/

Web Api Hackathon Vol.3 - connpass  http://connpass.com/event/3890/

