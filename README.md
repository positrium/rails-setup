## したいこと

- railsインストール
    - rspec_railsインストール
    - factory_girl_railsインストール
    - database_cleanerインストール
- 自動でrspec実行
    - guardインストール
- configを環境毎に分けたい
    - rails_configインストール

## 01. rails new

### 前処理

`~/.gemrc`へドキュメント取得を抑制するスイッチを書く

```~/.gemrc
install: --no-document
update: --no-document
```
- see also: [gem environment](http://guides.rubygems.org/command-reference/#gem-environment)

### ベースのrails環境をインストール

```
$ mkdir sample_app
$ cd sample_app

$ bundle init
$ vi Gemfile
```

```ruby:Gemfile
source 'https://rubygems.org'

gem 'rails'
```

```
$ bundle install --path=vendor/bundle

Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies...
Installing rake 11.2.2
Installing concurrent-ruby 1.0.2
Installing i18n 0.7.0
...
```

```
$ bundle exec rails new -Tf .

# skip options
#
# [T]est case
# [--no-ri] do not generate README.rdoc
#
```

```
$ brew install gibo

$ gibo Rails >> .gitignore
$ gibo OSX >> .gitignore
$ echo '/vendor/bundle' >> .gitignore
```

```
$ git init
$ git add .
$ git commit -m 'initialize.'
```

## 02. append gems

```ruby:Gemfile
group :development do
  gem 'spring-commands-rspec'
  gem 'guard-rspec'
end

group :test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
  gem 'database_cleaner'
end

gem 'config'
```

### development

- spring-command-rspec
  - `bundle exec spring rspec`
  - springをコマンドでrspecを爆速に使いたい
- guard-rspec
  - *_spec.rbを変更しながらテストを実行したい

### test

- rspec-rails
  - railsコマンド系でrspecを生成したい
- factory_girl_rails
  - railsコマンド系でfactory_girlを生成したい
- database_cleaner
  - rspec単位でtest用データを初期化したい

### global

- config
  - `bundle exec rails g config:install`
  - see also: [configを利用する](http://qiita.com/yumiyon/items/32c6afb5e2e5b7ff369e)


## 03. rspec-rails

- rspecの設定なので`spec_helper.rb`へ記述
- see also : http://www.atmarkit.co.jp/ait/articles/1409/30/news037.html
- specは他のspecの実行結果に依存しない作りにするので、実行はランダムにする。

```
$ bundle exec rails g rspec:install

      create  .rspec
      create  spec
      create  spec/spec_helper.rb
      create  spec/rails_helper.rb
```

```ruby:spec/spec_helper.rb
RSpec.configure do |config|

  # rspec configuration
  config.order = :random

end
```

## 04. factory_girl

- rspecの設定ではないので`rails_helper.rb`へ記述
- see also : http://qiita.com/muran001/items/436fd07eba1db18ed622

```ruby:spec/rails_helper.rb
RSpec.configure do |config|
  
  config.use_transactional_fixtures = false
  
  config.include FactoryGirl::Syntax::Methods
  
  config.before(:all) do
    FactoryGirl.reload
  end

end
```

## 05. database_cleaner

- rspecの設定ではないので`rails_helper.rb`へ記述
- see also : http://dev.classmethod.jp/server-side/ruby-on-rails/ruby-on-rails_database-cleaner_data_erase/
- see also : http://qiita.com/yoshitsugu/items/3470dbcadfdd677be543
- specは他のspecの実行結果に依存したくないので、実行前後でtruncationを行う。

```ruby:spec/rails_helper.rb
RSpec.configure do |config|

  config.before(:suite) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end

end
```

## 06. confirmation

```
$ bundle exec rails g model borrowing user:references book:references due_back:date

      invoke  active_record
      create    db/migrate/20150318135741_create_borrowings.rb
      create    app/models/borrowing.rb
      invoke    rspec
      create      spec/models/borrowing_spec.rb
      invoke      factory_girl
      create        spec/factories/borrowings.rb
```

factory_girl_railsを利用しているためか、http://www.atmarkit.co.jp/ait/articles/1409/30/news037.html では生成されていないfactory_girl関連のファイルも生成できていることが分かる。

もちろん、rspec関連のファイルも生成されていることが分かる。

## 07. config

- see also : http://qiita.com/yumiyon/items/32c6afb5e2e5b7ff369e

```
$ bundle exec rails g config:install

      create  config/initializers/config.rb
      create  config/settings.yml
      create  config/settings.local.yml
      create  config/settings
      create  config/settings/development.yml
      create  config/settings/production.yml
      create  config/settings/test.yml
      append  .gitignore
```

## 99-A. guard-rspec

- see also : http://ruby-rails.hatenadiary.com/entry/20141021/1413819783
- guardでファイルの変更を監視してもらい、ファイル変更をトリガに全rspecを実行してもらいたい


```shell
$ bundle exec guard init rspec

15:46:42 - INFO - Writing new Guardfile to /Users/ito/src/ghe.apctdl.com/inspector-gadget/rails-swagger-mockserver/Guardfile
15:46:42 - INFO - rspec guard added to Guardfile, feel free to edit it
```

```shell
$ bundle exec spring binstub --all

* bin/rake: spring inserted
* bin/rspec: generated with spring
* bin/rails: spring inserted
```

```ruby:Guardfile
guard :rspec, cmd: 'bundle exec spring rspec --color --format documentation' do
  # ..
end
```

```
$ bundle exec guard

15:52:24 - INFO - Guard::RSpec is running
15:52:25 - INFO - Guard is now watching at '/path/to/root'
[1] guard(main)>
[1] guard(main)> exit

15:52:51 - INFO - Bye bye...
```

### Could not find 'bundler' エラーの場合

> /Users/hoge/.rbenv/versions/2.1.5/lib/ruby/2.1.0/rubygems/dependency.rb:298:in `to_specs': Could not find 'bundler' (>= 0) among 93 total gem(s) (Gem::LoadError)

```
$ bundle update
```

### rails5でENCODING_FLAGのエラーの場合

> bundler: failed to load command: rspec (/path/to/root/vendor/bundle/ruby/2.3.0/bin/rspec)
NameError: uninitialized constant ActionView::Template::Handlers::ERB::ENCODING_FLAG

- 参考：[Getting an unitialized constant error with RSpec. Have no idea what's causing it](http://stackoverflow.com/questions/4429068/getting-an-unitialized-constant-error-with-rspec-have-no-idea-whats-causing-it)

```ruby:spec/spec_helper.rb
require 'rails/all' # 先頭に追加
require 'rspec/rails'
require 'factory_girl'
require 'database_cleaner'
```
