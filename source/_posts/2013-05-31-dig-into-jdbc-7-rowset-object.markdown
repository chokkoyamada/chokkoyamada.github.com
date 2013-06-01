---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(7) RowSet Objects
date: 2013-05-31 08:42
comments: true
categories: java jdbc mysql rowset
---

## RowSetはResultSetの拡張

RowSet ObjectはResultSetを拡張したクラスです。ResultSetがデータベースの実装をそのまま引き継ぐタイプの実装であるのに対し、RowSetはアプリケーション側で扱いやすいように機能が追加されています。

RowSetはJavaBeansでもあります。JavaBeansとは、下記のような特徴を持ちます。

* 引数の無いコンストラクタで直接インスタンス化できる
* getter/setterがある
* Serializableである

## RowSet Objectの種類

RowSetには主に下記の5種類のインタフェースがあります。

[接続維持型]

* JDBCRowSet

接続型のRowSet

[接続非維持型]

* CachedRowSet 

データをDBから取得後、オブジェクト内にキャッシュし、接続を切断する。

* WebRowSet

CachedRowSetにXMLの拡張がついている

* JoinRowSet

WebRowSetの拡張で、別テーブル同士のJOINができる

* FilteredRowSet

WebRowSetの拡張で、データのフィルタリング機能を持つ

この中で、使いそうだと感じたCachedRowSetとJoinRowSetを試してみます。

Using CachedRowSetObjects (The Java™ Tutorials > JDBC(TM) Database Access > JDBC Basics)
http://docs.oracle.com/javase/tutorial/jdbc/basics/cachedrowset.html

Using JoinRowSet Objects (The Java™ Tutorials > JDBC(TM) Database Access > JDBC Basics)
http://docs.oracle.com/javase/tutorial/jdbc/basics/joinrowset.html

## CachedRowSet: 更新時にコンフリクトした場合の解決, 他のオブジェクトへのメッセージ機能

1つ1つの処理が短く、長くても数百ミリ秒でレスポンスを返すウェブだとあまり恩恵がないかもしれないのですが、CachedObjectはDBからデータを取得した後、即座に接続を切断(実際にはコネクションプールに戻す)しても、オブジェクト自体はデータを保持しています。

ResultSetはConnectionから用意したStatementオブジェクトから返されるものなので、DBへの接続を保持している必要があったので、コネクションプールが枯渇しやすい状況においてはメリットになるかもしれません。

``` java
cachedRowSet = new CachedRowSetImpl();
cachedRowSet.setCommand("SELECT * FROM COFFEES");
cachedRowSet.execute(connection);
```

接続の取得方法はいくつかありますが、今回はexecuteのときにconnectionを渡すようにしています。
下記のようにDataSouceを渡す方法も使えます。

``` java
cachedRowSet = new CachedRowSetImpl();
cachedRowSet.setDataSouceName("java:comp/env/jdbc/continuousops");
cachedRowSet.setCommand("SELECT * FROM COFFEES");
cachedRowSet.execute();
```

他のオブジェクトに通知をしたい場合、addRowSetListener()で指定できます。
RowSetListenerは下記の3つのイベントをサポートしています。

* rowSetChanged
* rowChanged
* cursorMoved

``` java
cachedRowSet.addRowSetListener(new ExampleRowSetListener());
```

CachedRowSetはオブジェクトにキャッシュしているデータなので、データを更新してコミットしようとした際に、元のデータとズレが生じる可能性があります。CachedRowSetはそれが発生した際に例外を投げる仕組みが用意されているので、それを試してみました。

``` java
//getting first cachedRowSet
cachedRowSet = new CachedRowSetImpl();
cachedRowSet.setCommand("SELECT * FROM COFFEES");
cachedRowSet.execute(connection);

//getting second cachedRowSet
cachedRowSet2 = new CachedRowSetImpl();
cachedRowSet2.setCommand("SELECT * FROM COFFEES");
cachedRowSet2.execute(connection);

cachedRowSet.next();
cachedRowSet.updateInt("SUP_ID", 101);
cachedRowSet.updateRow();
cachedRowSet.acceptChanges();

cachedRowSet2.next();
cachedRowSet2.updateInt("SUP_ID", 150);
cachedRowSet2.updateRow();
cachedRowSet2.acceptChanges();
```

cachedRowSetのインスタンスを2つ用意し、データをそれぞれ取得した後データを更新かけてみます。こうすると、1つめのcachedRowSetを更新した後、2つめのcachedRowSetはオブジェクト内のデータとDBのレコードにずれが出るので、一番最後の文である`cachedRowSet2.acceptChanges();`を実行したときに例外が投げられます。

それが、`SyncProviderException`です。これをcatchしてコンフリクトを解決させようとしました。下記はほとんどOracleのサンプルコードのままです。

