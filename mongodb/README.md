# ドキュメントデータベース

MongoDB を使ってドキュメントデータベースの実習を行います。  
MySQL や DynamoDB と比較しながら操作してみましょう。  

## データモデル

ドキュメントデータベースのデータはドキュメント形式（例は JSON）で格納されます。これが最も分かりやすい特徴です。  
リレーショナルデータベースと違い、テーブル正規化は必要ありません。  

```json
  { item_id: 1, item_name: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A", tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
  { item_id: 2, item_name: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A", tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
  { item_id: 3, item_name: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D", tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
  { item_id: 4, item_name: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D", tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
  { item_id: 5, item_name: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A", tags: ["blue"], dim_cm: [ 10, 15.25 ] }
```

### Primary Key

MongoDB では `_id` が Primary Key として使われます。  
インサート時に `_id` を省略すると MongoDB が自動で生成します。  
演習中、何度か検索を行いますが、`_id` が表示されることを確認してみてください。  

```bash
   _id: ObjectId('6614f519f17a620598e10a96'),
```

## コンテナ起動

```bash
# mongodb ディレクトリへ移動
$ cd $WORKDIR/mongodb

# コンテナ起動
$ docker-compose up -d

# コンテナ起動確認
$ docker-compose ps
NAME      IMAGE                                     COMMAND                   SERVICE   CREATED         STATUS         PORTS
mongodb   mongodb/mongodb-community-server:latest   "python3 /usr/local/…"   mongodb   7 seconds ago   Up 6 seconds   27017/tcp   # STATUS が Up であること、PORTS が 27017 であることを確認
```

## MonoDB 実習

```bash
# コンテナ内部へ
$ docker-compose exec mongodb bash
mongodb@28f2392f23f1:/$     # プロンプトが変わっていることを確認

# DB 接続
$ mongosh -u root -p root
test>     # プロンプトが変わっていることを確認

# 接続確認
> show dbs;
admin   100.00 KiB
config   72.00 KiB
local    72.00 KiB

# データベース作成
> use myNewDatabase
switched to db myNewDatabase

# データ投入
> db.inventory.insertMany([
  { item_id: 1, item_name: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A", tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
  { item_id: 2, item_name: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A", tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
  { item_id: 3, item_name: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D", tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
  { item_id: 4, item_name: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D", tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
  { item_id: 5, item_name: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A", tags: ["blue"], dim_cm: [ 10, 15.25 ] }
])
```

ここからは様々な検索方法を紹介します。検索パターンによって結果がどのように変わるか確認しながら進めてください。  

```bash
# 全検索
> db.inventory.find()

# イコール検索、status が 'D' のドキュメントを検索
> db.inventory.find( { status: "D" } )

# AND 検索、status が 'A' かつ qty が 30 より小さいドキュメントを検索
> db.inventory.find( { status: "A", qty: { $lt: 30 } } )

# OR 検索、status が 'A' または qty が 30 より小さいドキュメントを検索
> db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )

# AND 検索と OR 検索の組み合わせ、status が 'A' かつ qty が 30 より小い、または item_name が 'p' で始まるドキュメントを検索
> db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item_name: /^p/ } ]
} )

# ネストフィールドの検索
> db.inventory.find( { "size.uom": "in" } )

# アレイの検索
> db.inventory.find( { tags: ["red", "blank"] } )

# アレイ要素ごとの検索
> db.inventory.find( { tags: "red" } )

# アレイ要素の検索条件を複合条件で検索
> db.inventory.find( { dim_cm: { $gt: 10, $lt: 15 } } )
```

ここから更新処理です。  

```bash
# 更新前のデータ確認
> db.inventory.find( { qty: { $lt: 50 } } )

# qty が 50 より小さいドキュメントを対象に status を 'P' に更新
> db.inventory.updateMany(
  { "qty": { $lt: 50 } },
  { $set: { status: "P" } }
)

> db.inventory.find( { qty: { $lt: 50 } } )
[
  {
    _id: ObjectId('6614f519f17a620598e10a92'),
    item_id: 1,
    item_name: 'journal',
    qty: 25,
    size: { h: 14, w: 21, uom: 'cm' },
    status: 'P',                            # status が 'P' になっていることを確認
    tags: [ 'blank', 'red' ],
    dim_cm: [ 14, 21 ]
  },
  {
    _id: ObjectId('6614f519f17a620598e10a96'),
    item_id: 5,
    item_name: 'postcard',
    qty: 45,
    size: { h: 10, w: 15.25, uom: 'cm' },
    status: 'P',                            # status が 'P' になっていることを確認
    tags: [ 'blue' ],
    dim_cm: [ 10, 15.25 ]
  }
]
```

最後に削除して終わりです。  

```bash
# Delete Operations
> db.inventory.deleteMany({})
{ acknowledged: true, deletedCount: 5 }

> db.inventory.find()
# 結果がないことを確認
```

## コンテナ停止

```bash
# データベースからログアウト
> exit

# コンテナからログアウト
$ exit

# コンテナ停止
$ docker-compose down
[+] Running 2/2
✔ Container mongodb        Removed
✔ Network mongodb_default  Removed   

$ docker-compose ps
NAME      IMAGE     COMMAND   SERVICE   CREATED   STATUS    PORTS
# 結果がないことを確認
```

## 参考

[Start with Guides](https://www.mongodb.com/docs/guides/)  
[CRUD Operations](https://www.mongodb.com/docs/manual/crud/)  

