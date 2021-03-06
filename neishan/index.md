Title: Hadoop界隈が何を言っているかわからない件
Subtitle: 分かって実践するビッグデータ処理
Author: @neishan
Author(romaji): neishan

# 速習ビッグデータ処理の歴史

　嫁「私そういうのよくわからないんだけどさ・・・面白いの？」
　私「Hadoopは、俺の嫁!!」
　ということで、皆さまエンジョイしてますか？
　Hadoop界隈では、似てるようで似てないような、それでいて似ているプロダクトがわんさか出てきています。そもそも似ているのかどうかすらわからないあなたに送る、ハードボイルドHadoop界隈歴史観。それでは早速始めましょう!!

　最近、Hadoopを中心にクラスタを用いたビッグデータ処理が一般的になっています。本記事では、特に、2005年あたりから何が起こっていたかを解説します。どうしてHadoopというプロジェクトが発生したのか、またその周りのプロダクトがどのような役割を担っているのかを概説します。

## ビッグデータ前夜(〜2000年代前半)

　Haddopの生みの親であるDugg Cutting氏は、1999年から全文検索エンジンのLuceneを開発しています。この開発の中で、大規模データの転置インデックス得るためにどのようなアーキテクチャを取るべきかについて悩んでいました。数台レベルであればスケールしますが、数十台レベルでスケールしないという問題です。


　アカデミックなデータ工学分野では、半構造データとしてのXMLの研究が流行っていました。その頃、DBMSのコア技術は、プロプライエタリで開発されたOracle, DB2, MS SQLServerなどをもつ一部の企業に独占され、課題やソースがオープンにならないため、研究者が研究テーマを設定しずらい状況となっていました。また、分散DBMSの研究は、1980年代にひと通りやりきったというのが全体的な流れであり、その結果として、フルトランザクションを提供しつつ、スケールさせることはできないというものでした。

　Napsterと呼ばれる音楽共有サービスがクライアント・サーバ型であるもののP2Pが極めて役に立つものだということが認識されていました。そのため、アカデミックな分散処理分野では、P2Pの技術としてDHT(Distributed Hash Table)が研究されていました。初期のDHTアルゴリズムは、スケールしづらくデータの探索に時間がかかるため、レスポンスタイムが要求されるようなデータ処理には利用できないと思われてきました。しかし、2001年にChordと呼ばれるアルゴリズムが発表されます。

　この頃、Googleは次世代の検索エンジンとして知名度を上げてきており、スタンフォード時代に構築したページランクアルゴリズムは特許化されており知られていました。しかし、膨大なWebの情報を一体どうやって処理していたのかは誰にもわかりませんでした。また、Amazonも世界中の物流システムに入り込もうと拡大していました。その基板となるソフトウェアはどのようなものなのか、同じく誰も知りませんでした。

## ビッグデータ処理基盤のオープン化(2000年後半)

　2003年に、Google File System(GFS)、2004年にGoogle MapReduce、2006年にBigtable、Chubby、2007年 Amazon Dynamo と立て続けに、GoogleやAmazonの裏で巨大クラスタをソフトウェアでどのように管理しているかが明らかになっていきました。Doug Cutting氏は、MapReduceを用いれば分散クラスタにおいて、様々な処理をスケールさせることが出来ると考え、Hadoopの開発を始めることになります。以上の流れから、HDFSとHadoop::MapReduceの開発に当初取り組んでいました。少しずつ、HDFSやMapReduceが動作するようになると、MapReduceを常に書き続けるのは大変だとかがえる人たちが出てきます。2008年ごろから、Pig, Hive, Cascadingといったプロダクトが提案さてていました(図1)。

![図1](pic1.png)

　MapReduceでは、分散処理をMapとReduceという関数を書くだけで実現できることが最も重要な提案でした。MapやReduceと概念自体は、関数型言語やHPC(High Performace Computing)の世界においてもよく使われてきました。しかし、いわゆるスパコンでは、HDFSに相当するレイヤが存在しないため、分散処理を書くためには低レイヤの実ファイルの分割や配置まで含めて全てプログラマの仕事でした。HDFSでは、ファイルをアップロードすると、レプリカを作りつつ、64MBのブロックで分割して勝手に分散配置します。MRでは、分散配置された64MBのブロックごとに処理を行います。しかしながら、MapとReduceのプログラミングモデルは分散されて処理されることを意識しないとうまくスケールできないという問題もあります。

