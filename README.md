# C++ の歩き方 | CppMap (ソースコード)

ビルド済みの Web サイトは https://cppmap.github.io/ で閲覧できます。

## どのようにコントリビュートするか

- 誤字、情報、スタイルの修正
    - PR を送るか Issues に書き込んでください 
- 情報の追加
    - PR を送るか Issues に書き込んでください 
- 新規ページ
    - Issues で相談してください
- リクエスト
    - Issues に書き込むか、管理者 [@Reputeless](https://twitter.com/Reputeless) にリプライを送ってください

コントリビューションが採用された方は、[コントリビュータ](https://cppmap.github.io/contribution/contributors/)として Web サイトに名前を掲載します。

## ビルド方法

### MKDocs のインストール

MkDocs を使います。  
Python 3 をインストールし、MkDocs と MkDocs Material テーマ、`fontawesome_markdown` をインストールするために、以下のコマンドを実行します。
```
pip install mkdocs
pip install mkdocs-material
pip install fontawesome_markdown
```

### ビルド

`mkdocs.yml` があるプロジェクトフォルダに移動し、以下のコマンドを実行すると Web サイトをビルドできます。
```
mkdocs build
```
ビルドされた Web サイトは `../cppmap.github.io` ディレクトリに保存されます。


