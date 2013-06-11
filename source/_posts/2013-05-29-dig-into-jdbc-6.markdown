---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(6) トランザクション
date: 2013-05-29 08:50
comments: true
categories: java jdbc mysql
---

## 一般的なトランザクション処理の流れ

複数の処理をまとめてコミットorロールバックできるトランザクション機能。JDBCでの基本フローを試してみます。
前回と同様下記の内容を参考にしています。

Using Transactions (The Java™ Tutorials > JDBC(TM) Database Access > JDBC Basics)
http://docs.oracle.com/javase/tutorial/jdbc/basics/transactions.html

下記のようなレコードがあるとして、
```
mysql> SELECT * FROM COFFEES WHERE COF_NAME IN ("Amaretto", "Espresso");
+----------+--------+-------+-------+-------+
| COF_NAME | SUP_ID | PRICE | SALES | TOTAL |
+----------+--------+-------+-------+-------+
| Amaretto |     49 |  9.99 |     0 |     0 |
| Espresso |    150 |  9.99 |     0 |     0 |
+----------+--------+-------+-------+-------+
2 rows in set (0.00 sec)
```

これらのPRICEをひとまとまりで更新してみます。

``` java
try {
	context = new InitialContext();
	DataSource dataSource = (DataSource)context.lookup("java:comp/env/jdbc/continuousops");
	connection = dataSource.getConnection();

	connection.setAutoCommit(false);

	preparedStatement1 = connection.prepareStatement("UPDATE COFFEES SET PRICE=? WHERE COF_NAME = 'Amaretto' LIMIT 1");
	preparedStatement2 = connection.prepareStatement("UPDATE COFFEES SET PRICE=? WHERE COF_NAME = 'Espresso' LIMIT 1");

	preparedStatement1.setFloat(1, 7.99f);
	preparedStatement1.executeUpdate();

	preparedStatement2.setFloat(1, 6.99f);
	preparedStatement2.executeUpdate();

	connection.commit();

} catch (NamingException | SQLException e) {
	connection.rollback();
	e.printStackTrace();
}finally{
	connection.setAutoCommit(true);
}
```

実行結果
```
mysql> SELECT * FROM COFFEES WHERE COF_NAME IN ("Amaretto", "Espresso");
+----------+--------+-------+-------+-------+
| COF_NAME | SUP_ID | PRICE | SALES | TOTAL |
+----------+--------+-------+-------+-------+
| Amaretto |     49 |  7.99 |     0 |     0 |
| Espresso |    150 |  6.99 |     0 |     0 |
+----------+--------+-------+-------+-------+
2 rows in set (0.00 sec)
```

JDBCでのトランザクションは、

* connection.setAutoCommit(false)でトランザクション開始
* connection.commit()でコミット
* connection.rollback()でロールバック
* connection.setAutoCommit(true)で元のモードに戻す　

というのが一連の流れです。

## トランザクション分離レベル: REPEATABLE READ

トランザクションは、更新を一括で行うという意味の他に、トランザクション処理中はデータの一貫性が保証されるという意味も持っています。

```
mysql> show variables like '%isolation';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| tx_isolation  | REPEATABLE-READ |
+---------------+-----------------+
1 row in set (0.00 sec)
```

MySQL5.5のデフォルトのトランザクション分離レベルはREPEATABLE READです。
JDBCのトランザクション分離レベルは使用しているデータベースの設定に従うので、下記のコードで調べてみると、

``` java
int isolationLevel = connection.getTransactionIsolation();
```

結果は、4となります。

Constant Field Values (Java Platform SE 7 )
http://docs.oracle.com/javase/7/docs/api/constant-values.html#java.sql.Connection.TRANSACTION_READ_COMMITTED

4はREPEATABLE_READとなっているので、MySQLの設定を使っていると確認できます。

アプリケーションによって性能を優先したい場合、一段階下げてREAD COMMITEDで運用するようにしてもよいと思います。InnoDBはREPEATABLE READでネクストキーロックをするので、INSERT性能が下がってしまうためです。

## Rollbackする地点を指定：Sevepointの使用

JDBCには一歩進んだ機能として、トランザクション中の任意のポイントまでロールバックできるSavepointという仕組みが用意されています。これをConnector-Jでも使用可能か試してみます。

下記のようなコードの場合、savepoint2まで途中でロールバックしているので、ステートメント1の更新は反映されるが、ステートメント2の更新はロールバックされているという挙動になります。

``` java
        Savepoint savepoint1 = null;
        Savepoint savepoint2 = null;
        try {
            context = new InitialContext();
            DataSource dataSource = (DataSource)context.lookup("java:comp/env/jdbc/continuousops");
            connection = dataSource.getConnection();

            connection.setAutoCommit(false);

            savepoint1 = connection.setSavepoint("point1");

            preparedStatement1 = connection.prepareStatement("UPDATE COFFEES SET PRICE=? WHERE COF_NAME = 'Amaretto' LIMIT 1");
            preparedStatement2 = connection.prepareStatement("UPDATE COFFEES SET PRICE=? WHERE COF_NAME = 'Espresso' LIMIT 1");

            preparedStatement1.setFloat(1, 5.99f);
            preparedStatement1.executeUpdate();

            savepoint2 = connection.setSavepoint("point2");

            preparedStatement2.setFloat(1, 7.99f);
            preparedStatement2.executeUpdate();

            connection.rollback(savepoint2);

            connection.commit();

        } catch (NamingException | SQLException e) {
            try {
                connection.rollback(savepoint1);
```

実行前
```
mysql> SELECT * FROM COFFEES WHERE COF_NAME IN ("Amaretto", "Espresso");
+----------+--------+-------+-------+-------+
| COF_NAME | SUP_ID | PRICE | SALES | TOTAL |
+----------+--------+-------+-------+-------+
| Amaretto |     49 |  9.99 |     0 |     0 |
| Espresso |    150 |  9.99 |     0 |     0 |
+----------+--------+-------+-------+-------+
2 rows in set (0.00 sec)
```

実行後
```
mysql> SELECT * FROM COFFEES WHERE COF_NAME IN ("Amaretto", "Espresso");
+----------+--------+-------+-------+-------+
| COF_NAME | SUP_ID | PRICE | SALES | TOTAL |
+----------+--------+-------+-------+-------+
| Amaretto |     49 |  5.99 |     0 |     0 |
| Espresso |    150 |  9.99 |     0 |     0 |
+----------+--------+-------+-------+-------+
2 rows in set (0.00 sec)
```

1個めのUPDATEだけが適用されました。

Savepointは、大規模なトランザクションでは使いどころがありそうです。

今回のソースはこちらです。

https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter6

参考：

トランザクション分離レベル - Wikipedia
http://ja.wikipedia.org/wiki/%E3%83%88%E3%83%A9%E3%83%B3%E3%82%B6%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E5%88%86%E9%9B%A2%E3%83%AC%E3%83%99%E3%83%AB

MySQL5 分離レベル　～ transaction_isolation | QuickKnowLedge
http://www.s-quad.com/wordpress/?p=956


[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)