``` java
} catch (SyncProviderException spe){
    System.out.println("conflict detected.");

    SyncResolver resolver = spe.getSyncResolver();

    Object crsValue;
    Object resolverValue;
    Object resolvedValue;

    try {
        while (resolver.nextConflict()) {

            if (resolver.getStatus() == SyncResolver.UPDATE_ROW_CONFLICT) {
                int row = resolver.getRow();
                cachedRowSet2.absolute(row);

                int colCount = cachedRowSet2.getMetaData().getColumnCount();
                for (int j = 1; j <= colCount; j++) {
                    System.out.println("conflict value: " + resolver.getConflictValue(j));
                    if (resolver.getConflictValue(j) != null) {
                        crsValue = cachedRowSet2.getObject(j);
                        resolverValue = resolver.getConflictValue(j);

                        resolvedValue = crsValue;

                        resolver.setResolvedValue(j, resolvedValue);
                        System.out.println("conflict resolved.");
                    }
                }
            }
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

ただ、これは実際にはうまく動作させることができていません。
下記のようになってしまい、conflictが検知できるものの、実際にループを回すとコンフリクトが生じているデータが取得できず、そのままループが終わってしまいます。

```
conflict detected.
conflict value: null
conflict value: null
conflict value: null
conflict value: null
conflict value: null
```

結果としては、1つ目の更新が反映され、2つ目の更新は無視されるという挙動になります。
これがMySQLとかConnector-Jの実装によるものなのか、私が書いたコードの問題なのかは結局のところ分かりませんでした。

## JoinRowSet: 2つのRowSetからテーブル同士のJOINと同等のことを行う

もう一つの興味深い機能である、RowSetのジョイン機能も試してみました。

```
mysql> SELECT * FROM COFFEES;
+--------------------+--------+-------+-------+-------+
| COF_NAME           | SUP_ID | PRICE | SALES | TOTAL |
+--------------------+--------+-------+-------+-------+
| Amaretto           |    101 |  5.99 |     0 |     0 |
| Amaretto_decaf     |     49 | 10.99 |     0 |     0 |
| Colombian          |    101 |  0.07 |     0 |     0 |
| Colombian_Decaf    |    101 |  8.99 |     0 |     0 |
| Espresso           |    150 |  9.99 |     0 |     0 |
| French_Roast       |     49 |  8.99 |     0 |     0 |
| French_Roast_Decaf |     49 |  9.99 |     0 |     0 |
| Hazelnut           |     49 |  9.99 |     0 |     0 |
| Hazelnut_decaf     |     49 | 10.99 |     0 |     0 |
+--------------------+--------+-------+-------+-------+
9 rows in set (0.00 sec)

mysql> SELECT * FROM SUPPLIERS;
+--------+---------------------------+---------------------+--------------+-------+-------+
| SUP_ID | SUP_NAME                  | STREET              | CITY         | STATE | ZIP   |
+--------+---------------------------+---------------------+--------------+-------+-------+
|     49 | Superior Coffee           | 1 Party Place       | Mendocino    | CA    | 95460 |
|    101 | Acme, Inc.                | 99 Market Street    | Groundsville | CA    | 95199 |
|    150 | The High Ground           | 100 Coffee Lane     | Meadows      | CA    | 93966 |
|    456 | Restaurant Supplies, Inc. | 200 Magnolia Street | Meadows      | CA    | 93966 |
|    927 | Professional Kitchen      | 300 Daisy Avenue    | Groundsville | CA    | 95199 |
+--------+---------------------------+---------------------+--------------+-------+-------+
5 rows in set (0.00 sec)
```

このように、COFFEESテーブルとSUPPLIERテーブルがあるとして、サプライヤー名を指定したときにそのIDをキーにしてコーヒーリストから商品名を持ってきたい・・・というようなケースはテーブルジョインのよくあるケースです。その際、普通は

``` sql
SELECT c.COF_NAME
FROM COFFEES AS c, SUPPLIERS AS s
WHERE c.SUP_ID = s.SUP_ID
AND SUP_NAME = "Acme, Inc."
```

などとやるのが普通ですが、これを各テーブルのRowSetをJoinRowSetを使ってJavaコード内でジョインさせることができます。

``` java
coffees = new CachedRowSetImpl();
coffees.setCommand("SELECT * FROM COFFEES");
coffees.execute(connection);

suppliers = new CachedRowSetImpl();
suppliers.setCommand("SELECT * FROM SUPPLIERS");
suppliers.execute(connection);

jrs = new JoinRowSetImpl();
jrs.addRowSet(coffees, "SUP_ID");
jrs.addRowSet(suppliers, "SUP_ID");

System.out.println("Coffees bought from " + supplierName + ": ");
while (jrs.next()) {
    if (jrs.getString("SUP_NAME").equals(supplierName)) {
        String coffeeName = jrs.getString(1);
        System.out.println("     " + coffeeName);
    }
}
```

これでJOINしたSQLと同等の結果が得られます。

実際には使う機会は多くなさそうですが、データベースをシャーディングしていて別データベースにあるテーブルをJOINしたい場合などには便利に使えると思いました。

今回のソースはこちらです。

https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter7


[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)
