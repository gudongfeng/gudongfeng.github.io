---
title: 【译】自定义Devise
layout: post
date: 2016-10-12
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 翻译
- devise
- rails
blog: true
author: gdf
description: 如何自定义Devise的主要几个功能
---

## 参考链接
原文[链接](https://rails.devcamp.com/professional-rails-development-course/application-build/customizing-devise)

## 文章正文

### 自定义Devise
在这篇文章当中我们将会介绍如何使用`devise`的一些自定义功能来满足一些特定的需求，下面列表就是需求列表

- 用户需要有以下属性`first_name`, `last_name`, `role` 和 `username`
- 用户的`role`属性默认为`student`
- 用户可以注册
- 用户可以登录
- 用户可以删除自己的账号
- 用户可以编辑自己的账号
- 用户可以修改自己的`role`属性
- 用户的`username`是一个子域名(`subdomain`)
- 用户的`username`是唯一的
- 用户的`username`不可以含有特殊的字符

**用户必须要填写`first_name`, `last_name`, `role` 和 `username`**

假设我们已经将`first_name`, `last_name`, `role` 和 `username`添加到`users`的表中。我们还需要添加一些新的参数，有一个需要注意的地方是，如果我们想要添加一些`Devise`中没有内置的参数，我们需要将这个参数添加到strong parameters当中，这样Rails才知道这些参数是有用的。

让我们从写一些`specs`测试开始，如果没有使用任何的自定义的区域（field），那么我们一般来说不需要测试`Devise`因为`Devise`本身就有非常完善的测试。但是当我们需要添加一些自定义的行为的时候，那么我们自定义的这些行为就需要进行测试。首先创建一个新的文件`user_spec.rb`

```ruby
# spec/features/user_spec.rb

require 'rails_helper'

describe 'user navigation' do
  describe 'creation' do
    it 'can register with full set of user attributes' do
      visit new_user_registration_path

      fill_in 'user[email]', with: "test@test.com"
      fill_in 'user[password]', with: "password"
      fill_in 'user[password_confirmation]', with: "password"
      fill_in 'user[first_name]', with: "Jon"
      fill_in 'user[last_name]', with: "Snow"
      fill_in 'user[username]', with: "watcheronthewall"

      click_on "Sign up"

      expect(page).to have_content("Welcome")
    end
  end
end
```

运行测试`rspec spec/features/user_spec.rb`将会出现以下的错误`Capybara::ElementNotFound: Unable to find field "user[first_name]"`。这个错误是因为我们还没有添加相应的自定义区域（field），现在让我们在registration的模版（template）当中添加我们自定义的区域。

```ruby
<!-- app/views/devise/registrations/new.html.erb -->

<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>

  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true %>
  </div>

  <div class="field">
    <%= f.label :username %><br />
    <%= f.text_field :username %>
  </div>

  <div class="field">
    <%= f.label :first_name %><br />
    <%= f.text_field :first_name %>
  </div>

  <div class="field">
    <%= f.label :last_name %><br />
    <%= f.text_field :last_name %>
  </div>

  <div class="field">
    <%= f.label :password %>
    <% if @minimum_password_length %>
    <em>(<%= @minimum_password_length %> characters minimum)</em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "off" %>
  </div>

  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "off" %>
  </div>

  <div class="actions">
    <%= f.submit "Sign up" %>
  </div>
<% end %>

<%= render "devise/shared/links" %>
```

现在让我们将新属性添加到strong parameters当中。创建一个新的控制器`registrations_controller.rb`：

```ruby
# app/controllers/registrations_controller.rb

class RegistrationsController < Devise::RegistrationsController

  private

  def sign_up_params
    params.require(:user).permit(:first_name, :last_name, :email, :password, :password_confirmation, :username)
  end

  def account_update_params
    params.require(:user).permit(:first_name, :last_name, :email, :password, :password_confirmation, :current_password, :username)
  end
end
```

这个将会拓展默认的`Devise`控制器，然后添加一些新的自定义的属性，我们将会在我们的注册的表中使用到这些属性。最后我们需要修改路由（route）文件

```ruby
# config/routes.rb

Rails.application.routes.draw do
  devise_for :users, controllers: { registrations: 'registrations' }
  root to: 'static#home'
end
```

让我们再次运行测试`rspec`时，所有的测试将会通过。这时我们可以开启服务器`rails s`然后使用浏览器访问页面`localhost:3000/users/sign_up`，将会看到新的区域。

![Alt text](/assets/images/posts/devise_custom_params_form.png)

---
**用户的role属性默认值为student**

用户的role属性应该由系统控制，用户将无法做出任何的更改。让我们来通过修改rails生成（`rails generate model User` ）的`user_spec.rb`文件来添加这个功能的测试：

```ruby
# spec/models/user_spec.rb

require 'rails_helper'

RSpec.describe User, type: :model do
  describe 'creation' do
    before do
      @user = User.create(email: "test@test.com", password: "password", password_confirmation: "password", first_name: "Jon", last_name: "Snow", username: "watcheronthewall")
    end

    it 'should be able to be created if valid' do
      expect(@user).to be_valid
    end

    it 'should have a default role of: student' do
      expect(@user.role).to eq('student')
    end
  end
end
```

这里面有两个测试，其中一个测试将会正常通过因为我们知道`User`将会被成功创建，第二个测试将会失败，因为这个测试是来检查`User`默认的`role`是否为`Student`。运行`rspec spec/models`，我们将会发现以下的错误：`NoMethodError: undefined methodrole' for #User:0x007fd2b09991e0`。下面让我们来创建一个迁移文件来修复这个错误：

```ruby
rails g migration add_role_to_users role:string
```

当我们运行了`rake db:migrate`之后，将会添加一个`role`属性到`User`的数据库当中，现在我们重新运行测试`rspec spec/models`，我们将会获得一个不同的错误`expected: "student", got: nil`。

![Alt text](/assets/images/posts/1475505387332.png)

现在让我们添加一个回调函数到`user.rb`当中，来默认创建用户的`role`属性

```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  after_initialize :set_defaults

  private

    def set_defaults
      self.role ||= 'student'
    end
end
```

- `after_initialize`是一个回调函数，这个函数将会在`User`模型初始化的时候被调用。这里我们不使用`after_create`的原因是因为`after_create`将会复写我们实时修改的信息。
- `set_defaults`是一个自定义的方法，用于设置默认值。
- `self.role ||= 'student'`如果`role`的值为`nil`则把`role`设置为`'student'`。

当我们再次运行`rspec`时，所有的测试将会通过。

---

现在让我们回到**用户必须有`first name`，`last name`，`role`和`username`**的这个需求上来。为了实现这个需求，我们需要添加一些`validations`到`user.rb`文件当中，但是首先让我们添加一些新的`rspec`测试：

```ruby
# spec/models/user_spec.rb

require 'rails_helper'

RSpec.describe User, type: :model do
  describe 'creation' do
    before do
      @user = User.create(email: "test@test.com", password: "password", password_confirmation: "password", first_name: "Jon", last_name: "Snow", username: "watcheronthewall")
    end

    it 'should be able to be created if valid' do
      expect(@user).to be_valid
    end

    it 'should have a default role of: student' do
      expect(@user.role).to eq('student')
    end

    describe 'validations' do
      it 'should not be valid without a first name' do
        @user.first_name = nil
        expect(@user).to_not be_valid
      end

      it 'should not be valid without a last name' do
        @user.last_name = nil
        expect(@user).to_not be_valid
      end

      it 'should not be valid without a username' do
        @user.username = nil
        expect(@user).to_not be_valid
      end
    end
  end
end
```

运行`rspec spec/models`将会给我们三个错误，错误内容为`Failure/Error: expect(@user).to_not be_valid`。让我们给我们的`user.rb`文件添加以下的`validations`：

```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  after_initialize :set_defaults

  validates_presence_of :first_name, :last_name, :username

  private

    def set_defaults
      self.role ||= 'student'
    end
end
```

再次运行`rspec`将会发现所有的测试都会通过。

---

现在让我们回顾一下我们的需求：

- 用户需要有以下属性`first_name`, `last_name`, `role` 和 `username` - **完成**
- 用户的`role`属性默认为`student` - **完成**
- 用户可以注册 - **devise自动生成**
- 用户可以登录 - **devise自动生成**
- 用户可以删除自己的账号 - **devise自动生成**
- 用户可以编辑自己的账号 - **devise自动生成**
- 用户可以修改自己的`role`属性 - **完成**
- 用户的`username`是一个子域名(`subdomain`) - **稍后会完成**
- 用户的`username`是唯一的
- 用户的`username`不可以含有特殊的字符

现在让我们来完成需求：

**用户的`username`是唯一的**

首先让我们来添加一些新的测试到测试文件中来：

```ruby
# spec/models/user_spec.rb

it 'should ensure that all usernames are unique' do
  duplicate_username_user = User.create(email: "test2@test.com", password: "password", password_confirmation: "password", first_name: "Joni", last_name: "Snowy", username: "watcheronthewall")
  expect(duplicate_username_user).to_not be_valid
end
```

运行测试我们将会发现以下错误`Failure/Error: expect(duplicate_username_user).to_not be_valid`。下一步让我们来更新`user.rb`文件：

```ruby
# app/models/user.rb

validates_uniqueness_of :username
```

当我们再次运行`rspec`时，所有的测试将会通过。

---

现在让我们实现最后的一个需求。**用户的`username`不可以含有特殊的字符**

因为我们的用户名最终将会成为子域名（subdomain），所以用户名中将不能包含任何的特殊字符，类似于`&*$&*@`。首先我们添加`rspec`测试：

```ruby
# spec/models/user_spec.rb

it 'should ensure that usernames do not allow special characters' do
  @user.username = "*#*(@!"
  expect(@user).to_not be_valid
end
```

运行测试将会给出相应的错误提示消息，现在让我们来修改`user.rb`文件：

```ruby
# app/models/user.rb

class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  after_initialize :set_defaults

  validates_presence_of :first_name, :last_name
  validates :username, uniqueness: true, presence: true, format: { with: /\A[a-zA-Z]+([a-zA-Z]|\d)*\Z/ }

  private

    def set_defaults
      self.role ||= 'student'
    end
end
```

现在当我们再次运行测试时，所有的测试将会通过。

### 参考[代码](https://github.com/rails-camp/dailysmarty/tree/a795d2b8a40fc742e534f0dae27b8adb956d19d7)