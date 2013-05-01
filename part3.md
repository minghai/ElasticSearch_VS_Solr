#Solr vs ElasticSearch: Part 3 – 検索

October 1, 2012 by Rafał Kuć 8 Comments

In the last two parts of the series we looked at the general architecture and how data can be handled in both Apache Solr 4 (aka SolrCloud) and ElasticSearch and what the language handling capabilities of both enterprise search engines are like. In today’s post we will discuss one of the key parts of any search engine – the ability to match queries to documents and retrieve them.

このシリーズの前の二回にてApache Solr 4 (SolrCloudとも呼ばれる）とElasticSearchにおける全体のアーキテクチャとデータがどのように扱われるか、そして両者の検索エンジンの言語の取扱能力がどのようなものであるかを調べた。今日の記事では検索エンジンのキーとなる部分の1つであるドキュメントに対してクエリをマッチし、結果を返す能力について議論しよう。

1. Solr vs. ElasticSearch: Part 1 – Overview
2. Solr vs. ElasticSearch: Part 2 – Indexing and Language Handling
3. Solr vs. ElasticSearch: Part 3 – Searching
4. Solr vs. ElasticSearch: Part 4 – Faceting
5. Solr vs. ElasticSearch: Part 5 - Management API Capabilities
6. Solr vs. ElasticSearch: Part 6 – User & Dev Communities Compared

##全体的なアプローチ

Both search engines expose their search APIs via HTTP. If you are not familiar with Solr or ElasticSearch, here are a few simple examples of what Apache Solr and ElasticSearch queries look like:

両方の検索エンジンがHTTPを経由して検索APIを公開している。SolrやElasticSearchを良く知らない場合のため、ここからいくつか、Apache SolrとElasticSearchのクエリがどのような物か、簡単な例を示す。

###Solr
	curl -XGET 'http://localhost:8983/solr/sematext/select?q=post_date:[2012-09-10T12:00:00Z+TO+2012-09-10T15:00:00Z]'

###ElasticSearch

	curl -XGET http://localhost:9200/sematext/_search?pretty=true -d '{
	    "query" : {
	        "range" : {
	            "post_date" : {
	                "from" : "2012-09-10T12:00:00",
	                "to" : "2012-09-10T15:00:00"
	            }
	        }
	    }
	}'

As you can see the ElasticSearch query is more structured allowing for more precise control of what you are trying to get – similar to Lucene queries. Solr on the other hand uses a query parser to parse your query out of the textual value of the “q” URL parameter (n.b. you can use query parser in ElasticSearch too). But as one can see on Solr mailing lists, many new users have problems because of such approach – they are overwhelmed with all the options and parameters. At the same time, Solr does make simple queries with boosting and extended dismax parser very easy to do, although that comes at a price. If you want to have a higher degree of control over your query, you are (in most cases) forced to use local params that, while powerful, can be quite hard for users not familiar with its cryptic syntax.

ご覧のとおり、ElasticSearchのクエリはより構造化され希望の結果取得のため正確なコントロールを可能とする。Luceneのクエリに似ている。一方SolrはクエリパーザをURLパラメータの"q"の値から取り出されたテキスト値をクエリとしてパースする。（注意：ElasticSeachでもクエリパーザは利用できる）しかしSolrのメーリングリストで確認できるように、多くの新しいユーザがそのやり方のせいで問題を抱えている。彼らは全てのオプションとパラメータに困惑している。同時に、Solrは簡単なクエリをboostingと拡張されたdismaxパーザで簡単に実行できる。しかしそれには相応の負担と引き換えになる。もし高度なコントロールをクエリに対して行いたい場合に、（多くの場合）ローカルパラメータを使用せねばならず、（パワフルではあるが）その暗号のような文法に親しみの無いユーザにはとても大変である。

To sum up our short introduction – both search engines give you a similar degree of control when it comes to querying, although if you want to create your queries from scratch and control every aspect of them, just as you would when using Lucene directly, ElasticSearch is the way to go, not because Solr doesn’t let you, but the structured JSON way of querying ElasticSearch is a better fit in that case and feels more intuitive.

我々の短いイントロをまとめると、両方の検索エンジンが同じ程度のコントロールをクエリに対して提供する。しかしスクラッチからクエリを作成し、Luceneを直接利用する様に全ての局面をコントロールしたい場合、ElasticSearchを選択するべきだ。Solrではできないからではなく、ElasticSearchの提供する構造化されたJSONの仕組みがそのケースにはよりフィットし、より直感的に感じられるからだ。

##全文検索

