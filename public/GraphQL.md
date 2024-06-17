---
title: GraphQLの公式ドキュメントを要約してみた
tags:
  - 'GraphQL'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# GraphQLとは

2012年にFacebook社によって開発され、2015年にオープンソース化されたデータクエリ言語およびサーバサイドランタイムのこと。

特徴としてシングルエンドポイントであり、クライアント側は必要なデータの形式をクエリで指定する。


# Queriesについて

## クエリの基本形式

Request例
```
{
  alias1: field1(argument: argument_value) {
    name
  }
  alias2: field2(argument: argument_value) {
    name
  }
}
```

Response例
```
{
  "data": {
    "alias1": {
      "name": "Luke Skywalker"
    },
    "alias2": {
      "name": "R2-D2"
    }
  }
}
```

## Fragmentsの使い方

クエリ定義をフラグメントとして、他のクエリの一部に使い回すことができる。

```
{
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  appearsIn
  friends {
    name
  }
}


{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        },
        {
          "name": "C-3PO"
        },
        {
          "name": "R2-D2"
        }
      ]
    },
    "rightComparison": {
      "name": "R2-D2",
      "appearsIn": [
        "NEWHOPE",
        "EMPIRE",
        "JEDI"
      ],
      "friends": [
        {
          "name": "Luke Skywalker"
        },
        {
          "name": "Han Solo"
        },
        {
          "name": "Leia Organa"
        }
      ]
    }
  }
}
```


### Fragments内でvariablesを使う

```
query HeroComparison($first: Int = 3) {
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
  rightComparison: hero(episode: JEDI) {
    ...comparisonFields
  }
}

fragment comparisonFields on Character {
  name
  friendsConnection(first: $first) {
    totalCount
    edges {
      node {
        name
      }
    }
  }
}


{
  "data": {
    "leftComparison": {
      "name": "Luke Skywalker",
      "friendsConnection": {
        "totalCount": 4,
        "edges": [
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          },
          {
            "node": {
              "name": "C-3PO"
            }
          }
        ]
      }
    },
    "rightComparison": {
      "name": "R2-D2",
      "friendsConnection": {
        "totalCount": 3,
        "edges": [
          {
            "node": {
              "name": "Luke Skywalker"
            }
          },
          {
            "node": {
              "name": "Han Solo"
            }
          },
          {
            "node": {
              "name": "Leia Organa"
            }
          }
        ]
      }
    }
  }
}
```

```
query クエリ名($変数名: 変数の型 = デフォルト値) {
  エイリアス名: フィールド名(引数名: $変数名) {
    フィールド名
    フィールド名 {
      フィールド名
    }
  }
}
```

変数は
変数名が＄から始まる。
スカラ値、enum, input　objectのいずれかの型でないといけない。

## Directives
要約ビューと詳細ビューをもつUIコンポーネントのように動的に構造や形を変化させる方法

```
query Hero($episode: Episode, $withFriends: Boolean!) {
  hero(episode: $episode) {
    name
    friends @include(if: $withFriends) {
      name
    }
  }
}
```
@include(if: $condition)
によりconditionがtrueの時のみフィールドを含めることができる。


フィールド名の後にディレクティブをつけてフィールド名を条件により含めるか決めることができる。
以下のディレクティブがある。
- @inclued(if: Boolean)
ifの値がtrureの時に含める
- @skip(if: Boolean)
ifの値がtrueの時に含めない

## mutationによりサーバ側のデータを修正できる



GraphQLのメリット
Presentation-Domain SeparationSeparation