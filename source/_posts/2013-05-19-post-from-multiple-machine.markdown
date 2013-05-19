---
layout: post
title: Octopressを始めました
date: 2013-05-19 19:09
comments: true
categories: octopres, blog, git
---

Octopressの気に入ったところ
---------------------

GitHub Pagesを使ってOctopressを始めてみました。ずっとwordpressを使っていたのですが、次の3点が気にいって使ってみようと思い立ちました。

* 静的HTMLでサイトが生成される
* Git(Hub)ベースなので履歴が残る
* Pushしておけば、必ずGitHubにバックアップがある
* vimとmarkdownで書ける

静的に生成できればS3やCloudFrontに置くことで高速化もできるし、サーバーに負荷がかかりにくいです。

WordpressはMySQLに記事データが入っていましたが、Octopressは生のテキストデータとしてデータが残っています。Markdownのテキストデータとしてデータが残っているというのは気分的に安心感があります。
HTML以外の他のデータフォーマットへのエクスポートもしやすいかと思いました。

最近、仕事でもプライベートでもGitHubをよく使っているので、セットアップも手こずらないだろうという見込みもありました。

vimで書けてそのままアップできるのも気持ちいいです。Markdownはvimとの相性も良いです。[vim-markdown](https://github.com/plasticboy/vim-markdown) を組み合わせて使っています。

日本語を使うところではまる
---------------------

日本語をタイトルや本文に入れて投稿しようとしたところ、
```
➜  octopress git:(source) be rake generate

## Generating Site with Jekyll
identical source/stylesheets/screen.css 
Configuration from /Users/caaios5/Documents/git/octopress/_config.yml
Building site: source -> public


ERROR: YOUR SITE COULD NOT BE BUILT:
------------------------------------
"\xE3" on US-ASCII in /Users/caaios5/Documents/git/octopress/source/_posts/2013-05-19-hello.markdown
```

という感じでなぜか記事をgenerateできず。文字コードの問題だと分かったので、ググって環境変数を設定したらいけました。

```
export LC_CTYPE=en_US.UTF-8 
export LANG=en_US.UTF-8
```

これを`~/.zshrc`に記載します(使ってるシェルに合わせて変更してください)

zshだと`rake new_post["hoge"]`とは書けず、`rake new_post\["hoge"\]`とエスケープする必要がある
---------------------

```
➜  octopress git:(source) be rake new_post["hoge"]
zsh: no matches found: new_post[hoge]
```

これはOctopress関係ないのですが、rakeコマンドが通らないので一瞬悩みました。bashならこの問題はありません。
またはrake 'new_post["hoge"]'でもいけます。


Octopressレポジトリのブランチ構成
---------------------
Octopressのレポジトリは、"master"と"source"という2つのブランチを使いわけた構成になっています。

* master

`rake generate`で生成された静的なHTMLコンテンツが置かれている。通常こちらを直接更新はしない。

* source

普段の作業ディレクトリ。`rake new_post['hoge']`などを実行して投稿を作るのもこちら。

「masterが公開する部分」、「sourceが管理側」といったイメージです。

なので、`rake deploy`では実質的にmasterがpushされます(実際、Rakefileの中はそのようになっています)、`git push origin source`で管理側をpushしておけば、`git clone`して`git checkout source`で別のマシンから作業の続きができます。


参考：

http://tokkonopapa.github.io/blog/2011/12/30/octopress-on-github-and-bitbucket/

記事を書き始めるのにはここが参考になりました。

http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/

文字コードの問題は環境変数を設定すると解決するのはここを読んで気づきました。
