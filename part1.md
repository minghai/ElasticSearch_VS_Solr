# Solr vs. ElasticSearch: Part 1 – 概観

August 23, 2012 by [Rafał Kuć](http://blog.sematext.com/author/kucrafal/ "http://blog.sematext.com/author/kucrafal/")

2012/08/23 原著者 Rafał Kuć

A good Solr vs. ElasticSearch coverage is long overdue.  We make good use of our own Search Analytics and pay attention to what people search for.  Not surprisingly, lots of people are wondering when to choose Solr and when ElasticSearch, and this SolrCloud vs. ElasticSearch question is something we regularly address in our search consulting engagements.

SolrとElasticSearchの良い比較記事は長い間望まれていた。我々は自身の検索分析を活用してきたし人々が何を検索したいのかに注意を払っている。不思議なことではないが、多くの人々がいつSolrを選択し、いつElasticSearchを選択すべきかに困惑している。そしてこのSolrCloud vs. ElasticSearchという問題は我々が検索コンサルティングの契約の中でいつも示してきたものだ。


As the Apache Lucene 4.0 release approaches and with it Solr 4.0 release as well, we thought it would be beneficial to take a deeper look and compare the two leading open source search engines built on top of Lucene – Apache Solr and ElasticSearch. Because the topic is very wide and can go deep, we are publishing our research as a series of blog posts starting with this post, which provides the general overview of the functionality provided by both search engines.

Apache Lucene 4.0のリリースが近づくにつれ、それと共にSolr 4.0のリリースもまた近づいている。Apache SolrとElasticSearchというLuceneの上に構築されている2つの主要な検索エンジンを比較し、深い考察を得ることはとても有意義だろうと考える。このトピックはとても広範で深くなりえるため我々は自身の研究成果をこの記事を始めとする一連のブログポストとして公開する。この記事では2つの検索エンジンが提供する機能の全体の概観を示す。

1. Solr vs. ElasticSearch: Part 1 – 概観
2. Solr vs. ElasticSearch: Part 2 – インデックス作成と言語の取扱
3. Solr vs. ElasticSearch: Part 3 – 検索
4. Solr vs. ElasticSearch: Part 4 – Faceting
5. Solr vs. ElasticSearch: Part 5 - 管理APIの機能
6. Solr vs. ElasticSearch: Part 6 – ユーザと開発者のコミュニティ比較

## はじめる前に

This post is based on released versions of Solr and ElasticSearch. For Solr, all the functionality description is based on version 4.0 beta and all of the ElasticSearch functionality is based on 0.19.8. Because we are comparing ElasticSearch and Solr, on the Solr side the focus is on Solr 4.0 (aka SolrCloud) functionality functionality and not Solr 3.*, so we could call this series as SolrCloud vs. ElasticSearch, too.

このポストは以下のバージョンに基づいている。Solrの全ての機能記述はバージョン4.0βに基づき、ElasticSearchの全ての機能記述はバージョン0.19.8に基づく。ElasticSearchとSolrを比較するのだからSolrはSolrCloudとして知られるSolr 4.0にフォーカスする。Solr 3.*ではない。そのため我々はこの一連の記事をSolrCloud vs. ElasticSearchとも呼べるだろう。

## 検索エンジンの裏側

For indexing and searching both Solr and ElasticSearch use Lucene. As you may suspect, Solr 4.0 beta uses the 4.0 version of Lucene, while ElasticSearch 0.19.8 still uses version 3.6.  Of course, that doesn’t mean much when it comes to future versions of ElasticSearch because you can be sure that ElasticSearch will start using Lucene 4.0 once it’s GA release is ready, or maybe even before that.

インデックス作成と検索にはSolrとElasticSearchの両方がLuceneを用いる。お気づきだろうがSolr4.0βはLucene 4.0を用いるが、ElasticSearch 0.19.8は依然v3.6を用いている。もちろんそれはElasticSearchの将来のバージョンについては多くの意味を持たないだろう。Lucene 4.0のGAリリースの準備が終われば直ぐに（または恐らくそのずっと前から）ElasticSearchもまたそれを使い始めることが確実だからだ。

## 基本

There are a few differences in the way Solr and ElasticSearch name certain concepts. Let’s start with the basics – many servers connected together forms a cluster for both ElasticSearch and Solr. A single instance of Solr or ElasticSearch is called a node. That’s about it for nomenclature overlap.

SolrとElasticSearchにはいくつかのコンセプトに対するネーミングに若干の違いがある。基礎から始めてみよう。ElasticSearchとSolrの両方に対して、多くのサーバが集合してクラスタを形成する。Solr、またはElasticSearchの1つのインスタンスはノードと呼ばれる。両者の命名法が重なるのはそれだけだ。

The main logical data structure for Solr is called the Collection. A Collection is composed of Shards that are really Lucene indices. A single Collection can have multiple Shards and Shards can live on different Nodes. Because a Collection is composed of one or more Shards, a single Collection can be spread across multiple Nodes giving you a distributed environment. In addition to that, a Collection can have Replicas – basically an exact copy of the Shard whose main purpose is to enable scaling and data duplication in case of Node failures (i.e., High Availability).

Solrのメインの論理データ構造はコレクションと呼ばれる。コレクションは実際にはLuceneのインデックスであるShardから構成される。1つのコレクションは複数のShardを持つことが可能で、Shardは異なるノードの上に置くことができる。コレクションが1つ、または複数のShardにより構成されるから、1つのコレクションは複数のノードにまたがることが可能で、分散システムとなる。付け加えて、コレクションはレプリカを持つことができる。レプリカは基本的にshardの完全なコピーで主な目的はスケーラビリティとノード障害時のためのデータの複製である。（つまり高可用性だ。）

On the other hand we have ElasticSearch where the top logical data structure is called an Index. Similar to a Collection in Solr, ElasticSearch Index can have multiple Shards and Replicas. And here, too, Shards and Replicas are small Lucene indices, that can be spread across the Cluster in order to create a distributed environment. But that’s not all – in ElasticSearch you can have multiple Types of documents in a single Index. This means that you can index documents of different index structure (for example users and their documents) in a single Index. ElasticSearch is able to distinguish those Types during indexing as well as querying. In order to achieve the same with Solr you would have to simulate that inside your application or develop a custom search component.

一方、ElasticSearchはインデックスと呼ばれる論理的なデータ構造の上にある。Solrのコレクションに似て、ElasticSearchのインデックスは複数のshardとレプリカを持つことが可能である。そしてShardとレプリカはまた小さなLuceneのインデックスであり、クラスタ上で分散され、分散環境を構築する。しかしそれだけではない。ElasticSearchでは1つのインデックス上に複数の型のドキュメントを持つことができる。これは異なるインデックス構造の複数のドキュメント（例えばユーザと彼らのドキュメント）を単一のインデックスに索引付けできることを意味する。ElasticSearchはインデックス作成時にもまた検索時にもそれらの型の区別を付けることが可能だ。Solrで同じことをするにはアプリケーションの中でそれをシミュレートするか、カスタム検索コンポーネントを開発する必要があるだろう。

## 設定

Lets take a quick look at how Solr and ElasticSearch are configured. Let’s start with the index structure.

SolrとElasticSearchがどのように設定されるかを簡単に見てみよう。まずはインデックスの構造から始めよう。

In Solr you need the schema.xml file in order to define how your index structure, to define fields and their types. Of course, you can have all fields defined as dynamic fields and create them on the fly, but you still need at least some degree of index configuration. In most cases though, you’ll create a schema.xml to match your data structure.

Solrではschema.xmlファイルをインデックス構造のフィールドとその型の定義に必要とする。もちろん全てのフィールドを動的なフィールドに定義し、動的生成を行うことも可能だ。しかしそれでもいくらかの程度はインデックスの定義を必要とするだろう。多くの場合、データ構造にマッチするschema.xmlを作成することになる。

ElasticSearch is a bit different – it can be called schemaless. What exactly does this mean, you may ask. In short, it means one can launch ElasticSearch and start sending documents to it in order to have them indexed without creating any sort of index schema and ElasticSearch will try to guess field types. It is not always 100% accurate, at least when comparing to the manual creation of the index mappings, but it works quite well. Of course, you can also define the index structure (so called mappings) and then create the index with those mappings, or even create the mappings files for each type that will exist in the index and let ElasticSearch use it when a new index is created. Sounds pretty cool, right? In addition to than, when a new, previously unseen field is found in a document being indexed, ElasticSearch will try to create that field and will try to guess its type.  As you may imagine, this behavior can be turned off.

ElasticSearchは少し異なる。スキーマレスと呼べるものだ。あなたはそれが何なのか尋ねるかもしれない。短かく言えば、誰でもElasticSearchを起動してそのままドキュメントをその中に送ることで、1つもインデックススキーマを定義することも無しにインデックス作成を行うことができる。ElasticSearchがフィールドの型を推論してくれる。いつでも100％正しいとは限らない。少なくとも手動でスキーマを定義した場合に比べればそうだ。しかしそれはとてもうまく働く。もちろん、インデックス構造（つまりマッピング）を定義することは可能であるし、マッピングに従いインデックスを作成することもできる。また複数のマッピングファイルをインデックスに存在するだろう個々の型に対して作成することも可能であり、ElasticSearchに新しいインデックスが作成される度にをそれを用いらせることも可能だ。とても魅力的だろう？ それに加えてインデックス作成中のドキュメントにまだ見かけていない新しいフィールドが見つかった場合に、ElasticSearchはその型を推測して新しいフィールドを作成しようと試みる。もちろんこの機能はオフにすることも可能だ。

Let’s talk about the actual configuration of Solr and ElasticSearch for a bit. In Solr, the configuration of all components, search handlers, index specific things such as merge factor or buffers, caches, etc. are defined in the solrconfig.xml file. After each change you need to restart Solr node or reload it. All  configs in ElasticSearch are written to elasticsearch.yml file, which is just another configuration file. However, that’s not the only way to store and change ElasticSearch settings. Many settings exposed by ElasticSearch (not all though) can be changed on the live cluster – for example you can change how your shards and replicas are placed inside your cluster and ElasticSearch nodes don’t need to be restarted.  Learn more about this in ElasticSearch Shard Placement Control.

SolrとElasticSearchの実際の設定についてもう少し話を続けよう。Solrでは全てのコンポーネント、検索ハンドラ、インデックス特定の物事（例えばマージファクタやバッファ、キャッシュ等）の定義は、solrconfig.xmlファイルの中に定義される。全ての変更の後にはSolrノードの再起動かリロードが必要である。ElasticSearchの設定全てはelasticsearch.ymlファイルに記述される。それはまた別の設定ファイルに過ぎない。しかしElasticSearchの設定を定義、又は変更する方法はそれだけではない。ElasticSearchにおける多くの設定（しかし全てでは無い）は運用中のクラスタ上にて変更可能だ。例えばShardとレプリカをクラスタ内部のどこに置くかである。またElasticSearchのノードは再起動する必要がない。このことについてはElasticSearchのShardの配置コントロールにおいてより詳しく学ぶ。

## 探索とクラスタの管理

Solr and ElasticSearch have a different approach to cluster node discovery and cluster management in general.  The main purpose of discovery is to monitor nodes’ states, choose master nodes, and in some cases also store shared configuration files.

SolrとElasticSearchはクラスタノードの探索とクラスタ管理全般において異なるアプローチを取る。探索の主な目的はノードの状態監視、マスターノードの選択、またShard定義ファイルの格納を伴なういくらかのケースである。

By default ElasticSearch uses the so called Zen Discovery, which has two methods of node discovery: multicast and unicast.  With multicast a single node sends a multicast request and all nodes that receive that request respond to it. So if your nodes can see each other at the network layer with the use of multicast method your nodes will be able to form a cluster. On the other hand, unicast depends on the list of hosts that should be pinged in order to form the cluster. In addition to that, the Zen Discovery  module is also responsible for detecting the master node for the cluster and for fault discovery. The fault discovery is done in two ways – the master node pings all the other nodes to see if they are healthy and the nodes ping the master in order to see if the master is still working as it should.  We should note that there is an ElasticSearch plugin that makes ElasticSearch use Apache Zookeeper instead of its own Zen Discovery.

デフォルトではElasticSearchは禅探索(Zen Discovery)と呼ばれる方法を用い、それにはマルチキャストとユニキャストの2つのノード探索手法が存在する。マルチキャストでは1つのノードはマルチキャストリクエストを送信し、そのリクエストを受け取った全てのノードはそれに対してレスポンスを送る。もしノードがマルチキャストメソッドを用いてネットワーク層においてお互いを認識できれば、ノードはクラスタを形成できる。一方、ユニキャストはホストのリストに頼り、クラスタを形成するためにpingを送信する必要がある。加えて禅探索モジュールはクラスタのマスターノードの検知と障害探索の責任を持つ。障害探索には2つの方法が取られる。マスターノードは他の全てのノードにpingを送信してそれらが健康であるかを確認し、他のノードはマスターが働いているかを確認するためにpingを送信する。注目するべき点として、禅探索の代わりにApache ZooKeeperを用いるElaticSearchプラグインが存在する。

Apache Solr uses a different approach for handling search cluster. Solr uses Apache Zookeeper ensemble – which is basically one or more Zookeeper instances running together. Zookeeper is used to store the configuration files and monitoring – for keeping track of the status of all nodes and of the overall cluster state. In order for a new node to join an existing cluster Solr needs to know which Zookeeper ensemble to connect to.

Apache Solrはクラスタ探索の扱いに異なるアプローチを取る。SolrはApache Zookeeper ensembleを利用する。それは基本的に1つ以上のZooKeeperのインスタンスが同時に実行することで構成される。ZooKeeperは設定ファイルの格納と監視に利用される。全てのノードとクラスタ全体の状態を追跡する。既存のクラスタに新しいノードを追加する場合に、SolrはどのZooKeeper ensembleに接続するか知っている必要がある。

There is one thing worth noting when it comes to cluster handling – the split brain situation. Imagine a situation, where you cluster is divided into half, so half of your nodes don’t see the other half, for example because of the network failure. In such cases ElasticSearch will try to elect a new master in the cluster part that doesn’t have one and this will lead to creation of two independent clusters running at the same time. This can be limited with a small degree of configuration, but it can still happen. On the other hand, Solr 4.0 is immune to split brain situations, because it uses Zookeeper, which prevents such ill situations. If half of your Solr cluster is disconnected, it wouldn’t be visible by Zookeeper and thus data and queries wouldn’t be forwarded there.

1つ注意すべきことがクラスタの取扱いにある。スプリットブレインという状態だ。クラスタが半分に分割されてノードの半分が、一方の半分を検知できない状態を想像して欲しい。例えばネットワークの障害だ。そのようなケースではElasticSearchはマスターノードを持たないほうの部分クラスタにおいて新しいマスターを選択しようと試みる。これが2つの独立したクラスタが同時に実行される状況へと導く。これは少ない量の定義で制限できるが、それでも起こりうる。一方でSolr4.0はスプリットブレイン状態に免疫がある。ZooKeeperを用いているため、そのような状態を防いでくれる。もし半分のSolrクラスタが接続不能に陥った場合、ZooKeeperにはそれが見えず、従ってデータとクエリはそちらには転送されない。

## API

If you know Apache Solr or ElasticSearch you know that they expose an HTTP API.

もしApache SolrかElasticSearchを知っているなら、それらがHTTP APIを公開していることも知っているだろう。

Those of you familiar with Solr know that in order to get search results from it you need to query one of the defined request handlers and pass in the parameters that define your query criteria. Depending on which query parser you choose to use, these parameters will be different, but the method is still the same – an HTTP GET request is sent to Solr in order to fetch search results. The good thing is that you are not limited to a single response format – you may choose to get results in XML, in JSON in JavaBin format and several other formats that have response writers developed for them.  You can thus choose the format that is the most convenient for you and your search application.  Of course, Solr API is not only about querying as you can also get some statistics about different search components or control Solr behavior, such as collection creation for example.

Solrに詳しい人は検索結果を得るために定義済みのリクエストハンドラを要求し、クエリの基準を定義するパラメータの中にて渡す必要があることは知っているだろう。どのクエリパーザを選択するかによりこれらのパラメータは異なる。しかし方法は同じである。HTTP GETリクエストが検索結果を取得するためにSolrに送られる。良い点は単一のレスポンス形式に制限されていないことだ。結果の取得にXML、JavaBin形式のJSONや他のいくつかの、それらのために開発されたレスポンスライターを持つ形式を選択することが可能だ。従って自分の検索と検索アプリケーションに最も便利な形式を選択することができる。もちろん、Solr APIはクエリのためだけの物ではなく、異なる検索コンポーネントの統計を得たり、Solrの振舞、例えばコレクションの作成等をコントロールすることも可能だ。

And what about ElasticSearch?  ElasticSearch exposes a REST API which can be accessed using HTTP GET, DELETE, POST and PUT methods. Its API allows one not only to query or delete documents, but also to create indices, manage them, control analysis and get all the metrics describing current state and configuration of ElasticSearch. If you need to know anything about ElasticSearch, you can get it through the REST API (we use it in our Scalable Performance Monitoring for ElasticSearch, too!). If you are used to Solr there is one thing that may be strange for you in the beginning – the only format ElasticSearch can respond in JSON – there is no XML response for example. Another big difference between ElasticSearch and Solr is querying. While with Solr all query parameters are passed in as URL parameters, in ElasticSearch queries are structured in JSON representation. Queries structured as JSON objects give one a lot of control over how ElasticSearch should understand the query and thus what results to return.

ElasticSearchについてはどうだろうか？ ElasticSearchはHTTPのGET、DELETE、POST、PUTメソッドを用いてアクセスできるREST APIを公開している。ドキュメントにクエリをかけたり削除するだけでなく、インデックスを作成したり管理したり分析をコントロールして現在の状態を表すメトリクスを全て取得したりElasticSearchの設定を取得したりもできる。もしElasticSearchについて何か知りたいことがあればREST APIを通して取得可能だ。（我々はそれを自社の"Scalable Performance Monitoring for ElasticSearch"でも使っている！）もしあなたがSolrに慣れ親しんでいるのなら最初におかしく感じることが1つあるだろう。ElasticSearchのレスポンスにはJSONフォーマットしか存在しない。例えばXMLは無い。ElasticSearchとSolrの他の大きな違いはクエリだ。Solrでは全てのクエリパラメータがURLパラメータとして渡さえるのに対し、ElasticSearchのクエリではJSON表現にて構成される。JSONオブジェクトとしてクエリを構成することでElasticSearchがどのようにクエリを理解し、そしてどのような結果を返すかの数多くのコントロールを提供する。

## データの取扱い

Of course, both Solr and ElasticSearch leverage Lucene near real-time capabilities.  This makes it possible for queries to match documents right after they’ve been indexed. In addition to that, both Solr (since 4.0) and ElasticSearch (since 0.15) allow versioning of documents in the index.  This feature allows them to support optimistic locking and thus enable prevention of overwriting updates. Let’s look at how distributed indexing is done in Solr vs. ElastiSearch.

もちろんSolrとElasticSearchの両方がLuceneのほぼリアルタイムの特性を利用している。これがクエリがドキュメントに対して、インデックスが作成された後に正しくマッチすることを可能にしている。付け加えてSolr（4.0から）とElasticSearch(0.15から）はインデックス内でのドキュメントのバージョニングも可能にしている。この機能は楽観的ロックのサポートを可能にし、更新のオーバーライトの防止も可能にしている。SolrとElasticSearchにて分散インデックスがどのように行われているかを見てみよう。

Let’s start with ElasticSearch this time. In order to add a document to the index in a distributed environment ElasticSearch needs to choose which shard each document should be sent to. By default a document is placed in a shard that is calculated as a hash from the documents identifier. Because this default behavior is not always desired, one can control and alter this behavior by using a feature called routing.  This is controlled via the routing parameter, which can take any value you would like it to have. Imagine that you have a single logical index divided into multiple shards and you index multiple users’ data in it.  On the search side you know queries are narrowed mostly to a single user’s data. With the use of the routing parameter you can index all documents belonging to a single user within a single shard by using the same routing value for all his/her documents.  On the search side you can then use the same routing value when querying. This would result in a single shard being queried instead of the query being spread across all shards in the index, which would be more expensive and slower. In case each index shard contains multiple users’ data we could additionally use a filter to limit matches to only one user’s documents.  In cases like this, routing functionality allows one to think of some nice optimization for both indexing and querying. If you want to hear some more about distributed indexing capabilities of ElasticSearch please take a look at my Berlin Buzzwords 2012 talk – Scaling Massive ElasticSearch Clusters (video).

今度はElasticSearchから始めよう。分散環境におけるElasticSearchのインデックスにドキュメントを追加するには個々のドキュメントがどのShardに送られるかを選択する必要がある。デフォルトではドキュメントはドキュメントIDから計算されたハッシュ値により決定されたShardにドキュメントは置かれる。このデフォルトの動作は常に理想的ではないため、ルーティングと呼ばれる機能を用いて変更することが可能だ。これはルーティングパラメータを通してコントロールされ、その値は任意の値が設定できる。複数のShardに分割された単一の論理インデックスを持っており、複数のユーザのデータをその中に索引付けしてみると考えてみよう。検索サイドではクエリは多くの場合に単一のユーザのデータに対するように絞られることを知っている。ルーティングパラメータを用いることによって、単一のユーザに所属する全てのドキュメントを単一のShardに索引付けすることが全ての彼/彼女のドキュメントに対して同じルーティング値を用いることで可能だ。検索サイドでは従ってクエリ時に同じルーティング値を用いることが可能だ。これにより結果はクエリが実行された単一のShard内にのみ存在し、クエリがインデックス内の全てのShardに対して広がることにならない。その場合はより高いコストで遅くなるだろう。個々のインデックスのShardが複数のユーザのデータを持っている場合には追加としてフィルタを用いることでただ一人のユーザのドキュメントへの適合に制限することができる。このようなケースではルーティングの機能はあなたに良い最適化をインデックス作成とクエリの両方に対して考えることを可能にする。ElasticSearchの分散インデックスの機能についてもっと聞きたければ著者のBerlin Buzzwords 2012における講演、"[Scaling Massive ElasticSearch Clusters](http://blog.sematext.com/2012/06/05/slides-scaling-massive-elasticsearch-clusters/ "http://blog.sematext.com/2012/06/05/slides-scaling-massive-elasticsearch-clusters/")"の[ビデオ](https://vimeo.com/44718089)を参照して欲しい。

Details of Solr’s implementation of distributed indexing (and searching) capabilities can be found in our The New SolrCloud: Overview post. But let’s recall some of those details. In order to forward a document to a proper shard Solr uses Murmur hashing algorithm which calculates the hash for the given document on the basis of its unique identifier. This part is similar to default ElasticSearch behavior.  However, Solr doesn’t yet let you specify explicitly to which shard the document should be sent  - there is no document and query routing equivalent in Solr yet.

Solrの分散インデックス（と検索）に関する実装の詳細は我々の<a href="http://blog.sematext.com/2012/02/01/solrcloud-distributed-realtime-search/">"The New Solrcloud: Overview"</a>の記事を参照して欲しい。しかしその詳細のいくつかを復習してみよう。ドキュメントを適切なShardに送るために、SolrはMurmur hashingアルゴリズムを用いて与えられたドキュメントのハッシュ値をその固有な識別子を基準として計算している。この部分はElasticSearchのデフォルトの動作に似ている。しかしSolrは明示的にどのShardにドキュメントを送るべきか指定することがまだできない。ドキュメントとクエリのルーティングに等しいものはSolrにはまだ無い。


Of course, both Solr and ElasticSearch allow one to configure replicas of indices (ElasticSearch) or collections (Solr). This is crucial because replicas enable creation of highly available clusters – even if some of nodes are down, for example because of hardware failure or maintenance, the cluster and data within it can remain available. Without replicas if one nodes is lost, you lose (access to) the data that were on the missing node. If you have replicas present in your configuration both search engines will automatically copy documents to replicas, so that you don’t need to worry about data loss.

もちろん、SolrとElasticSearchの両方がインデックス（ElasticSearch)、またはコレクション（Solr）のレプリカの設定を許可している。これはレプリカがクラスタの高可用性を実現していることからとても重要だ。例えいくつかのノードがハードウェア障害やメンテナンスにてダウンしても、クラスタとその中のデータは利用可能であり続ける。レプリカ無しでは1つのノードが失なわれたら、失なわれたノード上に存在したデータ（に対するアクセス）は失なわれる。もし設定にレプリカが存在する場合、両検索エンジンは自動的にドキュメントをレプリカにコピーするので、データロスについて心配する必要はない。

## まとめ


We hope that after reading this post you have the basic understanding of what you can expect from both Solr 4.0 and ElasticSearch 0.19.* and you can start to get the feeling for differences and similarities between them.  Of course,  both Solr and ElasticSearch have very strong and active user and development communities and are constantly evolving and improving, and are doing that rather fast. In pre-Solr 4.0 (aka SolrCloud) world the difference between Solr and ElasticSearch was quite stark.  Since then, and under the pressure from ElasticSearch, the gap has narrowed and both projects are moving forward quite quickly.  At Sematext our clients often ask us to recommend the search engine for their use and we recommend both of them.  Which one we recommend for a particular project depends on project requirements, which we always go through at the beginning of every engagement.  If you need help deciding, let us know.

この記事を読んだ後にSolr 4.0とElasticSearch 0.19.*の両者から何が期待できるか、その基礎をあなたが理解されたことを願う。そして両者の間の違いと類似点についても理解され始めていることを。もちろん、SolrとElasticSearchの両者共、とても強力で、活発なユーザと開発者のコミュニティが存在し、定期的に進化し洗練され、そしてより早く行っている。Solr Cloudとしても知られるSolr4.0以前にはSolrとElasticSearchの違いはとても大きかった。それから、ElasticSearchのプレッシャーの元で、ギャップは狭まり2つのプロジェクトは共にとても速く進んでいる。Sematextではクライアントからしばしば彼らの使用目的においての検索エンジンを推奨して欲しいと尋ねられる。我々は両方を勧める。どちらのプロジェクトを勧めるかは要件による。これが我々が常に契約の最初の時点にて通る道だ。もしあなたが決断に助けを必要とするのなら、ぜひ知らせて欲しい。

Also please keep in mind this post is not meant to be the most comprehensive guide to all the similarities and differences between ElasticSearch and Solr. We wanted to start with a general overview of how these two great search engines work and cover the big picture. In subsequent parts of the “Solr vs. ElasticSearch” series we’ll describe how the most frequently used features of both search engines work, what the differences between them are, and we’ll get into details of those features showing you pros and cons of Solr vs. ElasticSearch approach (for example approaches used in faceting or caching). Just as a sneak peak into the next post in the series – you can expect information about language handling capabilities, analysis configuration, and querying.

またこの記事がElasticSearchとSolrの間の類似点と相違点に対する全ての最も完璧なガイドではないことを常に注意して欲しい。我々はこれらの2つの素晴しい検索エンジンがどのように働くかの一般的な概観と全体像から始めたいと願った。これに続く"Solr vs. ElasticSearch"シリーズのパートでは両者の検索エンジンの最も頻繁に使われっる機能がどのように働くかを解説する。そしてそれらの機能の詳細に立ち入り"ElasticSearch vs. Solr"のアプローチにてその利点と欠点を示す。例えばファセットやキャッシュで利用されるアプローチにて）シリーズの次回において、言語の取扱の機能と分析設定、そしてクエリについてご期待あれ。

[@kucrafal](http://twitter.com/kucrafal), [@sematext](http://twitter.com/sematext "sematext")