　そこで、PigやCascadingは、よく使われるMapReduceの関数を再利用しやすい形で提供しました。パイプラインをつないで処理を書くことでMRを意識しないで記述可能となりました。Hiveは、Facebookが社内で利用していたプロダクトをオープンソース化したものでです。HiveQLと呼ばれる、SQLライクな宣言型言語を記述すると、MapReduceのプログラムに変換し実行してくれるというものです。

　Amazon Dynamoは、これまでデータ工学においては疑う余地のなかったトランザクションの持つACIDの特性をユースケースによっては緩和できることを表明したことが極めて大きな学術的な意義でした。Amazonにおいて一番重要なことは、オンライン上で買い物をしているユーザが、買い物かごに入れるときに「買い物かごに入れることができませんでした、再度やり直して下さい」というエラーを返却することでした(お客さんが帰ってしまうのですね)。そのため、いついかなる時でも「買った」という情報を書き込みできるDBが必要だったのです。そこで、結果整合性(Eventual Consistency)と呼ばれる概念を導入します。ここでの一貫性(Consistency)は、データ工学の文脈とは違っています。複数のサーバ上においてレプリカを持っている場合に、それらのレプリカに一時的なずれを許さない場合、一貫性を持つといいます。Dynamoでは、一時的なずれを許します。そのため、読み込み時にずれが生じた場合、それを修正するのはアプリケーション側の責任です。一方、Bigtableでは、一貫性を提供するためそのようなずれは生じません。しかし、データを管理しているサーバが障害等によりダウンした場合、その間はサービスが停止します。リカバリ処理が必要となるためです。

　2005年にデータ工学の大御所 Stonebraker博士は、"One size fits all" という発表を行いました。過去のDBMSはそれひとつで様々ユースケースに対応しようとしていました。しかし、もはやDBMSひとつで全てのユースケースに対応することはさらなる最適化の邪魔になると考え、OLAP(OnLine Analytical Processing)、OLTP(OnLine Transaction Processing)、Stream処理それぞれに特化したデーターベースエンジンを作ることを提唱します。その結果、以下の様なプロジェクトとベンチャー企業を設立しました。

- OLAP : C-Store -> Vertica社(HPにより買収)。分析処理で、既存DBMSの100倍高速化。
- OLTP : H-Store -> VoltDB社。トランザクション処理で、既存DBMSの100倍高速化。
- Stream : Aurora -> StreamBase社(TIBCOにより買収)。既存DBMSでは処理できないユースケース。

　ここで提案されたユースケースの分離が、その後のプロダクト(の考え方)にとても大きい影響を与えていきます。少なくとも最近出てきているプロダクトは、ユースケースに特化しています。

## さまざまなプロダクト(2010年前半)

![図2](pic2.png)

　図2は、2005から現在にかけて各プロダクトがどのような進化を遂げたかについてまとめています。二重枠で囲われたプロダクトは、OLAP向けです。実線枠で囲われたプロダクトは、OLTP向けです。2010年になると、分散処理基盤は加速度的に進化を遂げていきます。Hadoopがオープンソースであることにより、学術分野に属する人々の参加もしやすくなったり、ビジネスにおいて利用する場合のイニシャルコストが激減したこと(0から作る必要がない)が要因になっていると思います。ユースケースごとに見て行きたいと思います。

###OLAP

　HadoopのデータをSQLライクな言語で処理可能なHIVEは、SQLクエリの処理エンジンは結局のところMapReduceに変換されるため、(1).初期起動に時間がかかる（数十台ともなると、起動だけで10分のオーダ）。(2).既存のデータ工学で培われてきた最適化をそのまま適応できない場合がある。

　例えば、ログデータから有益な情報を得ようとした場合、一般的にETLと呼ばれる処理が必要と言われています。Extract-Transform-Loadの略ですが、必要な情報を抽出し(ログデータを取得する)、変換し(表記ゆれの修正やロードしやすい形に変換する)、実際にクエリを投げるためのデータベースに格納する(MySQLに格納する)。このような処理を行うことで、分析処理にようやく入ることが出来るようになります。さて、大量のログデータに対して、最初から何がしたいのかはっきりとわかっている場合はまれです。ETLをやっていく中で、試行錯誤を行います。その際、HIVEのようなエンジンを利用すると、ちょっとしたクエリであってもレスポンスが極端に大きくなり、人間の試行錯誤するスピードについていけません。この試行錯誤するときのクエリ処理をインタラクティブクエリと呼びます。