In this section we try to compare search capabilities of both both Apache Solr and ElasticSearch. This is by no means a comprehensive tutorial of all the features that both search engines expose, but rather  a simple comparison of similarities and difference of them.

このセクションではApache SolrとElasticSearchの両方の検索能力を比較することを試みる。これはもちろん包括的な両方の検索エンジンの持つ全ての機能のチュートリアルではない。どちらかと言えば簡単な類似点と相違点の比較に過ぎない。

###検索

Of course, both Apache Solr and ElasticSearch enable you to run standard queries such as Boolean queries, phrase queries, fuzzy queries, wildcard queries, etc. You can combine them into multiple Boolean phrases using Boolean operators. In addition to that, both engine let one specify query-time boosts and control how score is calculated during search execution.

もちろん、Apache SolrとElasticSearchの両者共、標準的な検索が可能だ。ブーリアンクエリ、フレーズクエリ、ファジークエリ、ワイルドカードクエリ等である。Boolean演算子を用いてそれらを結合して複数のブーリアンフレーズに結合することも可能だ。加えて両方のエンジンはクエリ時のboostを指定し、検索実行中にどのようにスコアが計算されるかをコントロールすることができる。

###Span Queries

If you are not familiar with span queries here is a one-sentence description: Lucene provides span queries in order to enable searching documents with position requirements, but not necessarily appearing one after another like in the phrase query. And now the comparison:

スパンクエリを知らない人に簡単に説明すると、Luceneが提供するスパンクエリは位置を必要とするドキュメント検索を可能にする。しかしフレーズクエリのように連続して表れる必要は無い。
では比較してみよう

####Solr

Update: As Erik Hatcher noticed support for span queries is already there in Apache Solr (SOLR-2703). We can use span queries by using the surround query parser.

更新： Erik Hatcherによるとスパンクエリは既にApache Solrに入っている。(SOLR-2703)。クエリパーザにて囲むことでスパンクエリを使用可能だ。

####ElasticSearch

ElasticSearch has the support for Lucene SpanNearQuery, SpanFirstQuery, SpanTermQuery, SpanOrQuery and SpanNotQuery. With these queries you can construct different span queries similar to what you can do with Lucene.

ElasticSearchはLuceneのSpanNearQuery、SpanFristQuery、SpanTermQuery、SpanOrQuery、そしてSpanNotQueryをサポートする。これらのクエリを用いてLuceneで行えるような異なるスパンクエリを構築することが可能だ。

###More Like This

“More like this” (aka MLT) functionality lets you to get documents similar to a given query according to some assumptions and parameters used to find documents that are similar to one another. Both search engines have the ability to run MLT queries. In Solr, MLT  query is implemented as a search component. On the other hand there is ElasticSearch where more like this is just another type of query one can construct using JSON. When comparing parameters available in both search servers it seems that ElasticSearch provides slightly more control over more like this functionality with features like specifying a set of words that shouldn’t be taken into consideration and the percentage of terms to match on.

"More like this"(またはMLT)とはクエリにて与えらたドキュメントに、いくつかの想定で似たドキュメントを得る機能だ。相似なドキュメントの発見にはパラメータが用いられる。両方の検索エンジンがMLTクエリを実行可能だ。SolrではMITクエリは検索コンポーネントとして実装されている。一方、ElasticSearchではJSONを用いて構築する1つの種類のクエリとなる。両者の検索サーバに存在するパラメータを比較するとElasticSearchはわずかに多くのコントロールを提供する。例えば考慮に入れてはいけない単語の集合を指定したり、マッチする項目の割合を指定可能だ。

###Did You Mean

“Did you mean” (aka DYM) functionality makes it possible to correct users’ query typos and spelling mistakes and suggest corrected queries. For example, for a misspelled phrase “saerch problems” our Researcher module on http://search-lucene.com (which is a kind of a did you mean module) works like this:

“Did you mean”（またはDYM）とはユーザのクエリのタイプミスやスペリングミスを直し、訂正されたクエリを提案する機能だ。例えば“saerch problems”といったミススペルのフレーズにhttp://search-lucene.comに存在する我々の研究者モジュール（Did you meanモジュールのような物だ）はこのような動作を行う。

Let’s see what Solr and ElasticSearch have to offer here.

SolrとElasticSearchにどのような機能が提供されているか見てみよう。

####Solr

Solr exposes spell check component API, which is built on top of Lucene spell checker module. Before Solr 4.0 the spell checker required its own index that, while built automatically by Solr, was another moving piece and potential inconvenience.  Now there is a DirectSolrSpellchecker implementation available which can give spell checker suggestion based on the index you are using for search instead of relying on the side-car spell checker index. Solr spell checker supports distributed search and has numerous parameters which allow control over its behavior, like number of suggestion, collation properties, accuracy, etc.

