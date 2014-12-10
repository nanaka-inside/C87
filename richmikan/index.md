Title: メトロパイパー
Subtitle: 偉大なる秘密結社シェルショッカーに捧ぐ
Author: @richmikan
Author(romaji): richmikan

# お前たち夏コミで洗脳されたか？

世界70億の人間どもよ、また会ったな。私は偉大なるPOSIX原理主義集団「秘密結社シェルショッカー」日本支部のリッチー大佐である。夏コミで見せてやったシェルスクリプト製ショッピングカートにして我日本支部の怪人第一号である「シェルショッカー1号男」で、さぞ洗脳されたことであろう。幸いにして我々の前には「正義の味方」なる宿敵もおらん。シ○ッカーという組織は、改造人間を作る際、身体の改造を先にやって洗脳作業を後回しにしたからな。だから滅びおったわけだ。洗脳を先にやっておけばよいものを……。

これを読んでいるお前たちはとっくに洗脳されておるな。……なに、今何と言った!!……むむ、まだ洗脳が足りんようだな。いいだろう。お前たちに今から、我組織が改造して造り上げた第二の怪人、恐怖！「メトロパイパー男」を見せてやる。これは、「オープンデータ活用コンテスト」開催と称してまんまと路線データを公開しおった東京メトロを侵略するための怪人。今度こそ確実に洗脳されるはずだ。ワッハッハッハ！

# 序章. 我組織が誇る三つの兵器

シェルショッカー1号男の開発の時には出番がなかったが、POSIX原理主義を貫く我々組織には三つの兵器があるのだ。これは日本支部のみならず、世界中の支部に配備されている。教えてやろう。それはCSVデータ、JSONデータ、そしてXMLデータを解析するパーサーだ。今世界にはこれらのテキストデータが溢れている。人間どもがやりとりをするこれらのデータを傍受し、世界征服への次なる一手を日々考えているのだよ。

恐怖！メトロパイパー男を見せる前にその、パーサー(先にソースコードが見たいならGitHubに置いてあるから見るがいい。URLは`https://github.com/ShellShoccar-jpn/Parsrs`だ。)の威力を見せてやろう。

## jqやxmllintに首領様が怒った

CSVの話は置いておくが、JSONテキストのパースならjqというコマンドが、そしてXMLテキストのパースならxmllint、hxselect(W3Cが出している“HTML-XML-utils”というユーティリティーに収録されている。)、xpath(MacOS Xにあるコマンド)といったものが既にある。だが、我組織の首領様はそのどれもがお気に召さなかった。「POSIX原理主義者が身に付けておくべきUNIX哲学を無視している」とお怒りになった。理由は次のとおりだ。

### 理由1. 一つのことをうまくやっていない

UNIX哲学の一つとしてよく引用されるマイク・ガンカーズの教義(「UNIXという考え方―その設計思想と哲学」を読むがいい。)に

1. 小さいものは美しい。
2. 1つのプログラムには1つのことをうまくやらせよ。

というのがあるが、まずこれができておらん。jqやxmllint等は、データの正規化（都合の良い形式に変換する）機能とデータの欲しい部分だけを抽出する部分抽出機能を分けていない。むしろ前者をすっ飛ばして後者だけやっているように見える。

だがUNIX使いとしては**部分抽出といったらgrepやAWK**を使い慣れているわけで、それらでできるようにしてもらいたいのだよ。部分抽出をするために、**jqやxmllint等独自の文法をわざわざ覚えたくもない**。

だから、正規化だけをやるようなコマンドであってほしかった。

### 理由2. フィルターになりきれてない

同じく教義の一つに

9. 全てのプログラムはフィルターとして振る舞うように作れ。

というものがある。フィルターとは、入力されたものに何らかの加工を施して出力するものをいうが、jqやxmllint等は、出力の部分に注目するとフィルターと呼ぶには心もとないのだよ。そうは思わんか？

なぜなら、JSONパーサーはJSONテキストの一部をJSON形式のまま出力し、XMLパーサーはXMLテキストの一部をXML形式のまま出力する。我々は**JSONやXML形式が扱いづらいからパーサーに掛けたいのであって、サブセットを渡されてそれをどーしろと**言うのか！そんなものを標準UNIXコマンドに放り込んでも、どうにも料理できないではないか。AWK、sed、grep、sort、tr……といった**UNIX標準コマンド群は、行単位あるいは列単位（空白区切り）のデータ加工に向いた仕様**になっておるのだ。

というようにUNIX哲学の心を全く理解しておらんのだよ。

## そして兵器は開発された

そこで我組織は秘密裏に、自らの手でパーサー兵器を開発した。使い勝手を揃えたCSVパーサーも併せてな。それがParsrs（パーサーズ）だ。→`https://github.com/ShellShoccar-jpn/Parsrs`

これがいかに優れているか、早速デモしてやる。

### JSONデータをパースする

まず次のようなJSONデータがあったとしよう。会員の商品購入履歴だ。