　上記の課題に対して、オープンソースでは、Cloudera社のImpalaやFacebookのPresto、若干趣が変わりますが、SalesForceのPhoenixが提案されています(いずれも、HIVEより数十倍以上早いと謳っています)。プロプライエタリでは、Load後の分析処理を劇的に高速化するNeteeza(IBMが買収)、Vertica(HPが買収)が生まれました。特性を見て行きましょう。また、Amazon RedShiftやGoogle BigQueryサービスを提供しているプロダクトDremelも注目に値します。

　Impalaは、インタラクティブクエリに特化するため、クエリの対象となるデータは基本的にメモリ上に乗っているという前提を持っています。その場合のボトルネックはCPUになるため、LLVM+CLangを用いた高速化を図っています。あくまでインタラクティブクエリに特化するため、Impalaがクエリを処理中にサーバがダウンしたら、そのクエリは失敗します(可用性が低くなる)。それでも、インタラクティブクエリではレスポンスタイムが数秒だから、もう一度やり直せば良いという判断をしています。

　Prestoは、HIVEを作ったFacebookが、必要に迫られてインタラクティブクエリ用途のエンジンです。アーキテクチャは、ImpalaもPrestoも本質的には変わらないです。SQLをMapReduceに変換した部分をやめて、古くからデータ工学で用いられているオペレータによる実行を行います。2012年秋に開発が始まり、2013年頭には、社内で利用開始(4ヶ月!!)されています。メーリングリストなどみても、とても活発だという印象を受けます。

　次に、Phoenixです。これは、対象となるデータがHBaseに格納されたデータという部分が他２つと違います。これまで、HBaseのデータをSQLライクにクエリを投げるとするとHIVEやImpala経由となっていましたが、それらはHBaseのインターフェースを呼び出すため極めて効率が悪いという本質的な問題を抱えていました。そこで、SQLクエリの処理をHBase内部でフィルタを用いたり、カラムストアの最適化やKVSの特徴をうまく使うことで、Phoenixは、HBaseに対するHIVEやImpalaのクエリ処理に比べて数十倍早いプロダクトとなっています。

　Neteeza は、元々はハードウェアエンジニアである開発者が、SQLの処理をFPGAにやらせてしまえば劇的に早くなる事に気づいて、PostgreSQLをベースに作られたプロダクトです。あまりの早さに、Oracle CEOのラリー・エリソンもビビっていたようです。Oracle社の新しい分析エンジンの発表会において、「これでNeteezaに勝てる」と言ってしまったがために、逆にNeteezaが有名になるという逸話もあります。ただ、個人で試しに使うにはVertica程ではないにせよお高いです。そのため、ETLはHadoopで行い最後のLoadするためのDBとして利用されます。

　次に、Verticaですが、これはC-Storeと呼ばれる研究プロジェクトを元に0からフルスクラッチで構築されたカラムストア型データベースです。既存のRDBMSでは、複数の行をまとめてページとし、その単位でストレージへの書き込みを行います。しかし、カラムストアでは、列ごとに書き込みを行います。こうすることで、1.何百カラムも持つような巨大なテーブルから一部のカラムを取得することが劇的に早くなる。2.カラムに入るデータは似たようなものなので、圧縮率が極めて高い。3.軽量圧縮と呼ばれる圧縮法を用いると、SQLクエリ処理を解凍しなくても実行可能。4.ユーザに返却するデータをギリギリまで生成しないことで、中間データの生成を削減し高速化する遅延実体化などの手法を用いることで、高速化を実現しています。ただし、こちらはNeteeza以上にお高い。

　Amazon RedShiftは、ビッグデータをクラウド上で処理するためのサービスとして提供されました。アーキテクチャとしては、PostgreSQLをベースに分散クエリ処理基盤を構築しており、水平分散することでスケールするようになっています。分散RDBMSで問題となる、複数サーバにまたがるjoinの高速化を、ユーザのテーブル設計に任せており、同じjoin-keyは同じサーバに存在するようにテーブル設計をしないと高速化しません。ただし、内部アーキテクチャはシンプルでわかりやすいため性能を得るためのチューニング(テーブル設計含む)は、比較的取り組みやすくなっていると思います。