SolrはスペルチェックコンポーネントAPIを提供する。それはLuceneのスペルチェッカーモジュールの上に構築されている。Solr 4.0以前ではスペルチェッカーはそれ自身のインデックスを必要とし、Solrにより自動的に構築されるが、別に動作する部品であり恐らく不便なものだった。今はDirectSolrSpellcheckerの実装が存在し、検索に使用しているインデックスに基づきスペルチェッカーが提案を行うことが可能だ。スペルチェッカー用のインデックスを必要としない。Solrのスペルチェッカーは分散検索をサポートし、多くのパラメータを持つことでその動作をコントロールできる。例えば提案の数や照合のプロパティや正確性等である。

####ElasticSearch

Unfortunately, ElasticSearch doesn’t offer did you mean functionality out of the box. There is issue  #911 currently open, so we can expect that module in one of the future releases. Although we’ll be talking about plug-ins in the last part of the Solr vs ElasticSearch series, if you need did you mean functionality in ElasticSearch you can use the Suggest Plugin developed by @spinscale (https://github.com/spinscale/elasticsearch-suggest-plugin).

残念だがElasticSearchは“Did you mean”の機能をそのままでは提供しない。Issue#911が現在オープンしており将来のリリースでそのモジュールを期待できるだろう。Solr vs ElasticSearchの最後のパートにてプラグインについて話すが、もしあなたが“Did you mean”の機能をElasticSearchにて必要とするのならば@spinscaleが開発したSuggestプラグインを利用することが可能だ。(https://github.com/spinscale/elasticsearch-suggest-plugin).

###Nested Queries

As we already wrote, ElasticSearch supports indexing of nested document which Solr doesn’t support. In order to query nested documents ElasticSearch exposes nested query type. This query is run against nested documents, but as the result we get the root documents. In addition to that, you can also set how scoring of the root document is affected.

既に書いたがElasticSearchは入れ子のドキュメントのインデックス作成をサポートする。Solrはサポートしない。入れ子ドキュメントをクエリするためにElasticSearchは入れ子クエリタイプを公開している。このクエリは入れ子ドキュメントに対し実行されるが結果としてルートドキュメントを得ることになる。これに加えてルートドキュメントのスコアリングにどのように影響されるかも設定可能だ。


###Parent – Child Relationship Queries
####Solr

In Apache Solr there is no functionality called parent - child, instead of that we have the possibility to use joins. Solr joins are specified in local params format and look like this:

Apache Solrでは親子と呼ばれる機能は無い。その代わりにjoinを使える可能性が存在する。Solrのjoinはローカルパラメータの形式で指定され以下のようになる

	q={!join from=parent to=id}color:Yellow

The above query says that we want to get all parent documents that have child documents that have the Yellow term in the color field. The join should be done on parent field in the children to the id field in the parent document.

上のクエリではcolorフィールドにYellowを持つ子ドキュメントを持つ全ての親ドキュメントを得る。joinは子ドキュメントのparentフィールドから親ドキュメントのidフィールド上にて行われるはずだ。

####ElasticSearch

ElasticSearch lets you use two type of queries – has_children and top_children queries to operate on child documents. The first query accepts a query expressed in ElasticSearch Query DSL as well as the child type and it results in all parent documents that have children matching the given query. The second type of query is run against a set number of children documents and then they are aggregated into parent documents. We are also allowed to choose score calculation for the second query type.

ElasticSearchは2つのタイプのクエリを使用可能だ。has_childrenとtop_childrenのクエリは子ドキュメント上にて命令される。最初のクエリはElasticSearch Query DSLまたはchildタイプで表現されたクエリを受け付け、与えられたクエリにマッチする子ドキュメントを持つ全ての親ドキュメントを結果とする。2つ目のタイプのクエリはいくつかの子ドキュメントの集合に対し実行されそれらは親ドキュメントの中に集約される。2つ目のクエリタイプに対してもスコア演算を選択することが許可されています。

###Filtering And Caching Control
####Solr

Of course Solr lets you to narrow results of your query execution with filters. You can filter documents based on a single value, Boolean expression, query, field existence, geographical location and many, many more. In addition to that you can use local params and construct complicated queries like:

もちろんSolrはクエリの実行結果をフィルタにて制限できる。単一の値、ブール式、クエリ、フィールドの存在、地理上の位置、それにもっともっと多くのことを基準としてドキュメントをフィルタ可能だ。加えてローカルパラメータを用いて複雑なクエリを構築することができる。
例：

	fq={!frange l=10 u=30}if(exists(promotionPrice),sum(promotionPrice,dailyPrice),sum(price,dailyPrice))

