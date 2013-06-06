---
layout: post
title: "aws-summit-2013-freakout"
date: 2013-06-06 15:19
comments: true
categories: 
---

クラウドでRTB

FreakOUt

2010/10創業

4billion+imp
60+staff

## FreakOutとRTB

FreakOutは広告主のネット広告におけるROI最適化を目的としたDSPを展開

このページのオーディエンスはどんな人
興味関心
生活主観
過去の行動

RTBによって広告主の都合で買い付け可能に

960億/月のRTBreqを受けている　


## FreakoutとAWS

100msecで広告を表示する
100msecでSSPがリクエストを投げ、DSPがレスポンスを返しきるまで
TCPのRTTを含んでいる
SSPは複数あり、多くの同時多発的リクエストが発生する

### 課題

レイテンシの軽減
多数のRTB処理
落札後の画像&Flash配信

RTBはCPUバウンド

自作PCを使っている 数百台規模で使っている　
多コアを安く並べる

その一方でコンテンツ配信には広い帯域が必要

S3+CloudFrontを使用
コストパフォーマンスが良い

オンプレミス＋クラウドで費用対効果を追求
すべてをクラウドに任せるのではなく、徹底して費用対効果を勘案したうえで技術選択を行っている

## 海外での展開とAWS

レイテンシ
singapore 80msec
us-west 110msec
us-east 160msec

海外でも物理的に近い必要がある

VPCとVPC connsを使って拡張

EC2 専用のAMIを作成して活用
ELB LVSの代わりとして活用
Elasticache 当時VPCで使用不能だった
RDS　Slaveでバッチを稼働させるため不使用　
ELB以外はEC2で構築

構築初期　
c1.xlarge
app
m2.2xlarge
kvs
m1.medium
other

ログ処理関連
S3 海外でputして日本でget
オンプレミスのhadoopへ

まとめ
自分たちの都合で必要なことを判断して使う
いい意味でドライなので線引きをエンジニア自身で考える
AWS以上に都合のよいクラウドはいまのところない

## 改善点・課題

Reserved Capacity
10tboverなら配信料ダウン
Region単位契約なので注意

HTML/JS
cookieをさわりたいので、ドメイン指定する必要がある
カスタムオリジンをhttps対応してくれると助かる

cc2.8xlargeによるインスタンス集約
c1.xlarge x 4以上

Elasticacheを今後使っていきたい

EC2はインスタンスを増やすのが簡単だが増やしすぎると運用負担増
1台あたりのパフォーマンス改善が重要　


## おわりに

* FreakOutは必要な技術を都度精査して選択する
* 結果責任は技術選択をする事業者自身にある
* AWSは有力な選択肢として魅力をもっていると考えており、今この瞬間もAWSを利用して広告を配信している