　Google Dremelは、OLAP向けエンジンのGoogleが出した答えです。最も注目するべきは、半構造データを処理できる点です。通常、RDBMSから発生したOLAP処理システムは、リレーショナルモデルに基づくためデータ構造がネストするようなデータは処理できません。そのため、どうしても正規化したテーブルに対してはjoinが頻発し性能が劣化します。Dremelでは、カラム指向でありながら、後々joinするデータは、最初からネストした形で保存することが可能となっており、初期のテーブル設計をうまくやるとクエリが激速となります。アーキテクチャ的には、水平・垂直分散を採用しており、スケール性能も大変よいです。Amazonがサービス提供したRedShiftに呼応して、サービス提供が行われています。まだ、まともに動作するところまで言っていませんが、オープンソースクローンにApache Drillがあります。

## OLTP

　OLTP のユースケースにおいてスケールさせる必要が生じたことで、AmazonやGoogleはそれぞれKVSを構築しました。学術的には、分析用途を完全に捨てたH-Storeが提案されVoltDBと呼ばれるベンチャーが設立されています。H-Storeでは、主に(1)並列制御のためのロックを排除、(2)永続性のためのコミットログ排除、(3)完全ストアードプロシージャ化などの高速化をはかりました。

　また、SSDを始めとするハードウェアの進歩により、既存のRDBMSの処理スループットが劇的に増加することで、既存のRDBMSのスループットも大きく向上しました。

　Googleは、特に分散システムのインフラ構築技術が突き抜けており、現在誰も追い付くことができないレベルに到達しています。その例として、Google F1と呼ばれる全世界を股にかける広告配信システムのバックエンドデータベースSpannerがあります。それでは、OLAPと同様にダイジェストで紹介していきます。厳密にOLTPというと、トランザクション処理をたくさん行うユースケースになります。KVSは、トランザクション処理を持っているわけではないので、若干ずれていますがユースケースとしてはOLTPの目指すところと同じということで、その中で話をしていきます。

　Google Bigtableは、Hadoop HBaseの大元となるアーキテクチャを提案しています。Google社内で様々なサービスに利用されていました(すでに、Bigtable の子孫として、MegaStore、Spannerが存在しているので現状はそちらを利用していると思います)。例えば、Google Earth、Analytics、Orkut(なくなっちゃったね)、Personalized Search、Finance、などなど。Bigtableは、CAP定理でいうところの、CPを採用しています。CAP定理では、分散システムにおいて、Consistency(一貫性), Availability(可用性)，Partition tolerance(分断耐性)の3つの属性のうち2つしか選択できないこと証明しています。Consistencyは、複数のユーザが同一時刻に同じKeyに対して検索した時、同じ値を返却すること。Availabilityは、サービスが停止しないこと。Partition toleranceは、ネットワーク分断が生じても状態がおかしくならないことになります。CPを採用したということは、常に誰が検索しても同じ値を返却する(Consistencyがある)が、サーバがダウンした場合にはそのサーバが管理しているデータが一時的に検索できなくなる(Availabityがない)。また、ネットワークが分断した場合、分散ロックシステムによりリーダが一意に選出されるため、状態は整合性を持ったまま維持されます。

Bigtableでは、平たく言うと、トランザクションが出来る範囲をかなり小さくし(RowレベルでのAtomic性しか保証していない)、水平垂直分割することで、500台程度までスケールすることを可能にしました。また、特徴的なのがkeyによりソートをしているため、範囲検索を高速に行うことができます。一応、OLTPのジャンルで取り上げていますが、このソートされているという特性がMapReduceとの親和性を高めるため、OLAPとしても利用しやすくなっています。一般的なP2P由来のKVSでは、範囲検索をした場合でもクラスタに参加する全てのサーバへ問い合わせがかかってしまいます(DynamoDBやMongoDB)。CPであるため、更新が完了すると、必ず読み込み時には常に同じデータが返却されます。利用するユーザからすると、DynamoDBのように不整合が生じないためとてもプログラミングしやすいです。一方、そのメリットを得るために、ある時点で、あるデータを管理するのはただひとつのサーバです。そのため、そのサーバが落ちるとリカバリ処理が必要になるので可用性が低くなります。また、斬新だったのはKVSなのに、テーブルスキーマが定義できる部分です。keyには、(Rowkey, Column, Timestamp)を入れることで、あたかもRDBMSのテーブルに似たアクセスを行うことが可能となっています。をBigtableクローンのオープンソースとしては、HBase、Hypertableなどがあります。

