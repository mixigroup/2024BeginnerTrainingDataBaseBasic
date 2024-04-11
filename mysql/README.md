# リレーショナルデータベース

MySQL を使ってリレーショナルデータベースの実習を行います。  

## テーブル

リレーショナル、キーバリュー、ドキュメントの違いを感じてもらうために、すべてのデータベースで同じようなデータを使用します。  
テーブルは強引に正規化してみました。リレーショナルデータベースらしさを感じてもらえると嬉しいです。  

### item

| item_id | item_name | qty | status |
| ------- | --------- | --- | ------ |
| 1       | journal   | 25  | A      |
| 2       | notebook  | 50  | A      |
| 3       | paper     | 100 | D      |
| 4       | planner   | 75  | D      |
| 5       | postcard  | 45  | A      |

### item_size

| item_id | size_h | size_w | uom |
| ------- | ------ | ------ | --- |
| 1       | 14     | 21     | cm  |
| 2       | 8.5    | 11     | in  |
| 3       | 8.5    | 11     | in  |
| 4       | 22.85  | 30     | cm  |
| 5       | 10     | 15.25  | cm  |

### item_tag

| item_id | tag   |
| ------- | ----- |
| 1       | blank |
| 1       | red   |
| 2       | red   |
| 2       | blank |
| 3       | red   |
| 3       | blank |
| 3       | plain |
| 4       | blank |
| 4       | red   |
| 5       | blue  |

### item_dim

| item_id | dim_cm_l | dim_cm_h |
| ------- | -------- | -------- |
| 1       | 14       | 21       |
| 2       | 14       | 21       |
| 3       | 14       | 21       |
| 4       | 22.85    | 30       |
| 5       | 10       | 15.25    |

## コンテナ起動

```bash
# mongodb ディレクトリへ移動
$ cd $WORKDIR/mysql

# コンテナ起動
$ docker-compose up -d

# コンテナ起動確認
$ docker-compose ps
NAME      IMAGE          COMMAND                   SERVICE   CREATED         STATUS         PORTS
mysql     mysql:latest   "docker-entrypoint.s…"   mysql     3 seconds ago   Up 3 seconds   33060/tcp, 0.0.0.0:13306->3306/tcp       # STATUS が Up であること、PORTS が 0.0.0.0:13306->3306/tcp であることを確認
```

## MySQL 実習

```bash
# コンテナ内部へ
$ docker-compose exec mysql bash
bash-4.4#      # プロンプトが変わっていることを確認

# DB 接続
$ mysql -u root -proot
mysql>    # プロンプトが変わっていることを確認

# 接続確認
> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.02 sec)

```

MySQL インスタンスに接続できました。  
続いてデータベース、テーブルを作成し、データを投入してみましょう。  

```bash
# データベース作成
> create database testdb;

> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| testdb             |     # testdb  が追加されていることを確認

# データベース選択
> use testdb;
Database changed

# テーブル作成
> create table item (
    item_id int primary key,
    item_name varchar(255),
    qty int,
    status char(1)
);

> create table item_size (
    item_id int primary key,
    size_h decimal(5,2),
    size_w decimal(5,2),
    uom varchar(10)
);

> create table item_tag (
    item_id int,
    tag varchar(255)
);

> create table item_dim (
    item_id int,
    dim_cm_l decimal(5,2),
    dim_cm_h decimal(5,2)
);

# テーブル確認
> show tables;
+------------------+
| Tables_in_testdb |
+------------------+
| item             |
| item_dim         |
| item_size        |
| item_tag         |
+------------------+

# データ投入
> insert into item (item_id, item_name, qty, status) values 
(1, 'journal', 25, 'A'),
(2, 'notebook', 50, 'A'),
(3, 'paper', 100, 'D'),
(4, 'planner', 75, 'D'),
(5, 'postcard', 45, 'A');

> insert into item_size (item_id, size_h, size_w, uom) values 
(1, 14, 21, 'cm'),
(2, 8.5, 11, 'in'),
(3, 8.5, 11, 'in'),
(4, 22.85, 30, 'cm'),
(5, 10, 15.25, 'cm');

> insert into item_tag (item_id, tag) values 
(1, 'blank'),
(1, 'red'),
(2, 'red'),
(2, 'blank'),
(3, 'red'),
(3, 'blank'),
(3, 'plain'),
(4, 'blank'),
(4, 'red'),
(5, 'blue');

> insert into item_dim (item_id, dim_cm_l, dim_cm_h) values 
(1, 14, 21),
(2, 14, 21),
(3, 14, 21),
(4, 22.85, 30),
(5, 10, 15.25);

> commit;
```

