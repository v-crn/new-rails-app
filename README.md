自分用の備忘録としてRailsプロジェクトの作成手順をここにまとめます。

# 前提

* Mac環境
* Bundlerインストール済み

# １．`bundle init`
プロジェクトファイルを用意し、Gemfileを生成します。

```
$ mkdir new-rails-app
$ cd new-rails-app
$ bundle init
```

生成されたGemfileにおいて`# gem "rails"`のコメントアウトを外し、必要であればバージョンを指定します。その他最初に用意したいGemがあれば、Gemfileに記述しておきます。

```
# frozen_string_literal: true
source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem "rails"
```

# ２．`bundle install --path vendor/bundle`
保存先を`vendor/bundle`に指定してGemをインストールします。

```
$ bundle install --path vendor/bundle
```

# ３．`bundle exec rails new . -B -T`
インストールしたrailsを使用してRailsプロジェクトを生成します。

```
$ bundle exec rails new . -B -T
```

* 上記コマンド実行時、Gemfileの上書きをして良いか尋ねられたら、`Y`（yesの意）を入力して続行します。
* newの後ろに`.（ドット）`を置いて`rails new`コマンドを実行すると、現在のディレクトリにプロジェクトファイルを作成できます。

## 主に使用するオプション

|オプション|説明|
----|----
|-B, --skip-bundle|bundle installを行わない|
|-d, --database=DATABASE|データベースの指定（デフォルトはsqlite3）|
|--skip-turbolinks|turbolinksの無効化|
|-T, --skip-test|デフォルトのminitestを使わない|

その他のオプションについては

```
$ rails new -h
```

で確認できます。

# ４．gitリポジトリの管理
## ４−１．.gitignoreの編集
gitの管理から外すべきファイルを.gitignoreに記述します。

様々な環境に合わせて.gitignoreのコードを作成してくれるWebサービス[gitignore.io](https://www.gitignore.io/)を利用すると、非常に簡単に用意できます。

https://www.gitignore.io/

私の場合、`Ruby`, `Rails`, `VisualStudioCode`をキーワードにコードを生成し、編集を加えた上で使っています。実際のコードは下記のリンクから確認できます。

https://github.com/v-crn/new-rails-app/blob/master/.gitignore


<b>gitignore.ioで生成したコードからの変更点</b>

* 環境変数設定ファイル`.env`についての記述が重複していたので２番目を削除
* `.env`を`.env*`に変更：ファイル名が「.env」で始まるファイルを無視（`.env.development`といったファイルもgitの管理対象から除外するため）
* `# config/secrets.yml`のコメントアウトを外した
* `config/credentials.yml.enc`の追加
* `config/database.yml`の追加
* 不要なコメント`# Ignore Byebug command history file.`を削除


## ４−２．ローカルリポジトリでコミット

```
$ git init
$ git add .gitignore
$ git commit -m "first commit"
```

## ４−３．リモートリポジトリへプッシュ

プッシュする前に、任意のGitホスティングサービスで新規リポジトリを作成します。

https://github.com/

https://bitbucket.org/

リポジトリ作成直後に表示されるリポジトリパスをコピーしておきます。
プロトコルとしてはHTTPSよりSSHの方が無難です。((参考：[GitHubのremote URLにはどのプロトコルを使えばよいのか？](https://qiita.com/chroju/items/67da13c672efcd2bc787)))


<b>SSH keyの形式</b>

|サービス名|SSH key|
|---|---|
|GitHub|git@github.com:GitHub-ID/リポジトリ名.git|
|Bitbucket|git@bitbucket.org:Bitbucket-ID/リポジトリ名.git|

あとはコンソール上で先程コピーしたリポジトリパスを登録してプッシュするだけです。

```
$ git remote add origin リポジトリURL
$ git push -u origin master
```

-u, --set-upstreamオプションを付けてプッシュすると、プッシュ成功時にプッシュ先のリモートブランチを上流ブランチとして設定します。上流ブランチ（あるローカルブランチが、履歴を追跡するように設定したリモートブランチのこと）を設定しておくと、`git push`や`git pull`などのコマンドを実行するときに引数を省略でき、便利です。


以上でRailsプロジェクトの最初のステップが完了しました！　ここからガンガン開発を進めていきましょう！