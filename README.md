# Simple Clang Compiler

# Description
本プログラムはRui Ueyama氏による「[低レイヤを知りたい人のためのCコンパイラ作成入門](https://www.sigbus.info/compilerbook)」に習って作成しているC言語によるCコンパイラプロジェクト。副読本として「[コンパイラ: 作りながら学ぶ (日本語) 単行本 – 2017/10/25](https://www.amazon.co.jp/%E3%82%B3%E3%83%B3%E3%83%91%E3%82%A4%E3%83%A9-%E4%BD%9C%E3%82%8A%E3%81%AA%E3%81%8C%E3%82%89%E5%AD%A6%E3%81%B6-%E4%B8%AD%E7%94%B0-%E8%82%B2%E7%94%B7/dp/4274221164/ref=asc_df_4274221164/?tag=jpgo-22&linkCode=df0&hvadid=295719984664&hvpos=&hvnetw=g&hvrand=12517034168150798774&hvpone=&hvptwo=&hvqmt=&hvdev=c&hvdvcmdl=&hvlocint=&hvlocphy=1009279&hvtargid=pla-527007822676&psc=1&th=1&psc=1)」も。このreadmeは半分くらいメモ

# 環境構築
rui ueyama氏のブログよりdockerfileを拝借する
```zsh
$ docker build -t compilerbook https://www.sigbus.info/compilerbook/Dockerfile
```
個人的には対話的にシェルを走らせたいので、以下のコマンドで実行環境を走らせる
```zsh
$ docker run -it -v $PWD/:/9cc compilerbook
```

# 利用方法について
仮想環境にログインしたら、ccを利用し、9cc.c（今回書いているプログラム本体）をコンパイルし、実行ファイルを9ccという名前で作成する。    
ちなみに`-o`オプションは実行ファイル名に任意の名前をつけることができるオプションで、指定しないと`a.out`になる。  
実行ファイルに引数として数式を渡し、作成されるアセンブラを`tmp.s`と命名する。  

```zsh
$ cc -o 9cc 9cc.c
$ ./9cc 123 > tmp.s
```

```zsh
$ cc -o tmp tmp.s
$ ./tmp
$ echo $?
123
```
shellでは直前の終了コードが$?に格納されていて、こいつを出力することで結果を確認することができる。数式を記述したい場合は以下のようにすれば良い
```
$ ./9cc " 5 + 4 - 2" > tmp.s
```

# 理論

# 抽象構文木
構文解析の目標は抽象構文木を作成することにある。では、抽象構文木とはなんだろうか。  
まず構文器について説明する。

## 構文木

特に、優先度に関係のない、グループ化のためのカッコなどの冗長な表現を取り除いたものを抽象構文木と呼ぶ。コンパイラは、構文解析を行いプログラムをトークン化、抽象構文木に変換し、構文木をアセンブラに変換する。

## BNF（EBNF）
苦手なバッカスナウア記法についてきちんと理解してまとめる。
具象構文木を作るのに使える。ただしこの文法に従って生成される構文木は計算の優先順位などが定義されているわけではないので、ここからひと工夫してあげる必要がある。

## 優先順位を文法に埋め込む
#### １つ目の改良：乗除を優先させる
```
expr = mul ("+" mul | "-" mul)*
mul  = num ("*" num | "/" num)*
```
#### ２つ目の改良：カッコによる優先順位
```
expr    = mul ("+" mul | "-" mul)*
mul     = primary ("*" primary | "/" primary)*
primary = num | "(" expr ")"
```
もう一段節を生やしてあげるとなんとも綺麗に優先順位が表現できる。

## 再帰下降構文解析
上で述べたことをそのままコードにするとなんと不思議にもうまく四則演算に対応する構文木を生成することができる。

## スタックマシン
構文木をアセンブラに変換するのを実現するために使うのがスタックマシンの概念。#1にまとめた気がする

# C言語の構文について
知らない関数などを調べてまとめる。

## atoiとstrtol：
どちらも文字を数値に変換する処理を行う。
### strtol
#### 定義

```
long strtol(
    const char * restrict nptr,
    char ** restrict endptr
    int base
);
```

```c
  #include <stdlib.h>
	long int strtol(const char *nptr, char **endptr, int base);
```
#### 働き
nptr: 変換する文字列 (認識可能な形式は以下の通りです ※1)
16 進数以外に変換する場合: [符号(+ or -)] 空でない数字の列
16 進数に変換する場合: [符号 (+ or -)] [0x or 0X] 空でない 16 進数の列
endptr: 変換出来ない文字列を格納 ※2
base: 基数 ※3