　次に、Amazon dynamoです。dynamoは、前述したようにAmazonならではの「もしもエラーを返したらユーザは帰ってしまう」という問題に対応したKVSです。CAP定理で言うところのAPです。いついかなる時でも書き込みを完了することが出来るという特性を持ちます。Chordと呼ばれるP2Pアルゴリズムを採用しており、スケール性はかなり高いです。ただし、OLAP用途に向いていません。どんなクエリでも全てのサーバに対してクエリ処理を投げてしまうためです。しかし、利用ユーザには使いづらい一面があります。それは、検索した時に「幾つか値があるけども、どれを選択する？」と聞かれることです。RDBMSやBigtableではありえません。可用性を上げるため、ある時点で有るキーに対してデータを管理できるサーバが複数台いるために、場合によっては、複数の状態が存在してしまうのです。クローンとしては、Riakがあります。

　オープンソースのKVSとしてシェアの大きいCasandraについて説明します。Cassandraは、Dynamoを作った開発者が、BigtableとDynamoのイイトコどりをしたアーキテクチャ設計に基づいて開発されています。テーブルスキーマを持ちつつも分散アルゴリズムはDynamoのアーキテクチャを採用し、各サーバの処理はBigtableのTabletServerを採用しています。また、可用性と一貫性のトレードオフを調整することが出来るためユースケースによって使い分けができるようになっています。また、HBase等で運用が難しいメジャーコンパクションと呼ばれる重たいGC処理において、Tired Compactionと呼ばれる機構を導入することで、メジャーコンパクション時に極端に大きく負荷がかかる減少を低減しています。

　Google Bigtableから進化したシステムを見てみましょう。MegaStoreは、Bigtableの各サーバ内においてはトランザクションを提供しつつ、Paxosアルゴリズムを用いてデータセンタ間でのレプリケーションを行うという機能を持っています。後者は、データセンタダウンにも対応するために取り入れられています。Google App Engineを始めとして、さまざまなサービスで利用されています。クローンプロジェクトは、現時点では存在していません。

　Spannerは、「プラネットスケールのデータベース」とされているものです。ある種行き着くところまで行き着いたというレベルのプロダクトです。1980年代の分散RDBMSでは2phase commitベースで分散トランザクションが提供されていたため、スケール性能は高々数台レベルです。しかし、Spannerは、プラネットスケールで分散トランザクションを提供しています。こちらもオープンソース実装は、存在していません。ここまで巨大な分散データベースは、Googleくらいしか必要がないのかもしれません。

### 分散データ処理

　MapReduceについておさらいしましょう。MapReduceは、概念的にはかなりシンプルに出来ています。その後、色々と改善手法が提案されてややこしくなっていますが、本質的におさえておけばよいのは、データがどのように流れるのか？だけです。MapReduceは、名前の通りmap処理とreduce処理からなります。map処理は入力されたデータに対して、全く同じ処理をやるだけです。reduce処理は、入力として与えられるグループされたデータの集計をしたりします。map関数は、以下のように定義されます。javaのインターフェースだと、mapの入力は(key, value)だったりするのですが、本質的には大した問題では無いのです。
map: (value) -> (key, output value)
　つまり、なんからの入力データが入ってきたら、(key, value)を返却してあげるだけです。次にreduce関数は、
reduce: (key, {input values}) -> (key, output value)
　と定義されます。ポイントは、mapで出力したkeyでグループ化されたvalueの集合が入力として飛んできます。だから、例えば、集計として平均や和を取りたければ、グループ化された範囲内で集計すれば良いことになります。mapから出力する形式が(key, value)とすることがとても重要です。そうすることで、いろんなサーバで処理されたmapは、同じkeyは同じreduceサーバに集めることが出来るのです。2章で実際にMapReduceを書いてみます。詳細はそちらを御覧ください。

# オープンデータで遊ぼう

## オープンデータの現状

　読者の皆さんは、きっと「数百台でスケールさせるようなデータをそもそも持ってないし」と思われることでしょう。近年、オープンデータと呼ばれる活動が活発化しています。政府機関の持っている情報を公開することで、世の中に役立つサービスが生まれるのではないかという意図をもって、様々な情報が公開され始めています。例えば、
 - Data@go.jp http://www.data.go.jp/for-developer?lang=japanese
 - Linked Open Data Challenge http://lod.sfc.keio.ac.jp/challenge2012/dataset.html
