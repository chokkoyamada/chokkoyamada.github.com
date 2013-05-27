---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(4) ResultSetの特性
date: 2013-05-27 23:23
comments: true
categories: jdbc java mysql
---

## MySQL-Connector-J 5.1のResultSetの実装

前回SELECTクエリを発行したときに、ResultSetというオブジェクトが返ってくるのでクエリ結果はそこから取り出すということをやりました。

Oracle公式のドキュメントを読んでみるとResultSetは多くの機能をサポートしていますが、MySQLのconnector-Jで全てサポートしているかどうかがよく分からなかったので調べてみました。複数の機能の切り替えをサポートしている場合、切り替えることでパフォーマンスが改善する可能性が考えられるからです。

Retrieving and Modifying Values from Result Sets (The Java™ Tutorials > JDBC(TM) Database Access > JDBC Basics)
http://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

このドキュメントページによると、ResultSetが提供する特性(characteristics)には、Types, Concurrency, Holdabilityの3種類があります。

## ResultSet Types

カーソルがどのように操作されるかと、並行実行された変更が反映されるかどうかの違いによって3種類あるようです。[日本語訳を引用](http://docs.oracle.com/javase/jp/1.4/guide/jdbc/getstart/resultset.html)します。


     TYPE_FORWARD_ONLY
     結果セットは、スクロール不可能です。カーソルは上から下へ、順方向にのみ移動します。
     結果セット内のデータの見え方は、DBMS が結果をインクリメンタルに生成するかどうかに依存します。

     TYPE_SCROLL_INSENSITIVE
     結果セットはスクロール可能です。 カーソルは順方向および逆方向に移動可能で、移動先の行を直接指定することも、現在位置からの相対位置で指定することもできます。
     一般に、結果セットは、基となるデータベースが開いている間に行われた変更を表しません。 一般に、行のメンバシップ、順序、および列値は、結果セットの作成時に固定されます。

     TYPE_SCROLL_SENSITIVE
     結果セットはスクロール可能です。カーソルは順方向および逆方向に移動可能で、移動先の行を直接指定することも、現在位置からの相対位置で指定することもできます。
     結果セットは、開いている間に行われた変更を反映します。 基となる列値が変更されると、新たな値が反映されます。つまり、基となるデータの動的なビューを提供します。 結果セットに含まれる行とその順序は、実装によって固定されることも、されないこともあります。

Connector-Jの現行バージョンではどれなのか調べてみました。


``` java
DatabaseMetaData databaseMetaData = connection.getMetaData();
System.out.printf("[TYPE_FORWARD_ONLY]%s%n", databaseMetaData.supportsResultSetType(ResultSet.TYPE_FORWARD_ONLY));
System.out.printf("[TYPE_SCROLL_INSENSITIVE]%s%n", databaseMetaData.supportsResultSetType(ResultSet.TYPE_SCROLL_INSENSITIVE));
System.out.printf("[TYPE_SCROLL_SENSITIVE]%s%n", databaseMetaData.supportsResultSetType(ResultSet.TYPE_SCROLL_SENSITIVE));
```

結果
```
[TYPE_FORWARD_ONLY]false
[TYPE_SCROLL_INSENSITIVE]true
[TYPE_SCROLL_SENSITIVE]false
```

TYPE_SCROLL_INSENSITIVEのみがサポートされています。

## ResultSet Concurrency 並行処理のタイプ

次は結果セットを更新可能かどうかです。

     CONCUR_READ_ONLY
     プログラム的に更新不可能な結果セットを示します。
     JDBC 1.0 API だけを実装するドライバから利用可能な並行処理のタイプ
     最高レベルの並行処理を提供します (同時使用可能なユーザ数を最大にします)。 読み取り専用の並行処理が可能な ResultSet オブジェクトでロックを設定する場合には、読み取り専用ロックが使用されます。 これにより、ユーザはデータを読み取ることができますが、変更はできません。 データに対して同時に設定可能な読み取り専用ロックの数は無制限であるため、DBMS またはドライバが制限を課さない限り、同時使用するユーザ数には制限がありません。

     CONCUR_UPDATABLE
     プログラム的に更新可能な結果セットを示します。
     JDBC 2.0 コア API を実装するドライバから利用可能
     並行処理のレベルを低下させます。 一度にデータ項目にアクセス可能なユーザを 1 人だけに限定する場合、更新可能な結果セットは書き込み専用ロックを使用します。 これにより、2 人以上のユーザが同じデータを変更する可能性が排除されるため、データベースの一貫性が保証されます。 ただし、この一貫性の代償として、並行処理のレベルが低下します。

これもサポート状況を調べてみました。
```
System.out.printf("[CONCUR_READ_ONLY]%s%n", databaseMetaData.supportsResultSetConcurrency(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_READ_ONLY));
System.out.printf("[CONCUR_UPDATABLE]%s%n", databaseMetaData.supportsResultSetConcurrency(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE));
```

結果
```
[CONCUR_READ_ONLY]true
[CONCUR_UPDATABLE]true
```

両方サポートしているようです。デフォルトはCONCUR_READ_ONLYです。

## ResultSet Cursor Holdability

Cursor Holdabilityというのは、トランザクションをコミットしたときに通常はカーソルがクローズされるものを、保持しておくことを可能にするか否かのオプションのようです。
     HOLD_CURSORS_OVER_COMMIT
     現在のトランザクションがコミットされたときに、この保持機能を持つオープンしている ResultSet オブジェクトがクローズする。

     CLOSE_CURSORS_AT_COMMIT
     現在のトランザクションがコミットされたときに、この保持機能を持つオープンしている ResultSet オブジェクトがオープンしたままになる。

これも同様にサポート状況を調べてみました。

```
DatabaseMetaData dbMetaData = connection.getMetaData();
System.out.println("ResultSet.HOLD_CURSORS_OVER_COMMIT = " +
          ResultSet.HOLD_CURSORS_OVER_COMMIT);

System.out.println("ResultSet.CLOSE_CURSORS_AT_COMMIT = " +
          ResultSet.CLOSE_CURSORS_AT_COMMIT);

System.out.println("Default cursor holdability: " +
          dbMetaData.getResultSetHoldability());

System.out.println("Supports HOLD_CURSORS_OVER_COMMIT? " +
          dbMetaData.supportsResultSetHoldability(
                    ResultSet.HOLD_CURSORS_OVER_COMMIT));

System.out.println("Supports CLOSE_CURSORS_AT_COMMIT? " +
          dbMetaData.supportsResultSetHoldability(
                    ResultSet.CLOSE_CURSORS_AT_COMMIT));
```

結果
``` java
ResultSet.HOLD_CURSORS_OVER_COMMIT = 1
ResultSet.CLOSE_CURSORS_AT_COMMIT = 2
Default cursor holdability: 1
Supports HOLD_CURSORS_OVER_COMMIT? true
Supports CLOSE_CURSORS_AT_COMMIT? false
```

通常、コミットされたらカーソルをクローズしてくれたほうがいいように直感的には思いますが、

     Holdable cursors might be ideal if your application uses mostly read-only ResultSet objects.

とのことなので、トランザクションの実行に関わらず読み取りを高速に行いたい場合が考慮されているのかもしれません。そもそもカーソルのクローズは自動的にされるか、明示的に呼ぶと思いますので、よほど長い処理でない限りはどちらでも大差ない気はします。

基本的にはデフォルトのまま使うので問題ないように思いますが、このへんはMySQLの設定のほうにあるトランザクションの分離レベルとも関わってくると思うのですが、今回調べただけだとどう影響しあうかがよく分かりませんでした。

次回ももう少しこのへんを深掘りしてみます。

今回のソースはこちら。
https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter4

[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)
