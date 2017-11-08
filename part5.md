# Solr vs ElasticSearch: Part 5 – Management API Capabilities

January 8, 2013 by Rafał Kuć 3 Comments

In previous posts, all listed below, we’ve discussed general architecture, full text search capabilities and facet aggregations possibilities. However, till now we have not discussed any of the administration and management options and things you can do on a live cluster without any restart. So let’s get into it and see what Apache Solr and ElasticSearch have to offer.

下記に示す以前の回では全般的な概観、全文検索機能、ファセットの集約機能について議論した。しかしここまで管理と運営のオプションと運用中のクラスタにて再起動無しでできることについては議論していない。そこでその詳細に触れ、Apache SolrとElasticSearchが何を提供できるか見てみよう。

1. Solr vs. ElasticSearch: Part 1 – Overview
2. Solr vs. ElasticSearch: Part 2 – Data Handling
3. Solr vs. ElasticSearch: Part 3 – Searching
4. Solr vs. ElasticSearch: Part 4 – Faceting
5. Solr vs. ElasticSearch: Part 5 – Management API Capabilities
6. Solr vs. ElasticSearch: Part 6 – User & Dev Communities Compared

## Input/Output Format
### ElasticSearch

As you probably know ElasticSearch offers a single way to talk to it – its HTTP REST API – JSON structured queries and responses. In most cases, especially during query time, it is very handy, because it let’s you perfectly control the structure of your queries and thus control the logic.

ご存知かとは思うがElasticSearchは単一の通信方法を提供する。HTTP REST APIだ。JSON構造のクエリとレスポンスである。多くの場合、特にクエリ時においてとても便利だ。クエリの構造を完全にコントロール可能、従ってロジックをコントロールできるからだ。

### Apache Solr

On the other hand we have Apache Solr. If you are familiar with it you know that in order to send a query to Solr one needs to send it using URL request parameters.  This makes communication much less structured compared to ElasticSearch JSON format. In response you can get multiple response formats that are supported out of the box, like the default XML, JSON, CSV, PHP serialized, or Ruby.

一方のApache Solrであるが、もしそれに詳しければ、Solrにクエリを送るのにURLリクエストパラメータを送る必要があるのはご存知だろう。これがElasticSearchのJSON形式に比べるとコミュニケーションをより低い程度の構造化としてしまう。レスポンスでは素の状態で複数のレスポンス形式がサポートされている。デフォルトのXML、JSON、CSV、シリアライズされたPHP、またはRubyである。


## Statistics API

Most of the time your search cluster will be fine and you won’t have any problems with it. However, there are times where you may need to see what is happening inside Apache Solr or ElasticSearch to diagnose problems, such as performance problems (hello SPM!), stability issues, or anything like that. In such cases, both search engines provide some amount of statistics.

ほとんどの場合、検索クラスタは健康で問題を起こすことはないだろう。しかしApache SolrやElasticSearchの中で何が起こっているのかを見て問題の診断を行う必要があるだろう。例えばパフォーマンス上の問題や（やぁ、 SPM!)安定性の問題、またはそのような問題全てだ。そのような場合には両方の検索エンジンはいくつかの量の統計値を提供する。

### Apache Solr

In Solr we can use JMX or HTTP requests to retrieve information about handler usage, cache statistics or information about most Solr components.


SolrではJMXやHTTPリクエストをハンドラの使用法、キャッシュの統計、または多くのSolrのコンポーネントの情報を得るのに利用できる。

### ElasticSearch
ElasticSearch was designed to be able to return various statistics about itself. With the REST API calls we can get information from the simplest ones like cluster health or nodes statistic, to extent information like the detailed ones about indices with merges, refreshes. The same stats are available via JMX, too.

ElasticSearchはそれ自身の様々な統計を返すことができるように設計されている。REST APIの呼出を用いて最も単純な物、例えばクラスタの健康状態やノードの統計等から、インデックスのマージ、リフレッシュのような詳細な物等広範囲の情報を得ることができる。同じ情報はJMX経由でも取得可能だ。


## Settings API
### ElasticSearch