　などがあります。Data@go.jpでは、日本政府の省庁ごとにオープンデータが集まっています。また、Linked Open Data Challengeでは、一般からオープンデータを募りコンテストを行っています。ちょっと具体的に見てみますと、
 - 気象庁のデータ http://www.data.jma.go.jp/gmd/risk/obsdl/index.php
 - 事故情報データバンクシステム http://www.jikojoho.go.jp/ai_national/help/help.html
 - 2ch スレッドデータ http://open.ceek.jp/

　ちょっと探すだけでも、面白そうなものがたくさんあります。また、これらを用いた新しいサービス・ビジネスがどんどん作られようとしています。通産省による支援サイトもあるので、見てみると面白いかと思います。
 - Knowledge Connector http://idea.linkdata.org/all
　現時点では、オープンデータの可視化を行うプロジェクトが多いように感じます。今まで、データは存在していたがうまく可視化されないため利用されてこなかったという流れがあるようです。例えば、近くのAED設置場所を探したり、税金はどこえいった？等々。以下では、海外や日本における活用事例が載っています。
 - http://datameti.go.jp/wp-content/uploads/2014/01/03b558b1126807662402d3ebb9a98605.pdf

## 可視化ツール

　膨大なデータをわかりやすく可視化することは、とても重要です。単に表に数値だけ入っていても誰も理解できないためです。この分野においてもさまざまな提案がされています。また、Google Fusion TablesやSpread Sheetのいち機能であるmotion graphなどは無料で簡単に利用可能です。中でもIBMが2000年前半からスタートしているManyEyesプロジェクトは、眺めるだけでも大変面白いです。自社で開発するのではなく、ユーザとともに新しい可視化方法を共有するというモデルを用いて、さまざまな可視化を提案しています。
 - ManyEyes http://www-01.ibm.com/software/analytics/many-eyes/
　また、GoogleではFusion TablesやMottion Graphなど、簡単にデータを可視化したり、アニメーションに対応したGraphなどを提供しています。

## 2chスレッドデータで遊ぶ

　それでは、実際にオープンデータをMapReduceを用いて処理してみましょう。今回は、2ch スレッドデータを対象として、日本語のワードカウントをMRで行いたいと思います。各年で人気だったキーワードを探してみることが目的です。この中で、Hadoop MapReduceを使える言語はJavaだけはないこと、2つの関数を実装するだけで動作することを確認していきたいと思います。導入において、個人がMapReduceを使わなければならないほど、データを持っていないという話をしました。しかし、2ch スレッドデータは、2006年から2014年の間で圧縮後の合計で、1.7GBです。強力なマシンを持っているなら良いですが、一般的なノーパソ程度では刃が立ちませんね。そこで、MapReduceの出番になってくるわけです。閑話休題。今回のプログラムについてご紹介していきます。

### mapプログラム

　Hadoop Streamingにおけるmapは、標準入力にデータが流れてくるだけです。そこから、(key, value)を出力すれば、mapプログラムは完成です。簡単そうですね。今回言語はpythonを選択しました。この記事を参考にしてもらうことで、皆さんのお好きな言語で試すことが可能です。2chということで、日本語の形態素解析が必要になります。詳細は省きますが、まずは日本語の辞書を作成します。

http://sourceforge.jp/projects/igo/releases/
から、igo-0.4.5.jarをダウンロードします。次に、
http://sourceforge.net/projects/mecab/files/mecab-ipadic/2.7.0-20070801/
から、mecab-ipadic-2.7.0-20070801.tar.gzをダウンロードし、以下の様に実行してください。

```
tar xvzf mecab-ipadic-2.7.0-20070801.tar.gz
java -cp igo-0.4.5.jar net.reduls.igo.bin.BuildDic ipadic mecab-ipadic-2.7.0-20070801 EUC-JP
```
　上記により、ipdicディレクトリに辞書ファイルが作成されます。次に、map.pyとして以下のようなスクリプトを作成します。

