---
layout: post
title: "GradleとScalaでスタンドアローンのjarファイルを生成"
date: 2014-01-18 21:12
comments: true
categories: gradle scala
---

gradleをビルドツールに使ってscalaの開発をしているのだけど、基本的なところがちょくちょく分からないのでメモしていきます。

バッチなどで実行できるスタンドアローンのjarファイルを作りたい場合です。scalaのライブラリなども内包して、`java -jar HelloJar.jar` で実行できるようにしたい。

* build.gradle

``` groovy
apply plugin: "scala"
apply plugin: "idea"
apply plugin: "gradle-one-jar"

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "com.github.rholder:gradle-one-jar:1.0.3"
    }
}

task HelloJar(type: OneJar){
    mainClass = "HelloScala"
}

repositories {
    mavenCentral()
}

dependencies {
  runtime "org.scala-lang:scala-compiler:2.10.3"
  compile "org.scala-lang:scala-library:2.10.3"
}


artifacts {
    archives HelloJar
}

```
