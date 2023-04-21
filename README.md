## GraphQL の公式ドキュメントに基づいた Rails によるプロジェクト

- プロジェクト

公式ドキュメントの基づき、次の機能を備えた[Hackernews](https://news.ycombinator.com)クローンのバックエンドの実装

・リンクのリストを表示する

・認証システム

・ユーザーは新しいリンクを作成可能

・ユーザーはリンクに投票可能

- 今回の目的

完璧に分からなくても良いから、GraphQL が Rails 上でどのように使われているか、どのように動くのかを掴む。

- Ruby のバージョン

  3.2.3

- Rails のバージョン

  7.0.4.3

- データベース

  sqlite3

## 公式ドキュメントの訂正点

### GraphQL のバージョン

バージョンが古いからか、**rails g graphql:install** 時にエラーが発生するので、バージョンを上げた。

```
# gem 'graphql', '1.11.6'
gem 'graphql', '~> 1.12.0'
```

## Rails での GraphQL の使い方

### クエリの作成

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

### ミューテーションの作成

ミューテーションはスキーマで自動的に公開される

```/app/graphql/graphql_tutorial_schema.rb
class GraphqlTutorialSchema < GraphQL::Schema
    mutation Types::MutationType
    query Types::QueryType
end
```

app/graphql/mutations/create\_ タイプ名.rb を作成し、新規作成のミューテーションを定義する。

```
module Mutations
    class CreateLink < BaseMutation

        argument :description, String, required: true
        argument :url, String, required: true

        type Types::LinkType

        def resolve(description: nil, url: nil)
            Link.create!(
                description: description,
                url: url,
            )
        end
    end
end
```

### ミューテーションのテスト

定義したミューテーションのリゾルバに対して、単体テストを行うことも可能。([Resolvers::CreateLink のテスト](https://github.com/DaisukeKarasawa/graphql-tutorial/blob/main/test/models/mutations/create_link_test.rb))

始めに、'Mutations::CreateLink'のインスタンスを作成し、与えられた引数をそのクラスに渡す。その後、resolve メソッドを呼び出して、GraphQL のミューテーションを実行し、リンクを作成する。

```
def perform(user: nil, **args)
    Mutations::CreateLink.new(object: nil, field: nil, context: {}).resolve(**args)
end
```

そこから、url, description を仮指定したものを perform メソッドの引数として渡す。その後、テストケースとして、「データベースに保存されたか」「description, url が引数と同じものか」を確認する。

```
assert link.persisted?
assert_equal link.description, 'description'
assert_equal link.url, 'http://example.com'
```

テストの実行

```
rails test
```