```python:map.py
#!/usr/bin/python
# *-* coding: utf-8 *-*
import sys
import codecs
from igo.Tagger import Tagger

sys.stdin = codecs.getreader('utf-8')(sys.stdin)
sys.stdout = codecs.getwriter('utf-8')(sys.stdout)

dic = 'ipadicのパス'

tagger = Tagger(dic)

for line in sys.stdin:
  line.strip()
  splited=line.split(u'\t',9)
  subject=splited[2]
  date=splited[6]
  splited_date=date.split(u'-')
  year=splited_date[0]
  words = tagger.parse(subject)
  for word in words:
    if not u"名詞" in word.feature:
      continue
    print "%s-%s\t1" % (year, word.surface)
 
```
　上記は、標準入力からテキストを読み込み、形態素解析を行います。もし、名詞であると判断された場合には、その名詞を出力します。Map関数は、(Key, Value)を出力する必要が有ることを思い出してください。最後のprintでは、"<名詞><タブ>1"を出力しています。また、2006年〜2014年を一度に処理させるため、keyは、年-名詞 タブ カウントとして出力しています。

これは、「名詞」が一つ出てきたことを示しています。次に示すreducer.pyでは、この幾つも出来てくる"<名詞><タブ>1"というデータが<名詞>でグループ化された上で、バリューの集合が入力として与えられます。辞書ファイルのpathは、ipdicのフルパスを指定してください。形態素解析に読み込ませたいのは、スレッドの題名だけであり、3カラム目に存在しているため分割処理を最小限にし、かつ、3カラム目を取得して形態素解析ライブラリに渡しています。

```python:reduce.py
# *-* coding: utf-8 *-*
import sys
import codecs

sys.stdin = codecs.getreader('utf-8')(sys.stdin)
sys.stdout = codecs.getwriter('utf-8')(sys.stdout)

words = {}
for line in sys.stdin:
  word, count = line.split("\t")
  words[word] = words.get(word, 0) + int(count)

for word, count in words.items():
  print "%d\t%s" % (count, word)
```

　map関数の出力は、reduce関数の出力にそのまま入力されてきます。ただし、ここで重要なのは、複数のmapで処理されて出力されたwordは、必ず一つのreducerに集められるという約束があります。しかも、wordによってソートされた状態で入力にやってきます。これは、MapReduceが自動で行ってくれます。ファイルが分割されて複数のmapで処理されたとしても、wordのカウントを正しく求めることが出来るわけです。reducerでは、標準入力をタブで分離し、word, countを得ます。その後、各wordごとに足しこみを行いカウントを求めます。その後出力(count, word)を行います。このプログラムは、単に標準入出力を使っているだけであるので、ローカルでも実行できます。例えば、以下のように実行すると、wordcount結果が出力されます。

```
cat 2ch_thread_archive_2006.txt | python ./scripts/mapper.py | python ./scripts/reducer.py > tmp.result
```
　手元のマシンで実行してみると、8時間以上かかってしまいます。2006年〜2014年を処理したら、単純に見積もると64時間以上です。。wordcountではそれほど急ぎませんが、良いビジネスを思いついたらスピードが命です。時間を金で買いましょう。そうです、AWSのElastic MapReduce(通称:EMR)を用いて処理してみましょう。EMRは、比較的簡単に、Hadoop MapReduceを動作させることが出来るようになっています。すでに、map.pyとrecduce.pyは作成済みです。あとは、EMR上でmap.pyとreduce.pyが動作するように、ipdicやigo-pythonを起動にインストールしてやれば良いことになります。

　EMRでMapReduceを開始する前段の処理を行うためのshellを以下のように用意します。

```boot.sh
#!/bin/bash

# 必要なファイルをDLする
cd /mnt/var/lib/hadoop/tmp
hadoop fs -get s3n://nanaka/2ch/scripts/ipadic .

# 実行環境を整える
sudo easy_install igo-python
```

　s3n://nanaka/2ch/scripts/ipadic は、ipadicが存在するs3上のパスを指定しています。s3は、AWSが提供しているファイル格納庫であり、EMRからシームレスにアクセスすることが可能です。s3n://nanaka/2ch/scripts/に、前述したmap.py, reduce.pyもuploadしましょう。inputデータ(2chのスレッドデータ)は、s3n://nanaka/2ch/input に配置してください。この際、gzip圧縮されたファイルであれば、EMRは自動で判別し展開してくれるのでよろしいです。

AWSにアカウントを作成すると、EMRという項目があるので選択をします。色々と設定する項目がありますが、奥することはありません。重要なことはそれほど多くないです。