ElasticSearch allows us to modify most of the configuration values dynamically. For example, you can clear you caches (or just specific type of cache), you can move shards and replicas to specific nodes in your cluster. In addition to that you are also allowed to update mappings (to some extent), define warming queries (since version 0.20), etc. You can even shut down a single node or a whole cluster with the use of a single HTTP call. Of course, this is just an example and doesn’t cover all the possibilities exposed by ElasticSearch.

ElasticSearchは動的に設定値の多くを変更することが可能だ。例えばキャッシュをクリアしたり、または特定のタイプのキャッシュのみをクリアしたり、Shardとレプリカをクラスタの指定したノードに移動することが可能だ。加えてある範囲のマッピングを更新したり、ウォーミングクエリ（v0.20以降）を定義したりもできる。単一ノードやクラスタ全体を一度のHTTP呼出でシャットダウンすることも可能だ。もちろんこれはただの例でありElasticSearchにより公開されている機能を全てカバーする訳ではない。

### Apache Solr

In case of Apache Solr we do not (yet) have the possibility of changing configuration values (like warming queries) with API calls.

Apache Solrの場合、API呼出により設定値の変更（例えばウォーミングクエリ）を行う機能は「まだ」無い


## インデックス／コレクションの管理機能
In addition to the capabilities mentioned above both ElasticSearch and Apache Solr provide APIs that allows us to modify our deployment when it comes to collections and indices.

上で説明した機能の他にElasticSearchとApache Solrはコレクションとインデックスに関するデプロイを変更することを可能とする。

### Apache Solr

Pre 4.0 we were able to manipulate cores inside our Solr instances. We could create new cores, reload them, get their status, rename, swap two of them, and finally remove a core from the instance. With Solr 4.0, a new API was introduced that is built on top of core admin API – the collections API. It allows us to create collections on started SolrCloud cluster, reload them and of course delete them. As the collections API is built on top of the core admin API,  if you create a new collection all the needed cores on all instances will be created. Of course, the same goes for reloading and deleting – all the cores will be appropriately informed and processed.

4.0以前ではSolrインスタンスの中の複数のコアを操作できた。新しいコアを作成したりリロードしたり、状態を得たり、名前変更、その内2つのスワップ、それに最終的にはインスタンスからコアを削除することができた。Solr 4.0ではコア管理APIのトップに新しいAPIが構築されて、紹介された。コレクションAPIである。開始したSolrCloudのクラスタ上にコレクションを作成すること、リロードともちろん消去もできる。コレクションAPIがコア管理APIの上に構築されて、新しいコレクションを作成すれば全てのインスタンス上の必要なコアは全て作成される。もちろん同様にリロードや削除も可能だ。全てのコアは適切に情報を受け、処理される。

### ElasticSearch

In case of ElasticSearch we can create and delete indices by running a simple HTTP command (GET or DELETE method) with the index name we are interested in. In addition to that, with a simple API call we can increase and decrease the number of replicas without the need of shutting down nodes or creating new nodes. With the newer ElasticSearch versions we can even manipulate shard placement with the cluster reroute API. With the use of that API we can move shards between nodes, we can cancel shard allocation process and we can also force shard allocation – everything on a live cluster.

ElasticSearchの場合、簡単なHTTPコマンド（GET、またはDELETEメソッド）を対象とするインデックスの名前と共に実行するだけでインデックスの作成や削除が可能だ。加えて簡単なAPI呼出でレプリカの数をノードのシャットダウンや作成無しに増減することも可能だ。新しいElasticSearchのバージョンではShardの配置をクラスタのリルートAPIで行うこともできる。そのAPIの使用によりShardをノード間で移動したり、Shardの獲得手続をキャンセルしたり、または強制することも可能だ。全て運用中のクラスタ上でである。

## Query Analysis
### Apache Solr

If you’ve used Apache Solr you probably come across the debugQuery parameter and the explainOther parameter. Those two allows to see the detailed score calculation for the given query and documents found in the results (the debugQuery parameter) and the specified ones (the explainOther). In addition, we can also see how the analysis process is done with the use of analysis handler or by using the analysis page of the Solr administration panel provided with Solr.

