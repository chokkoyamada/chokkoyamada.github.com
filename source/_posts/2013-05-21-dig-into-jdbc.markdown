---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(1)
date: 2013-05-21 21:40
comments: true
categories: java jdbc mysql tutorial
---

## Javaの記事はどれも古い！

Javaを仕事で書けるようになるためにあらためて勉強しているのですが、Javaの記事をぐぐってみると古い記事があたることが多いです。

Javaは後方互換性に優れているので10年前の記事でも余裕で参考になるのですが、それでも例えば2003年の記事を読んだだけだと現在の状況にどこまで適用してよい話なのか不安が残ります。

Oracleの公式ドキュメントをあたってみても、全てが日本語訳されているわけではなく、重い腰あげて読み込まないとなという気持ちがします。

いまの私の仕事は運用エンジニアということもあってJavaのアプリケーションコードを自分で書いてはいないのですが、重要な場面ではコードの細部まで追って対応できることが不可欠。その中でもデータベース周りはクリティカルに効いてくることが多いので、深く理解しておきたいと思っています。


## Java(Servlet + JDBC(Connector-J))は相当速い

最近、下記のウェブアプリケーションのベンチマークが話題になりました。

http://www.techempower.com/benchmarks/

これを見ると、JavaのServletは常に上位にいます。
私自身の最近の仕事の経験からしても、Servlet+JDBCの組み合わせはかなりのスループットが出ます。チューニングしていくたびにどんどん速くなってきたし、まだ速くする余地がかなりあるんじゃないかと思っています。

ベンチマークのソースを見たのですが、DataSourceのオプションなどはかなり参考になります。

https://github.com/TechEmpower/FrameworkBenchmarks/

Javaでウェブアプリケーションを書くことが常にベストの選択とは思いませんが、レスポンスタイムとスループットを安定して出したい場合の選択としてはかなりいい線行ってると思います。これをもっと極めていきたいです。

そこで、シリーズものとして、JavaのデータベースライブラリであるJDBCについてまとまったものを書いてみたいと思います。JDBCの基礎から始めて、1台あたり1000req/secを[50msec以内で安定して返し続けるサーバー](http://www.slideshare.net/Satully/jaws2013lt-1000qps)に必要なチューニングまで、順にステップアップしながら自分自身の勉強を兼ねて書いていきます。


## 環境は、CentOS6 - Java7 - tomcat7 - JDBC5.1 - MySQL5.5

前提となる環境ですが、まずサーバーはMac OS X 10.7のローカルで開発しつつ適宜[VagrantでCentoS 6.3](http://www.vagrantbox.es)を起動して使います。ある程度細かくベンチマークを取りたい場合はAWSで[Amazon Linux(最新版)](http://aws.amazon.com/jp/amazon-linux-ami/)を使います。

Javaはバージョン1.7で[Oracle公式のJDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)を使います。
アプリケーションサーバーは[tomcatのバージョン7](http://tomcat.apache.org/tomcat-7.0-doc/)、JDBCドライバは[mysql-connector-java](http://mvnrepository.com/artifact/mysql/mysql-connector-java)の5.1系です。
[MySQLは5.5](http://dev.mysql.com/doc/refman/5.5/en/index.html)です。

まずはOracle公式やその他の入門記事に沿って簡単なウェブアプリを作っていきながらひと通りのJDBCの機能を使っていきます。

[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)


