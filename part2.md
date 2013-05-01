#Solr vs. ElasticSearch: Part 2 – Data Handling

September 4, 2012 by Rafał Kuć 25 Comments

In the previous part of Solr vs. ElasticSearch series we talked about general architecture of these two great search engines based on Apache Lucene. Today, we will look at their ability to handle your data and perform indexing and language analysis.

前回の"Solr vs. ElasticSearch"シリーズの記事においてApache Luceneをベースとするこれらの2つの素晴しい検索エンジンの全般的なアーキテクチャについてお話した。今日はデータの取扱とインデックス作成の実行、言語の解析について、それらの能力を調べることにする。

1. Solr vs. ElasticSearch: Part 1 - Overview
2. Solr vs. ElasticSearch: Part 2 - Indexing and Language Handling
3. Solr vs. ElasticSearch: Part 3 - Searching
4. Solr vs. ElasticSearch: Part 4 - Faceting
5. Solr vs. ElasticSearch: Part 5 - Management API Capabilities
6. Solr vs. ElasticSearch: Part 6 – User & Dev Communities Compared

##データのインデックス作成

Apart from using Java API exposed both by ElasticSearch and Apache Solr, you can index data using an HTTP call. To index data in ElasticSearch you need to prepare your data in JSON format. Solr also allows that, but in addition to that, it lets you to use other formats like the default XML or CSV. Importantly, indexing data in different formats has different performance characteristics, but that comes with some limitations. For example, indexing documents in CSV format is considered to be the fastest, but you can’t use field value boosting while using that format. Of course, one will usually use some kind of a library or Java API to index data as one doesn’t typically store data in a way that allows indexing of data straight into the search engine (at least in most cases that’s true).


ElasticSearchとApache Solrの両者により公開されるJava APIの使用から離れて、HTTPの呼出を用いてデータのインデックス作成を行うことが可能だ。ElasticSearchにおいてデータのインデックスを作成するにはデータをJSONフォーマットに準備する必要がある。Solrもまたそれが可能だが、それに付け加えてデフォルトのXMLやCSV等、他の形式を利用することが可能だ。重要なこととして、異なる形式のデータのインデックスを作成する場合、異なるパフォーマンスの特性が存在する。しかしそれはいくらかの制約と共に存在する。例えばCSV形式のドキュメントのインデックスを作成する場合、最も速いと考えられるが、その形式を用いる場合にフィールド値のboostingを利用することができない。もちろん通常は何らかの種類のライブラリやJava APIを用いてデータのインデックス作成をするだろう。一般的に検索エンジンに直接入るデータのインデックスを作成しながらデータを格納することは無いだろう。（少なくとも多くの場合はそれが真実だ）

##ElasticSearchについて補足

It is worth noting that ElasticSearch supports two additional things, that Solr does not – nested documents and multiple document types inside a single index.

ElastiCSerachがSolrがサポートしない2つの追加機能をサポートすることには触れておく価値があるだろう。入れ子ドキュメントと単一ドキュメント内での複数のドキュメントタイプである。

The nested documents functionality lets you create more than a flat document structure. For example, imagine you index documents that are bound to some group of users. In addition to document contents, you would like to store which users can access that document.  And this is were we run into a little problem – this data changes over time. If you were to store document content and users inside a single index document, you would have to reindex the whole document every time the list of users who can access it changes in any way. Luckily, with ElasticSearch you don’t have to do that – you can use nested document types and then use appropriate queries for matching. In this example, a nested document would hold a lists of users with document access rights. Internally, nested documents are indexed as separate index documents stored inside the same index. ElasticSearch ensures they are indexed in a way that allows it to use fast join operations to get them. In addition to that, these documents are not shown when using standard queries and you have to use nested query to get them, a very handy feature.

入れ子ドキュメント機能は平坦なドキュメント構造を越えた物を作成可能にする。例えばあるユーザのグループに紐付くドキュメントのインデックスを作成すると考えよう。ドキュメントの中身に加えて、どのユーザがそのドキュメントにアクセスできるのかを格納したい。もし単一のインデックス文書の中にドキュメントの中身とユーザを格納する場合、そのドキュメントにアクセスできるユーザのリストを変更する度にドキュメント全体のインデックスを再作成しなければならないだろう。幸運なことにElasticSearchを用いることでそれを行う必要がなくなる。入れ子ドキュメントタイプを用いて適切なクエリをマッチングに用いることが可能だ。この例では入れ子ドキュメントはユーザのリストとドキュメントのアクセス権限を持つだろう。内部では入れ子ドキュメントは同じインデックス内部に格納されあた分離されたインデックス文書として索引付けされる。ElasticSearchはそれらが高速なJOIN命令を用いて取得することが可能な方法で索引付けられることを保証する。加えてこれらのドキュメントは標準クエリを用いる場合には不可視であり、入れ子のクエリを取得するためには使用する必要があり、とても便利な機能である。

