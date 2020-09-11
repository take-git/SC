# Simple Clang Compiler

# Description
本プログラムはRui Ueyama氏による「[低レイヤを知りたい人のためのCコンパイラ作成入門](https://www.sigbus.info/compilerbook)」に習って作成しているC言語によるCコンパイラプロジェクト。

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
shellでは直前の終了コードが$?に格納されていて、こいつを出力することで結果を確認することができる。

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
