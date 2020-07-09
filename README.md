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

### 日本語検索スクリプトの修正

日本語の検索スクリプトに問題があるため、次の手順で修正します ([参考](https://qiita.com/t-kuni/items/410aac718e531c6aee17#%E4%BF%AE%E6%AD%A3%E7%AE%87%E6%89%80))
- `Python??\Lib\site-packages\mkdocs\contrib\search\lunr-language\` フォルダに [TynySegmenter](http://chasen.org/~taku/software/TinySegmenter/tiny_segmenter-0.2.js) を追加、TynySegmenter の最終行に `module.exports = TinySegmenter;` を加える
- `Python??\Lib\site-packages\mkdocs\contrib\search\lunr-language\lunr.jp.js` で `var segmenter = new lunr.TinySegmenter();  // インスタンス生成` を `var segmenter = new (require('./tiny_segmenter-0.2.js'))();  // インスタンス生成` に置き換え

### ビルド

`mkdocs.yml` があるプロジェクトフォルダに移動し、以下のコマンドを実行すると Web サイトをビルドできます。
```
mkdocs build
```
ビルドされた Web サイトは `../cppmap.github.io` ディレクトリに保存されます。


