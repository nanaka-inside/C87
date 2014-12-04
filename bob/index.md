# Pebbleに関してなにか書く
候補
* Pebble入門的な内容
	* Pebbleとはなんぞや
	* 現在の開発環境
		* Pebble SDK
		* Pebble.js
		* Simply.js
	* Watchfaceをつくろう
		* API叩いてPebbleで表示
		* 公式のチュートリアルと似た感じ
* 具体的になにか作った話
	* Yahoo!雨雲レーダを使って自宅周辺の天気状態を確認
		* 雨雲レーダの画像をパースした結果を返すAPI生やす(golang+martini)
		* 親機からそれを叩いてPebbleで表示
			* Herokuにデプロイ
		* ほぼAPI叩いて結果を見せるという点でチュートリアルの内容と変わらない 
			* うーむ…
	* Garoonを使って次のMTG部屋を確認
		* ドワンゴLTで発表したもの
		* Pebbleで何かしたというよりもGaroonのAPIが辛い、というだけ
			* XMLだからより辛い
			* 泥臭いことを多くやって対応してる
		* 設定画面の作成までは済み
* Watchappを作る話
	* ※まだ作ったことない
	* ◯✕ゲームぐらいなら簡単に作れそう