####ElasticSearch

ElasticSearch, similar to Solr, lets you use many filter types, which are similar to filters, so we’ll skip mentioning them all. However, in addition to similarities with Solr, there are also some differences like supports for filters run against nested documents and children documents. ElasticSearch can also use scripts to filter documents with the script filter.


ElasticSearchはSolrに似て多くのフィルタタイプを用いることができる。それはフィルタと同様なので全てを語る必要がない。しかしSolrに対する類似性に加えて、いくつかの異なる点が存在する。例えば入れ子ドキュメントや子ドキュメントに対して実行するフィルタのサポートである。ElasticSearchはまたスクリプトフィルタを用いてドキュメントのフィルタリングにスクリプトを使用することも可能だ。

###Filter Cache Control
Both ElasticSearch and Apache Solr can control if the filter should or shouldn’t be cached, but in addition to that Solr lets you control the order of filters execution (the non cached ones). Its a great feature of Solr, because if you know that one of your filters is a performance killer, you can set its execution after all other filters and that way it’ll only work on the subset of the original result set.

ElasticSearchとApache Solrの両方でフィルタがキャッシュされるべきかそうでないかをコントロール可能だ。加えてSolrはフィルタの実行順をコントロールできる。（キャッシュされていないもののみ）これはSolrの優れた機能でもしあなたのフィルタの1つがパフォーマンスに悪い影響を与えることが事前にわかっていればそのフィルタの実行を他の全ての後に設定可能だ。そうすることでそのフィルタはオリジナルの実行結果の部分集合のみを対象に実行されることになる。

###Score Calculation Control

In both engines we are more or less allowed to control how scores for documents are calculated. In Solr this is mostly done by using function queries and different boosts and queries made using local params. In ElasticSearch we can use different query types which allow us to give specific scores to some of the documents (for example ones matching a certain filter) or calculate score on the basis of used script.

両方のエンジンにおいて多かれ少なかれドキュメントに対するスコアがどのように演算されるかをコントロールすることが可能だ。Solrではこれは多くの場合においてファンクションクエリか複数の異なるboostを用いて行われ、クエリはローカルパラメータを使用して作成される。ElasticSearchでは異なるクエリタイプを用い、あるドキュメントに対して特定のスコアを与えることを可能にする。（例としていくつかのフィルタにマッチするドキュメント。）または使用したスクリプトによりスコアを演算することも可能だ。

###Real Time Get

Real time get allows us to retrieve a document using its identifier as soon as it was sent for indexing even if it hasn’t yet been hard committed. Both ElasticSearch and Apache Solr return the newest document, even if it wasn’t indexed. But lets go into specifics.

リアルタイムGetはドキュメントのIDを用いて、それがインデックス作成のために送られた時に、例えハードコミットが行なわれていなくてもドキュメントの回収を可能にする。ElasticSearchとApache Solrの両方が最新のドキュメントを例え索引付けされていなくても返すことが可能だ。しかし具体的に見てみよう。

####Solr

Introduction of so called transaction log in Solr 4.0 allowed for the real time get functionality. Basically, the real time get looks for the newest version of the document in the transaction log first and returns it as a result of such call (if it is found, of course). If it is not found the real time get handler gets the document using the latest opened searcher available. Keep in mind that in order to return the newest version of the document in near real time manner Solr doesn’t need to reopen the index after indexing, so this functionality is useful even if you don’t reopen your searcher every second.

Solr4.0におけるトランザクションログと呼ばれる機能の紹介によりリアルタイムGetが利用可能になった。基本的にリアルタイムGetはドキュメントの最新バージョンをトランザクションログから最初に探しその呼出の結果として返す（もちろん見つかった場合に）もし見つからなかったらリアルタイムGetハンドラは最も最近に開かれたsearcherが存在するドキュメントを得る。覚えておいて欲しいのはほぼリアルタイムの様式最新バージョンのドキュメントを返すためにSolrはインデックス作成後にインデックスを再び開く必要がない。従ってこの機能はsearcherを毎秒再開しなくても有効である。

####ElasticSearch

ElasticSearch also uses transaction log and because of that the real time get is not affected by the refresh rate of your indices. In addition to returning the document itself ElasticSearch exposes a few other API parameter that allow you to specify if the request should go to the primary or local shard (or even a custom one). You can also use routing with real time get to route the request to one specific shard if you know which shard should have the appropriate document. The real time get API of ElasticSearch also allows to check if the document exists using HTTP HEAD method, for example:

