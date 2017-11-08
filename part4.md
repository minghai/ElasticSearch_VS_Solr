# Solr vs ElasticSearch: Part 4 – Faceting

October 30, 2012 by Rafał Kuć 2 Comments

Solr 4 (aka SolrCloud) has just been released, so it’s the perfect time to continue our ElasticSearch vs. Solr series. In the last three parts of the ElasticSearch vs. Solr series we gave a general overview of the two search engines, about data handling, and about their full text search capabilities. In this part we  look at how these two engines handle faceting.

Solr 4 (またはSolrCloud）がリリースされた。ElasticSearch vs. Solrのシリーズを続けるのに完璧な時だと言えるだろう。ElasticSearch  vs. Solrシリーズの以前の3回では2つの検索エンジン全体の概観、データの取扱、全文検索能力について話した。このパートでは2つのエンジンがどのようにファセットを取り扱うかを調べる。

1. Solr vs. ElasticSearch: Part 1 – Overview
2. Solr vs. ElasticSearch: Part 2 - Indexing and Language Handling
3. Solr vs. ElasticSearch: Part 3 – Searching
4. Solr vs. ElasticSearch: Part 4 – Faceting
5. Solr vs. ElasticSearch: Part 5 - Management API Capabilities
6. Solr vs. ElasticSearch: Part 6 – User & Dev Communities Compared

## Faceting

When it comes to faceting, both Solr and ElasticSearch have some faceting methods that other search engine does not.  Both search engines allow you to calculate facets for a given field, numerical range, or date range. The key differences are in the details, of course – in the control of how exactly the facets are calculated, in the memory footprint, and whether we can change the calculation method. In most cases ElasticSearch allows more control over faceting, however Solr has some serious advantages, too.  Lets get into details of each of the methods.

facetingについてはSolrとElasticSearchの両方が他の検索エンジンが持たないいくつかのfacetingメソッドを持つ。両者の検索エンジン共が与えられたフィールド、数値範囲、日付範囲のファセットを演算することが可能である。キーとなる違いはもちろん詳細の中になり、実際にファセットがどのように演算さるかのコントロール、メモリフットプリント、そして演算手法の変更が変更できるかである。多くの場合にElasticSearchがfacetingにおいて多彩なコントロールを可能にする。しかしSolrもいくらかの重要な優位点を持っている。個々のメソッドの詳細に入ってみよう。

## Term Faceting

This method of faceting allows one to get information about the number of term occurrences in a certain field.


このfacetingのメソッドはあるフィールド内であるタームがいくつあるかの情報を得ることを可能にする。

### Solr
Solr let’s you control how many facets are returned, how they are sorted, the minimum quantity required, and so on. In addition to that, in Solr field faceting, you can choose between a couple of different methods for computing facets.  One of these method should be used for fields with a high number of distinct terms, while the second method is best used in the opposite scenario – when you expect relatively few distinct terms in a field being faceted on.


Solrはファセットがいくつ返されるか、それらがどのように格納されるか、最小必要数等をコントロールできる。加えてSolrのフィールドfacetingではファセットの演算法を2、3の異なる手法の間から選択することができる。これらのメソッドの1つが最も数の多いタームに用いられ、2つ目の手法は逆のシナリオに適している。フィールド中の相対的に少ないタームがファセットされて欲しい場合である。

### ElasticSearch
On the other side we have ElasticSearch which allows us to do all that Solr can do (in terms of faceting calculation, not the calculation methods), but in addition it also let’s us exclude specific terms we are not interested in and use regular expressions to define which terms will be included in faceting results. In addition to that we can combine term faceting results from different field automatically or just use scripts to modify the fields values before the calculation process steps in

一方、ElasticSearchではSolrができること全てが可能である。（ファセット演算に関してであり演算手法についてではない）加えて興味ない特定のタームを排他することが可能で、またファセットの結果に含まれるタームの定義に正規表現を用いることも可能だ。さらに異なるフィールドのタームfacetingの結果を自動的に連結したり、演算プロセスに入る前にフィールドの値を変更するスクリプトを使用したりもできる。

## Query Faceting

Both Solr and ElasticSearch allow calculating faceting for arbitrary query results. In both cases queries can be expressed in the query API of the search engine which we use. For example, in ElasticSearch you can use the whole query DSL to calculate faceting results on them.


SolrとElasticSearchの両方が任意のクエリの結果に対してファセットを演算できる。両者共クエリは検索エンジンのクエリAPIにて表現できる。例えばElasticSearchではクエリDSL全体をその結果のファセットの算出に用いることが可能だ。

## Range Faceting
Range faceting lets you get the number of documents that match the given range in a field. Both engines allow for range faceting although in different fashion.

レンジファセットはあるフィールドにおいて与えられた範囲にマッチするドキュメントの数を返す。両方のエンジンがレンジファセットを可能だが異なる方法で行う。

### Solr

Apache Solr lets you define the start value, end value, and the gap (with some adjustments like inclusion of values at the end of the ranges) and calculate all the ranges defined by that.

Apache Solrは始まりと終わりの値、ギャップ（といくつかの調整するもの、例えば範囲の終わりの値は含まれるか否か等）を指定し、それにより定義された全ての範囲を演算する。

### ElasticSearch

ElasticSearch takes a different approach – it lets you specify set of ranges and returns document counts as well as aggregated data. In addition to that, ElasticSearch let’s you specify a different field to check if a document falls into a given range and a different field for the aggregated data calculation. Furthermore, you can modify the field and aggregated data with a script. And that’s not all – in addition to the above method of range faceting ElasticSearch also supports the so called histogram calculation.  This is similar to the Apache Solr approach – for a given field you can get a histogram of values. However, ElasticSearch doesn’t let you control the start and end like Solr does, but only the gap.

ElasticSearchは異なるアプローチを取る。レンジの集合を指定しドキュメントのカウントと集約されたデータを返す。加えてElasticSearchは異なるフィールドを指定し、ドキュメントが与えられたレンジに当てはまるかチェックする他、集約データの演算に他のフィールドを指定可能だ。さらにフィールドと集約データをスクリプトで変更可能だ。それだけでなく上記のレンジファセットに加えてElasticSearchヒストグラム演算と呼ばれる物も利用できる。これはApache Solrのアプローチに似ていて与えられたフィールドの値のヒストグラムを取得できる。しかしElasticSearchはSolrのように開始値と終了値のコントロールはできず、ギャップのみである。


## Date Faceting

Again, both search engines support faceting on date based fields.

同様に両方の検索エンジンはフィールドに基づき日付のファセットをサポートする。

### Solr

Date faceting in Apache Solr is quite similar to the range faceting, although it is calculated on fields of  solr.DateField type. You have the same control and use similar parameters as withing the range faceting so I’ll omit describing it.

Apache SolrのDateファセットはレンジファセットに全く同様だがsolr.DateFieldタイプのフィールド上にて演算される。レンジファセットと同様のコントロールが可能で、同様のパラメータが使用できる。そのためここでは記述を省略する。

### ElasticSearch

On the other hand, we have ElasticSearch with its date faceting which is an enhancement over the standard histogram faceting. It supports interval specification with date specific parameters like for example: year, month, or day. In addition to that, ElasticSearch lets you specify the time zone to be used in computation and of course manipulate the calculation with the use of a script.

一方、ElasticSearchではdateファセットは標準のヒストグラムファセットの拡張である。日付を特定するyear、month、またはdayのようなパラメータと間隔の指定をサポートする。加えてElasticSearchは演算に用いられるタイムゾーンの指定が可能で、もちろんスクリプトを利用して演算を操作することも可能だ。

### Decision Tree Faceting – Solr Only

One of the things that ElasticSearch lacks and that is present in Solr is the pivot faceting aka decision tree faceting. It basically lets you calculate facets inside a parents facet. For example, this is what pivot faceting results look like in Solr (n.b. this example is trimmed for this post) :

現時点でSolrに存在するがElasticSearchが欠く物の1つがpivotファセット、または決定木ファセットと呼ばれるものだ。これは親のファセット内部でファセットを演算できる。例えばこれがSolrにおけるpivotファセットがどのように見えるかの例だ。（この例はこの記事のために一部削除してあることに注意せよ）

	<?xml version="1.0" encoding="UTF-8"?>
	<response>
	<lst name="responseHeader">
	  <int name="status">0</int>
	  <int name="QTime">1</int>
	  <lst name="params">
	    <str name="facet">true</str>
	    <str name="indent">true</str>
	    <str name="facet.pivot">inStock,cat</str>
	    <str name="q">*:*</str>
	    <str name="rows">0</str>
	  </lst>
	</lst>
	<result name="response" numFound="32" start="0">
	</result>
	<lst name="facet_counts">
	  <lst name="facet_queries"/>
	  <lst name="facet_fields"/>
	  <lst name="facet_dates"/>
	  <lst name="facet_ranges"/>
	  <lst name="facet_pivot">
	    <arr name="inStock,cat">
	      <lst>
	        <str name="field">inStock</str>
	        <bool name="value">true</bool>
	        <int name="count">17</int>
	        <arr name="pivot">
	          <lst>
	            <str name="field">cat</str>
	            <str name="value">electronics</str>
	            <int name="count">10</int>
	          </lst>
	          <lst>
	            <str name="field">cat</str>
	            <str name="value">currency</str>
	            <int name="count">4</int>
	          </lst>
	          .
	          .
	          .
	        </arr>
	      </lst>
	      <lst>
	        <str name="field">inStock</str>
	        <bool name="value">false</bool>
	        <int name="count">4</int>
	        <arr name="pivot">
	          <lst>
	            <str name="field">cat</str>
	            <str name="value">electronics</str>
	            <int name="count">4</int>
	          </lst>
	          <lst>
	            <str name="field">cat</str>
	            <str name="value">connector</str>
	            <int name="count">2</int>
	          </lst>
	          .
	          .
	          .
	        </arr>
	      </lst>
	    </arr>
	  </lst>
	</lst>
	</response>

## Statistical Faceting

Both ElasticSearch and Apache Solr can compute statistical data on numeric fields – values like count, total, minimal value, maximum value, average, etc. can be computed.

ElasticSearchとApache Solrの両方が統計データを数値フィールド上で演算できる。例えばカウント、総計、最小値、最大値、平均値等が演算可能である。

### Solr

In Apache Solr the functionality that enables you to calculate statistics for a numeric field is called Stats Component. It returns the above mentioned values as a part of the query result, in a separate list, just as faceting results.

Apache Solrでは数値フィールドにおいて統計を演算する機能はstatsコンポーネントと呼ばれる。上で記述した値をクエリの結果の一部として、別のリストの中にファセットの結果として返す。

### ElasticSearch

In ElasticSearch this functionality is called Statistical Facet. You should keep in mind thought that, as usual, ElasticSearch allows us to calculate this information for values returned by a script or combined for multiple fields, which is very nice if you need combined information for two or more fields or you want to do additional processing before getting the data returned by ElasticSearch.

ElasticSearchではこの機能はstatisticalファセットと呼ばれる。心に留めておかなければいけないのは、いつもどおり、ElasticSearchはこの情報をスクリプトの返り値に対し演算したり、複数のィールドの結合に演算可能であることである。これは2つ以上のフィールドの結合情報が必要であったり、ElasticSearchにより返された値を取得する前に追加の処理を実行したい場合にとても便利だ。

## Geodistance Faceting

(Geo)spatial search is quite popular nowadays where we try to provide the best search results we can and we considering multiple pieces of information and conditions. Of course both Apache Solr and ElasticSearch provide spatial search capabilities, but we are not talking about searching – we are talking about faceting. Sometimes there is a need to return a distance from a given point, just to show that in our application – and we can do that both in ElasticSearch and Solr.

(Geo)spatial検索は今日とても人気がある。我々は可能な限り、複数の情報と条件によるベストの検索結果が得られるように努力している。もちろんApache SolrとElasticSearchの両方がspatial検索の能力を持つ。しかし今は検索の話はしていない。ファセットだ。我々のアプリケーションでは時々、与えられた地点からの距離を返す必要がある。ElasticSearchとSolrの両方で可能だ。

### Solr

In Solr to be able to facet by distance from a given point we would have to use facet.query parameter and use frange or geofilt, for example like this:
Solrでは与えられた地点からの距離でファセットすることが可能だ。facet.queryパラメータを使用する必要があり、frange、geofiltを以下のように使用する

	q=*:*&sfield=location&pt=10.10,11.11&facet=true&facet.query={!geofilt d=10 key=d10}

This would return the number of document within 10 kilometers from the defined point.

これは定義された地点より10Km以内のドキュメントの数を返す。

### ElasticSearch

ElasticSearch exposes dedicated geo_distance faceting type that lets us pass the point and the array of ranges we want the distance to be calculated for. An example query might look like this:

ElasticSearchはgeo_distanceファセットタイプが公開されており、それを用いて位置と計算対象の距離のレンジの配列を渡す。クエリの例は以下のようになる。

	{
	 "query" : {
	  "match_all" : {}
	 },
	 "facets" : {
	  "d10" : {
	   "geo_distance" : {
	    "doc.location" : {
	     "lat" : 10.10,
	     "lon" : 11.11
	    },
	    "ranges" : [
	     { "to" : 10 }
	    ]
	   }
	  }
	 }
	}

In addition to that, we can specify the units to be used in distance calculations (kilometers and miles) and the distance calculation type – arc for better precision and plane for faster execution.

これに加えて距離の演算に使用する単位（Kmやマイル）と距離の演算タイプを指定することができる。arcはより良い精度でplaneは演算が速い。

## Solr, LocalParams and Faceting

One of the good things about faceting in Solr is that it allows the use of local params. For example, you can remove some filters from the faceting results. Imagine you have a query that gets all results for a term ‘flower’ and you only get results that fall into ‘cloth’ category and ‘shirt’ subcategory, but you would like to have faceting for tags field not narrowed to any filter. With the help of local params this query may look like this:

Solrのファセットにおける良い点の1つはローカルパラメータが使用できることだ。例えばファセットの結果からいくつかのフィルタを消すことが可能だ。‘flower’というタームに対する全ての結果を得るクエリがある時、‘cloth’のカテゴリで‘shirt’のサブカテゴリのみの結果を得たいと考える。しかしtagsフィールドに対してはどのフィルタでも絞り込みは行いたくない。ローカルパラメータの助けによりこのクエリは以下のようになるだろう

	q=flower&fq={!tag=facet_cat}category:cloth&fq={!tag=facet_sub}subcategory:shirt&facet=true&facet.field={!ex=facet_cat,facet_sub}tags

## ElasticSearch Faceting Scope and Filters

By default ElasticSearch facets are restricted to the scope of a given query, which is understandable. However, ElasticSearch also lets us change the scope of faceting to global and thus calculate the faceting for the whole data set, and not just for a given result set. In addition to that we can calculate facets for different nested objects by defining the scope matching the name of the nested object. This can come in handy in many situations, for example when optimizing memory usage on faceting on multivalued fields with many unique terms. In addition to that with ElasticSearch we can narrow down the subset of the documents on which faceting will be applied by using filters. We can define filters inside faceting (just please remember that filters that narrow down query results are not restricting faceting) and choose which documents should be taken into consideration when calculating facets. Of course, as you may expect, filters for faceting may be defined in the same way as filters for queries.

デフォルトではElasticSearchのファセットは与えられたクエリのスコープに限定される。これは理解できる。しかしElasticSearchはまたファセットのスコープをグローバルに変更可能で、与えられた結果集合だけでなく全てのデータセットに対してファセットを演算することができる。さらに異なる入れ子オブジェクトに対しファセットを演算することも可能だ。入れ子オブジェクトの名前にマッチングするスコープを定義する。これが多くの場合に便利である。例えば多くの異なるタームと複数値のフィールド上でファセットを行う場合のメモリを最適化したい場合だ。加えてElasticSearchではフィルタを用いることでファセットを適用するドキュメントの部分集合を絞り込むことが可能だ。ファセットの内部でフィルタを設定可能であり（クエリの結果を絞り込むフィルタはファセットだけに限定されないことを思いだすことを願う）どのドキュメントがファセットを演算する時に考慮すべきかを選択することができる。もちろん予想されるだろうが、ファセットに対するフィルタはクエリに対するフィルタと同じようにて定義できる。

## まとめ

In this part of the Apache Solr vs ElasticSearch posts series we talked about the ability to calculate facet information and only about this. Of course, this is only a look at the surface of faceting, because both Apache Solr and ElasticSearch provide some additional parameters and features that we couldn’t cover without turning this post into a tl;dr monster. However, we hope this post gives you some general ideas about what you can expect from each of these search engines. In the next part of the series we will focus on other search features, such as geospatial search and the administration API. If you are going to the upcoming ApacheCon EU and are interested in hearing more about how ElasticSearch and Apache Solr compare, please come to my talk titled “Battle of the giants: Apache Solr 4.0 vs ElasticSearch“. See you there!

Apache Solr vs. ElasticSearchのシリーズにおけるこのパートではファセット情報の演算能力のみについて話した。もちろんこれはファセットの表面のみを見ただけに過ぎない。Apache SolrとElasticSearchの両方にさらなる追加のパラメータと機能が存在するのでtl;drのモンスターにしない限りカバーできない。しかしこの記事が各検索エンジンに何を期待できるかの一般的な考えを得られたことを期待する。次回では別の検索機能であるgeospatial検索と管理者向けAPIについてフォーカスするつもりだ。もし今度のApacheCon EUにあなたが出席しElasticSearchとApache Solrの比較についてより聞きたければぜひ私の講演、タイトルは“Battle of the giants: Apache Solr 4.0 vs ElasticSearch“を聞きにきて欲しい。そこでお会いしましょう！

- @kucrafal, @sematext