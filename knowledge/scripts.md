# ファイル・ディレクトリの存在確認
## 複数のサーバーにディレクトリが存在するかチェックする

実行結果

```
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
