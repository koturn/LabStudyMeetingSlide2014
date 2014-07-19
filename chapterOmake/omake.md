## おまけ
<h1>プログラミング勉強会</h1>

担当: koturn




## 目次
- バージョン管理システム
- Markdown
- reveal.js




## バージョン管理システム

- バージョン管理システム(VCS: Version Control System)
  - ソースコードの変更履歴などを記録
- ソフトウェア開発では必須


## 主なバージョン管理システム
- CVS: Concurrent Versions System
  - 集中型バージョン管理システム
  - C言語で実装
  - 1990年代に広く利用された
  - 現在ではあまり用いられていない
- SVN: Apache Subversion
  - 集中型バージョン管理システム
  - C言語で実装
  - 現在でも用いられている
- git
  - 分散型バージョン管理システム
  - C言語，シェルスクリプト，Perlなどで実装
  - 現在，最も用いられている


## git

- リーナスにより開発された
  - Linuxのソースコード管理のため
- 現在，GitHubなどのホスティングサービスがかなり有名になっている
- MITの学生の課題提出にも用いられているらしい


## Gitリポジトリのホスティングサービス

- リモートリポジトリを置けるサービス
- 他人と共有開発ができる
- ソフトウェア公開に利用されているのが多くのケース？
- 以下は無料で利用可能なWebサービス
  - GitHub
  - Bitbucket
  - codebreak;


## GitHub

- 最も有名かつ用いられているGitホスティングサービス
  - Gitホスティングサービスのデファクトスタンダード
- 以下の条件なら無料で利用可能
  - 容量300MB以内
  - 共有人数無制限
  - 公開リポジトリのみ
- 以下の機能を利用するには，課金が必要
  - 非公開(プライベート)リポジトリ
    - 非公開リポジトリの最大個数に応じて，課金額が異なる
- GitHub Pagesというサービスを用いて，静的Webサイトを構築可能


## Bitbucket

- GitHubの次に有名なGitホスティングサービス
- 無料で以下のことが可能
  - 容量300MB以内
  - 共有人数は5人まで
  - 非公開リポジトリの作成(個数は無制限)
- GitHub Pagesのような機能は無い


## codebreak;

- 日本で作られたGitホスティングサービス
- あまり有名ではない？
  - 情報が少ない
- 無料で以下のことが可能
  - 容量無制限
  - 共有人数無制限
  - 非公開リポジトリの作成(個数は無制限)




## GitHub Pages

- GitHubの機能の1つ
- 静的なWebサイトを作成できる
  - **簡単！**
  - **タダ！**


## GitHub Pagesの作成方法

- 2つの方法がある
  1. USERNAME.github.io という名前のリポジトリを作成する
    - 1アカウントに1つしか作成できない
  2. gh-pagesブランチを作成する
    - プロジェクト毎にWebページを作成できる


## USERNAME.github.io での作成方法

- 作成してHTMLファイル等を置くだけ


## USERNAME.github.io でのURL

- GitHubのリポジトリ
  - https://github.com/USERNAME/USERNAME.github.io
- GitHub Pages
  - http://USERNAME.github.io/


## gh-pages ブランチを作成する場合

```sh
# gh-pagesブランチを作成
$ git branch gh-pages

# gh-pagesブランチに移動
$ git checkout gh-pages

# コミットなど
$ git add .
$ git commit -m "Initial commit on gh-pages"

# GitHubにpush
$ git push origin gh-pages
```


## ね，簡単でしょ？


## GitHub PagesのURL

- GitHubのリポジトリ
  - https://github.com/USERNAME/REPOSITORY
- GitHub Pages
  - http://USERNAME.github.io/REPOSITORY/


## GitHub PagesのURL 例

このスライドのURLが実例

- GitHubのリポジトリ
  - https://github.com/koturn/chapterOmake
- GitHub Pages
  - http://koturn.github.io/chapterOmake/


## GitHub Pages ルートディレクトリ