```
{"会員名" : "文具 太郎",
 "購入品" : [ "はさみ",
              "ノート(A4,無地)",
              "シャープペンシル",
              {"取寄商品" : "替え芯"},
              "クリアファイル",
              {"取寄商品" : "6穴パンチ"}
            ]
}
```

これを我らのJSONパーサー、parsrj.shに掛けるとこうなる。

```
$.会員名 文具 太郎
$.購入品[0] はさみ
$.購入品[1] ノート(A4,無地)
$.購入品[2] シャープペンシル
$.購入品[3].取寄商品 替え芯
$.購入品[4] クリアファイル
$.購入品[5].取寄商品 6穴パンチ
```

JSONのデータ構造は、ファイル・ディレクトリー構造に似ている。後者で重要なものがファイルパスとファイルの中身であるように、JSONも値の場所と値が重要なのだ。幸いJSONには“JSONPath”(`http://goessner.net/articles/JsonPath/`参照)という値の場所を示すための規格があるから、JSONPathと値を1行に並べたデータにすればよい。keyとvalueの2列から構成されるテーブルに変換できたことになる。

### XMLデータをパースする

同じデータがXML形式で表現されている場合も同じだ。例えば次のようなデータになっていたら、

```
<文具購入リスト 会員名="文具 太郎">
  <購入品>はさみ</購入品>
  <購入品>ノート(A4,無地)</購入品>
  <購入品>シャープペンシル</購入品>
  <購入品><取寄商品>替え芯</取寄商品></購入品>
  <購入品>クリアファイル</購入品>
  <購入品><取寄商品>６穴パンチ</取寄商品></購入品>
</文具購入リスト>
```

我らのXMLパーサーにかければこうなる。

```
/文具購入リスト/@会員名 文具 太郎
/文具購入リスト/購入品 はさみ
/文具購入リスト/購入品 ノート(A4,無地)
/文具購入リスト/購入品 シャープペンシル
/文具購入リスト/購入品/取寄商品 替え芯
/文具購入リスト/購入品
/文具購入リスト/購入品 クリアファイル
/文具購入リスト/購入品/取寄商品 ６穴パンチ
/文具購入リスト/購入品
/文具購入リスト \n  \n  \n  \n  \n  \n  \n
```

XMLには、XPathというJSONPath同様(同様というか規格としてはXPathの方が先なのだが。)に値の場所の表現法がある。従ってXPathとその値を1つの行並べればよい。

こうすると何が嬉しかといえば、UNIXコマンドで好きなように料理できるようになるということだ。例えば、店に並んでいない取り寄せ商品の一覧が知りたければ、パース後のテキストデータに対して`grep '取寄商品' parsed_data.txt`などと書けばよい。さらに、それが何個あったか知りたければ`grep '取寄商品' parsed_data.txt | wc -l`と、後ろにパイプでコマンドを繋げばよい。UNIXコマンドとの相性もばっちりではないか。

## Parsrsもまた、パイプで繋いだコマンドの塊

我々はPOSIX原理主義集団であるから、Parsrsの中身ももちろんシェルスクリプトとUNIX標準コマンド群でできておる。嘘だと思うなら、GitHubに上げてあるソースコード飽きるほど眺めるがいい。→`https://github.com/ShellShoccar-jpn/Parsrs` 見よ、このパイプで繋がれたコマンドの塊を！

![POSIX原理主義XMLパーサー（抜粋）](images/parsrx_src.png)

これまたUNIX哲学の教義だが、

6. ソフトウェアは「てこ」。最小の労力で最大の効果を得よ。
7. 効率と移植性を高めるため、シェルスクリプトを活用せよ。

というのがある。これに基づいて開発された我らのパーサーは、C言語に頼らずともそこそこのスピードが出ておる（マルチコア環境でコア数が多いほど威力が増すのだ。）し、かつC言語で書いたプログラムと違ってコンパイル不要。UNIX系OSでありさえすれば、コピーするだけでどこでも動くぞ。どぅぉーだ、スゴいだろう。わっはっは！

そしていよいよ、恐怖！メトロパイパー男が誕生するのだ。

# 第一章. 東京メトロ、コンテスト開催

2014年9月某日。日本の帝都の高速度交通インフラを握るあの東京メトロが、自社の鉄道網に関するデータをWebAPIを通してオープンデータとして公開し、コンテストを実施すると発表した。発表によれば、公開されるデータの概要は次のとおり。

* 方向（どこ方面行きか）
* 列車番号
* 列車種別（各停、特急、急行、快速、臨時） ※非営業列車（試運転、回送）は公開しない
* 始発駅・行先駅
* 所属会社（どの鉄道事業者の車両か）
* **在線位置**（ホーム、駅間の2区分）
* 遅延時間（5分以上の遅延を「遅延」として表示）
* 列車情報（列車時刻表、運賃表、駅間所要時間、各駅の乗降人員数、女性専用車両）
* 施設情報（バリアフリー情報、駅出入口情報、車両ごとの最寄り施設・出入口案内）

何と！在線位置、つまり列車が今どこを走っているかまで公開するではないか。東京メトロとしては「このデータを使って、人々の役に立つアプリケーションを作ってください」という目的なようだが、我々秘密結社シェルショッカーに目を付けられるとは奴らも運が悪かったな。

