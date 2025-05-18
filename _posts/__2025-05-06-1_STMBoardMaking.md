---
title: "Fusion360で作るSTM32基板" #タイトルを入力
categories: #分類
  - 回路設計
tags: #タグ
  - 基板設計
  - Fusion360
  - STM32
comments: true
---
## 目次
1. [目次](#目次)
2. [目的](#目的)
3. [ピン配置決め](#ピン配置決め)
4. [電源周り](#電源周り)
5. [書き込み周り](#書き込み周り)
6. [マイコン本体](#マイコン本体)
7. [周辺部品の接続](#周辺部品の接続)

## 目的
こんにちわ。今回はSTM32の基板を作ろうと思います。まあ目的としてはメイン基板が必要だからですね。<br>
まあメイン基板とはいっても疑似的なUSB-CAN基板です。基本的には過去に作ったサブ制御基板にCANを送ることが主目的です。STMを使う意味とは…？<br>
やだやだやだ！STM32使いたい！！！もともとサブ制御基板でもSTM使いたかったけどお金内から断念したんだからメイン基板くらいはSTM使いたい！！！！<br>
まあSTMを使う理由としては中枢を占めるメイン基板はある程度高機能かつ高クロックなSTMを使いましょうということです。あとMPLAB 〇 IDEとかいうIDEを使って開発したくない。<br>
あっ先言っとくと私はプロではないので普通回路班が「この配線はないわー」っていう配線も平気で行います。見つけたら注意してあげてね。<br>

## ピン配置決め
まあPICとは異なり結構ピン配置に制約が設けられているのでうまいこと決めます。<br>
まず大前提となるのは書き込みまわり。今回は[こちら](https://akizukidenshi.com/catalog/g/g118187/)のSTlink-V3MINIEを利用します。<br>
書き込みピンはいかに示す通りです。14ピンあるうち、気にしないと行けないのは表に示します。
![STlinkPin]({{site.baseurl}}/assets/Picture/STMBoardMaking/STlinkPin.png)<br>
<table>
	<tbody>
		<tr>
			<td>T_VCC</td>
			<td>VCCで供給してくれるように見えるが、実際はマイコンの電源電圧測ってHighレベル信号の電圧調整してるだけなんで出力してくれない。マイコンの電源とつなぐことは推奨しとく</td>
		</tr>
		<tr>
			<td>T_SWDIO</td>
			<td>書き込みのピン。名前からしてIO（InputOutput）してそう。</td>
		</tr>
		<tr>
			<td>GND</td>
			<td>GND。実家のような安心感</td>
		</tr>
		<tr>
			<td>T_SWCLK</td>
			<td>当然のようにCLK（CLocK）。こういう業界でしか通じない略し方嫌い</td>
		</tr>
		<tr>
			<td>T_SWO</td>
			<td>書き込みのピン</td>
		</tr>
		<tr>
			<td>GNDDETECT</td>
			<td>GNDじゃないから見逃しそうになるがこいつもGNDにつなぐ</td>
		</tr>
		<tr>
			<td>T_NRST</td>
			<td>なんかリセットピンにあたるらしい。</td>
		</tr>
		<tr>
			<td>T_VCP_RX</td>
			<td>UARTのRX。PA10をつなぐ</td>
		</tr>
		<tr>
			<td>T_VCP_TX</td>
			<td>UARTのTX。PA9をつなぐ</td>
		</tr>
	</tbody>
</table>
まあこんな感じですね。ここに書いてないピンでも警戒しないといけないピンはあります。それがBOOT0です。ここをミスると一生プログラムが実行されません。簡単に言うと起動モード切り替えで、プログラム実行モードとSTlink以外のフォーマットによる特殊書き込みモードの切り替えを行います。<br>
あとクロック回路です。マイクラで初めて聞いたことがある人多そう。まあ普通にクリスタルにつなぐやつですね。今回、外部クロックはHSEしか使わないためPH0,PH1（PF0,PF1）に水晶を取り付けます。<br>
さて。これでようやく自由にペリフェラルを設定できるようになります。まあそうはいっても事前にちゃんとどこにどの機能のピン割り当ててーとかやるよりも作りながら調べたほうが楽なので調べながら組みます。

## 電源周り
とはいっても初っ端からSTMのピン接続するわけではありません。とりあえず電源回路組みましょう。電源基板はType-Cケーブルで引っ張ってくるつもりなので、Type-Cで5Vを引き出しましょう。また、供給がわかりやすいように、5VでLEDを光らせます。また、電源の逆流防止にダイオードをつけておきます。<br>
![Power_1]({{site.baseurl}}/assets/Picture/STMBoardMaking/Power_1.png)<br>
んで、このままだとSTMには入れられません。なぜならSTMはイマドキマイコンらしく3.3Vしか入力電源が入らないから。3.3Vしか入れられないマイコンをZ世代マイコンと呼ぼう（提案）<br>
さて。まあレギュレーターが必要なわけですが、どれにしようか悩んだ結果[NJM2845](https://akizukidenshi.com/catalog/g/g111299/)を使用します。
まあ800mAも流せれば十分だべ。で、逆流防止にダイオードつけて、入力電圧をセラコン及び電解コンデンサでノイズ除去してこんな回路図の出来上がり<br>
![Power_2]({{site.baseurl}}/assets/Picture/STMBoardMaking/Power_2.png)<br>
なんかNJMがエグゾディアに見えてきた…
## 書き込み周り
さっき書いちゃったせいでなにもいうことがありませんが、以下の配線のとおりです。
![Write_1]({{site.baseurl}}/assets/Picture/STMBoardMaking/STlink.png)<br>

## マイコン本体
さて。ピン配置が以下の通りです。リセットスイッチやブートモード切り替えスイッチも含まれてます。大まかにいえばCAN2Ch,UARTがSTlinkとの通信込みで3Ch,I2Cが1Ch,GPIOが24ピンです。BOOT0は10kΩとりつけてます。こちらも特に説明することはないですね。<br>
![Write_1]({{site.baseurl}}/assets/Picture/STMBoardMaking/STMPinMap.png)<br>
## 周辺部品の接続
大量にタグをつけたGPIOに何をつなぐかって話ですね。<br>
まずはCAN！CAN1,CAN3はMCP

[Type-C](https://akizukidenshi.com/catalog/g/g116438/)<br>
[NJM2845](https://akizukidenshi.com/catalog/g/g111299/)<br>