よく使うシェルコマンド置き場
===

* コマンドラインオプションいちいち覚えてないのでメモっとく用ページ
* このページの内容を信用して起きた事故など一切感知しないので、自己責任で見てください

## ファイル・ディレクトリの存在確認
### 複数のサーバーにディレクトリが存在するかチェックする

実行結果

```sh
$ for i in `awk /gomi-/'{print $2}' /etc/hosts`; do echo "$i:"; ssh $i '[ -d /etc/elasticsearch ]; echo $?'; done;
gomi-web01
1
gomi-back01
0
```

* `-w`: writable
* `-x`: executable
* `-e`: path
* `-d`: directory
* `-f`: file
* `-s`: empty file

## ファイル・ディレクトリのサイズ確認

権限周りでうまく集計できないケースがあるので sudo をつけたり、適切なユーザーになってから実行するとよい

```sh
$ sudo du -sh /home/common/*
4.0K	/home/common/app
112K	/home/common/logs
```

* `-s`: --summarize
* `-h`: --human-readable

## 鍵生成

```sh
ssh-keygen -t rsa -b 4096
```
