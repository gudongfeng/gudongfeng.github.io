---
title: Devise的用法
layout: post
date: 2016-10-11
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 翻译
- devise
- rails
- ruby
blog: true
author: gdf
description: Devise的基本用法
---

## 参考链接
原文[链接](https://rails.devcamp.com/trails/ruby-gem-walkthroughs/campsites/authentication/guides/devise)

## 文章正文

### Devise 能做什么
用户的认证功能对于新的Rails开发者来说是一件非常困难的事情， 而 `Devise` 是一个非常全面的用户认证`gem`，提供了以下一些重要的功能：

- 注册
- 登录／登出
- 会话管理（可以知道用户是否登录当他们使用网站的时候）
- 密码重设
- 可以自动生成`View`模板，便于你用于参考。
- 可以自动生成模型(model)
- 可以集成诸如`Facebook`, `Twitter`, `wechat`等网站的一键登录功能(OAuth2)
- ...

### 为什么使用 Devise
`Deivse`之所以如此受欢迎的主要原因，是因为它能够很快的帮助开发者建立起很全面的用户的认证系统，并且它非常稳定有专业的人员来做持续的维护工作。
在安全方面，`Devise`也包含了许多重要的功能来保证安全性，它可以自动加密／解密密码，重设密码等。总体来说，它可以很大程度的保证我们用户认证系统的安全性。

### 缺点
如果你想要复写`Devise`中的一些默认的功能，将会变的非常困难。比如说如果你想让一个用户使用用户名登录而不是邮件登陆，那么这个的难度可能跟你从头创建一个`Auth`的系统一样。
还有当你想要添加自己的参数到`Devise`的模块当中时，比如说把`role`属性添加到`users`模型当中，我们就需要来修改`ApplicationController`来吧`role`属性添加到白名单当中，这会感觉很奇怪因为正常来说我们应该修改的是对应模型本身的`controller`。

### 实现／Devise实例
让我们现在开始集成`Devise`到我们的项目当中，首先我们需要到[rubygems.org](https://rubygems.org/gems/devise/versions/3.5.3)中获取最新的稳定版本，然后将相应的`gem`添加到我们项目的`Gemfile`当中。

```ruby
gem 'devise', '~> 3.5', '>= 3.5.3'
```

然后我们运行`bundle install`来安装相应的文件，安装完成之后，我们可以运行生成器(generator):

```bash
rails generate devise:install
```

这将会创建以下两个文件

```bash
create  config/initializers/devise.rb
create  config/locales/devise.en.yml
```

在我们开始做其他任何事情之间，我们需要首先打开文件`config/initializers/devise.rb`然后将文件当中的默认邮件的属性修改成我们对应的邮件地址：

```ruby
config.mailer_sender = 'please-change-me-at-config-initializers-devise@example.com'
```

修改为

```ruby
config.mailer_sender = 'no-reply@gdf.name'
```

然后我们可以查看`Devise`在终端中打印出来的帮助文档

```tex
Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

       rails g devise:views 
```

我们可以根据上面的提示来一步一步的完成`Devise`的配置

1.添加一个新的配置选项到`config/environments/development.rb`配置文件当中（如果是一个生产服务器那么你将会需要修改文件`config/environments/production.rb`）。在`configure`的代码块中，添加如下的代码：

```ruby
# config/environments/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

2.请确认你有一个`homepage`， 如果没有的话你需要首先修改文件`config/routes.rb`如下：

```ruby
# config/routes.rb

root to: 'pages#home'
```

其次你需要创建一个控制器文件`app/controllers/pages_controllers.rb`然后添加以下代码：

```ruby
# app/controllers/pages_controller.rb

class PagesController < ApplicationController
  def home
  end
end
```

3.创建一个`pages`目录然后创建文件`app/views/pages/home.html.erb`，并将以下的代码写入到文件`application.html.erb`当中（在`yield`代码的上方）:

```html
<%#= app/views/layouts/application.html.erb %>

<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

4.因为我们现在的`Rails`版本远超3.2，所以我们不需要做这一步的操作

5.使用`Deivse`generator创建视图（view）控件，在终端中运行命令`rails g devise:views`将会创建如下的文件：

```bash
app/views/devise/shared/_links.html.erb
app/views/devise/confirmations/new.html.erb
app/views/devise/passwords/edit.html.erb
app/views/devise/passwords/new.html.erb
app/views/devise/registrations/edit.html.erb
app/views/devise/registrations/new.html.erb
app/views/devise/sessions/new.html.erb
app/views/devise/unlocks/new.html.erb
app/views/devise/mailer/confirmation_instructions.html.erb
app/views/devise/mailer/password_change.html.erb
app/views/devise/mailer/reset_password_instructions.html.erb
app/views/devise/mailer/unlock_instructions.html.erb
```

上面的这些文件的创建将会给我们足够的代码来实现下列的功能：用户注册， 用户登录，用户编辑账户详情，用户重设密码和一些可以选择的功能，例如确认账号。

现在我们已经将所有的视图文件配置完毕，我们可以开始创建模型（model）文件，在终端中运行命令`rails g devise User`。这行代码将会创建一个模型（model）文件和一个迁移（migration）文件。迁移文件如下：

```ruby
# db/migrate/20160115010323_devise_create_users.rb

class DeviseCreateUsers < ActiveRecord::Migration
  def change
    create_table(:users) do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

从上述的文件当中，我们可以看到`Devise`给我们的标准属性有些什么，例如`email`和`password`，`Deivse`也提供了相应的属性来跟踪记录用户的登录次数，还有许多其他的功能。默认情况下`Devise`注释掉了`confirmable`属性，如果你取消了注释，那么用户将会收到邮件来激活相应的账号才可以使用。这是一个很有用的功能，但是很多情况下确认的邮件将会发送到用户的垃圾收件箱当中并且会有一定的延迟。

现在让我们在终端中运行命令`rake db:migrate`这将会在数据库中创建`user`列表。最后使用命令`rails s`运行服务器，在浏览器中输入地址`http://localhost:3000/users/sign_up`你就可以注册账号然后使用注册好的账号访问`http://localhost:3000/users/sign_in`来登录系统。

### 有趣的功能

1.了解用户是否登录
将如下的代码添加到`homepage`当中：

```html
<%#= app/views/pages/home.html.erb %>

<% if current_user %>
  <p>I'm signed in</p>
<% else %>
  <p>I'm not signed in</p>
<% end %>
```

首先登陆系统，你将会看到消息`I'm signed in`，然后打开一个匿名的浏览器窗口（chrome打开的方法为 MAC: `cmd+shift+n` Windows: `ctrl+shift+n`）访问主页你将会看到提示`I'm not signed in`。这是因为`current_user`这个属性是`Devise`的内置功能来查看一个用户是否登录。

2.强制用户登录

当我们想让强制用户登录之后才能查看某些页面时，我们可以使用方法`authenticate_user!`来保证用户必须登录了之后才能查看页面。在本项目中我们可以修改`PagesControllers`：

```ruby
# app/controllers/pages_controller.rb

class PagesController < ApplicationController
  def home
    authenticate_user!
  end
end
```
现在如果未登录的用户访问主页的时候将会自动跳转到登录界面，并且提示`You need to sign in or sign up before continuing.`

### 项目代码

[链接](https://github.com/rails-camp/gem-review-devise)