<table summary='表1::AWS EMRの設定項目一覧'>
  <tr>
     <th>設定項目</th>
     <th>設定値</th>
     <th>備考</th>
  </tr>
  <tr>
     <td>Cluster name</td>
     <td>何でも良いです</td>
     <td></td>
  </tr>
  <tr>
     <td>Termination protection</td>
     <td>No</td>
     <td>強く推奨。実行が終われば課金されません</td>
  </tr>
  <tr>
     <td>Logging</td>
     <td>Enabled</td>
     <td>推奨。ログをWebUIから取得できます</td>
  </tr>
  <tr>
     <td>Log folder S3 location</td>
     <td>s3:path/to/</td>
     <td>s3上のパスを指定</td>
  </tr>
  <tr>
     <td>Debugging</td>
     <td>チェックON</td>
     <td>推奨。ログをWebUIから取得できます</td>
  </tr>
  <tr>
     <td>Tags</td>
     <td></td>
     <td>入れても大丈夫</td>
  </tr>
  <tr>
     <td>Hadoop distribution</td>
     <td>Amazonの最新を選択</td>
     <td></td>
  </tr>
  <tr>
     <td>Additional applications</td>
     <td>デフォルトで入っているものを全て削除</td>
     <td>使わないのに起動時間がかかって、課金が増えてしまいます</td>
  </tr>
  <tr>
     <td>File System Configuration</td>
     <td>チェックしない</td>
     <td></td>
  </tr>
  <tr>
     <td>Hardware Configuration</td>
     <td>タスクに合わせて選んでください</td>
     <td></td>
  </tr>
  <tr>
     <td>Security and Access</td>
     <td></td>
     <td>EMR起動中に、ログインしたければ必要です</td>
  </tr>
  <tr>
     <td>IAM Roles</td>
     <td>デフォルト</td>
     <td></td>
  </tr>
  <tr>
     <td>Bootstrap Actions</td>
     <td>先ほど作成したboot.shを指定</td>
     <td></td>
  </tr>
  <tr>
     <td>Steps</td>
     <td>追加してください</td>
     <td>これがMRの設定です。詳細は、以下に続きます。</td>
  </tr>
</table>


引き続き、Steps内での設定を行います。設定項目を以下にまとめます。
<table summary='表1::AWS EMRの設定項目一覧'>
  <tr>
     <th>設定項目</th>
     <th>設定値</th>
     <th>備考</th>
  </tr>
  <tr>
     <td>Name</td>
     <td>自由につけてください</td>
     <td></td>
  </tr>
  <tr>
     <td>Mapper</td>
     <td>S3上にuploadしたmap.pyを指定</td>
     <td></td>
  </tr>
  <tr>
     <td>Reducer</td>
     <td>S3上にuploadしたreduce.pyを指定</td>
     <td></td>
  </tr>
  <tr>
     <td>Input S3 location</td>
     <td>Inputデータを格納したS3上のディレクトリを指定</td>
     <td>ディレクトリに有るすべてのファイルが処理されます</td>
  </tr>
  <tr>
     <td>Output S3 location</td>
     <td>出力フォルダを指定</td>
     <td>存在しないディレクトリを指定しましょう。</td>
  </tr>
  <tr>
     <td>Action on failure</td>
     <td>Terminate</td>
     <td>必ず、こうしましょう</td>
  </tr>
</table>


　さあ、いよいよCreate Clusterボタンを押すときがやって来ました。ポチッとな。あとは、EMRがよろしくやってくれます。待ちましょう。m1.middle x 4で実行した場合、10時間程度で終わりました。あるmapタスクが最後のほうで落ちてしまったおかげで、時間がかかっていますがうまく行っていれば5時間程度の処理だったでしょう。かかった金額は、EC2でm1.middle x 5 x 10 hours = $6.1 と、EMRで、$1くらいです。S3転送量料も含めると合計$8くらいでしょうか。ということで、ローカルでの予想処理時間64時間 - AWSでかかった時間 10時間 = 54時間を$8で買った感じになりました。

　最後に、カウントしたワードの中から、上位の中から幾つかピックアップしてキーワードのスレッドタイトル出現集についてグラフ化してみました(図3)。

![図3](pic3.png)

　時代によって、出現数が変動していたり、逆に変動していなかったりということがわかります。他にも色々な出現単語で確認しても面白いかもしれません。


## おわりに

　嫁「あーぜんぜん分からなかった。これでアフィブログが儲かるの？」
　私「そこまで行くには時間がかかる。だけど、サービス残業で死ぬまで働くなってゴメンだからな。そこに行くまで寝ないで頑張る。」
　嫁「…」