Apache Solrを以前に使ったことがあればdebugQueryパラメータとexplainOtherパラメータを知っているだろう。この2つを用いて与えられたクエリとその結果見つかったドキュメント（debugQueryパラメータ）または特定の1つ（explainOther）のスコア演算詳細を見ることができる。さらに解析ハンドラの使用により、またはSolrにより提供されるSolr管理パネルの解析ページの使用により、解析プロセスがどのように行われているかを見ることができる。

For example this is how debug information returned by Solr can look like:

Solrにより返されたデバッグ情報がどのように見えるかを以下に示す。

	<?xml version="1.0" encoding="UTF-8"?>
	<response>
	.
	.
	.
	<lst name="debug">
	 <str name="rawquerystring">ten</str>
	 <str name="querystring">ten</str>
	 <str name="parsedquery">(+DisjunctionMaxQuery((prefixTok:ten)~0.01) ())/no_coord</str>
	 <str name="parsedquery_toString">+(prefixTok:ten)~0.01 ()</str>
	 <str name="QParser">DisMaxQParser</str>
	 <null name="altquerystring"/>
	 <null name="boostfuncs"/>
	 <lst name="timing">
	  <double name="time">2.0</double>
	  <lst name="prepare">
	   <double name="time">1.0</double>
	   <lst name="org.apache.solr.handler.component.QueryComponent">
	    <double name="time">1.0</double>
	   </lst>
	   <lst name="org.apache.solr.handler.component.FacetComponent">
	    <double name="time">0.0</double>
	   </lst>
	   <lst name="org.apache.solr.handler.component.MoreLikeThisComponent">
	    <double name="time">0.0</double>
	   </lst>
	   <lst name="org.apache.solr.handler.component.HighlightComponent">
	    <double name="time">0.0</double>
	   </lst>
	   <lst name="org.apache.solr.handler.component.StatsComponent">
	    <double name="time">0.0</double>
	   </lst>
	   <lst name="org.apache.solr.handler.component.DebugComponent">
	    <double name="time">0.0</double>
	   </lst>
	 </lst>
	 <lst name="process">
	  <double name="time">1.0</double>
	  <lst name="org.apache.solr.handler.component.QueryComponent">
	   <double name="time">0.0</double>
	  </lst>
	  <lst name="org.apache.solr.handler.component.FacetComponent">
	   <double name="time">0.0</double>
	  </lst>
	  <lst name="org.apache.solr.handler.component.MoreLikeThisComponent">
	   <double name="time">0.0</double>
	  </lst>
	  <lst name="org.apache.solr.handler.component.HighlightComponent">
	   <double name="time">0.0</double>
	  </lst>
	  <lst name="org.apache.solr.handler.component.StatsComponent">
	   <double name="time">0.0</double>
	  </lst>
	  <lst name="org.apache.solr.handler.component.DebugComponent">
	   <double name="time">1.0</double>
	  </lst>
	 </lst>
	</lst>
	<lst name="explain">
	 <str name="Ten mices">
	1.3527006 = (MATCH) sum of:
	 1.3527006 = (MATCH) weight(prefixTok:ten in 35158) [DefaultSimilarity], result of:
	 1.3527006 = fieldWeight in 35158, product of:
	 1.4142135 = tf(freq=2.0), with freq of:
	 2.0 = termFreq=2.0
	 6.1216245 = idf(docFreq=6355, maxDocs=1065313)
	 0.15625 = fieldNorm(doc=35158)
	 </str>
	</lst>
	</lst>
	</response>

As you can see, we can get information about timings of each of the used components. In addition to that, we see the parsed query and of course the explain information showing us how the document score was calculated.

ご覧のとおり、使用された各コンポーネントのタイミングに関する情報を得ることが可能だ。加えてパースされたクエリを見ることや、もちろん、どのようにドキュメントのスコアが計算されたかを知るexplain情報を見ることも可能だ。

### ElasticSearch

ElasticSearch exposes three separate REST end-points to analyze our queries, documents and explain the documents score. The Analyze API allows us to test our analyzer on a specified text in order to see how it is processed and is similar to the analysis page functionality of Solr. The Explain API provides us with information about the score calculation for a given documents. Finally, the Validate API can validate our query to see is it is proper and how expensive it can be.

