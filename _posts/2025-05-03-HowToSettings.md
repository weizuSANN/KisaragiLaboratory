---
title: "コメント・タグ・検索機能の実装法" #タイトルを入力
categories: #分類
  - Jekyll
tags: #タグ
  - Jekyll
  - Minimal Mistakes
  - Blog
comments: true
---
## 目次
1. [目次](#目次)
2. [目的](#目的)
2. [コメント機能の実装](#コメント機能の実装)
3. [タグ・カテゴリー検索の実装](#タグ・カテゴリー検索の実装)
4. [検索機能の実装](#検索機能の実装)
5. [まとめ](#まとめ)

## 目的
どうもこんにちわ。きさらぎかえでです。<br>

今回、このブログの検索機能やらコメント機能なんかを実装するにあたって2時間くらい詰まったので、記録を残します。<br>

## コメント機能の実装
本ブログにおいて、コメント機能は、[disqus](https://disqus.com/admin/install/platforms/jekyll/)というプロバイダを利用しています。先のリンクから飛んで、アカウントを作成します。その後、Settings→Generalの順にアクセスし、Shortnameの欄をコピーします。<br>
_config.ymlを開きます。commentsでproviderをdisqusにし、disqusのshortnameに、先ほどコピーしたShortnameを貼り付けます。
![disqus_config](/assets/Picture/HowToSetting/Config.png)
ブログ本体の.mdファイルの一番上の設定に、comments:trueをつけ足せば、コメントができるようになります。

## タグ・カテゴリー検索の実装
これが一番時間かかりました。まず、プロジェクトのルートフォルダに、tag.html,category.htmlを作成します。
tag.htmlには、以下のコードをそのまま貼り付けます。
```
---
layout: page
title: タグ一覧
permalink: /tags/
---
{% for tag in site.tags %}
<article>
    <h1 id="tag.{{ tag[0] }}" style="margin-left: 30%; text-align: left;">{{ tag[0] }}</h1>
    <ul style="margin-left: 30%; margin-right: 20%; text-align: left;">
        {% for post in tag[1] %}
        <li style="list-style: none;"><h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1></li>
        {% assign paragraphs = post.content | split: "</h2>" %}
        {% if paragraphs.size > 2 %}
        <p>{{ paragraphs[2] | remove: "<p>" | strip_html | truncate: 100 }}</p>
        {% endif %}
        {% endfor %}
    </ul>
</article>
{% endfor %}
```
理屈を軽く解説します。
6行目で、Python式のfor文を利用してこのサイトで使用されてるタグ(site.tagが保持している)を取得して、タグをすべて表示させます。<br>
8行目がタグ表示の仕組みです。少し大きめにh1で表記しています。また、全体から見た時、中心にあるけど、テキストそのものは左寄せにしたいので、margin-leftとtext-alignを指定しています。<br>
9行目は、リスト要素を利用して、記事のタイトル、記事の目次を除いた最初数行を表示させます。<br>
10行目は、11行目～16行目までの処理を繰り返し実行します。タグの付いた記事の数だけ繰り返します。<br>
11行目がタイトル表示<br>
12行目～15行目までは、記事の目次を除去して、その最初数行を取得しています。仕組みとしては、.contentの中から、見出し(h2タグのついてるもの)で分割して、変数paragraphsに逐次代入していきます。この段階で、見出しごとに内容が分けられています。しかし、これだけでは目次が残っているので、それを除去するために、paragraphs.sizeが2以上(見出しの2番目)のものであれば、その中で、paragraphsの2番目(1.見出し、2.内容の順番)を表示させています。<br>
カテゴリーのプログラムは以下の通りで、上のものとほぼ同じです。タグをカテゴリーに読み替えてもらえればOKです。
```
---
layout: page
title: カテゴリ一覧
permalink: /categories/
---
{% for category in site.categories %}
<article>
    {% capture category_name %}{{ category | first}}{% endcapture %}
    <h1 id="tag.{{ category_name }}" style="margin-left: 30%; text-align: left;">{{ category_name }}</h1>
    <ul style="margin-left: 30%; margin-right: 20%; text-align: left;">
        {% for post in site.categories[category_name] %}
        <li style="list-style: none;"><h1><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h1></li>
        {% assign paragraphs = post.content | split: "</h2>" %}
        {% if paragraphs.size > 2 %}
        <p>{{ paragraphs[2] | remove: "<p>" | strip_html | truncate: 100 }}</p>
        {% endif %}
        {% endfor %}
    </ul>
</article>
{% endfor %}
```
_config.ymlを必要に応じて改変します。
```
category_archive:
  type: liquid
  path: /categories/
tag_archive:
  type: liquid
  path: /tags/
```
あとはプッシュして、記事のtagをクリックすると、tag一覧が表示されます。
なお、post.urlだけだと、URLからリポジトリが抜けていて、アクセスできないという仕様に嵌り、何度も修正しました。上記コードはうまくいったものをそのままのっけてるので動くと思います。

## 検索機能の実装
検索機能はめちゃくちゃ簡単です。
![SearchConfig](/assets/Picture/HowToSetting/SearchConfig.png)
このように、search関連をtrueにして、search_providerをlunrにして終わりです。

## まとめ
知っていれば簡単なんですけど、資料が英語しかなかったり、Markdownだけかければいいかと思いきやhtmlの技術(産技2年生相当)が必要だったりするので、一応書いた感じです。もし追従してブログ書く方いたら参考程度にでも。