Multiple types of documents per index allow just what the name says – you can index different types of documents inside the same index.  This is not possible with Solr, as you have only one schema in Solr per core.  In ElasticSearch you can filter, query, or facet on document types. You can make queries against all document types or just choose a single document type (both with Java API and REST).

インデックス当りの複数タイプドキュメントは名前どおりのことを可能にする。異なるタイプのドキュメントを同じインデックス内に索引付けすることが可能だ。これはSolrでは不可能で、Solrはコア当たりに1つのスキーマを持つことしかできない。ElasticSearchではドキュメントタイプによりフィルタ、クエリ、ファセットを行うことができる。全てのドキュメントタイプに対して、または1つのドキュメントタイプを選択してクエリを行うことができる。（Java APIとRESTの両方で可能である。）


##インデックスの操作

Let’s look at the ability to manage your indices/collections using the HTTP API of both Apache Solr and ElasticSearch.

Apache SolrとElasticSearchの両者にてインデックス／コレクションをHTTP APIを用いて管理する能力を見てみよう


###Solr

Solr let’s you control all cores that live inside your cluster with the CoreAdmin API – you can create cores, rename, reload, or even merge them into another core. In addition to the CoreAdmin API Solr enables you to use the collections API to create, delete or reload a collection. The collections API uses CoreAdmin API under the hood, but it’s a simpler way to control your collections. Remember that you need to have your configuration pushed into ZooKeeper ensemble in order to create a collection with a new configuration.

Solrは運用中のクラスタ内の全てのコアのコントロールをCoreAdmin APIにより可能にする。コアの作成、名前変更、リロード、またはそれらをマージして1つのコアにすることも可能だ。CoreAdmin APIに加えてSolrはCollections APIを用いてコレクションの作成、削除、まはたリロードが行える。Collections APIは裏でCoreAdmin APIを使用しており、より簡単にコレクションをコントロールする方法である。新しい設定のコレクションを作成するにはZooKeeper ensembleに設定をプッシュしなければならないことに注意すること。

When it comes to Solr, there is additional functionality that is in early stages of work, although it’s functional – the ability to split your shards. After applying the patch available in SOLR-3755 you can use a SPLIT action to split your index and write it to two separate cores. If you look at the mentioned JIRA issue, you’ll see that once this is commited Solr will have the ability not only to create new replicas, but also to dynamically re-shard the indices.  This is huge!

Solrにはまだ未熟な段階ではあるが有効な追加の機能がある。Shardの分割だ。SOLR-3755に存在するパッチを当てた後にインデックスを分割して2つの分離したコアに書き込むことがSPLITアクションを用いて可能になる。そのJIRAのイシューを読めばこれがコミットされればSolrは新しいレプリカを作成するだけでなく、動的にインデックスをre-shardする機能を持つことになるだろう。これはとても大きい。

###ElasticSearch

One of the great things in ElasticSearch is the ability to control your indices using HTTP API. We will take about it extensively in the last part of the series, but I have to mention it ere, too. In ElasticSearch you can create indices on the live cluster and delete them. During creation you can specify the number of shards an index should have and you can decrease and increase the number of replicas without anything more than a single API call. You cannot change the number of shards yet.  Of course, you can also define mappings and analyzers during index creation, so you have all the control you need to index a new type of data into you cluster.
ElasticSearchの素晴しい機能の1つにHTTP APIを用いてインデックスをコントロールする能力がある。このシリーズの最後のパートにて広範にそれについて触れるがここでも触れておく必要があるだろう。ElasticSearchでは運用中のクラスタ上にてインデックスを作成し、消すことが可能だ。作成の間にインデックスが持つべきShardの数を指定可能で、単一のAPI呼出のみでレプリカの数の増減が可能だ。Shardの数はまだ変更できない。もちろんマッピングと解析器をインデックス作成時に設定可能で、クラスタ内に新しい型のデータの索引付けに必要な全てのコントロールを持つことになる。

##ドキュメントの部分更新

Both search engines support partial document update. This is not the true partial document update that everyone has been after for years – this is really just normal document reindexing, but performed on the search engine side, so it feels like a real update.

両方の検索エンジンがドキュメントの部分更新をサポートする。これは皆が期待する真のドキュメント部分更新ではない。単なる通常のドキュメントのインデックス再作成である。しかし検索エンジンサイドで実行されるので本物の更新に感じる。

###Solr

