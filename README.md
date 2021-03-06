# 横浜ガジェット祭り2017に出展する<br>3D LEDのブロック認識プログラム


## 本プログラム

カメラから入力した画像の中からLEGO Duploを探し出して、ブロックの色を認識するプログラム。  
認識した色の情報は3D LED点灯命令へ変換しPythonプロセスへ送信する。  
Pythonプロセスのプロジェクトは以下。  
https://github.com/tatsuo98se/3d_led_cube2

## 最新バージョン

- 0.9.10

## 言語

- c++11(Visual Studio 2013で開発)

## ライブラリ

ビルドでは必要だが実行には不要
- opencv
- boost
- picojson

## 動作環境

- windows  
開発はMacでも行っているので、XCODEプロジェクトもコミットしている。
- logicool C270カメラ  
専用のドライバソフトもインストールすること（カメラの設定を編集するため）  
Macはベンダー提供のドライバがないので、カメラ自体は動作するが設定は編集不可。
- Arduinoボタン  
ボタンを押すと、Pythonプロセスへコマンドを送信する。ボタンがないときは、標準入力がトリガーとなる。

## 使い方

- ローカルホスト（ポート80）にTCP送信するモード（通常使用モード）  
`block_identifier -a ::1 -p 80`  
カメラプレビューが表示されるので、うまくブロックを認識するようカメラ角度やブロックを調整する。  
コマンドラインで適当な文字を入力しENTERを押すと、TCP送信する。  
うまく繋がらないとき（Windowsの設定によると思われる）は::1をlocalhost、127.0.0.1、WindowsのIPアドレスなどへ変更する必要あり。
- TCP送信しないモード（カメラデバッグ等）  
`block_identifier`
- WEBカメラではないカメラ（インカメラなど）が表示されてしまうとき  
`block_identifier -d 1`
- カメラを繋がないとき（TCPデバッグ等）  
`block_identifier --debug`
- 色数や命令を変更する  
`block_identifier -g`  
block_identifier.xmlが出力される。  
colorタグ、instuctionタグの中身を修正する。（色名、色基準値、命令と色の紐付け）  
`block_identifier -o block_identifier.xml`

## LEGO DUPLOとLEGOの変更方法

XMLファイルを出力する。  
`block_identifier -g`

以下の設定を変更する（デフォルトはLEGO DUPLO）

* LEGO DUPLO
```xml
<stud_threshold>40</stud_threshold> <!-- ポッチサイズ -->
<block_height>102</block_height> <!-- ブロック高さ -->
<block_width>150</block_width> <!-- ブロック幅 -->
```
* LEGO
```xml
<stud_threshold>20</stud_threshold> <!-- ポッチサイズ -->
<block_height>51</block_height> <!-- ブロック高さ -->
<block_width>75</block_width> <!-- ブロック幅 -->
```

XMLを読み込んで実行する  
`block_identifier -o block_identifier.xml`

## 認識する色を編集する

XMLファイルを出力する。  
`block_identifier -g`

XMLファイルを開き、colorを編集する。  

```xml
<color>
  <count>6</count> <!-- 色数 -->
  <item>
    <name>green</name> <!-- 色名 -->
    <bgr>
      <b>128</b> <!-- B値 -->
      <g>255</g> <!-- G値 -->
      <r>128</r> <!-- R値 -->
    </bgr>
  </item>
</color>
```

## ブロックと命令の対応を編集する

「ブロックの色、幅」がキー、「命令、パラメータ」がバリューのマップとして紐づけます。  
XMLファイルを出力する。  
`block_identifier -g`

XMLファイルを開き、block-instruction-mapを編集する。  

キーとバリューの対応は以下で記述する。  

```xml
<item>
  <first>
    <color>yellow</color> <!--ブロック色-->
    <width>3</width> <!--ブロック幅-->
  </first>
  <second>
    <name>ctrl-loop</name> <!--命令名-->
    <param>
      <count>1</count> <!--パラメータ数-->
      <item_version>0</item_version> <!--固定値-->
      <item>
        <first>count</first> <!--パラメータ名-->
        <second>1</second> <!--パラメータ値-->
      </item>
    </param>
  </second>
</item>
```

## コマンドラインオプション

  -h [ --help ]            Show help  
  -v [ --version ]         Print software version  
  -g [ --generate ]        Generate option file  
  -o [ --option ] arg      Option file path  
  -d [ --device ] arg (=0) Camera device number if PC has multiple camera devices  
  -a [ --address ] arg     Python process IP address  
  -p [ --port ] arg (=80)  Python process port number  
  -c [ --com ] arg (=0)    COM Post if you use Arduino Button  
  --debug                  DEBUG mode

## 未実装項目

- ブロック色と命令の紐付けがテキトーすぎる（暫定のつもり）  
一応、XMLファイルで変更や追加はできる。デフォルト値がいい加減なだけ。
- キャリブレーションモード未実装  
一応、XMLファイルを手で修正すれば校正は可能（手動であり自動でないだけ）
