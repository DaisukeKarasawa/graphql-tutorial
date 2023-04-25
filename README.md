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

```Gemfile
# gem 'graphql', '1.11.6'
gem 'graphql', '~> 1.12.0'
```

## GraphQL on Rails

### クエリの作成(スキーマ)

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

```/app/graphql/types/link_type.rb
module Types
    class LinkType < Types::BaseObject
        field :id, ID, null: false
        field :url, String, null: false
        field :description, String, null: false
    end
end
```

ここの field は、Link タイプのフィールドを表している。

```/app/graphql/types/link_type.rb
# id, url, description フィールドを持つ
field :id, ID, null: false
field :url, String, null: false
field :description, String, null: false
```

### リゾルバの定義

'[app/graphql//types/query_type.rb](https://github.com/DaisukeKarasawa/graphql-tutorial/blob/main/app/graphql/types/query_type.rb)' にクエリに関連するリゾルバの定義をする。

まずはクエリのフィールドを定義する。今回の場合、'all_links' というフィールドを定義し、その戻り値として LinkType の配列を受け取ることを定義する。
加えて、'null: false' で戻り値が空になることはないと定義できる。

```/app/graphql/types/query_type.rb
field :all_links, [LinkType], null: false
```

フィールドに定義した 'all_links' を定義する。この関数では、データベースに保存されている全ての Link モデルのオブジェクトを取得する。

```/app/graphql/types/query_type.rb
def all_links
  Link.all
end
```

### エントリーポイントのスキーマの定義（自動定義）

'GraphqlTutorialSchema'(このアプリケーションのスキーマ)が 'GraphQL::Schema' を継承し、'Types::MutationType' と 'Types::QueryType' を
それぞれクエリとミューテーションとして定義する。

この定義によって、このアプリケーション上で '[MutationType](https://github.com/DaisukeKarasawa/graphql-tutorial/blob/main/app/graphql/types/mutation_type.rb)' と '[QueryType](https://github.com/DaisukeKarasawa/graphql-tutorial/blob/main/app/graphql/types/query_type.rb)' の２つのオブジェクトタイプを持ち、それぞれがスキーマのミューテーションとクエリに紐づけられるので、
それぞれの操作が実行できるようになる。

**結論：このスキーマを定義することで、GraphQL を使用してアプリケーションのデータを取得・更新・削除ができる。**

```/app/graphql/graphql_tutorial_schema.rb
class GraphqlTutorialSchema < GraphQL::Schema
    mutation Types::MutationType
    query Types::QueryType
end
```

### ミューテーションの作成

app/graphql/mutations/create\_ タイプ名.rb を作成し、新規作成のミューテーションを定義する。

```/app/graphql/mutations/create_link.rb
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

この作成したミューテーションを '/app/graphql/types/mutation_type.rb' 内の 'MutationType' のフィールドに定義する。

この場合、'[Mutations::CreateLink](https://github.com/DaisukeKarasawa/graphql-tutorial/blob/main/app/graphql/mutations/create_link.rb)' は、フィールドに定義された 'create_link' が呼び出された時に実行されるリゾルバとなる。

'mutation' というオプションを使用して、'create_link' フィールドが持つリゾルバがどのミューテーションに対応するかを GraphQL に伝えている。

```/app/graphql/types/mutation_type.rb
module Types
  class MutationType < BaseObject
    field :create_link, mutation: Mutations::CreateLink
  end
end
```

### ミューテーションの共通機能の定義

全てのミューテーションの親クラスである 'BaseMutation' クラスに全てのミューテーション共通の機能を定義する。
全てのミューテーションで共通する設定を一箇所にまとめることで、コードの重複を避け、保守性が上がる。

この場合、全てのミューテーションは null を返すことができず、必ず値を返すようになる設定を定義している。

```/app/graphql/mutations/base_mutation.rb
module Mutations
  class BaseMutation < GraphQL::Schema::Mutation
    null false
  end
end
```

### ミューテーションのテスト

定義したミューテーションのリゾルバに対して、単体テストを行うことも可能。([Resolvers::CreateLink のテスト](https://github.com/DaisukeKarasawa/graphql-tutorial/blob/main/test/models/mutations/create_link_test.rb))

始めに、'Mutations::CreateLink'のインスタンスを作成し、与えられた引数をそのクラスに渡す。その後、resolve メソッドを呼び出して、GraphQL のミューテーションを実行し、リンクを作成する。

```/test/models/mutations/create_link_test.rb
def perform(user: nil, **args)
    Mutations::CreateLink.new(object: nil, field: nil, context: {}).resolve(**args)
end
```

そこから、url, description を仮指定したものを perform メソッドの引数として渡す。その後、テストケースとして、「データベースに保存されたか」「description, url が引数と同じものか」を確認する。

```/test/models/mutations/create_link_test.rb
assert link.persisted?
assert_equal link.description, 'description'
assert_equal link.url, 'http://example.com'
```

テストの実行

```
rails test
```
