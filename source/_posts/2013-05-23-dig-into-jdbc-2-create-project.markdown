---
layout: post
title: 最速ウェブアプリのためのJDBC再入門(2) プロジェクト作成
date: 2013-05-23 08:48
comments: true
categories: java jdbc servlet
---

## JSP & Servletを使った基本的なプロジェクトを作成

まずは基本的なプロジェクトを作成しました。
mavenのプロジェクト構成に沿った形にして、JSPとServletを連携させた最低限の構成です。

ソースはこちらです。　
chokkoyamada/DigIntoJDBC at chapter2 · GitHub
https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter2

[「独習Java サーバーサイド編 第2版」](http://www.amazon.co.jp/独習Javaサーバサイド編-第2版-山田-祥寛/dp/4798130494) を教科書的に使いながら作りました。
ただ、書籍の中ではJSPの中で直接データベースに接続しているので、それはJavaのクラスファイルのほうへ外出ししました。

```
├── lib
├── pom.xml
├── src
│   └── main
│       ├── java
│       │   └── com
│       │       └── kirishikistudios
│       │           └── continuousops
│       │               └── digintojdbc
│       │                   └── Chapter2.java
│       ├── resources
│       └── webapp
│           ├── META-INF
│           │   └── context.xml
│           ├── WEB-INF
│           │   └── web.xml
│           └── chapter2
│               └── index.jsp
```

MySQLサーバーはlocalhostに立てて、ユーザー名jdbcuser、パスワードjdbcuserで作ってcontinuousopsというデータベースを作成してあります。

## context.xmlの置き場所と意味

上記の中で、context.xmlはどういうルールで書いてどこに置けばよいのかが分かりませんでした。
まとめると下記のようなことのようです。

* "context.xml"という名前自体に特別な意味はないが、ルールとして推奨されている
* xmlファイル内で定義する`<Context>`要素に、アプリケーション単位の情報を定義する。今回の場合はJDBCの接続情報。
* この`<Context>`要素を独立した設定ファイル(コンテキスト定義ファイル)にしたものがcontext.xml
* 配置先は下記の5つのどれかと決まっている。

- %CATALINA_HOME%/conf/server.xml
- %CATALINA_HOME%/conf/context.xml
- %CATALINA_HOME%/conf/<エンジン名>/<ホスト名>/context.xml.default
- %CATALINA_HOME%/conf/<エンジン名>/<ホスト名>/<アプリケーション名>.xml
- %CATALINA_HOME%/webapps/<アプリケーション名>/META-INF/context.xml

参考
context.xmlの配置について分かったこと． - 小さな星がほらひとつ
http://d.hatena.ne.jp/WorldWorldWorld/20101202/1291293444

「独習Java サーバーサイド編」にも同じことが書いてありました。

下記を見ると、説明の方法がちょっと違います。ひとまず無難なところでMETA-INFの下に置きました。

Apache Tomcat 7 Configuration Reference (7.0.40) - The Context Container
http://tomcat.apache.org/tomcat-7.0-doc/config/context.html#Defining_a_context


## ローカルのMySQLに接続してみる


``` java Chapter2.java
package com.kirishikistudios.continuousops.digintojdbc;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.sql.DataSource;
import java.io.IOException;
import java.sql.Connection;
import java.sql.SQLException;

public class Chapter2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException{
        request.setAttribute("subject", "JDBC");
        Context context;
        Connection connection = null;
        try {
            context = new InitialContext();
            DataSource dataSource = (DataSource)context.lookup("java:comp/env/jdbc/continuousops");
            connection = dataSource.getConnection();
            request.setAttribute("message1", "Successfully connected to database.");
            connection.close();
        } catch (NamingException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            if(connection != null) {
                try {
                    connection.close();
                    request.setAttribute("message2", "Successfully disconnected from database.");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
        this.getServletContext().getRequestDispatcher("/chapter2/index.jsp").forward(request, response);
    }
}

```

``` jsp index.jsp
<%@ page contentType="text/html; charsert=UTF-8" %>
<!DOCTYPE html>
<html>
<body>
<h2>Hello ${requestScope['subject']}!</h2>
<p>${requestScope['message1']}</p>
<p>${requestScope['message2']}</p>
</body>
</html>

```

どこからどこをtry-catchで囲んで、例外処理すればよいのかがよくわかってないですが・・・このへんはエラーが出た際にアプリケーションでどう処理したいかにもよってくると思います。

突っ込みどころがあればPull Requestお願いします。

https://github.com/chokkoyamada/DigIntoJDBC/tree/chapter2

https://github.com/chokkoyamada/chokkoyamada.github.com/blob/source/source/_posts/2013-05-23-dig-into-jdbc-2-create-project.markdown

次は実際にSQLを発行していきます。

[→最速ウェブアプリのためのJDBC再入門TOPへ](/special/jdbc)