Let’s start from the requirements – because this functionality reconstructs the document on the server side you need to have your fields set as stored and you have to have the _version_ field available in your index structure. Then you can update a document with a simple API call, for example:

要件から始めよう。この機能はサーバ側のドキュメントを再構成するのでフィールドの集合が格納済みで_version_フィールドがインデックス構成に存在しなければならない。そうすれば単一のAPI呼出によりドキュメントを更新することが可能だ。

例：

	curl 'localhost:8983/solr/update' -H 'Content-type:application/json' -d '[{"id":"1","price":{"set":100}}]'

###ElasticSearch

In case of ElasticSearch you need to have the _source field enabled for the partial update functionality to work. This _source is a special ElasticSearch field that stores the original JSON document.  Theis functionality doesn’t have add/set/delete command, but instead lets you use script to modify a document. For example, the following command updates the same document that we updated with the above Solr request:

ElasticSearchの場合、部分更新の機能が働くためには_sourceフィールドが許可されている必要がある。この_sourceは特別なElasticSearchのフィールドでオリジナルのJSONドキュメントを格納する。この機能にはadd/set/delete命令は存在しない。その代わりにドキュメントを変更するスクリプトの使用を許可する。例えば次のコマンドは上のSolrのリクエストにて更新した物と同じドキュメントを更新する

	curl -XPOST 'localhost:9200/sematext/doc/1/_update'-d '{
	    "script" : "ctx._source.price = price",
	    "params" : {
	        "price" : 100
	    }
	}'

##多国語データの取扱

As we mentioned previously, and as you probably know, both ElasticSearch and Solr use Apache Lucene to index and search data. But, of course, each search engine has its own Java implementation that interacts with Lucene. This is also the case when it comes to language handling. Apache Solr 4.0 beta has the advantage over ElasticSearch because it can handle more languages out of the box. For example, my native language Polish is supported by Solr out of the box (with two different filters for stemming), but not by ElasticSearch. On the other hand, there are many plugins for ElasticSearch that enable support for languages not supported by default, though still not as many as we can find supported in Solr out of the box.  It’s also worth mentioning there are commercial analyzers that plug into Solr (and Lucene), but none that we are aware of work with ElasticSearch…. yet.


以前述べたとおり、そして恐らくご存知のとおり、ElasticSearchとSolrはApache Luceneをインデックス作成とデータの検索に用いている。しかし、もちろん各検索エンジンはそれ自身のJava実装を持ちそれがLuceneと相互作用している。これが各国語の取扱においても同じである。Apache Solr 4.0βがElasticSearchに対して優位である。そのままでより多くの言語を扱うことができるからだ。例えば筆者のネイティブ言語であるポーランド語はSolrでは最初からサポートされている。（2つの異なるstemming用フィルタ付きで。）しかしElasticSearchはそうではない。一方で、ElasticSearchにはデフォルトではサポートされない言語をサポートするプラグインが多く有る。それでもその数はSolrが最初からサポートしている数程は無い。もう一つ記述しておく価値があるのはSolr（とLucene）には商業解析器がある。しかしElasticSearch向けの物を筆者はまだ知らない。

##サポートされる自然言語
For the full list of languages supported by those two search engine please refer to the following pages:
2つの検索エンジンにてサポートされる言語の完全なリストについては次のページを参照して欲しい。

Apache Solr
    <a href="http://wiki.apache.org/solr/LanguageAnalysis" >http://wiki.apache.org/solr/LanguageAnalysis</a>

ElasticSearch