よーし、その在線情報を傍受し、我々が東京メトロを、ついでにコンテストを侵略して世界征服の礎にしてやるわ!!! わーっはっはっは。

## WebAPI仕様は単純だった

9月12日。コンテストの応募受付が開始されると同時にWebAPIが公開された、具体的な仕様、そして実データの姿が明らかになったわけだ。

我々は侵略の第一歩として、早速使用方法の解析に入った。アクセスするためには「アクセストークン」なるIDの発行を申請する必要があるということだった。これはTwitterで言うなら独自アプリを作る時に申請する“Application ID”なわけだ。「もしや、APIを叩くにはこのIDでOAuth認証をして……」と思ったのだが、単にアクセス用のURLにCGI変数として付ければよいだけだった。じつに簡単ではないか。侵略をしようとしている手前、あまり足が付くようなことはしたくないのでな……。

後程紹介するメトロパイパー男のソースコードを見てもらえばわかるが、WebAPIのURLはこうだ

`https://api.tokyometroapp.jp/api/v2/datapoints`

これに知りたい情報の種別と、アクセストークンの2つをGETメソッドで渡す。絞り込みをしたければ（例えば9路線のうちの「丸ノ内線だけ」など）その条件を付記しても構わない。

例えば、在線情報を千代田線について知りたければこういうURLを生成してアクセスするだけだ。

`https://api.tokyometroapp.jp/api/v2/datapoints?rdf:type=odpt:Train=odpt.Railway:TokyoMetro.Chiyoda&&acl:consumerKey=ここにアクセストークン`

これなら、curlコマンド一発で済むな(なに、**「curlコマンドはPOSIXじゃないだろう!」**だと？まぁ、そこは我々にとって唯一の弱点なのでそれ以上追及するな。)。ところで、肝心のアクセストークンはコンテストの応募締切を過ぎた現在、発行が停止されてしまった。「それのどこがオープンデータなのか」と批判も多いのだが、2015年3月以降コンテストが完全に終わった頃に真のオープンデータとしてこのAPIが帰ってくる可能性があるので、お前たちも期待して待っているがいい。

## JSON-LDというのがちと気になった

東京メトロがコンテストの概要を発表した際、データは“JSON-LD”という形式で送信されるとされた。JSON-LDとはJSON for Linking Dataの略で、一言で言うならハイパーリンクの埋め込まれたJSONということになる。もしや肝心なデータに辿り着くには、最初に読み込んだJSONの中にあるリンク先のJSONの中にあるリンク先のJSONの中にある……（以下略）……、ということを想像したのだがこちらの欲しいデータ（列車在線位置情報）に関しては最初に読み込むJSONの中に直接書かれている内容で完結していた。

```
[
 {"@context":"http://vocab.tokyometroapp.jp/con
text_odpt_Train.jsonld",
  "@type":"odpt:Train",
  "@id":"urn:ucode:_00001C000000000000010000030
CB30E",
  "dc:date":"2014-12-05T01:08:22+09:00",
  "dct:valid":"2014-12-05T01:09:52+09:00",
  "odpt:frequency":90,
  "odpt:railway":"odpt.Railway:TokyoMetro.Chiyo
da",
  "owl:sameAs":"odpt.Train:TokyoMetro.Chiyoda.B
2519K",
  "odpt:trainNumber":"B2519K",
  "odpt:trainType":"odpt.TrainType:TokyoMetro.L
ocal",
  "odpt:delay":0,
  "odpt:startingStation":"odpt.Station:TokyoMet
ro.Chiyoda.KitaSenju",
  "odpt:terminalStation":"odpt.Station:JR-East.
Joban.Matsudo",
  "odpt:fromStation":"odpt.Station:TokyoMetro.C
hiyoda.KitaSenju",
  "odpt:toStation":null,
  "odpt:railDirection":"odpt.RailDirection:Toky
oMetro.Ayase",
  "odpt:trainOwner":"odpt.TrainOwner:JR-East"
 }
]
```

これは実際に取得された在線情報データの例だが、データの一番最初（`@context`）にこのデータ構造がどう定義されているかを示した別のJSONデータへのリンクが記されているだけで、列車とその在線位置を特定するに必要なデータは全てこの中にあるではないか。脅かしおって、これなら我らが開発したJSONパーサーで十分だ。

# 第二章. 恐怖！メトロパイパー男、誕生

9月16日。コンテスト開催から4日の短期間で試作体が完成した。

## その恐るべき戦闘能力

### 簡単操作！

### 情報の鮮度が一目でわかる！！

### どこでも動き、10年後も動く！！！

## 11月17日、完全体に


# 第三章. メトロパイパー男の秘密

## ブロック図

## 変わりゆくデータ

## HTMLにハメる


# 第四章. コンテストを制圧するのだ！

ゆけ、メトロパイパー男よ！コンテストを制圧し、世のプログラマー達を洗脳して世界征服の礎を築くのだ!!!

わっはっは。