最後の `commit;` はトランザクションを確定するためのコマンドです。忘れずに実行してください。  
次は検索です。  

```bash
# item テーブルの全検検索
> select * from item;
+---------+-----------+------+--------+
| item_id | item_name | qty  | status |
+---------+-----------+------+--------+
|       1 | journal   |   25 | A      |
|       2 | notebook  |   50 | A      |
|       3 | paper     |  100 | D      |
|       4 | planner   |   75 | D      |
|       5 | postcard  |   45 | A      |
+---------+-----------+------+--------+

# イコール検索、status が 'D' のレコードを検索
> select * from item 
  where status = 'D';

# AND 検索、status が 'A' かつ qty が 30 より小さいレコードを検索
> select * from item 
  where status = 'A' and qty < 30;

# OR 検索、status が 'A' または qty が 30 より小さいドキュメントを検索
> select * from item 
  where status = 'A' or qty < 30;

# AND 検索と OR 検索の組み合わせ、status が 'A' かつ qty が 30 より小い、または item_name が 'p' で始まるドキュメントを検索
> select * from item 
  where status = 'A' and (qty < 30 or item_name like 'p%');

# JOIN、item_name = 'journal' の tag を検索
> select i.item_name, t.tag 
  from item i join item_tag t on i.item_id = t.item_id 
  where i.item_name = 'journal';

# ミニクイズ
# item_name, size_h, size_w, uom を一覧で表示してみましょう
```

更新もしてみましょう。  

```bash
# 更新前レコードの確認
> select * from item 
  where qty < 50;

# qty が 50 より小さいレコードを対象に status を 'P' に更新
> update item 
  set status = 'P' 
  where qty < 50;

> select * from item 
  where qty < 50;
+---------+-----------+------+--------+
| item_id | item_name | qty  | status |
+---------+-----------+------+--------+
|       1 | journal   |   25 | P      |   # status が 'P' に更新されていることを確認
|       5 | postcard  |   45 | P      |   # status が 'P' に更新されていることを確認
+---------+-----------+------+--------+

> commit;
```

最後に削除して終わりです。  

```bash
# item テーブルの全レコード削除
> delete from item;

# 削除の確認
> select * from item;
Empty set (0.00 sec)

# ロールバックしてみる
> rollback;

# もう一度検索
> select * from item;
+---------+-----------+------+--------+
| item_id | item_name | qty  | status |
+---------+-----------+------+--------+
|       1 | journal   |   25 | A      |     # ロールバックされていることを確認
|       2 | notebook  |   50 | A      |     # ロールバックされていることを確認
|       3 | paper     |  100 | D      |     # ロールバックされていることを確認
|       4 | planner   |   75 | D      |     # ロールバックされていることを確認
|       5 | postcard  |   45 | A      |     # ロールバックされていることを確認
+---------+-----------+------+--------+

# DDL truncate を使ってみましょう
# item_dim テーブルの truncate
> truncate table item_dim;

# 削除の確認
> select * from item_dim;
Empty set (0.00 sec)

# ロールバックしてみる
> rollback;

# もう一度検索
> select * from item_dim;
Empty set (0.00 sec)      # truncate は戻らない

# delete と truncate を続けて実行してみましょう
> delete from item_size;
> truncate table item_tag;

# ロールバックしてみる
> rollback;

# 検索
> select * from item_size;
Empty set (0.00 sec)

> select * from item_tag;
Empty set (0.00 sec)

# 今度は両テーブルとも空になってしまいました
# trucate は暗黙の commit が発生するため、rollback で戻せません
```

## コンテナ停止

```bash
# データベースからログアウト
> exit

# コンテナからログアウト
bash-4.4# exit

# コンテナ停止
$ docker-compose down
[+] Running 2/2
✔ Container mysql      Removed
✔ Network mysql_default  Removed   

$ docker-compose ps
NAME      IMAGE     COMMAND   SERVICE   CREATED   STATUS    PORTS
# 結果がないことを確認
```