Analyzers: [http://www.elasticsearch.org/guide/reference/index-modules/analysis/lang-analyzer.html](http://www.elasticsearch.org/guide/reference/index-modules/analysis/lang-analyzer.html)
Stemming: [http://www.elasticsearch.org/guide/reference/index-modules/analysis/stemmer-tokenfilter.html](http://www.elasticsearch.org/guide/reference/index-modules/analysis/stemmer-tokenfilter.html),
 [http://www.elasticsearch.org/guide/reference/index-modules/analysis/snowball-tokenfilter.html](http://www.elasticsearch.org/guide/reference/index-modules/analysis/snowball-tokenfilter.html) それに [http://www.elasticsearch.org/guide/reference/index-modules/analysis/kstem-tokenfilter.html](http://www.elasticsearch.org/guide/reference/index-modules/analysis/snowball-tokenfilter.html)

##解析チェインの定義

Of course, both Apache Solr and ElasticSearch allow you to define a custom analysis chain by specifying your own analyzer/tokenizer and list of filters that should be used to process your data. However, the difference between ElasticSearch and Solr is not only in the list of supported languages. ElasticSearch allows one to specify the analyzer per document and per query. So, if you need to use a different analyzer for each document in the index you can do that in ElasticSearch. The same applies to queries – each query can use a different analyzer.

もちろん、Apache SolrとElasticSearchの両方がカスタム解析器のチェインを定義できる。あなたのデータ処理に使用される解析器及びトークナイザ、フィルタのリストを指定する。しかしElasticSearchとSolrの違いはサポートされる言語のリスト中だけではない。ElasticSearchはドキュメント毎、クエリ毎に解析器を指定することが可能だ。そのためインデックス中の個別のドキュメントに異なる解析器の使用が必要なら、ElasticSearchではそれが可能だ。同じことがクエリにも言える。各クエリは異なる解析器を使用できる。

##結果のグルーピング

One of the most requested features for Apache Solr was result grouping. It was highly anticipated for Solr and it is still anticipated for ElasticSearch, which doesn’t yet have field grouping as of this writing.  You can see the number of +1 votes in the following issue: https://github.com/elasticsearch/elasticsearch/issues/256.  You can expect grouping to be supported in ElasticSearch after changes introduced in 0.20. If you are not familiar with results grouping – it allows you to group results based on the value of a field, value of a query, or a function and return matching documents as  groups. You can imagine grouping results of restaurants on the value of the city field and returning only five restaurants for each city. A feature like this may be handy in some situations. Currently, for the search engines we are talking about, only Apache Solr supports results grouping out of the box.

Apache Solrに最もリクエストされた機能の1つが結果のグルーピングである。Solrにとって最も期待された機能であり、ElasticSearchにとって今でも期待されている。これを書いている時点ではElasticSearchはフィールドのグルーピングを持っていない。次のissueを見れば何個の+1が投票されているかわかるだろう。https://github.com/elasticsearch/elasticsearch/issues/256
ElasticSearchには0.20の変更が紹介された時にはサポートされることが期待できあるであろう。
結果のグルーピングを知らない人に説明すると、それはフィールドやクエリ、または関数の値によって結果をグループに分けることが可能になる。そしてマッチしたドキュメントをグループにして返す。例えばレストランの結果を街のフィールドでグルーピングし、各街の5つのレストランだけを返す。このような機能はいくつかのシチュエーションにて便利だろう。現在はApache Solrのみがそのままの状態で結果のグルーピングを持っている。

##Prospective Search

One thing Apache Solr completely lacks when comparing to ElasticSearch is functionality called Percolator in ElasticSearch. Imagine a search engine that, instead of storing documents in the index, stores queries and lets you check which stored/indexed queries match each new document being indexed. Sound handy, right?  For example, this is useful when people want to watch out for any new documents (think Social Media, News, etc.) matching their topics of interest, as described through queries. This functionality is also called Prospective Search, some call it Pub-Sub as well as Stored Searches.  At Sematext we’ve implemented this a few times for our clients using Solr, but ElasticSearch has this functionality built-in.  If you want to know more about ElasticSearch Percolator see http://www.elasticsearch.org/blog/2011/02/08/percolator.html.

Apache SolrがElasticSearchに比べて完全に欠いているものの1つはElasticSearchにてパーコレータと呼ばれている機能である。例えばインデックスにドキュメントを格納するのではなく、クエリを格納し、新しいドキュメントのインデックス作成が行われる度に格納された（インデックスされた）クエリがマッチするかどうかチェックする検索エンジンを考える。良さそうに思わないだろうか？
これが便利なのは、例えばユーザが全ての新しいドキュメント（ソーシャルメディアやニュース等を考えよう）がユーザの興味が有るトピックに属するか、先程記述したクエリを通してチェックしたい場合等だ。この機能はまたProspective Searchとも呼ばれる。またある人はPub-Subと呼ぶし、stored searchとも呼ばれる。Sematextでは我々のSolrを使う顧客のためにこれを何度か実装したことがある。しかしElasticSearchは最初から組み込みでこの機能を持っている。もしElasticSearchのパーコレータについてもっと知りたければ次を参照すること。
[http://www.elasticsearch.org/blog/2011/02/08/percolator.html](http://www.elasticsearch.org/blog/2011/02/08/percolator.html)

##次回予告

In the next part of the series we will focus on comparing the ability to query your indices and leverage the full text search capabilities of Apache Solr and ElasticSearch. We will also look at the possibility to influence Lucene scoring algorithms during query time. Till next time :)

このシリーズの次回ではApache SolrとElasticSearch上のインデックスに対するクエリの能力と全文検索の能力についてフォーカスする。またクエリ時におけるLuceneのスコアリングアルゴリズムの操作能力についても調べるつもりである。それでは次回まで :)

@kucrafal, @sematext