- ルートディレクトリにアクセスしたときの動作
  - ルートディレクトリにある ```index.html``` を返そうとする


## よくありそうな疑問

- Dropboxでも静的Webサイトを作れるじゃん？
  - DropboxのURL，謎の番号含むから嫌だなぁ


## GitHub Pages の利点

- 自分のアカウント名を含んだ単純なURL
- バージョン管理も同時にできる




## Markdown

- 文書を記述するための軽量マークアップ言語
  - HTMLの代替として，気軽に書ける
- ヒューマンリーダブルな形式で書ける
  - HTMLはマシンリーダブル
- 様々なツールで様々な形式に変換可能
  - 汎用性がある
- 最近，広く用いられている印象がある


## Markdownのサンプル

```markdown
# Markdownの例
これは，Markdownの例を示すテキストです

## Markdownとは
軽量なMarkup言語です

- このように
- リストを扱うことが
- できます
  - 入れ子も可能
    1. 番号付リストも
    2. 可能です
```


# サンプルをreveal.jsで表示

これは，Markdownの例を示すテキストです


## Markdownとは
軽量なMarkup言語です

- このように
- リストを扱うことが
- できます
  - 入れ子も可能
    1. 番号付リストも
    2. 可能です


## Markdownはこんなところに使える

- GitHubのREADME.md
  - GitHub Flavored Markdownという形式
- プレゼン
  - reveal.js
  - pandoc
  - Vim(showtime.vim や previm など)


## Markdownの欠点

- Markdownそのものの表現能力は低い
  - HTMLタグを記述して，機能を補うこともある
- 各種Markdownの方言があり，統一されていない
  - GitHub
  - Bitbucket
  - Qiita
  - はてなブログ




## reveal.js の概要

- Javascriptで実装されたプレゼンツール
- オーソドックスな形式のスライドを作成可能
- そこそこアニメーションを行える
- HTMLだけでなく，Markdownで記述可能
  - 外部ファイルのMarkdownを読み込みできる
  - Markdownを転用可能
    - GitHubにMarkdownを置くだけで，簡易レジュメ
- 印刷用のスライドを簡単に作成できる


## reveal.js の特徴

- 細かいフォーマット調整はcssをいじる必要あり
- シンタックスハイライトのcssがデフォルトで1つしかない
  - デフォルトでもうちょっと用意してほしいところ


## reveal.js はこう用いる

- GitHub pages を用いて，無料でWeb上で公開可能
  - HTMLとCSSとJavascriptのみで構成されているため
- rvl.io というWebサービスでも無料でWeb上で公開可能
- PDF化すれば印刷可能，SlideShareでも無料で公開可能


## reveal.js スライドの作り方

1. HTML内に記述
2. HTML内にMarkdownを記述
3. 別ファイルにMarkdownを記述し，それを読み込む


## ローカルでの外部Markdownの読み込みに関して

- ローカルのreveal.js用のHTMLファイルを読み込んだ場合，外部Markdownの参照不可
- localhostにHTTPサーバをたてる必要あり


## Pythonの簡易HTTPサーバ (1)

以下のたった1行のコマンドで，カレントディレクトリをルートディレクトリとして
ポート8000番にHTTPサーバが立ち上がる

```sh
$ python -m SimpleHTTPServer
```

- http://localhost:8000
  - パスを指定しない場合，ルートディレクトリのindex.htmlにアクセス
- http://localhost:8000/sample/test.html
  - ルートディレクトリから見た，sample/index.htmlにアクセス


## Pythonの簡易HTTPサーバ (2)

- ポート番号を指定することも可能
- デフォルトポート:80番を指定してみる

```sh
$ python -m SimpleHTTPServer 80
```

- ポート番号を省略してアクセス可能
  - http://localhost
  - http://localhost/sample/test.html
  - http://127.0.0.1/sample/test.html




## まとめ

- GitHubでWebページ作れる！
- reveal.jsで手軽にいい感じのスライドが作れる！
