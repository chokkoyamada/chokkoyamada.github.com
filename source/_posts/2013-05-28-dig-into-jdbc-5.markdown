---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(5) カーソルと更新
date: 2013-05-28 08:20
comments: true
categories: java jdbc mysql
---

## Cursorの自由な移動

引き続き下記のページを見ていきます。後半部分の、カーソルを移動させたりrowを更新したりする部分がConnector-Jで実際にできるかどうかを確認していきます。

Retrieving and Modifying Values from Result Sets (The Java™ Tutorials > JDBC(TM) Database Access > JDBC Basics)
http://docs.oracle.com/javase/tutorial/jdbc/basics/retrieving.html

ドキュメント通りに、下記のようなテーブルがあるとします。

```
mysql> select * from COFFEES;
+--------------------+--------+-------+-------+-------+
| COF_NAME           | SUP_ID | PRICE | SALES | TOTAL |
+--------------------+--------+-------+-------+-------+
| Colombian          |    101 |  7.99 |     0 |     0 |
| Colombian_Decaf    |    101 |  8.99 |     0 |     0 |
| Espresso           |    150 |  9.99 |     0 |     0 |
| French_Roast       |     49 |  8.99 |     0 |     0 |
| French_Roast_Decaf |     49 |  9.99 |     0 |     0 |
+--------------------+--------+-------+-------+-------+
5 rows in set (0.00 sec)

mysql> show create table COFFEES\G
*************************** 1. row ***************************
       Table: COFFEES
Create Table: CREATE TABLE `COFFEES` (
  `COF_NAME` varchar(32) NOT NULL,
  `SUP_ID` int(11) NOT NULL,
  `PRICE` decimal(10,2) NOT NULL,
  `SALES` int(11) NOT NULL,
  `TOTAL` int(11) NOT NULL,
  PRIMARY KEY (`COF_NAME`),
  KEY `SUP_ID` (`SUP_ID`),
  CONSTRAINT `coffees_ibfk_1` FOREIGN KEY (`SUP_ID`) REFERENCES `SUPPLIERS` (`SUP_ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```

まず1レコード目を取得します。

```
private List<HashMap> viewTable(Connection connection, String dbName) throws SQLException {
        String query = "SELECT COF_NAME, SUP_ID, PRICE, SALES, TOTAL FROM " + dbName + ".coffees" ;
        preparedStatement = connection.prepareStatement(query);
        resultSet = preparedStatement.executeQuery();
        resultSet.next();
        System.out.println("|" + resultSet.getString("COF_NAME")
                         + "|" + resultSet.getInt("SUP_ID")
                         + "|" + resultSet.getBigDecimal("PRICE") + "|");
```

```
|Colombian|101|7.99|
```

2レコード進んで取得します。
```
resultSet.next();
resultSet.next();
System.out.println("|" + resultSet.getString("COF_NAME")
				 + "|" + resultSet.getInt("SUP_ID")
				 + "|" + resultSet.getBigDecimal("PRICE") + "|");
```

```
|Espresso|150|9.99|
```

1レコード戻ります。
```
resultSet.previous();
System.out.println("|" + resultSet.getString("COF_NAME")
		+ "|" + resultSet.getInt("SUP_ID")
		+ "|" + resultSet.getBigDecimal("PRICE") + "|");
```

```
|Colombian_Decaf|101|8.99|
```

最後のレコードに飛びます。
```
resultSet.last();
System.out.println("|" + resultSet.getString("COF_NAME")
		+ "|" + resultSet.getInt("SUP_ID")
		+ "|" + resultSet.getBigDecimal("PRICE") + "|");
```

```
|French_Roast_Decaf|49|9.99|
```

4レコード目にジャンプします。
```
resultSet.absolute(4);
System.out.println("|" + resultSet.getString("COF_NAME")
		+ "|" + resultSet.getInt("SUP_ID")
		+ "|" + resultSet.getBigDecimal("PRICE") + "|");
```

```
|French_Roast|49|8.99|
```

カーソルを最後まで進めます。その次は無いのでnext()でfalseになるはず。
```
resultSet.afterLast();
System.out.println(resultSet.next());
```

```
false
```

期待通りの結果が得られました。

## ResultSetを直接Update

使うかどうか分かりませんが、ResultSetからrowを直接更新する方法も用意されています。

``` java
private void updateTable(Connection connection, String dbName) throws SQLException {
        float percentage = 0.3f;
        String query = "SELECT cof_name, sup_id, price, sales, total FROM " + dbName + ".coffees" ;
        preparedStatement = connection.prepareStatement(query);
        resultSet = preparedStatement.executeQuery();
        resultSet.next();
        System.out.println("|" + resultSet.getString("COF_NAME")
                + "|" + resultSet.getInt("SUP_ID")
                + "|" + resultSet.getBigDecimal("PRICE") + "|");
        float f = resultSet.getFloat("PRICE");
        resultSet.updateFloat("PRICE", f * percentage);
        resultSet.updateRow();
        System.out.println("|" + resultSet.getString("COF_NAME")
                + "|" + resultSet.getInt("SUP_ID")
                + "|" + resultSet.getBigDecimal("PRICE") + "|");
    }
```

ただ、これをそのまま実行しようとするとエラーになります。属性が`CONCUR_UPDATABLE`である必要があるのですが、昨日みたように、デフォルトは`CONCUR_READ_ONLY`だからです。
```
com.mysql.jdbc.NotUpdatable: Result Set not updatable.This result set must come from a statement that was created with a result set type of ResultSet.CONCUR_UPDATABLE, the query must select only one table, can not use functions and must select all primary keys from that table. See the JDBC 2.1 API Specification, section 5.6 for more details.This result set must come from a statement that was created with a result set type of ResultSet.CONCUR_UPDATABLE, the query must select only one table, can not use functions and must select all primary keys from that table. See the JDBC 2.1 API Specification, section 5.6 for more details.
	at com.mysql.jdbc.ResultSetImpl.updateFloat(ResultSetImpl.java:8319)
```

CONCUR_UPDATABLEにしてやってみます。サンプル通りStatementオブジェクトを使ってしまいましたが、PreparedStatementにしたければprepareCall()の中でCONCUR_UPDATEBLEを指定すればいけると思います。

``` java
private void updateTableRevised(Connection connection, String dbName) throws SQLException {
	float percentage = 0.3f;
	String query = "SELECT cof_name, sup_id, price, sales, total FROM " + dbName + ".coffees" ;
	Statement statement = connection.createStatement(ResultSet.TYPE_SCROLL_SENSITIVE, ResultSet.CONCUR_UPDATABLE);
	resultSet = statement.executeQuery(query);
	resultSet.next();
	System.out.println("|" + resultSet.getString("COF_NAME")
			+ "|" + resultSet.getInt("SUP_ID")
			+ "|" + resultSet.getBigDecimal("PRICE") + "|");
	float f = resultSet.getFloat("PRICE");
	resultSet.updateFloat("PRICE", f * percentage);
	resultSet.updateRow();
	System.out.println("|" + resultSet.getString("COF_NAME")
			+ "|" + resultSet.getInt("SUP_ID")
			+ "|" + resultSet.getBigDecimal("PRICE") + "|");
}
```

```
|Colombian|101|2.40|
|Colombian|101|0.72|
```
更新できました。

## executeBatch()による更新

batchによる更新もサポートされています。
```
public void batchUpdate(Connection connection) throws SQLException {
	Statement stmt = null;
	try {
		connection.setAutoCommit(false);
		stmt = connection.createStatement();

		stmt.addBatch(
				"INSERT INTO COFFEES " +
						"VALUES('Amaretto', 49, 9.99, 0, 0)");

		stmt.addBatch(
				"INSERT INTO COFFEES " +
						"VALUES('Hazelnut', 49, 9.99, 0, 0)");

		stmt.addBatch(
				"INSERT INTO COFFEES " +
						"VALUES('Amaretto_decaf', 49, " +
						"10.99, 0, 0)");

		stmt.addBatch(
				"INSERT INTO COFFEES " +
						"VALUES('Hazelnut_decaf', 49, " +
						"10.99, 0, 0)");

		int [] updateCounts = stmt.executeBatch();
		System.out.println(updateCounts);
		connection.commit();

	} catch(BatchUpdateException b) {
		b.printStackTrace();
	} catch(SQLException ex) {
		ex.printStackTrace();
	} finally {
		if (stmt != null) { stmt.close(); }
		connection.setAutoCommit(true);
	}
}
```

```
mysql> select * from COFFEES;
+--------------------+--------+-------+-------+-------+
| COF_NAME           | SUP_ID | PRICE | SALES | TOTAL |
+--------------------+--------+-------+-------+-------+
| Amaretto           |     49 |  9.99 |     0 |     0 |
| Amaretto_decaf     |     49 | 10.99 |     0 |     0 |
| Colombian          |    101 |  0.07 |     0 |     0 |
| Colombian_Decaf    |    101 |  8.99 |     0 |     0 |
| Espresso           |    150 |  9.99 |     0 |     0 |
| French_Roast       |     49 |  8.99 |     0 |     0 |
| French_Roast_Decaf |     49 |  9.99 |     0 |     0 |
| Hazelnut           |     49 |  9.99 |     0 |     0 |
| Hazelnut_decaf     |     49 | 10.99 |     0 |     0 |
+--------------------+--------+-------+-------+-------+
```

これはトランザクションではないようなので使いどころがいまいち分かりませんが、名前の通りバッチ処理をする際に少しでも実行時間を短縮するために使えるのかもしれません。

今日のソースはこちらです。
https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter5

[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)