ElasticSearchは3つに分けられたRESTのエンドポイントを公開しており、クエリやドキュメント、ドキュメントスコアのexplainを分析できる。解析APIは解析器が特定のテキストにおいてどのように処理されるかをテストすることができる。Solrの解析ページの機能に似ている。Explain APIは与えられたドキュメントに対するスコア演算に関する情報を提供する。最後にValidate APIはクエリが適切でどの程度のコストであるかを確認できる。

For example, this is what Explain API response looks like:

例として、Explain APIのレスポンスがどのようなものであるかを示す。

	{
	 "ok" : true,
	 "_index" : "docs",
	 "_type" : "doc",
	 "_id" : "1",
	 "matched" : true,
	 "explanation" : {
	   "value" : 0.15342641,
	   "description" : "fieldWeight(_all:document in 0), product of:",
	   "details" : [ {
	     "value" : 1.0,
	     "description" : "tf(termFreq(_all:document)=1)"
	   }, {
	     "value" : 0.30685282,
	     "description" : "idf(docFreq=1, maxDocs=1)"
	   }, {
	     "value" : 0.5,
	     "description" : "fieldNorm(field=_all, doc=0)"
	   } ]
	 }
	}

You can see the description about score calculation that is returned from the Explain API.

Explain APIより返されたスコア演算の記述がわかるだろう。

## Before We End

There are a few words more we wanted to write before summarizing this comparison. First of all the above mentioned APIs and possibilities are not all that it is available, especially when it comes to ElasticSearch. For example, with ElasticSearch you can clear caches on the index level, you can check index and types existence, you can retrieve and manage your warming queries, clear the transaction log by running the Flush API, or  even close an index or open those that were closed. We wanted to point some differences and similarities between Apache Solr and ElasticSearch, but we didn’t want to make a summary of the documentation. :) So, if you are interested in some functionality and you don’t know if it exists, just send a mail to Apache Solr or ElasticSearch mailing list or leave a comment here, and we will be glad to help.

この比較をまとめる前にまだ少し書いておきたいことがある。まず最初に上記にて説明したAPIと機能は存在する全ての機能ではない。特にElasticSearchにおいてそうだ。例えばElasticSearchではインデックスレベル上のキャッシュをクリアしたり存在するインデックスとタイプを確認したり、ウォーミングクエリを取得、管理したり、フラッシュAPIを実行してトランザクションログを削除したり、さらにはインデックスをクローズしたり、クローズされたものをオープンしたりも可能だ。我々はApache SolrとElasticSearchの違いや類似点を示したかった。しかし我々はドキュメントのまとめを作りたくはなかった。:-) 従ってもしある機能に興味があり、それが存在するか知らない場合はApache SolrやElasticSearchのメーリングリストにメールを投げるか、ここにコメントを残して欲しい。我々は喜んで手助けをしたい。

## まとめ

When we first started the Solr vs ElasticSearch series we planned to initially divide the series into five posts, which are now published. However after seeing the popularity of the series and the amount of feedback we’ve received, we decided to extend the series. You can soon expect the next part, which will be dedicated to non-technical, but deeply important and interesting aspects of both search servers. After that, we’ll get back to the technical details with the subsequent post  dedicated to score influence capabilities, describing how we can change the default Lucene scoring and influence it from configuration, during indexing time and finally during querying.

Solr vs ElasticSearchのシリーズを開始した時、我々はシリーズを5つの記事に分ける予定だった。しかしこのシリーズの人気を見た後に、そして受け取ったフィードバックの量を考えた後に、このシリーズを拡張することを決めた。すぐに次回を期待できるだろう。次回は非技術的、しかしとても重要で面白い2つの検索サーバの側面について取り上げたい。その後、技術的な詳細に続くポストでは戻りスコアに影響する機能について取り上げ、インデックス作成時と最終的にクエリの間にどうやってLuceneのデフォルトのスコアリングを変更し、設定から影響を与えるかを取り上げるだろう。

If you liked this post, please tweet it!

この記事が気に入ったならtweetして下さい！

- @kucrafal, @sematext