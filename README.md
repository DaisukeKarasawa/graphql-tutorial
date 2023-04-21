## GraphQL の公式ドキュメントに基づいた Rails によるプロジェクト

- プロジェクト

公式ドキュメントの基づき、次の機能を備えた[Hackernews](https://news.ycombinator.com)クローンのバックエンドの実装

・リンクのリストを表示する

・認証システム

・ユーザーは新しいリンクを作成可能

・ユーザーはリンクに投票可能

- Ruby のバージョン

  3.2.3

- Rails のバージョン

  7.0.4.3

- データベース

sqlite3

### 公式ドキュメントの訂正点

**-GraphQL のバージョン-**

バージョンが古いからか、**rails g graphql:install** 時にエラーが発生するので、バージョンを上げた。

```
# gem 'graphql', '1.11.6'
gem 'graphql', '~> 1.12.0'
```

### Rails での GraphQL の使い方

**-クエリの作成-**

クエリの定義をターミナル上でできる。

```
rails g graphql:object LinkType id:ID! url:String! description:String!

# 通常のクエリの定義
type Link {
    id: ID!
    url: String!
    description: String!
}
```

これにより、app/graphql/types/タイプ名\_type.rb が作成される。

```
module Types
    class LinkType < Types::BaseObject
        field :id, ID, null: false
        field :url, String, null: false
        field :description, String, null: false
    end
end
```
