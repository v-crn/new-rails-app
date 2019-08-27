# Railsプロジェクトの新規作成手順

自分用の備忘録としてRailsプロジェクトの作成手順をここにまとめます。

# 目次

- [Railsプロジェクトの新規作成手順](#railsプロジェクトの新規作成手順)
- [目次](#目次)
- [前提](#前提)
- [1. `bundle init`](#1-bundle-init)
- [2. `bundle install --without production`](#2-bundle-install---without-production)
  - [グローバルインストール（デフォルト）](#グローバルインストールデフォルト)
  - [Gemの保存先をプロジェクト内の`vendor/bundle`に指定したい場合](#gemの保存先をプロジェクト内のvendorbundleに指定したい場合)
- [3. `bundle exec rails new . -B`](#3-bundle-exec-rails-new---b)
  - [主に使用するオプション](#主に使用するオプション)
- [4. gitリポジトリの管理](#4-gitリポジトリの管理)
  - [4-1. .gitignoreの編集](#4-1-gitignoreの編集)
  - [4-2. ローカルリポジトリでコミット](#4-2-ローカルリポジトリでコミット)
  - [4-3. リモートリポジトリへプッシュ](#4-3-リモートリポジトリへプッシュ)
- [5. Gemのインストール](#5-gemのインストール)
  - [Rubocop](#rubocop)
    - [gemインストール](#gemインストール)
    - [rubocopコマンド](#rubocopコマンド)
      - [コードチェック](#コードチェック)
      - [自動修正](#自動修正)
    - [rubocopの設定ファイル](#rubocopの設定ファイル)
      - [設定ファイルの作成](#設定ファイルの作成)
        - [プロジェクト専用の設定ファイル](#プロジェクト専用の設定ファイル)
    - [VS Codeの拡張機能ruby-rubocop](#vs-codeの拡張機能ruby-rubocop)
      - [Ruby › Rubocop: Config File Path](#ruby--rubocop-config-file-path)
      - [Ruby › Rubocop: Execute Path](#ruby--rubocop-execute-path)
  - [Solargraph](#solargraph)
  - [better_errors / binding_of_caller](#better_errors--binding_of_caller)
  - [minitest-reporters / rails-controller-testing](#minitest-reporters--rails-controller-testing)
  - [Guard](#guard)
    - [Gemfile](#gemfile)
    - [初期化](#初期化)
  - [dotenv](#dotenv)
  - [Bootstrap](#bootstrap)
    - [Bootstrapで利用できる変数](#bootstrapで利用できる変数)
    - [導入手順（Rails 6の場合）](#導入手順rails-6の場合)
      - [1. Bootstrap4、jQuery、Popper.jsの導入](#1-bootstrap4jquerypopperjsの導入)
      - [2. environment.jsの作成](#2-environmentjsの作成)
      - [3. application.jsの編集](#3-applicationjsの編集)
      - [4. application.scssの作成](#4-applicationscssの作成)
      - [5. application.html.erbの編集](#5-applicationhtmlerbの編集)
      - [参考](#参考)
    - [導入手順（Rails 6未満の場合）- Railsプロジェクトの新規作成手順](#導入手順rails-6未満の場合--railsプロジェクトの新規作成手順)
      - [1. application.cssの編集](#1-applicationcssの編集)
      - [2. application.jsに依存関係を記述する](#2-applicationjsに依存関係を記述する)
  - [pg（HerokuでpostgresqlをDBとして利用する場合）](#pgherokuでpostgresqlをdbとして利用する場合)
- [6. 本番環境の設定（production.rb）](#6-本番環境の設定productionrb)
  - [SSL/HTPPS](#sslhtpps)
  - [asset compile](#asset-compile)
- [本番環境用のWebサーバー](#本番環境用のwebサーバー)
- [Herokuデプロイ](#herokuデプロイ)
  - [Herokuの準備](#herokuの準備)
  - [初回のgit push heroku master](#初回のgit-push-heroku-master)
  - [更新の際よく使うコマンド](#更新の際よく使うコマンド)
    - [git push heroku](#git-push-heroku)
    - [DBリセット](#dbリセット)
    - [参考](#参考-1)
- [おわり](#おわり)


# 前提

* Mac環境
* bundlerインストール済み
* Rails 6.0.0

本記事はRails 5.2.3の設定を基にして書かれているため新しいバージョンに対応しきれていない項目があるかもしれません。

# 1. `bundle init`
まずプロジェクトのルートになるファイルを用意し、`bundle init`コマンドでGemfileを生成します。

```
$ mkdir new-rails-app
$ cd new-rails-app
$ bundle initÎ
```

生成されたGemfileにおいて`# gem "rails"`のコメントアウトを外し、必要であればバージョンを指定します。

```
# frozen_string_literal: true
source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

gem "rails"
```

# 2. `bundle install --without production`
## グローバルインストール（デフォルト）
bundlerでsystem(global)にGemをインストールします。

```
$ bundle install --without production
```

rbenvでRubyのバージョンが管理されているローカルの開発環境では、Gemはsystemのrubyのインストール先に保存されます。

例：`/Users/username/.rbenv/versions/2.5.3/lib/ruby/gems/2.5.0/gems`

ちなみにどのバージョンのrubyがsystemのglobalに設定されているかは`rbenv versions`で確認できます。

```
$ rbenv versions
  system
* 2.5.3 (set by /Users/username/Documents/WebDevelopment/workspace/new-rails-app/.ruby-version)
  2.6.0
```

## Gemの保存先をプロジェクト内の`vendor/bundle`に指定したい場合
`--path vendor/bundle`を付けます。

```
$ bundle install --path vendor/bundle
```

プロジェクト毎にGemを管理するため`vendor/bundle`にインストールするか、あるいはデフォルトのままsystem(global)にインストールするかについては種々の意見があります。個人的にはデフォルトのまま使用しています。

# 3. `bundle exec rails new . -B`
Gemfile.lockに記載されたバージョンのrailsを使用してRailsプロジェクトを生成します。

```
$ bundle exec rails new . -B
```

* 上記コマンド実行時、Gemfileの上書きをして良いか尋ねられたら、`Y`（yesの意）を入力して続行します。
* `new`の後ろに`.（ドット）`を置いて`rails new`コマンドを実行すると、現在のディレクトリにプロジェクトファイルを作成できます。
* アプリをHerokuにデプロイする場合、本番環境のDBをPostgresqlに設定する必要があります。上記コマンドに`-d postgresql`を付けて実行すると諸々設定してくれます。

## 主に使用するオプション

| オプション              | 説明                                      |
| ----------------------- | ----------------------------------------- |
| -B, --skip-bundle       | bundle installを行わない                  |
| -d, --database=DATABASE | データベースの指定（デフォルトはsqlite3） |
| --skip-turbolinks       | turbolinksの無効化                        |
| -T, --skip-test         | デフォルトのminitestを使わない            |

その他のオプションについては`$ rails new -h`で確認できます。

# 4. gitリポジトリの管理
## 4-1. .gitignoreの編集
gitの管理から外すべきファイルを.gitignoreに記述します。

ここで、さまざまな環境に合わせて.gitignoreのコードを作成してくれるWebサービス[gitignore.io](https://www.gitignore.io/)を利用すると簡単にこの作業が終わります。

[https://www.gitignore.io/:embed:cite]

私の場合、`Ruby`, `Rails`, `VisualStudioCode`をキーワードにコードを生成し、少し変更を加えた上で使っています。実際のコードを下記のリンクに置いておきます。

[https://github.com/v-crn/new-rails-app/blob/master/.gitignore:embed:cite]


<b>gitignore.ioで生成したコードからの変更点</b>

* 環境変数設定ファイル`.env`についての記述が重複していたので２番目を削除
* `.env`を`.env*`に変更：ファイル名が「.env」で始まるファイルを無視（`.env.development`といったファイルもgitの管理対象から除外するため）
* `# config/secrets.yml`のコメントアウトを外した
* `config/credentials.yml.enc`の追加
* `config/database.yml`の追加
* 不要なコメント`# Ignore Byebug command history file.`を削除
* `/spring/*.pid`の追加
* `/public/uploads`の追加


## 4-2. ローカルリポジトリでコミット

```
$ git init
$ git add .gitignore
$ git commit -m "first commit"
```

## 4-3. リモートリポジトリへプッシュ

プッシュする前に、任意のGitホスティングサービスで新規リポジトリを作成します。

[https://github.com/:title]

[https://bitbucket.org/:title]

リポジトリ作成直後に表示されるリポジトリパスをコピーしておきます。
プロトコルとしてはHTTPSよりSSHの方が無難です。((参考：[https://qiita.com/chroju/items/67da13c672efcd2bc787:title]))


<b>SSH keyの形式</b>

| サービス名 | SSH key                                         |
| ---------- | ----------------------------------------------- |
| GitHub     | git@github.com:GitHub-ID/リポジトリ名.git       |
| Bitbucket  | git@bitbucket.org:Bitbucket-ID/リポジトリ名.git |

あとはコンソール上でコピーしたリポジトリパスを登録してプッシュするだけです。

```
$ git remote add origin リポジトリURL
$ git push -u origin master
```

-u, --set-upstreamオプションを付けてプッシュすると、プッシュ成功時にプッシュ先のリモートブランチを上流ブランチとして設定します。上流ブランチ((上流ブランチ(upstream branch)：あるローカルブランチが、履歴を追跡するように設定したリモートブランチのこと。))を設定しておくと、`git push`や`git pull`などのコマンドを実行するときに引数を省略でき、便利です。

# 5. Gemのインストール
最初にGemfileの完成形を示し、それから導入したGemについて説明します。

[Gemfile]()
```
# frozen_string_literal: true

source "https://rubygems.org"
git_source(:github) { |repo| "https://github.com/#{repo}.git" }

ruby "2.5.3"

# Bundle edge Rails instead: gem 'rails', github: 'rails/rails'
gem "rails", "~> 6.0.0"
# Use Puma as the app server
gem "puma", "~> 3.11"
# Use SCSS for stylesheets
gem "sass-rails", "~> 5"
# Transpile app-like JavaScript. Read more: https://github.com/rails/webpacker
gem "webpacker", "~> 4.0"
# Turbolinks makes navigating your web application faster. Read more: https://github.com/turbolinks/turbolinks
gem "turbolinks", "~> 5"
# Build JSON APIs with ease. Read more: https://github.com/rails/jbuilder
gem "jbuilder", "~> 2.7"
# Use Redis adapter to run Action Cable in production
# gem 'redis', '~> 4.0'
# Use Active Model has_secure_password
# gem 'bcrypt', '~> 3.1.7'

# Use Active Storage variant
# gem 'image_processing', '~> 1.2'

# Reduces boot times through caching; required in config/boot.rb
gem "bootsnap", ">= 1.4.2", require: false

group :development, :test do
  # Call 'byebug' anywhere in the code to stop execution and get a debugger console
  gem "byebug", platforms: [:mri, :mingw, :x64_mingw]
  gem "dotenv-rails"
  # Use sqlite3 as the database for Active Record
  gem "sqlite3", "~> 1.4"
end

group :development do
  gem "better_errors"
  gem "binding_of_caller"
  # Access an interactive console on exception pages or by calling 'console' anywhere in the code.
  gem "web-console", ">= 3.3.0"
  gem "listen", ">= 3.0.5", "< 3.2"
  # Spring speeds up development by keeping your application running in the background. Read more: https://github.com/rails/spring
  gem "spring"
  gem "spring-watcher-listen", "~> 2.0.0"
end

group :test do
  # Adds support for Capybara system testing and selenium driver
  gem "capybara", ">= 2.15"
  gem "guard"
  gem "guard-minitest"
  gem "minitest-reporters"
  gem "rails-controller-testing"
  gem "selenium-webdriver"
  # Easy installation and use of web drivers to run system tests with browsers
  gem "webdrivers"
end

group :production do
  gem "pg"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]

# Appearance
gem "jquery-rails"
gem "bootstrap", "~> 4.3.1"
```

## [Rubocop](https://github.com/rubocop-hq/rubocop)
rubyコードがコーディング規約通りに記述されているかチェックする静的コード解析ツールです。

### gemインストール
`$ gem install rubocop`でインストールします。

### rubocopコマンド
#### コードチェック
```
$ rubocop
```

#### 自動修正
```
$ rubocop -a
```

### rubocopの設定ファイル
#### 設定ファイルの作成
拡張子.ymlのファイルをrubocopの設定ファイルとして用意します。

例：[]()

内容は以下のリンクを参考に編集しました。
- [rails/.rubocop.yml](https://github.com/rails/rails/blob/master/.rubocop.yml)
- [.rubocop.yml設定例](https://qiita.com/hi-nakamura/items/fb841be12566e7f579f0)

##### プロジェクト専用の設定ファイル
`rubocop --auto-gen-config`コマンドでそのプロジェクト専用の設定ファイルを作成することもできます。これを実行すると生成されるファイルが2つあります。
- .rubocop.yml：コーディング規約の設定ファイル
- .rubocop_todo.yml：`rubocop --auto-gen-config`実行時点で違反している規約を無効にする設定ファイル

.rubocop_todo.ymlの内容は空っぽの状態が望ましいですが、どうしても対応することが難しい規約は.rubocop.ymlに移動しましょう。

### VS Codeの拡張機能ruby-rubocop
インストール後、以下のVS Codeの設定で以下の項目を指定する必要があります。

#### Ruby › Rubocop: Config File Path
rubocopの設定ファイルのパスです。  
例：/Users/Username/***/Ruby/Rails/workspace/.rubocop.yml

#### Ruby › Rubocop: Execute Path
rubocop実行ファイルのディレクトリまでのパスです。  
例：/Users/Username/.rbenv/shims/rubocop/


## Solargraph
コード補完機能を提供するgemです。  
`gem install solargraph`でインストールできます。

## better_errors / binding_of_caller
errorを見やすく表示してくれるgemです。

Gemfile
```
group :development do
  ......
  gem 'better_errors'
  gem 'binding_of_caller'
end
```

詳しくは以下の記事が参考になります。

[【Rails】better_errorsとbinding_of_callerで自分でエラーを解決できるようになろう【初心者向け】 - Qiita](https://qiita.com/Ryokky/items/284892be879996e4f77c)

## [minitest-reporters](https://github.com/kern/minitest-reporters) / [rails-controller-testing](https://github.com/rails/rails-controller-testing)
minitest-reportersはRailsのデフォルトのテストmitetestにおいて合格／不合格を色分けして表示してくれます。  
rails-controller-testingを入れるとtestでassigns、assert_templateを使うことができるようになります。

```
$ gem 'minitest-reporters'
$ gem 'rails-controller-testing'
```

test/test_helper.rbに以下の内容を追記しましょう。
```
require "minitest/reporters"
Minitest::Reporters.use!
```

## [Guard](https://github.com/guard/guard)
Guardは、ファイルシステムの変更を検知すると自動的にテストを実行してくれるツールです。

### Gemfile
```
group :test do
  ......
  gem 'guard'
  gem 'guard-minitest'
end
```

Rspecの場合
```
group :test do
  ......
  gem 'guard'
  gem 'guard-rspec', require: false
end
```


### 初期化
```
$ bundle exec guard init
```

Rspecの場合
```
$ bundle exec guard init rspec
```

初期化コマンドで生成されるGuardfileを[Rails Tutorialのリスト3.45](https://railstutorial.jp/chapters/static_pages?version=5.1#sec-getting_started_with_testing#sec-guard)と同様に編集します。

Guard使用時のSpringとGitの競合を防ぐために.gitignoreファイルにspring/ディレクトリを追加する必要があります（上述の.gitignoreには反映済み）。


## dotenv
.envという名前の隠しファイルを作成し、そこに`ENV[変数名]=値`の形式で変数を保存すると、それを環境変数として利用できるようにしてくれます。

```
gem 'dotenv-rails', groups: [:development, :test]
```

## [Bootstrap](https://github.com/twbs/bootstrap-rubygem)
CSSやSCSSでViewを装飾するのに役立つパッケージです。

### Bootstrapで利用できる変数
次のリンクから確認できます。
[bootstrap-rubygem/assets/stylesheets/bootstrap/_variables.scss](https://github.com/twbs/bootstrap-rubygem/blob/master/assets/stylesheets/bootstrap/_variables.scss)

### 導入手順（Rails 6の場合）
#### 1. Bootstrap4、jQuery、Popper.jsの導入
JS/CSSのパッケージ管理ツール「yarn」を通じてRailsアプリにBootstrap、jQuery、Popper.jsを導入します。

```
$ yarn add bootstrap@4.3.1 jquery popper.js
```

#### 2. environment.jsの作成
続いてconfig/webpack/environment.jsを作成し、設定を書き込みます。
```
$ mkdir config/webpack
$ touch config/webpack/environment.js
```

config/webpack/environment.js
```js
const { environment } = require('@rails/webpacker')

const webpack = require('webpack')
environment.plugins.append(
  'Provide',
  new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    Popper: ['popper.js', 'default']
  })
)

module.exports = environment
```

#### 3. application.jsの編集
app/javascript/packs/application.jsに以下を追記し、bootstrapをインポートします。

```js
import 'bootstrap'
import './src/application.scss'
```

#### 4. application.scssの作成
app/javascript/packs/src/application.scssを作成し、中に`@import '~bootstrap/scss/bootstrap';`を記述します。
```
$ mkdir app/javascript/packs/src
$ echo "@import '~bootstrap/scss/bootstrap';" > app/javascript/packs/src/application.scss
```

`@import '~bootstrap/scss/bootstrap';`と書くのは、webpackerがビルド時にapp/javascript/packs以下を参照するのでapplication.scssがビルドに含まれるように指定するためです。

#### 5. application.html.erbの編集
`stylesheet_link_tag`を`stylesheet_pack_tag`に変更します。

app/views/layouts/application.html.erb
```html
<%= stylesheet_pack_tag 'application', media: 'all', 'data-turbolinks-track': 'reload' %>
```

#### 参考
[Install Bootstrap with Webpack with Rails 6 Beta](https://gorails.com/forum/install-bootstrap-with-webpack-with-rails-6-beta)

[Rails6 Bootstrap4 を使って Heroku で Deploy](https://qiita.com/taKassi/items/56172d140d7208230e32)

### 導入手順（Rails 6未満の場合）- [Railsプロジェクトの新規作成手順](#railsプロジェクトの新規作成手順)
- [Railsプロジェクトの新規作成手順](#railsプロジェクトの新規作成手順)
- [目次](#目次)
- [前提](#前提)
- [1. `bundle init`](#1-bundle-init)
- [2. `bundle install --without production`](#2-bundle-install---without-production)
  - [グローバルインストール（デフォルト）](#グローバルインストールデフォルト)
  - [Gemの保存先をプロジェクト内の`vendor/bundle`に指定したい場合](#gemの保存先をプロジェクト内のvendorbundleに指定したい場合)
- [3. `bundle exec rails new . -B`](#3-bundle-exec-rails-new---b)
  - [主に使用するオプション](#主に使用するオプション)
- [4. gitリポジトリの管理](#4-gitリポジトリの管理)
  - [4-1. .gitignoreの編集](#4-1-gitignoreの編集)
  - [4-2. ローカルリポジトリでコミット](#4-2-ローカルリポジトリでコミット)
  - [4-3. リモートリポジトリへプッシュ](#4-3-リモートリポジトリへプッシュ)
- [5. Gemのインストール](#5-gemのインストール)
  - [Rubocop](#rubocop)
    - [gemインストール](#gemインストール)
    - [rubocopコマンド](#rubocopコマンド)
      - [コードチェック](#コードチェック)
      - [自動修正](#自動修正)
    - [rubocopの設定ファイル](#rubocopの設定ファイル)
      - [設定ファイルの作成](#設定ファイルの作成)
        - [プロジェクト専用の設定ファイル](#プロジェクト専用の設定ファイル)
    - [VS Codeの拡張機能ruby-rubocop](#vs-codeの拡張機能ruby-rubocop)
      - [Ruby › Rubocop: Config File Path](#ruby--rubocop-config-file-path)
      - [Ruby › Rubocop: Execute Path](#ruby--rubocop-execute-path)
  - [Solargraph](#solargraph)
  - [better_errors / binding_of_caller](#better_errors--binding_of_caller)
  - [minitest-reporters / rails-controller-testing](#minitest-reporters--rails-controller-testing)
  - [Guard](#guard)
    - [Gemfile](#gemfile)
    - [初期化](#初期化)
  - [dotenv](#dotenv)
  - [Bootstrap](#bootstrap)
    - [Bootstrapで利用できる変数](#bootstrapで利用できる変数)
    - [導入手順（Rails 6の場合）](#導入手順rails-6の場合)
      - [1. Bootstrap4、jQuery、Popper.jsの導入](#1-bootstrap4jquerypopperjsの導入)
      - [2. environment.jsの作成](#2-environmentjsの作成)
      - [3. application.jsの編集](#3-applicationjsの編集)
      - [4. application.scssの作成](#4-applicationscssの作成)
      - [5. application.html.erbの編集](#5-applicationhtmlerbの編集)
      - [参考](#参考)
    - [導入手順（Rails 6未満の場合）- Railsプロジェクトの新規作成手順](#導入手順rails-6未満の場合--railsプロジェクトの新規作成手順)
      - [1. application.cssの編集](#1-applicationcssの編集)
      - [2. application.jsに依存関係を記述する](#2-applicationjsに依存関係を記述する)
  - [pg（HerokuでpostgresqlをDBとして利用する場合）](#pgherokuでpostgresqlをdbとして利用する場合)
- [6. 本番環境の設定（production.rb）](#6-本番環境の設定productionrb)
  - [SSL/HTPPS](#sslhtpps)
  - [asset compile](#asset-compile)
- [本番環境用のWebサーバー](#本番環境用のwebサーバー)
- [Herokuデプロイ](#herokuデプロイ)
  - [Herokuの準備](#herokuの準備)
  - [初回のgit push heroku master](#初回のgit-push-heroku-master)
  - [更新の際よく使うコマンド](#更新の際よく使うコマンド)
    - [git push heroku](#git-push-heroku)
    - [DBリセット](#dbリセット)
    - [参考](#参考-1)
- [おわり](#おわり)
Gemfileにjquery-railsとbootstrapを追加します。
```
# Appearance
gem 'jquery-rails'
gem "bootstrap", "~> 4.3.1"
```

bootstrap導入の前提として以下のgemが要求されることに注意です。
- [jquery-rails](https://github.com/rails/jquery-rails)
- sprockets-rails 2.3.2.以上

#### 1. application.cssの編集
app/assets/stylesheets/application.cssの拡張子をscssに変更します。
- application.css -> application.scss

続いて末尾に
```scss
// Custom bootstrap variables must be set or imported *before* bootstrap.
@import "bootstrap";
```
を追記しましょう。

#### 2. application.jsに依存関係を記述する
Bootstrapの依存関係をapplication.jsに追記します。

app/assets/javascripts/application.js
```js
//= require jquery3
//= require popper
//= require bootstrap-sprockets
```

これでBootstrapとjQueryが利用できるようになりました。


## pg（HerokuでpostgresqlをDBとして利用する場合）
デフォルトのsqlite3はdevelpmentとtest環境で、本番環境ではpgを使用するようにします。

Gemfile
```
group :development, :test do
  ......
  # Use sqlite3 as the database for Active Record
  gem 'sqlite3', '~> 1.4'
end

group :production do
  gem 'pg'
end
```

# 6. 本番環境の設定（production.rb）
## SSL/HTPPS
セキュリティ強化のためにSSLを導入するにはproduction.rbに次の項目を追記するだけでOKです。
```rb
# Force all access to the app over SSL, use Strict-Transport-Security,
# and use secure cookies.
config.force_ssl = true
```

一般に本番用のWebサイトでSSLを使えるようにするためには、ドメイン毎にSSL証明書を購入し、セットアップする必要があります。が、Herokuのサブドメインを利用する場合、HerokuのSSL証明書に便乗することでその作業をしなくても済みます。

## asset compile
`config.assets.compile = false`をtrueにします。
```rb
config.assets.compile = true
```

Herokuにデプロイしたアプリでロゴや背景画像などが表示されない場合はこの項目がデフォルトのfalseになっていることが多いです。


# 本番環境用のWebサーバー
HerokuのデフォルトではWEBrickというWebサーバを使っています。WEBrickは簡単に導入できることが特長ですが、大きなトラフィックを扱うことには適していません。多数のリクエストを捌くことに適したWebサーバの１つとしてPumaがあります。

WEBrickからPumaにWebサーバを置き換えるために[HerokuのPumaドキュメント](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)に沿ってセットアップします。

config/puma.rbを以下の内容に書き換えます。
```rb
# frozen_string_literal: true

workers Integer(ENV["WEB_CONCURRENCY"] || 2)
threads_count = Integer(ENV["RAILS_MAX_THREADS"] || 5)
threads threads_count, threads_count

preload_app!

rackup      DefaultRackup
port        ENV["PORT"]     || 3000
environment ENV["RACK_ENV"] || "development"

on_worker_boot do
  # Worker specific setup for Rails 4.1+
  # See: https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server#on-worker-boot
  ActiveRecord::Base.establish_connection
end
```

次にHeroku上でPumaのプロセスを実行するための設定ファイル「Procfile」をルートディレクトリに作成します。
```
$ touch Procfile
```

作成したProcfileに以下の内容を記述します。
```
 web: bundle exec puma -C config/puma.rb
```

以上でpumaの導入は完了です。

# Herokuデプロイ
## Herokuの準備
```
$ heroku create アプリ名（任意）
$ heroku addons:create heroku-postgresql
```

## 初回のgit push heroku master
```
$ git push heroku master
$ heroku run rails db:migrate RAILS_ENV=production
```

bundler 2を使用しているためにHerokuへの初回pushで`Activating bundler (2.0.1) failed:`などのエラーで失敗する場合は次のコマンドでパッチをあててください。

```
$ heroku buildpacks:set https://github.com/bundler/heroku-buildpack-bundler2
```


## 更新の際よく使うコマンド
### git push heroku
初回のpushで付けていた「master」は省略可です。
```
$ git push heroku
```

### DBリセット
```
$ heroku pg:reset DATABASE
$ heroku run rails db:migrate
$ heroku run rails db:seed
```

### 参考
[RailsアプリをHerokuにデプロイする](https://v-crn.hatenablog.com/entry/2019/07/02/Rails%E3%82%A2%E3%83%97%E3%83%AA%E3%82%92Heroku%E3%81%AB%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4%E3%81%99%E3%82%8B)

# おわり
以上でRailsプロジェクトの最初のステップが完了しました！　ここからガンガン開発を進めていきましょう！