ElasticSearchもまたトランザクションログを用いているのでリアルタイムGetはインデックスのリフレッシュレートに依らない。ドキュメント自身を返すのに加えてElasticSearchはいくつかの他のAPIパラメータを公開しておりそれを用いることでリクエストがプライマリかまたはローカルのShardへ（またはカスタムShardにさえ）送られるべきかを指定することが可能だ。またルーティングをリアルタイムGetと共に用いることでどのshardが適切なドキュメントを持っているか知っている場合にリクエストを1つの特定のshardへ誘導可能である。ElasticSearchのリアルタイムGet APIはまたドキュメントが存在するかどうかをHTTP HEADメソッドにて確認可能だ。例として

	curl -XHEAD 'http://localhost:9200/sematext/blog/123456789'

###Aliasing

One of the things introduced in Apache Solr 4.0 and no available in ElasticSearch right now is the ability to transform result documents. First of all Solr allows you to alias returned fields, so for example you can return field price_usd or price_eur as price depending on your needs. The second thing is the ability to return values returned by functions as a (pseudo) field in the result (or fields). Solr also has the ability to return fields which start with a given prefix (for example all fields starting with price). Apart from the ability to get a function value as a field added to matched documents on the fly other functionalities are not ground breaking, though they can be handy in some cases.

Apache Solr 4.0にて紹介されElasticSearchに現時点ではまだ存在しない機能として結果のドキュメントの変換がある。最初にSolrは返り値のフィールドに別名を付けることができる。そのため例えば必要に応じてpriceフィールドとしてprice_usdかprice_eurを返せる。2つ目に関数の返り値を（見せかけの）フィールドとして結果（またはフィールド）としてか返すことが可能だ。Solrはまた与えられた接頭子で始まるフィールド（例えばpriceで始まる全てのフィールド）を返す機能も存在する。
 
###他

One of the things we always mention when talking about the differences between Apache Solr and ElasticSearch, at least when it comes to query handling, is the possibility of specifying the analyzer during query time. But lets start from the beginning. In Solr, you have to create the schema.xml file which holds the information about the index structure as well as query and index-time analyzers for fields. Similarly, in ElasticSearch, you can create mappings and define analyzers. At query-time Solr will choose the right analyzer for each field and use it.  ElasticSearch will do the same with one major difference. In ElasticSearch you can change the analyzer and specify the analyzer you want to use for analysis at query-time. For example, this is very useful when you know the language of the query because then you can choose the most language-appropriate analyzer on the fly.  We have made use of this in combination with our Language Identifier.

Apache SolrとElasticSearchの違いについて我々が常に伝えていることの1つは、少くともクエリの取扱に関しては、クエリ時中に解析器を指定する可能性である。しかしまずは最初から始めよう。Solrではインデックス構造、それにクエリとフィールドのインデックス作成時の解析器に関する情報を持つschema.xmlファイルを作成する必要がある。同様にElasticSearchではマッピングを作成し解析器を定義する。クエリ時にSolrは各フィールドに対し適切な解析器を選択し、使用する。ElasticSearchも同じことをするが大きな違いが1つある。ElasticSearchでは解析器の変更と解析に必要な解析器の指定がクエリ時に行える。例えばクエリ対象の言語を知っている場合にこれはとても便利だ。最も言語に適切な解析器を動的に選択できるからだ。我々はこの機能を自身の言語特定機構といっしょに使用している。

##まとめ

As you can see, both ElasticSearch and Apache Solr expose lots of functionality when it comes to handling your search queries, and we barely scratched the surface here. Of course, each of them has some features that the other one doesn’t have, but Solr and ElasticSearch are competing for mind and market share, and are both rapidly evolving and improving, so we can expect more features from both of them in the future. In the next, fourth part of the series we will concentrate on the faceting capabilities of Apache Solr and ElasticSearch.  Stay tuned.  In the mean time, you can follow @sematext and tell us what you want us to cover.

ご覧のとおり、ElasticSearchとApache Solrの両方が検索エンジンの取扱に対して多くの機能を公開している。そしてここではその表面をわずかに露にしたに過ぎない。もちろんそれぞれが他方の持っていない機能を持っておいるが、SolrとElasticSearchは心理と市場のマーケットシェア上で競争をしている。そして両方が急速に進化し向上している。従って将来に渡りその両方からより多くの機能を期待することができるだろう。次回、このシリーズの第4回ではApache SolrとElasticSearchのファセットの機能についてフォーカスする。続けてご覧頂きたい。それまでは@sematextをfollowして我々に何をカバーして欲しいか伝えたて欲しい。