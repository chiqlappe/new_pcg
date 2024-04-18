2024/4/18

# 新PCG(仮称)仕様書


## 1. ハードウェア概要
PC-8001でユーザー定義キャラクタ(PCG)を使用可能にする


## 2.1 I/Oポート

|I/O|アドレス|用途|
|:--|:--|:--|
|OUT|$00|ラインデータ #0|
|OUT|$01|ラインデータ #0|
|OUT|$02|ラインデータ #0|
|OUT|$03|ラインデータ #0|
|OUT|$04|ラインデータ #0|
|OUT|$05|ラインデータ #0|
|OUT|$06|ラインデータ #0|
|OUT|$07|ラインデータ #0|
|OUT|$08|コントロール|


## 2.2 ラインデータ
 - 8x8ドットのキャラクタを垂直方向に8分割し、上からライン#0とする
 - ラインデータは1バイトで、MSBが左端のドットに対応する
 - PCGはビット反転して出力されるため、データはあらかじめ反転したものを作成すること

## 2.3 コントロール

 |BIT|用途|
|:--|:--|
|0,1|書き込みバンク# (bit0=LSB)|
|2|PCGスイッチ (1=on)|
|3|モードスイッチ (1=ブロックモード)|
|4~7|予約|


## 3.1 モード
- 2種類の表示モードが存在し、登録可能なPCGの最大数と使用可能エリアに違いがある

|モード|PCGの最大数|使用可能エリア|
|:--|:--|:--|
|ノーマル|256|制限なし|
|ブロック|800|制限あり (7.ブロックモードを参照)|


## 4.1 CG-RAM
- CG-RAMは2KBバイトずつ4つのバンクに分割されている

## 4.2 アドレスマップとバンク
|アドレス|バンク|用途|
|:--|:--|:--|
|$0000~$07FF|0|ノーマルモード、ブロックモード共用データエリア|
|$0800~$0FFF|1|ブロックモード専用データエリア|
|$1000~$17FF|2|ブロックモード専用データエリア|
|$1800~$1FFF|3|ブロックモード専用データエリア|


## 5.1 PCGのデータ登録
CG-RAMにPCGのデータを登録する手順は下記の通り

(1) コントロールポートで書き込みバンク#をセットする
(2) 垂直帰線消去期間を待機
(3) Bレジスタにキャラクタコードをセットする
(4) Cレジスタにライン番号をセットする
(5) Aレジスタにラインデータをセットする
(6) OUT(C),A を実行する

(4)から(6)を8回くりかえすことで登録が完了する

## 5.2 注意
予期しないデータが書き込まれるのを防ぐため、データ登録は必ず垂直帰線消去期間内に行うこと
1キャラクタのデータ登録が完了するまでBレジスタの値を変化させないこと
Bレジスタを使用するため、BASICのOUT命令では登録できない


## 6.1 PCGの表示
PCGを表示する手順は下記の通り

(1) 表示したい領域のVRAMアトリビュートをリバースにする
(2) コントロールポートのPCGスイッチ(bit2)をonにする

VRAMのアトリビュート情報でキャラクタ表示を切り替えられるため、PCGとROMキャラクタの同時使用が可能である

|アトリビュート|表示|
|:--|:--|
|ノーマル|ROMキャラクタ|
|リバース|PCG|


## 7.1 ブロックモード
画面を行で4分割し、それぞれにCG-RAMのバンクを割り当てることで256種類を超えるPCGの表示を可能にする

7.2 行番号とバンクの関係
|行番号|バンク|
|:--|:--|
|0~7|0|
|8~15|1|
|16~23|2|
|24|3|

## 7.3 動作原理と制限
ブロックモードではCRTCからのVSP信号をカウントして行番号を識別している。
25行モードでは1行あたり8回VSP信号が送られるので、8行(VSP信号64回)ごとにCG-RAMのバンクを１つ進めている。
一方で20行モードでは1行あたり10回VSP信号が送られるためカウントにズレが生じ、表示が崩れてしまう。
このためブロックモードは25行モードで使用しなければならない。

|VSPカウンタ値|行番号(25行モード)|
|:--|:--|
|0~63|0~7|
|64~127|8~15|
|128~191|16~23|
|192~199|24|



