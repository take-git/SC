# SC

## Description
本プログラムはRui Ueyama氏による「低レイヤを知りたい人のためのCコンパイラ作成入門」に習って作成しているC言語によるCコンパイラプロジェクト。

## 環境構築
rui ueyama氏のブログよりdockerfileを拝借する
```
$ docker build -t compilerbook https://www.sigbus.info/compilerbook/Dockerfile
```
個人的には対話的にシェルを走らせたいので、以下のコマンドで実行環境を走らせる
```
$ docker run -it -v $PWD/:/9cc compilerbook
```
