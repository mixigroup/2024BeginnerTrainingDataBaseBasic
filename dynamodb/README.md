# DynameDB ハンズオン

DynamoDB を使ってキーバリューストアの実習を行います。  

## 実習のポイント

### テーブル

良い例ではないですが、以下のようなテーブルを作成します。  
`item_name` がプライマリキーです。  

| item_name | qty | mystatus | mysize                         | tags                      | dim_cm        |
| --------- | --- | -------- | ------------------------------ | ------------------------- | ------------- |
| journal   | 25  | A        | { h: 14, w: 21, uom: "cm" }    | ["blank", "red"]          | [ 14, 21 ]    |
| notebook  | 50  | A        | { h: 8.5, w: 11, uom: "in" }   | ["red", "blank"]          | [ 14, 21 ]    |
| paper     | 100 | D        | { h: 8.5, w: 11, uom: "in" }   | ["red", "blank", "plain"] | [ 14, 21 ]    |
| planner   | 75  | D        | { h: 22.85, w: 30, uom: "cm" } | ["blank", "red"]          | [ 22.85, 30 ] |
| postcard  | 45  | A        | { h: 10, w: 15.25, uom: "cm" } | ["blue"]                  | [ 10, 15.25 ] |

DynamoDB は予約された単語が存在します。それらを属性名にしないようにしましょう。  
[DynamoDB の予約語](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/ReservedWords.html)

## コンテナ起動

```bash
# mongodb ディレクトリへ移動
$ cd $WORKDIR/dynamodb

# コンテナ起動
$ docker-compose up -d

# コンテナ起動確認
$ docker-compose ps
NAME                  IMAGE                          COMMAND                   SERVICE    CREATED         STATUS         PORTS
dynamodb-cli-1        amazon/aws-cli                 "bash -c /bin/bash"       cli        6 seconds ago   Up 6 seconds                # STATUS が Up であること
dynamodb-dynamodb-1   amazon/dynamodb-local:latest   "java -jar DynamoDBL…"   dynamodb   7 seconds ago   Up 6 seconds   8000/tcp      # STATUS が Up であること、PORTS が 27017 であることを確認
```

## DynamoDB の実習

```bash
# コンテナ内部へ
$ docker-compose exec cli bash
bash-4.2#     # プロンプトが変わっていることを確認

# テーブル作成
# プライマリキーのみを指定
bash-4.2# aws dynamodb create-table \
    --table-name MyTable \
    --attribute-definitions AttributeName=item_name,AttributeType=S \
    --key-schema AttributeName=item_name,KeyType=HASH \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1

{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "item_name",
                "AttributeType": "S"
            }
        ],
        "TableName": "MyTable",
        "KeySchema": [
            {
                "AttributeName": "item_name",
                "KeyType": "HASH"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2024-04-10T08:02:15.039000+00:00",
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": "1970-01-01T00:00:00+00:00",
            "LastDecreaseDateTime": "1970-01-01T00:00:00+00:00",
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 1,
            "WriteCapacityUnits": 1
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:ddblocal:000000000000:table/MyTable",
        "DeletionProtectionEnabled": false
    }
}

# データ投入

bash-4.2# aws dynamodb put-item \
    --table-name MyTable \
    --item '{ 
        "item_name": {"S": "journal"}, 
        "qty": {"N": "25"}, 
        "mystatus": {"S": "A"},
        "mysize": {"M": {"h": {"N": "14"}, "w": {"N": "21"}, "uom": {"S": "cm"}}},
        "tags": {"L": [{"S": "blank"}, {"S": "red"}]},
        "dim_cm": {"L": [{"N": "14"}, {"N": "21"}]}
    }'

bash-4.2# aws dynamodb put-item \
    --table-name MyTable \
    --item '{ 
        "item_name": {"S": "notebook"}, 
        "qty": {"N": "50"}, 
        "mystatus": {"S": "A"},
        "mysize": {"M": {"h": {"N": "8.5"}, "w": {"N": "11"}, "uom": {"S": "in"}}},
        "tags": {"L": [{"S": "red"}, {"S": "blank"}]},
        "dim_cm": {"L": [{"N": "14"}, {"N": "21"}]}
    }'

bash-4.2# aws dynamodb put-item \
    --table-name MyTable \
    --item '{ 
        "item_name": {"S": "paper"}, 
        "qty": {"N": "100"}, 
        "mystatus": {"S": "D"},
        "mysize": {"M": {"h": {"N": "8.5"}, "w": {"N": "11"}, "uom": {"S": "in"}}},
        "tags": {"L": [{"S": "red"}, {"S": "blank"}, {"S": "plain"}]},
        "dim_cm": {"L": [{"N": "14"}, {"N": "21"}]}
    }'

bash-4.2# aws dynamodb put-item \
    --table-name MyTable \
    --item '{ 
        "item_name": {"S": "planner"}, 
        "qty": {"N": "75"}, 
        "mystatus": {"S": "D"},
        "mysize": {"M": {"h": {"N": "22.85"}, "w": {"N": "30"}, "uom": {"S": "cm"}}},
        "tags": {"L": [{"S": "blank"}, {"S": "red"}]},
        "dim_cm": {"L": [{"N": "22.85"}, {"N": "30"}]}
    }'

bash-4.2# aws dynamodb put-item \
    --table-name MyTable \
    --item '{ 
        "item_name": {"S": "postcard"}, 
        "qty": {"N": "45"}, 
        "mystatus": {"S": "A"},
        "mysize": {"M": {"h": {"N": "10"}, "w": {"N": "15.25"}, "uom": {"S": "cm"}}},
        "tags": {"L": [{"S": "blue"}]},
        "dim_cm": {"L": [{"N": "10"}, {"N": "15.25"}]}
    }'

# データ取得
bash-4.2# aws dynamodb scan --table-name MyTable

```

