---
title: 【译】Rails 用户认证(Authen)
layout: post
date: 2016-10-23
image: 
headerImage: false
tag:
- devise
- rails
- ruby
blog: true
author: gdf
description: Rails 的用户认证
---

## 参考链接
原文[链接](https://rails.devcamp.com/professional-rails-development-course/application-build/rails-authentication)

## 文章正文
### 为什么要从用户认证（Authentication）开始

当我们决定先创建什么功能的时候，一个重要的依据就是功能之间的依赖性（dependencies）。依赖性的意思就是指不同功能之间的需求程度。功能A的依赖性高就是指有很多其他的功能需要这个功能A。下面让我们看一看我们所创建的需求表：

![functional requirements](/assets/images/posts/functional_requirements.png)

仔细阅读需求表，你将会发现几乎有一半的需求都跟`User`有关，所以我们将会从`User`的功能开始。如果你从其他的功能（比如`Post`）开始，那么我们将会需要在实现`User`的功能的时候回到`Post`当中来重构`Post`的代码。

从另一个方面来看，当我们查看数据库的`schema.rb`文件时，我们将会发现数据库中有许多其他的表单将`user_id`设为了外键（foreign key）。这是一个非常好的信号用来表示`User`这个表的依赖程度。所以说我们将会首先开始来创建`User`模型。

### 使用[Devise](https://github.com/plataformatec/devise)还是从头开始创建用户认证系统

我编写过很长一段时间的程序。到目前为止，只有两个应用程序我是从头开始创建用户认证系统而不是使用现有的用户认证系统库，类似于`Deivse`。而且这两个例外的应用程序还是因为它有特定的需求，需要集成微软的`ActiveDirectory`认证引擎，所以才我从头创建用户的认证系统。

我个人的观点认为我们应该使用像`Devise`这样，经过充分测试并且会有人员长期维护的库来创建一些功能，而不是从头开始创建。在接下来的教程当中，我将会介绍如何集成`Devise`的每一个步骤，但我并不会介绍`Devise`这个`gem`本身的内容，我已经写过一篇博客[Devise的用法](http://blog.gdf.name/devise-usage/)用来介绍有关`Devise`本身的内容，读者可以点进链接进行参。

### 实现用户认证

我们本来可以从创建模型的`spec`测试开始，但是我并不想给一些已经做过充分测试的功能再写一些重复的测试。`Devise`已经有一个非常详细与综合的测试系统，你可以查看测试[链接](https://github.com/plataformatec/devise/tree/master/test)。

在我们创建新的功能之间，有一个很重要的地方需要注意的就是，我们不能在`mater`分支上面直接修改代码，我们需要首先创建一个新的分支：

```bash
git checkout -b add-auth
```

将我们正在做的工作与`master`分支区分开来将是非常重要的，因为这样可以解决如下这种情况，当我们正在工作的时候，我们突然有另外一个非常紧急的功能需要实现，比如说修复一个错误，如果说我并没有将我现在工作放到另外一个分支当中，那么我将需要做：

- 还原（reset）到一个不同的commit（有可能会产生冲突）
- 将我做出来的所有的改变都先注释掉（这是一个非常糟糕的注意，你将会不可避免的忘记注释掉一些代码，导致程序崩溃）
- 删除自己做出的所有的改变（这也是一个非常糟糕的注意，因为这将会导致自己失去之前所做的所有的工作） 

现在让我们安全的在另外一个独立的分支当中开始工作，将`gem`信息添加到`Gemfile.rb`当中：

```ruby
# Gemfile

gem 'devise', '~> 3.5', '>= 3.5.5'
```

在运行了命令`bundle`之后，我们可以运行安装器：

```bash
rails generate devise:install
```

首先让我们在`config/initializers/devise.rb`中来更新默认的`mailer_sender`设置，这将可以保证认证邮箱的发件人是我们而不是默认的地址。

```ruby
# config/initializers/devise.rb

config.mailer_sender = 'team@dailysmarty.com'
```

现在让我们来一步一步的进行配置：

**更新本地的邮件服务器**

```ruby
# config/environments/development.rb

config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

我们可以跳过对主页面的配置因为我们已经在博客[Rails程序配置](http://blog.gdf.name/rails-rspec-initial/)/[英文](https://rails.devcamp.com/trails/professional-rails-development-course/campsites/application-build/guides/rails-app-configuration)完成了这个步骤。现在让我们来添加警告`alerts`的页面。我将不会把警告的页面添加到主页面当中，因为在有一些界面当中我们可能不想要实现警告的界面，或者说我们只想将警告的界面渲染到页面的特定位置。所以让我们在`view`目录下创建一个`shared`目录然后添加一个局部（partial）文件叫做`_alerts.html.erb`：

```ruby
<%#= app/views/shared/_alerts.html.erb %>

<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

现在让我们简单的在`application.html.erb`文件中添加一个渲染的命令，我们将会在开始设计的时候将这个命令移除：

```ruby
<%#= app/views/layouts/application.html.erb %>

<%= render 'shared/alerts' %>
```

你发现了这种设计有多么干净吗？当我们在逐渐创建应用程序的过程当中，你将会发现我们会使用大量的局部（partial）文件来保持我们代码的干净和不重复（DRY）。现在让我们来使用以下的命令创建`Devise`的视图（view）：

```bash
rails g devise:views
```

这段命令将会创建以下相关的视图（view）文件：
- 注册
- 登录
- 编辑账号
- 密码重设
- 邮件模板

我很喜欢`Devise`生成器，然后它确实会创建一个我们并不需要的文件，现在让我们来删除掉那些我们不会使用的文件来避免程序显得很杂乱：

```bash
rm -rf app/views/devise/confirmations
rm -rf app/views/devise/mailer/confirmation_instructions.html.erb
```

现在让我们使用`Devise`生成器来创建`User`模型：

```bash
rails g devise User first_name:string last_name:string avatar:text username:string
```

这将会创建一个默认的`User`模型，我在这个模型的迁移（migration）文件当中插入了一些新的属性：`first_name`,`last_name`,`avatar`和`username`。现在让我们来运行命令`rake db:migrate`来在数据库中创建相应的表和`schema.rb`文件：

```ruby
# db/schema.rb

create_table "users", force: :cascade do |t|
    t.string   "email",                  default: "", null: false
    t.string   "encrypted_password",     default: "", null: false
    t.string   "reset_password_token"
    t.datetime "reset_password_sent_at"
    t.datetime "remember_created_at"
    t.integer  "sign_in_count",          default: 0,  null: false
    t.datetime "current_sign_in_at"
    t.datetime "last_sign_in_at"
    t.inet     "current_sign_in_ip"
    t.inet     "last_sign_in_ip"
    t.string   "first_name"
    t.string   "last_name"
    t.text     "avatar"
    t.string   "username"
    t.datetime "created_at",                          null: false
    t.datetime "updated_at",                          null: false
end

add_index "users", ["email"], name: "index_users_on_email", unique: true, using: :btree
add_index "users", ["reset_password_token"], name: "index_users_on_reset_password_token", unique: true, using: :btree
```

以上的教程将会给我们生成足够的代码来让用户实现注册，登录和一些其他的标准的有关用户认证的操作。在下一个博客当中我们将会讲解如何[自定义Devise](http://blog.gdf.name/devise-customization/)。

### 资源

[代码](https://github.com/rails-camp/dailysmarty/tree/add-auth)

