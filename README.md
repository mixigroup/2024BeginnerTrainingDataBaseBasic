# 2024BeginnerTrainingDataBaseBasic
24新卒技術研修_データベース研修_DB設計基礎


## Git クローン

```bash
$ git clone git@github.com:mixigroup/2024BeginnerTrainingDataBaseBasic.git

$ cd 2024BeginnerTrainingDataBaseBasic
$ export WORKDIR=$(pwd)

# WORKDIR 変数が設定されていることを確認
$ echo $WORKDIR
/Users/hoge/hoge/2024BeginnerTrainingDataBaseBasic
```


## 4章 演習

### その１ テーブル構造の比較

リレーショナルデータベース、キーバリューストア、ドキュメント指向データベースの違いを感じてもらいます。  
ほぼ同じデータをそれぞれに入れて基本操作をしながら、3つのデータベースを比較してみましょう。    
製品は MySQL、DynamoDB Local、MongoDB です。  

1. [MySQL](./MySQL/README.md)
2. [DynamoDB Local](./DynamoDBLocal/README.md)
3. [MongoDB](./MongoDB/README.md)

### その2 ワイドカラムの操作

ワイドカラムの基本操作を学びます。  
製品は Cassandra です。  

4. [Cassandra](./Cassandra/README.md)

ワイドカラムは専門的な知識が必要なデータベースです。今回は基本的な操作のみとなっています。  
もしも配属先でワイドカラムを使うようなことがあれば、十分な学習が必要だと想像します。  

### その3 インメモリデータベースの操作

インメモリデータベースの基本操作を学びます。  
製品は Redis です。  
Redis の基本コマンドに加えて、簡単なチャットシステムを利用して Pub/Sub の仕組みを覗いてみます。  

5. [Redis](./Redis/README.md)

## 第8章 実習

SQL インジェクションの実習を行います。  
これからたくさんのコードを書くと思います。セキュリティを意識してコードを書いてもらえると嬉しいです。  

1. [Juice Shop](./juiceshop/README.md)

