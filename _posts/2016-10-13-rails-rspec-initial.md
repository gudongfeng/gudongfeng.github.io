---
title: 【译】Rails测试：如何给Rails项目初始化Rspec
layout: post
date: 2016-10-12
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 翻译
- rails
- rspec
blog: true
author: gdf
description: Rspec的初始化配置
---

## 参考链接

[link](https://rails.devcamp.com/professional-rails-development-course/application-build/rails-app-configuration)（本文为参考链接的非官方中文翻译版本，并非完全对照翻译，减少了原文中不必要的内容）

## 说明
这片文章将会一步一步来教大家如何初始化rspec将rspec集成到Rails的项目当中

## 文章正文：
要想充分的使用rspec，除了主要的 `rspec-rails` gem，我们还需要使用到以下三个gems：

- `Capybara` － 这个gem主要是用来写集成测试（integration tests），它可以模拟用户的行为来自动点击浏览器中的链接，填写浏览器中的表格等功能。
- `Database Cleaner` －这个gem可以自动清除我们的test数据库，来避免一些来自rspec可能的潜在错误，例如我们想要创建唯一性的数据时，因为数据库中已经存在了相应的数据，导致该数据无法创建。
- `Factory Girl` －用户创建sample数据，便于我们在rspec的测试中使用它们

首先让我们把相应的gem添加到`Gemfile`当中（因为上述的四个gems全部都是用于测试，所以我们可以把gems添加到 `:development` 和 `:test` 的组当中），参考一下的`Gemfile`文件：

```ruby
# Gemfile

source 'https://rubygems.org'

gem 'rails', '4.2.5'
gem 'pg', '~> 0.15'
gem 'sass-rails', '~> 5.0'
gem 'uglifier', '>= 1.3.0'
gem 'coffee-rails', '~> 4.1.0'
gem 'jquery-rails'
gem 'turbolinks'
gem 'jbuilder', '~> 2.0'
gem 'sdoc', '~> 0.4.0', group: :doc

group :development, :test do
  gem 'byebug'
  gem 'rspec-rails', '~> 3.0'
  gem 'capybara'
  gem 'database_cleaner'
end

group :development do
  gem 'web-console', '~> 2.0'

  gem 'spring'
end
```

参考 `:development`, `:test` 模块

```ruby
gem 'rspec-rails', '~> 3.0'
gem 'capybara'
gem 'database_cleaner'
```

通过运行命令 `bundle` 安装需要的gem之后，我们使用rspec的命令来安装 `RSpec` 。

```bash
rails generate rspec:install
```

这个命令将会创建相应的rspec配置文件

```ruby
create  .rspec
create  spec/spec_helper.rb
create  spec/rails_helper.rb
```

下一步我们将需要配置 `rails_helper.rb` 文件来使 `capybara` 和 `DatabaseCleaner` 运作：

```ruby
# spec/rails_helper.rb

ENV['RAILS_ENV'] ||= 'test'
require File.expand_path('../../config/environment', __FILE__)

abort("The Rails environment is running in production mode!") if Rails.env.production?
require 'spec_helper'
require 'rspec/rails'
require 'capybara/rails'

ActiveRecord::Migration.maintain_test_schema!

RSpec.configure do |config|
  config.fixture_path = "#{::Rails.root}/spec/fixtures"
  config.use_transactional_fixtures = false
  config.before(:suite) { DatabaseCleaner.clean_with(:truncation) }
  config.before(:each) { DatabaseCleaner.strategy = :transaction }
  config.before(:each, :js => true) { DatabaseCleaner.strategy = :truncation }
  config.before(:each) { DatabaseCleaner.start }
  config.after(:each) { DatabaseCleaner.clean }
  config.infer_spec_type_from_file_location!
  config.filter_rails_from_backtrace!
end
```

以上的配置文件对比默认的配置文件作出了一下的修改：

- 添加了 `capybara/rails`
- 将 `use_transactional_fixtures` 改变为 `false`
- 添加了一些 `DatabaseCleaner` 的配置

至此我们已经完成了相应的配置，下面让我们写一个测试文件来测试我们可以访问主页。创建一个目录 `features` 和文件 `static` ：

```ruby
# spec/features/static_spec.rb

require 'rails_helper'

describe 'navigate' do
  describe 'homepage' do
    it 'can be reached successfully' do
      visit root_path
      expect(page.status_code).to eq(200)
    end
  end
end
```

运行命令 `rspec` 将会出现错误 `NameError: undefined local variable or method root_path' for #RSpec::ExampleGroups::Navigate::Homepage:0x007fa73f05eef0`:

![Alt Text](/assets/images/posts/rails-rspec-1.png)

我们可以通过修改 `routes.rb` 文件来修复这个错误：

```ruby
# config/routes.rb

Rails.application.routes.draw do
  root to: 'static#home'
end
```

再次运行 `rspec` 命令，我们将会遇到另外一个错误因为我们没有创建 `static` 控制器（controller），然后错误提示还是会说无法识别 `root_path`，所以让我们手动创建 `static` 控制器：

```ruby
# app/controllers/static_controller.rb

class StaticController < ApplicationController
end
```
现在我们再次运行 `rspec` 命令，我们将会获得另外一个错误消息就是 `AbstractController::ActionNotFound: The action 'home' could not be found for StaticController` ：

![Alt Text](/assets/images/posts/rails-rspec-2.png)

我们可以通过给添加 `home` index：

```ruby
# app/controllers/static_controller.rb

class StaticController < ApplicationController
  def home
  end
end
```

当我们再次运行 `rspec` 命令， 我们将会遇到错误 `ActionView::MissingTemplate: Missing template static/home` 这是因为我们没有创建模板文件，现在让我们在 `app/views/static` 目录下创建文件 `app/views/static/home.html.erb` 。

现在当我们运行命令 `rspec` 时，测试通过：

![Alt Text](/assets/images/posts/rails-rspec-3.png)

在测试文件 `spec/features/static_spec.rb` 中

- `Capybara` 将会开启一个虚拟的浏览器会话
- 它将会访问地址 `root_path`
- 并察看相应的页面是否返回 `HTTP` 状态码 `200`

至此改教程结束