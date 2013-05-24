---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(3) INSERT文とSELECT文
date: 2013-05-24 09:05
comments: true
categories: java jdbc mysql
---

## SQL発行の基本的な手順

前回はデータベースに接続し、そのまま切断しただけでしたが、今回はWriteとReadの処理を1回ずつやってみます。

ソースはこちらです。
https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter3

基本的にはPHPのPDOやPerlのPythonのMySQLdbなどとクエリの発行の構造は同じで、PrepareStatementというオブジェクトを使います。

ConnectionオブジェクトからPrepareedStatementを作成し、そこからクエリを実行、という流れです。

参考：
Using Prepared Statements (The Java™ Tutorials > JDBC(TM) Database Access > JDBC Basics)
http://docs.oracle.com/javase/tutorial/jdbc/basics/prepared.html

## INSERT文:executeUpdate()メソッドを使う:戻り値は処理レコード数

``` java
private void insertRecord(Connection connection) throws SQLException {
	PreparedStatement preparedStatement = connection.prepareStatement("INSERT INTO address (name, address, tel, email) VALUES(?, ?, ?, ?)");
	preparedStatement.setString(1, getParameter("name"));
	preparedStatement.setString(2, getParameter("address"));
	preparedStatement.setString(3, getParameter("tel"));
	preparedStatement.setString(4, getParameter("email"));
	preparedStatement.executeUpdate();
	preparedStatement.close();
}

```

connection.prepareStatement()メソッド内でSQLを記述し、"?"をプレイスホルダとして使います。"?"の部分はsetXXXX()メソッド(XXXXの部分は型が入る)のなかで番号に対応する形で値を入れています。

INSERT/UPDATE/DELETE文の発行はexecuteUpdate()メソッドを使います。

	疑問：JDBCにはキーワード付プレイスホルダは無いんでしょうか？


## SELECT文:executeQuery()メソッドを使う:戻り値はResultSet

``` java
private HashMap selectRecord(Connection connection) throws SQLException {
	PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM address LIMIT 1");
	ResultSet resultSet = preparedStatement.executeQuery();
	HashMap<String, String> map = new HashMap<>();
	if(resultSet.next()){
		map.put("name-1", resultSet.getString("name"));
		map.put("address-1", resultSet.getString("address"));
		map.put("tel-1", resultSet.getString("tel"));
		map.put("email-1", resultSet.getString("email"));
	}
	preparedStatement.close();
	return map;
}
```

SELECT文も、connection.prepareStatement()メソッド内でSQLを記述します。

SELECT文の実行には、executeQuery()メソッドを使います。このメソッドはResultSetオブジェクトが返るので、resultSet.next()でループして結果を取得します。

今回はPreparedStatementを使いましたが、下記のように単純なStatementオブジェクトで記述することもできます。

``` java
Statement statement = null;
String query = "SELECT * FROM address LIMIT 1";

try {
	statement = connection.createStatement();
	ResultSet resultSet = statement.executeQuery(query);
(以下略)
```

Statementとその派生クラスには3種類あります。

* Statement         パラメータを持たない単純なSQLの実行
* PreparedStatement SQLをプリコンパイルして実行する
* CallableStatement プロシージャを呼び出すときに使う

StatementよりPreparedStaementのほうが高速な場合が多いので、SELECT/INSERT/UPDATE/DELETE文のSQLの実行にはPreparedStatementを使うとおぼえておいて基本的に間違いないと思います。

StatementオブジェクトはGCによって自動的にクローズされますが、明示的にclose()を呼んでおくのが良いようです。今回は明示的にclose()を呼んでいます。

ひとまずシンプルにINSERTとSELECTを行なってみました。
次回はSELECTの最適化・高速化という観点から、ResultSetオブジェクトの扱いについて詳しくみてみます。

[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)