検索方法を紹介します。  

```bash
# イコール検索、item_name が 'postcard' のデータを取得
bash-4.2# aws dynamodb get-item \
    --table-name MyTable \
    --key '{"item_name":{"S":"postcard"}}'

# イコール検索、status が 'D' のレコードを検索
# これは失敗します。DynamoDB の GetItem オペレーションにはプライマリキーが必要です。
bash-4.2# aws dynamodb get-item \
    --table-name MyTable \
    --key '{"mystaus":{"S":"D"}}'

# イコール検索、status が 'D' のレコードを検索
# scan を使えば取得することができますが、スキャンはコストが高いので使わないようにしましょう。
bash-4.2# aws dynamodb scan \
    --table-name MyTable \
    --filter-expression "mystatus = :status" \
    --expression-attribute-values '{":status": {"S": "D"}}'

# Global Secondary Index を追加すると Primary Key 以外で検索可能になります。
# Status に GSI を追加するのは賛否ありそうですが、例として追加します。
bash-4.2# aws dynamodb update-table \
    --table-name MyTable \
    --attribute-definitions AttributeName=mystatus,AttributeType=S \
    --global-secondary-index-updates '[{
        "Create": {
            "IndexName": "mystatus-index",
            "KeySchema": [{"AttributeName": "mystatus", "KeyType": "HASH"}],
            "Projection": {"ProjectionType": "INCLUDE","NonKeyAttributes": ["qty"]},
            "ProvisionedThroughput": {"ReadCapacityUnits": 1, "WriteCapacityUnits": 1}
        }
    }]'

# GSI のクエリ
bash-4.2# aws dynamodb query \
    --table-name MyTable \
    --index-name mystatus-index \
    --key-condition-expression "mystatus = :status" \
    --expression-attribute-values '{":status": {"S": "D"}}'

{
    "Items": [
        {
            "mystatus": {
                "S": "D"
            },
            "item_name": {
                "S": "planner"
            },
            "qty": {
                "N": "75"
            }
        },
        {
            "mystatus": {
                "S": "D"
            },
            "item_name": {
                "S": "paper"
            },
            "qty": {
                "N": "100"
            }
        }
    ],
    "Count": 2,
    "ScannedCount": 2,
    "ConsumedCapacity": null
}
```

次に更新処理を紹介します。  

```bash
# item_name が 'postcard' の MyStatus を 'P' に更新
bash-4.2# aws dynamodb update-item \
    --table-name MyTable \
    --key '{"item_name": {"S": "postcard"}}' \
    --update-expression "SET mystatus = :status" \
    --expression-attribute-values '{":status": {"S": "P"}}'

# 確認
bash-4.2# aws dynamodb get-item \
    --table-name MyTable \
    --key '{"item_name":{"S":"postcard"}}' \
    --projection-expression "item_name,mystatus"

{
    "Item": {
        "mystatus": {
            "S": "P"
        },
        "item_name": {
            "S": "postcard"
        }
    }
}
```

最後に削除して終わりです。  

```bash
bash-4.2# aws dynamodb delete-table --table-name MyTable
```

## コンテナ停止

```bash
# コンテナからログアウト
bash-4.2# exit

# コンテナ停止
$ docker-compose down
[+] Running 3/3
✔ Container dynamodb-cli-1       Removed
✔ Container dynamodb-dynamodb-1  Removed
✔ Network dynamodb_default       Removed    

$ docker-compose ps
NAME      IMAGE     COMMAND   SERVICE   CREATED   STATUS    PORTS
# 結果がないことを確認
```

## 参考

[Use Amazon DynamoDB with the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-dynamodb.html)  
[Setting up DynamoDB local (downloadable version)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html)  
[DynamoDB の操作](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/WorkingWithDynamo.html)  
