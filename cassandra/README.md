# ワイドカラム

Cassandra を使ってワイドカラムを体験してみましょう。  


## コンテナ起動

```bash
# mongodb ディレクトリへ移動
$ cd $WORKDIR/cassandra

# コンテナ起動
$ docker-compose up -d

# コンテナ起動確認
$ docker-compose ps
NAME                    IMAGE              COMMAND                   SERVICE     CREATED         STATUS         PORTS
cassandra-cassandra-1   cassandra:latest   "docker-entrypoint.s…"   cassandra   3 minutes ago   Up 3 minutes   7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9042->9042/tcp    # STATUS が Up であること、PORTS が 0.0.0.0:9042->9042 であることを確認
```

## Cassandra 実習

```bash
# コンテナ内部へ、コマンドが成功するまで少々時間がかかる場合があります、失敗した場合は数秒おいて再度実行してください
$ docker-compose exec cassandra cqlsh
cqlsh>   # プロンプトが変わっていることを確認, CQL = Cassandra Query Language

# keyspace 作成
cqlsh> CREATE KEYSPACE IF NOT EXISTS test WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : '1' };

# 作成された keyspace 確認
cqlsh> DESCRIBE KEYSPACES;

system       system_distributed  system_traces  system_virtual_schema
system_auth  system_schema       system_views   test                        # test が作成されていることを確認


# テーブル作成
# id が PK(Primary Key) で division が CK(Cluster Column) です
cqlsh> CREATE TABLE IF NOT EXISTS test.employees_tbl (
   id text,
   name text,
   region text,
   division text,
   project text,
   role text,
   pay_scale int,
   vacation_hrs float,
   manager_id text,
   PRIMARY KEY (id,division))
   WITH CLUSTERING ORDER BY (division ASC) ;

# データ挿入
cqlsh> INSERT INTO test.employees_tbl 
(id, name, project, region, division, role, pay_scale, vacation_hrs, manager_id)
VALUES ('012-34-5678','Russ','NightFlight','US','Engineering','IC',3,12.5, '234-56-7890') ;

# CSVファイルからロードすることも可能です
cqlsh> COPY test.employees_tbl (id, name, project, region, division, role, pay_scale, vacation_hrs, manager_id) 
FROM '/tmp/employees.csv' WITH HEADER = TRUE;

7 rows imported from 1 files in 0.379 seconds (0 skipped).


# データ確認
cqlsh> SELECT * FROM test.employees_tbl;

 id          | division    | manager_id  | name    | pay_scale | project     | region | role    | vacation_hrs
-------------+-------------+-------------+---------+-----------+-------------+--------+---------+--------------
 123-45-6789 | Engineering | 234-56-7890 |     Bob |         1 | NightFlight |     US |  Intern |            0
 678-90-1234 |   Marketing | 789-01-2345 |    Alan |         3 |       Storm |     US | Manager |         18.4
 234-56-7890 | Engineering | 789-01-2345 |     Bob |         6 | NightFlight |     US | Manager |           72
 012-34-5678 | Engineering | 234-56-7890 |    Russ |         3 | NightFlight |     US |      IC |         12.5
 789-01-2345 |   Executive |        None | Roberta |        15 |         All |     US |     CEO |          184
 456-78-9012 | Engineering | 234-56-7890 |    Beth |         7 | NightFlight |     US |      IC |        100.5
 567-89-0123 |   Marketing | 678-90-1234 |   Ahmed |         4 | NightFlight |     US |      IC |           88
 345-67-8901 | Engineering | 234-56-7890 |   Sarah |         4 |       Storm |     US |      IC |          108


# フィルター、SQL にそっくりです
cqlsh> SELECT * FROM test.employees_tbl WHERE id = '678-90-1234';

 id          | division  | manager_id  | name | pay_scale | project | region | role    | vacation_hrs
-------------+-----------+-------------+------+-----------+---------+--------+---------+--------------
 678-90-1234 | Marketing | 789-01-2345 | Alan |         3 |   Storm |     US | Manager |         18.4


# ただ、PK 以外のカラムでの検索は ALLOW FILTERING が必要です 
# ALLOW FILTERING は全てのデータをスキャンするため、パフォーマンスが低下します
cqlsh> SELECT * FROM test.employees_tbl WHERE division = 'Marketing' ALLOW FILTERING;

 id          | division  | manager_id  | name  | pay_scale | project     | region | role    | vacation_hrs
-------------+-----------+-------------+-------+-----------+-------------+--------+---------+--------------
 678-90-1234 | Marketing | 789-01-2345 |  Alan |         3 |       Storm |     US | Manager |         18.4
 567-89-0123 | Marketing | 678-90-1234 | Ahmed |         4 | NightFlight |     US |      IC |           88


# 比較演算
cqlsh> SELECT * FROM test.employees_tbl WHERE pay_scale > 4 ALLOW FILTERING;

 id          | division    | manager_id  | name    | pay_scale | project     | region | role    | vacation_hrs
-------------+-------------+-------------+---------+-----------+-------------+--------+---------+--------------
 234-56-7890 | Engineering | 789-01-2345 |     Bob |         6 | NightFlight |     US | Manager |           72
 789-01-2345 |   Executive |        None | Roberta |        15 |         All |     US |     CEO |          184
 456-78-9012 | Engineering | 234-56-7890 |    Beth |         7 | NightFlight |     US |      IC |        100.5

```

更新してみましょう。こちらも SQL 文に似ています。  

```bash
# id が '678-90-1234' の pay_scale を 5 に更新
# CREATE TABLE をした際に Primary Key が id と division になっているため、id と division の両方を指定する必要があります
cqlsh> UPDATE test.employees_tbl SET pay_scale = 5 WHERE id = '678-90-1234' AND division = 'Marketing';

cqlsh> SELECT * FROM test.employees_tbl WHERE id = '678-90-1234';

 id          | division  | manager_id  | name | pay_scale | project | region | role    | vacation_hrs
-------------+-----------+-------------+------+-----------+---------+--------+---------+--------------
 678-90-1234 | Marketing | 789-01-2345 | Alan |         5 |   Storm |     US | Manager |         18.4
```

削除して完了です。  

```bash
DROP KEYSPACE IF EXISTS test;
```

## コンテナ停止

```bash
# コンテナからログアウト
cqlsh> exit

# コンテナ停止
$ docker-compose down
[+] Running 2/1
 ✔ Container cassandra-cassandra-1  Removed
 ✔ Network cassandra_default        Removed

$ docker-compose ps
NAME      IMAGE     COMMAND   SERVICE   CREATED   STATUS    PORTS
# 結果がないことを確認
```


## 参考

[Get Started with Apache Cassandra](https://cassandra.apache.org/_/quickstart.html)  
[Welcome to Apache Cassandra’s documentation!](https://cassandra.apache.org/doc/stable/index.html)  
[Amazon Keyspaces (Apache Cassandra 向け) の使用開始](https://docs.aws.amazon.com/ja_jp/keyspaces/latest/devguide/getting-started.html)  
